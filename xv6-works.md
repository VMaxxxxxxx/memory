## 3.页表

### 3.1打印第一个进程的页表

> 使用risc-v的sv39页表硬件机制，在逻辑上实现访问页表
>
> - 递归打印页表：对传入的页表，按照512个PTE进行遍历，根据标志位来打印，利用PTE2PA做地址转换
> - 在第一个进程通过exec启动时根据pid来调用打印

### 3.2设计每个进程的内核页表

> - **多个进程在切换到内核态时，共享同一个内核页表，引发的问题：**
>   - 内核栈安全性不足：每个进程有自己的内核栈，但因为共享页表，所有进程的内核栈都会被映射，因为内核中的bug，被其他进程访问
>   - 目的：进程隔离：某进程进入内核态（同一时刻可能有多个进程处于内核态），**只能访问自己的内核栈**
> - 实现逻辑：
>   - 每个进程创建自己的内核页表：在proc结构体中添加一个字段kernelpgtbl
>     - 分配一个物理页，作为内核页表，然后做映射
>     - 内核页表需要固定的映射：uart、硬盘界面、中断控制等
>     - 以前是一个全局变量内核页表，现在有了独占的内核页表，一些函数修改参数（区分传入页表）（虚拟地址转换物理地址、虚拟地址映射到物理地址）
>     - 原来，在进程初始化的时候，为所有进程预分配内核栈
>     - **现在，进程创建的时候，为进程创建独立的内核页表，将专属的内核栈规定到内核页表固定位置**
>   - 在进入内核时，切换自己的内核页表，并刷新TLB，出来的时候再切回
>   - 进程释放时，要做清理和销毁（释放页表页空间，而不是指向的物理页（因为多个进程可能公用一个物理页码，比如fork之后涉及到写时复制，引用计数可能不是1，不能随意释放））：仍然是页表项遍历+递归

### 3.3简化copyin、copyinstr

> - copyin、copystr用于系统调用时**将用户空间的数据拷贝到内核空间**
> - xv6的内核页表中没有映射用户页表，因此想要访问用户空间数据就需要获取物理地址，这是通过**软件模拟**的：需要遍历进程的用户页表，将用户虚拟地址转换成物理地址，内核才能访问，非常繁琐且性能较低
> - 共享页表机制的原理：
>   - 通过**为每个进程构建一个共享页表（内核页表）**，即在进程独占的内核页表中，**直接加入用户空间的虚拟地址映射**
>   - 在内核态，可以通过共享页表直接访问用户空间的数据，无需每次查询页表后转换物理地址再做拷贝
> - 具体实现：
>   - 提供**页表复制辅助函数**，将src页表的一部分映射关系（页表项，并非物理页）拷贝到dst页表
>   - 提供**映射解除辅助函数**，根据传入参数的比较来确定页表缩减，并非清除物理页，仅解除映射
>   - 替换copyin、copyinstr的实现（PGROUNDDOWN、walkaddr、memmove），改为直接在内核态用用户的虚拟地址访问数据
>   - 在进程的共享页表（内核页表）中，**动态地同步用户空间的页表项**，如fork、exec、growproc、userinit在内存变化时，及时更新内核页表
>     - **fork**：为子进程复制父进程的用户页表时，也需要复制一份独占的内核页表（共享页表）
>     - **exec**：替换进程镜像时，用户页表已经替换，内核页表需要先解除映射，然后重新映射
>     - **growproc**：在进程的用户页表空间变化时，内核页表也需要相应变化（辅助函数）
>     - **userinit**：在用户内存初始化结束，需要同步程序内存到进程内核页表
>   - 对内存边界做严格检查，防止用户空间映射到内核空间或引发的整数溢出问题
> - 意义：
>   - 用用户虚拟地址进行拷贝数据，简化代码
> - 提问：
>   - 为什么更快：**减少了查询页表、地址转换步骤**，对数据直接进行读写
>   - 安全性：只要用户空间的最大虚拟地址严格控制在内核空间下，结合权限PTE_U，保证隔离
>   - Linux为什么不采用：效率高，但安全隐患meltdown/spectre，内核和用户空间分开切换页表
> - 注意：在内核态直接使用用户虚拟地址访问用户数据，虽然也要做虚拟地址转换，但这次转换由**CPU硬件自动寻址完成**，无需手动遍历页表，另外还可以使用**快表TLB**来加速访问
>

## 4.陷入Traps

### 4.2Backtrace

> - bt：打印曾经调用过的函数的地址，打印出函数调用栈，便于调试
>
> - 实验过程：
>
>   - 加入获取当前函数frame pointer的方法（汇编）
>
>     > xv6栈的结构：
>     >
>     > - sp指向当前栈帧的结束地址
>     > - p指向当前栈帧的起始地址
>     > - 栈从高地址往低地址生长
>     > - 依次是：返回地址ra（fp-8）、上一层栈帧的fp地址（fp-16）
>     > - xv6用一页来存储栈
>
>   - 通过判断当前帧指针fp是否在有效的页范围内，决定调用栈是否有效（有下一层）
>
>   - 通过fp-8、fp-16来得到ra和fp，进行打印
>
>   - 在sys_sleep中调用

### 4.3Alarm

> - 实现sigalarm和sigreturn两个系统调用：为用户进程添加定期通知功能，使得进程在一段时间内使用了CPU后，会被定期提醒，触发一种用户态的中断处理，模拟用户级的异常处理
> - 在进程结构体中添加一些**必要字段**：
>   - **时钟周期**alarm_interval，0禁止alarm
>   - **回调函数**alarm_handler
>   - 距离**下一次时钟的ticks数**，alarm_ticks
>   - 时钟中断的**陷阱帧**trapframe，alarm_trapframe，用于回调函数后的恢复
>   - 是否有回调正在执行的标志**alarm_goingoff**，防止回调执行期间，由于周期抵达再次回调提醒（导致alarm_trapframe被覆盖）
> - 在进程初始化、释放时加入对必要字段的处理
> - 注册时钟
>   - 实现sigalarm系统调用
>     - 使用argint和argaddr获取系统调用的参数（ticks、handler）
>     - 使用用户态传递的参数**初始化**进程结构体的必要字段（时钟周期、回调函数、下一次ticks）
>   - 实现sigreturn系统调用
>     - sigreturn就是将必要字段（陷阱帧trapframe、goingoff标志）**恢复**成alarm中断前的状态
> - 触发时钟中断
>   - 在usertrap中，判断当**`设置了时钟 && 时钟倒计时结束 && 没有其他时钟在运行`时，**
>   - **重置**时钟倒计时
>   - **保存**当前进程陷阱帧trapframe
>   - **跳转**执行时钟回调函数
>   - **标记**当前已有时钟在运行

## 5.惰性分配

> - RISC-V的三种页面错误：
>   - load：加载指令访问的虚拟地址找不到对应的物理地址时触发
>   - store：存储指令访问的虚拟地址找不到对应的物理地址时触发
>   - 指令页面错误：指令获取的虚拟地址找不到对应的物理地址时触发
> - 对应的寄存器或信息
>   - **scause**：指示页面错误的类型
>   - **stval**：保存无法转换的虚拟地址
>   - 引起page fault的PC值，表明了页面错误在用户空间发生的位置
> - 写时复制COW
>   - fork为例：父进程会将内存完全拷贝给子进程，确保父子进程的内存独立，但需要大量分配内存，如果子进程立即exec丢弃这个地址空间造成浪费
>   - 父子进程会**共享**相同的物理内存页面，而不是立即复制
>   - 共享的页面被标志为只读，父子进程无法直接写入，当父子进程修改页面时，触发页面错误异常，内核捕捉到异常，根据scause和stval的信息，**为父子进程都创建一个物理内存副本**，并映射到产生页面错误的虚拟地址上，更新页表权限为可读可写，返回到触发页面异常的指令位置，重新执行写操作
> - 惰性分配Lazy Allocation
>   - 当程序请求额外内存时，sbrk系统调用，增加地址空间，内核调**整进程的地址空间范围**，把**新地址标记为无效**
>   - 实际访问无效地址时，CPU会**触发页面错误**，捕捉到异常后，分析错误地址属于sbrk惰性分配引发
>   - 内核**实际分配**新的物理页面，将虚拟地址**映射**到新的页面上，更新页表中的新地址条目为有效状态
>   - **重新执行触发异常的指令**
>   - 应用程序往往申请比实际需求更多的内存，通过惰性分配，仅在真正使用内存时做实际分配，避免内存浪费
> - 页面换出Paging Out
>   - 进程的内存请求超出物理内存的容量，操作系统将部分**不常用的内存页面写入到磁盘**，释放出物理内存
>   - 被写入到磁盘的页面PTE标记为无效，当进程再次访问这些页面会产生页面错误
>   - 内核捕捉到页面错误，检查故障地址，属于换出的页面
>   - 内核分配一个新的物理页面，将该页面从磁盘上重新读回内存，更新PTE，标记为有效，恢复进程执行
>   - 页面换出机制可以让系统运行更多进程，或者让单进程使用比实际物理内存更多的地址空间

### 5.1从sbrk消除实际分配

> - 对于系统调用sys_sbrk，不再调用分配内存的growproc，仅让进程的sz加上新增的内存
> - 如果是n<0，即减少内存，需要立即调用uvmdealloc，检查减去内存后是否大于零

### 5.2Lazy Allocation

> - 修改usertrap，处理缺页异常
>   - 通过r_scause获取异常原因（13代表load、15代表write），stval表示引发缺页异常的虚拟地址
>   - 判断发生错误的虚拟地址是否位于栈空间（p->trapframe->sp）之上，进程大小之下（p->sz，即有效地址范围），然后再实际分配物理内存，添加映射
> - 修改解除映射，跳过未实际分配的物理页，无效的物理地址判断如下
>   - `if((pte == walk(pagetable, a, 0)) == 0`
>   - `if((*pte & PTE_V) == 0)`

### 5.3Lazytests  and Usertests

> - 压力测试：
>   1. sbrk参数为负，减少内存
>   2. 分配的虚拟内存处于进程空间
>- 两个函数：
> 
>  1. 检测虚拟地址是不是因为Lazy Allocation的原因没有分配和映射的地址
> 
>     - 处于地址范围内：`va < p->sz`
> 
>     - 不是栈的保护页guard page：`PGROUNDDOWN(va) != r_sp()`
> 
>     - 页表项不存在：`(((pte = walk(p->pagetable, va, 0)) == 0 || (*pte & PTE_V) == 0)`
> 
>  2. 给虚拟地址分配和映射物理页面
>      - 对传入的va，分配物理地址（一页），**分配失败结束进程**，成功则做虚拟地址和物理地址的映射
>     - 惰性分配的页刚分配时，没有对应的映射，把遇到无映射地址时的panic改为直接忽略（uvmunmap、uvmcopy跳过、copyin和copyinout立即分配）
> 

## 8.锁

### 8.1内存分配Memory allocator

> - 目的：优化xv6的物理内存分配器kalloc和kfree，让其在多核环境下减少锁争用，提升并发性能
>
> - 原因：**为什么有锁争用**
>
>   > - 原始：所有CPU公用同一个物理页空闲链表freelist，且用一把锁kmem.lock保护
>   > - 多核环境，多个CPU同时分配、释放物理页，都要竞争这把锁，争用严重
>   > - kalloctest中模拟多个进程大量分配或释放，统计出了锁争用次数很高
>
> - 优化：**为每个CPU分配独立的空闲页链表和锁**
>
>   > - 把原来的一个freelist，变成NCPU个，每个CPU有自己的freelist和锁
>   > - 每个进程分配和释放页时，只使用自己的锁和链表，不和其他CPU抢锁
>   > - 当前CPU的freelist没有空闲页，窃取其他CPU的freelist里的页（抢别人的锁）
>
> - 实现
>
>   > - **将原始的kmem结构体修改成数组（数量是CPU数量）**
>   > - 修改kinit()来初始化每个CPU对应的kmem的自旋锁
>   > - 修改在释放内存时，先**关中断**获取cpuid，再**加锁**将释放的页插入到freelist中（头插法），**释放kmem的锁**，重新**开中断**
>   > - 分配物理内存时，也是需要cpuid，因此关中断，加锁（当前cpu），去找空闲页
>   >   - 自己的freelist有空闲页，直接分配
>   >   - **自己没有，去窃取其他CPU的空闲页（窃取64页）**
>   >     - 轮询每个cpu
>   >     - **加对应CPU的锁**，查看人家的freelist状态（是否有空闲页）
>   >     - 如果有，一直窃取，**头插法**插到自己的freelist中，并移动其他CPU的freelist指针
>   >     - 直到窃取指定（64）页数后返回
>   >   - 解锁，开中断，往物理页中写入垃圾数据（memset）
>
> - 有个缺陷：
>
>   > - 一个CPU1在持有自身的锁，然后去偷CPU2的锁，此时CPU2也在持有自身锁的时候，去窃取CPU1的物理页，造成**环路等待和死锁**
>   >
>   > - 释放自己的锁，再去窃取其他CPU的页，把窃取的页的头指针保存起来，再申请本CPU的锁来CPU的页
>   > - 新问题：在释放自己的锁之后，再去窃取其他CPU的页，本CPU的空闲链表内容可能发生变化（被其他分配释放进程修改），只要偷页和插入页的过程都在各自锁的保护下修改链表，不会发生竞态或数据丢失

## 8.2磁盘缓冲Buffer cache

> - 背景
>
>   > - 块缓存（buffer cache）是用来缓存磁盘块的内存数据结构，原使用全局大锁bcache.lock保护整个缓存结构，以及所有块的引用计数
>   > - 当多进程并发访问文件系统时，都要抢这把大锁，导致锁争用严重，性能很差
>
> - 目的：让块缓存能高效支持多进程/多核并发访问，降低锁争用
>
> - 优化：将全局锁拆成细粒度的哈希桶锁
>
>   > - 哈希表分桶：用13个桶的哈希表来存储所有的缓存块buf，每个桶是一个带锁的链表
>   > - 查找、释放、分配块：只对相关的桶加锁，别的桶可以由其他进程并发
>   > - 块回收（驱逐）和插入：如果缓存满了，找一个最近最久且没人用（引用计数为0）的块进行替换，采用遍历所有桶进行查找实现，可能同时持有多个桶锁，避免死锁
>   > - 时间戳维护lru：每个块buf有个lastused时间戳标记，释放时更新
>   > - 保证一致性：不能有同一个块号blockno在缓存中出现多个副本，且驱逐、插入必须是原子操作，确保只有一个线程能插入、替换同一个块号
>
> - 实现：
>
>   > - > 下面的代码是`bio.c`，用于实现`基于哈希桶分布式加锁`和`避免缓存重复创建的并发驱逐策略`，属于`锁优化`和`多核缓存一致性`实验
>   >
>   > - > `buffer cache`：缓冲区缓存
>   >   >
>   >   > 定义：是一个由`buf`结构体组成的链表，用于缓存磁盘块的内容
>   >   >
>   >   > 优点：将磁盘块缓存在内存中，可以减少磁盘读取的次数，而提升性能
>   >   >
>   >   > 优点：为被多个进程使用的磁盘块，提供了同步点
>   >   >
>   >   > `bread`：获取特定磁盘块的缓冲区，如果不在缓存中会从磁盘中读取
>   >   >
>   >   > `bwrite`：修改缓冲区数据后，调用来将数据写回磁盘
>   >   >
>   >   > `brelse`：当不再需要缓冲区时，释放，自此不能再使用
>   >   >
>   >   > 限制：同一时间只有一个进程能够使用某个缓冲区
>   >   >
>   >   > 限制：不能长时间持有缓冲区
>   >
>   > - > xv6原始使用一个全局大锁`bcache.lock`，所有缓存操作（查询、插入、淘汰）都会竞争这个锁，这在多核并发下会使性能大幅下降，容易产生瓶颈
>   >   >
>   >   > 优化的策略：
>   >   >
>   >   > - 细化锁的粒度：将属于整个`buffer cache`的全局大锁`bcache.lock`分解成多个哈希桶锁`bufmap_locks`
>   >   > - 并发驱逐保护：避免两个线程在并发缓存未命中时重复创建缓存，使用一个额外的驱逐锁`eviction_lock`
>   >   > - 支持`LRU`替换策略：使用`lastuse`记录最近访问时间
>   >
>   > - > 把缓存分成13个桶，利用哈希运算，能够减少冲突，每个桶都有一个头节点`bufmap[i]`和该桶对应的自旋锁`bufmap_locks[i]`
>   >   >
>   >   > ```c
>   >   > #define NBUFMAP_BUCKET 13
>   >   > #define BUFMAP_HASH(dev, blockno) ((((dev) << 27) | (blockno)) % NBUFMAP_BUCKET)
>   >   > struct
>   >   > {
>   >   > struct buf buf[NBUF];
>   >   > struct spinlock eviction_lock;    // 驱逐锁
>   >   > // 哈希表
>   >   > struct buf bufmap[NBUFMAP_BUCKET];            // 哈希桶
>   >   > struct spinlock bufmap_locks[NBUFMAP_BUCKET]; // 桶锁
>   >   > } bcache;
>   >   > ```
>   >   >
>   >   > 驱逐锁的目的：解决重复创建缓存块的并发问题（多线程下保证缓存唯一性）
>   >   >
>   >   > - 当某个线程在自己的桶中找不到目标块时，会去全局查找`LRU`块，这个时候需要加锁保护
>   >   > - 避免两个线程同时找到相同的`LRU`块，创建两个缓存副本
>   >
>   > - > 初始化缓存：对每个哈希桶进行初始化锁，然后将所有缓存按分块buf以头插法分配到第一个桶内，最后初始化一个驱逐锁
>   >   >
>   >   > ```c
>   >   > void
>   >   > binit(void)
>   >   > {
>   >   > // 初始化桶锁
>   >   > for(int i = 0; i < NBUFMAP_BUCKET; ++i)
>   >   > {
>   >   >  initlock(&bcache.bufmap_locks[i], "bcache_bufmap");
>   >   > }
>   >   > for(int i = 0; i < NBUF; ++i)
>   >   > {
>   >   >  // 初始化缓存区块
>   >   >  struct buf* b = &bcache.buf[i];
>   >   >  initsleeplock(&b->lock, "buffer");
>   >   >  b->lastuse = 0;
>   >   >  b->refcnt = 0;
>   >   >  // 将所有缓存区块添加到bufmap[0]
>   >   >  b->next = bcache.bufmap[0].next;
>   >   >  bcache.bufmap[0].next = b;
>   >   > }
>   >   > initlock(&bcache.eviction_lock, "bcache_eviction");
>   >   > }
>   >   > ```
>   >
>   > - > 查找缓存块`buf`的接口函数`bget`：
>   >   >
>   >   > - 查找缓存块是否已存在
>   >   > - 若未命中，执行LRU驱逐并重建
>   >   >
>   >   > 三个阶段：
>   >   >
>   >   > - 尝试命中当前桶
>   >   > - 加驱逐锁并再次确认，双检防止释放桶锁期间其他线程抢先创建缓存
>   >   > - 全局寻找`LRU`块，重建并插入目标桶
>   >   >
>   >   > ```c
>   >   > static struct buf* bget(uint dev, uint blockno)
>   >   > {
>   >   >   struct buf *b;
>   >   >   // 通过哈希运算获取桶号，并获取桶锁
>   >   >   uint key = BUFMAP_HASH(dev, blockno);
>   >   >   acquire(&bcache.bufmap_locks[key]);
>   >   > 
>   >   >   // 第一阶段
>   >   >   // blockno的缓存区块是否已经在缓存区内
>   >   >   for(b = bcache.bufmap[key].next; b; b = b->next)
>   >   >   {
>   >   >     if(b->dev == dev && b->blockno == blockno)
>   >   >     {
>   >   >       b->refcnt++;
>   >   >       release(&bcache.bufmap_locks[key]);
>   >   >       acquiresleep(&b->lock);
>   >   >       return b;
>   >   >     } // 在当前桶中，命中
>   >   >   }
>   >   >   // 不在缓存区内：
>   >   > 
>   >   >   // 第二阶段
>   >   >   // 为了防止死锁，先释放当前桶锁
>   >   >   release(&bcache.bufmap_locks[key]);
>   >   >   // 但避免引入多线程对同一blockno在当前桶查询导致缓存区块重复创建，加上驱逐锁，其他的线程会卡在这里
>   >   >   acquire(&bcache.eviction_lock);
>   >   >   // 释放桶锁，加上驱逐锁的间隙，可能创建了新的blockno的缓存区块，因此需要再次检查一遍
>   >   >   for(b = bcache.bufmap[key].next; b; b = b->next)
>   >   >   {
>   >   >     if(b->dev == dev && b->blockno == blockno)
>   >   >     {
>   >   >       acquire(&bcache.bufmap_locks[key]); // 添加引用次数之前要加锁
>   >   >       ++b->refcnt;
>   >   >       release(&bcache.bufmap_locks[key]);
>   >   >       release(&bcache.eviction_lock);   // 找到buf需要释放驱逐锁
>   >   >       acquiresleep(&b->lock);   // 获取找到的buf的锁
>   >   >       return b;
>   >   >     }
>   >   >   }
>   >   > 
>   >   >   // 第三阶段
>   >   >   // 当再次查询仍然不在缓存区的时候，全局查找
>   >   >   // 此时只持有驱逐锁，不持有其他任何桶锁，查询所有桶中bru-buf
>   >   >   struct buf* before_least = 0;    // lru-buf的前一个块
>   >   >   uint holding_bucket = -1;       // 记录当前持有那个桶锁，方便后续重建
>   >   > 
>   >   >   // 循环查询所有桶
>   >   >   for(int i = 0; i < NBUFMAP_BUCKET; ++i)
>   >   >   {
>   >   >     acquire(&bcache.bufmap_locks[i]); // 获取当前便利的桶锁
>   >   >     int newfound = 0;   // 是否在当前桶中，找到新的lru-buf
>   >   > 
>   >   >     // 查询当前桶内所有的buf，尝试找到一个合适的，并标记
>   >   >     for(b = &bcache.bufmap[i]; b->next; b = b->next)
>   >   >     {
>   >   >       if(b->next->refcnt == 0 && (!before_least || b->next->lastuse < before_least->next->lastuse))
>   >   >       {
>   >   >         // 当前查询的引用数为1 并且（还没有找到过lru-buf 或 当前查询的buf的时间戳比找到的lru-buf更早）
>   >   >         // 更新当前找到的lru-buf
>   >   >         before_least = b;
>   >   >         newfound = 1;
>   >   >       }
>   >   >     }
>   >   >     // 如果没找到新的lru-buf，就释放当前的桶锁
>   >   >     if(!newfound)
>   >   >     {
>   >   >       release(&bcache.bufmap_locks[i]);
>   >   >     }
>   >   >     else
>   >   >     {
>   >   >       // 找到了新的lru-buf
>   >   >       if(holding_bucket != -1)
>   >   >       {
>   >   >         // 如果当前找到的不是第一个lru-buf，之前肯定持有某个桶锁，需要释放
>   >   >         release(&bcache.bufmap_locks[holding_bucket]);
>   >   >       }
>   >   >       holding_bucket = i; // 把标记 holding_bucket 更改成当前桶锁的编号
>   >   >     }
>   >   >   }
>   >   > 
>   >   >   // 第三阶段的全局查找结束，根据查询结果，进行重建
>   >   >   // 如果没有找到任何一个lru-buf，表示没有任何空闲缓存块了
>   >   >   if(!before_least)
>   >   >   {
>   >   >     panic("bget: no buffers");
>   >   >   }
>   >   > 
>   >   >   b = before_least->next;
>   >   >   if(holding_bucket != key)
>   >   >   {
>   >   >     // 想要偷的块如果不在key桶，就要把这个块从他所在的桶内驱逐出来
>   >   >     before_least->next = b->next;
>   >   >     release(&bcache.bufmap_locks[holding_bucket]);
>   >   >     // 将lru-buf加入到key桶
>   >   >     acquire(&bcache.bufmap_locks[key]);
>   >   >     b->next = bcache.bufmap[key].next;
>   >   >     bcache.bufmap[key].next = b;
>   >   >   }
>   >   > 
>   >   >   // 设置新buf的字段
>   >   >   b->dev = dev;
>   >   >   b->blockno = blockno;
>   >   >   b->refcnt = 1;
>   >   >   b->valid = 0;
>   >   > 
>   >   >   // 终于可以释放相关的锁了
>   >   >   release(&bcache.bufmap_locks[key]);
>   >   >   release(&bcache.eviction_lock);
>   >   >   acquiresleep(&b->lock);
>   >   >   return b;
>   >   > }
>   >   > ```
>   >
>   > - > `bread`：读`buf`：根据`dev`和`blockno`查询`buf`，如果找到的是个新块，就要从磁盘中读取数据到`buf`中
>   >   >
>   >   > `bwrite`：写`buf`：将`buf`的数据写入磁盘，当然当前cpu要持有`buf`，不能被其他持有
>   >
>   > - > `brelse`：当不再需要这个`buf`时，释放`buf`锁、申请再释放桶锁（减引用、如果无人引用就要重新更新`lastuse`）
>   >   >
>   >   > ```c
>   >   > void brelse(struct buf *b)
>   >   > {
>   >   > // 如果当前buf的锁被其他持有
>   >   > if(!holdingsleep(&b->lock))
>   >   > {
>   >   >  panic("brelse");
>   >   > }
>   >   > // 释放锁
>   >   > releasesleep(&b->lock);
>   >   > // 索引桶号
>   >   > uint key = BUFMAP_HASH(b->dev, b->blockno);
>   >   > // 获取桶锁，引用数-1
>   >   > acquire(&bcache.bufmap_locks[key]);
>   >   > b->refcnt--;
>   >   > // 如果不再有人引用，更新时间戳，用于跟踪lru-buf
>   >   > if(b->refcnt == 0)
>   >   > {
>   >   >  b->lastuse = ticks;
>   >   > }
>   >   > release(&bcache.bufmap_locks[key]);
>   >   > }
>   >   > ```
>   >
>   > - > `bpin`：手动加引用
>   >   >
>   >   > `bunpin`：手动减引用
>   >   >
>   >   > - 要注意先索引哈希桶，然后对桶加锁、释放锁
>   >   >
>   >   > ```c
>   >   > void bpin(struct buf* b)
>   >   > {
>   >   >   uint key = BUFMAP_HASH(b->dev, b->blockno);
>   >   >   acquire(&bcache.bufmap_locks[key]);
>   >   >   ++b->refcnt;
>   >   >   release(&bcache.bufmap_locks[key]);
>   >   > }
>   >   > void bunpin(struct buf* b)
>   >   > {
>   >   >   uint key = BUFMAP_HASH(b->dev, b->blockno);
>   >   >   acquire(&bcache.bufmap_locks[key]);
>   >   >   --b->refcnt;
>   >   >   release(&bcache.bufmap_locks[key]);
>   >   > }
>   >   > ```

