### POSIX socket编程，即Linux/Unix下的socket()系列API做网络编程

1. 高层流程

   - TCP服务端

     > ```shell
     > socket()	#创建套接字，得到一个文件描述符fd
     > setsocketopt()	#配置socket选项，如SO_REUSEADDR复用
     > bind()	# 绑定IP、端口到socket，即将fd关联到本地地址
     > listen()	# 把socket标记为被动监听，等待连接
     > accept()	# 接受客户端连接，返回新的用于通信的文件描述符fd
     > recv()	/ read() /	send()	/ write()	# 收发数据
     > shutdown()	# 半关闭连接（停止读或写）
     > close()	# 关闭fd，释放资源
     > ```

   - TCP客户端

     > ```shell
     > socket()	# 创建套接字
     > connect()	# 连接远端服务器，IP+端口
     > send()	/ recv()	# 收发数据
     > shutdown()	/ close()	# 关闭
     > ```

   - UDP

     > ```shell
     > # 服务端
     > socket()->bind()->recvfrom()->sendto()
     > # 客户端
     > socket()->sendto()/recvfrom()或connect()->send()/recv()
     > ```

2. 函数签名

   > ```c++
   > // domain: AF_INET/AF_INET6/AF_UNIX, type: SOCK_STREAM/SOCK_DGRAM
   > int socket(int domain, int type, int protocal);
   > // SO_REUSEADDR/SO_REUSEORT/SO_RCVTIMEO/SO_SNDTIMEO/SO_LINGER/TCP_NODELAY
   > int setsockopt(int sockfd, int level, int optname, const void *optval, cocklen_t optlen);
   > // addr 常为struct sockaddr_in 转型struct sockaddr*
   > int bind(int sockfd, const struct sockaddr *addr, socklen_t addrlen);
   > // backlog 指的是内核未accept的连接队列长度
   > int listen(int sockfd, int backlog);
   > // addr 会被填充为客户端地址
   > int accept(int sockfd, struct sockaddr *addr, socklen_t *addrlen);
   > int connect(int sockfd, const struct sockaddr *addr, socklen_t addrlen);
   > ssize_t send(int sockfd, const void *buf, size_t len, int flags);
   > ssize_t recv(int sockfd, void *buf, size_t len, int flags);
   > ssize_t sendto(...), recvfrom(...);
   > int close(int fd);
   > int shutdown(int sockfd, int how);
   > inet_pton(int af, const char *src, void *dst);
   > inet_ntop();
   > htons();
   > htonl();
   > ntohs();
   > ntohl();
   > fcntl(fd, F_SETFL, flags | O_NONBLOCK);
   > ```

3. 重要结构体

   > ```c++
   > // 统一地址容器，统一接口参数类型
   > struct sockaddr
   > {
   >  sa_family_t sa_family;	// 地址族
   >  char	   sa_data[14];	// 占位，具体类型按子类转换
   > }
   > struct sockaddr_in
   > {
   >  sa_family_t sin_family;		// AF_INET：IPV4
   >  uint16_t 	sun_port;		// 端口：转换网络字节序：htons(8080)
   >  struct in_addr sin_addr;	// IPv4地址，绑定到本机所有IP地址：INADDR_ANY
   >  unsigned char sin_zero[8];	// 填充，使得sizeof == sockaddr
   > }
   > /*
   > 	bind/accept/connect 等接口的参数是 通用的 struct sockaddr*。
   > 	你实际用的是 struct sockaddr_in* 或 struct sockaddr_in6*。
   > 	所以必须强制类型转换：
   > 	struct sockaddr_in addr;
   > 	bind(sockfd, (struct sockaddr*)&addr, sizeof(addr));
   > */
   > ```

4. 阻塞、非阻塞、阻塞IO、非阻塞IO、IO多路复用

   > - 阻塞、非阻塞：调用IO函数是否会让进程、线程等待，通常用 `fcntl(sockfd, F_SETFL, O_NONBLOCK)` 设置，系统调用不会阻塞，失败时返回-1并errno=EAGAIN
   > - 阻塞IO、非阻塞IO：内核处理IO的模式，是否挂起进程，内核调用阻塞用户进程或立即返回
   > - IO多路复用：用于一个进程、线程同时监视多个文件描述符的机制，对应传统阻塞式IO、多线程连接
   >   - select，使用位图bitmap存储fd状态，fd数量受限
   >   - poll，使用数组存储fd，内核用链表维护，没有fd_set限制，但需要遍历所有fd，效率不高
   >   - epoll，内核维护红黑树+就绪链表，Linux专属，只返回有事件的fd，性能高，支持大量并发连接，支持边沿触发和水平触发

5. `IO`多路复用

   > - 是什么：一种网络通信的机制
   > - 怎么样：可以以阻塞的方式同时检测多个`FD`，一旦检测到有`FD`就绪，阻塞解除，用户态程序基于这些`FD`与客户端通信
   > - 有什么：`select`、`poll`、`epoll`
   > - 对比：
   >   - 多进程/多线程并发：
   >     - 主：`accept`监听客户端连接请求（阻塞）
   >     - 子：与建立连接的客户端进行通信（阻塞`read/recv/write/send`））
   >   - `IO`多路复用：
   >     - 委托内核检测所有的`FD`（阻塞，通信、监听），就绪时解除阻塞，传回用户态
   >     - 判断`FD`：
   >       - 监听：`accept`建立连接（不会阻塞，因为`FD`已就绪）
   >       - 通信：`read/recv/write/send`（不会阻塞，读写缓冲区可读或可写）

6. `Select`

   > - 跨平台
   >
   > - 检测`FD`对应的读写缓冲区的状态（读缓冲区、写缓冲区、读写异常），通过传入传出参数记录
   >
   > - `int select(int nfds, fd_set *readfds, fd_set *writefds, fd_set *exceptfds, struct timeval *timeout);`
   >
   > - `fd_set`操作函数：
   >   - `void FD_CLR(int fd, fd_set *set);`将对应的标志位置0
   >   - `int FD_ISSET(int fd, fd_set *set)`判断`fd`是否在`set`集合中
   >   - `void FD_SET(int fd, fd_set *set)`将对应的标志位置1
   >   - `void FD_ZERO(fd_set *set)`将`set`的所有标志位置0，用于初始化
   >   
   > - `fd_set`的实质：128字节的`bitmap`位图，每个`bit`对应着一个文件描述符，上限1024，且0,1,2被占用
   >
   > - 当`fd`未就绪，进程会挂到进程等待队列上，等待网卡上有数据拷贝到内核环形缓冲区上时，由内核来唤醒
   >
   > - 内核使用poll方法遍历集合`fd_set`时，如果被检测的`FD`对应的缓冲区的数据变化，将修改`FD`对应的`fd_set`标志位
   >
   > - 执行原理：
   >   - 将当前进程的所有`FD`，一次性从用户态拷贝到内核态
   >   
   >   - 在内核态中快速的无差别遍历每个`FD`，判断是否有数据到达
   >   
   >   - 将所有`FD`状态，从内核态拷贝到用户态，返回就绪的`FD`的个数
   >   
   >   - 在用户态遍历具体哪个`FD`就绪，进行相应的事件处理
   >   
   >   - ==select底层机制==
   >   
   >     > ```c
   >     > // 通过socket函数创建一个用于监听的文件描述符lfd
   >     > int lfd = socket(PF_INET, SOCK_STREAM, 0);
   >     > bind(lfd, (struct sockaddr*)&address, sizeof(address));
   >     > listen(lfd, 5);
   >     > // 创建一个bitmap来具体表示文件描述符的状态（被关注、被监听）
   >     > fd_set read_set;
   >     > // 使用for + accept建立了5个客户端连接，将用于通信的fd放入一个数组中
   >     > for(int i = 0; i < 5; ++i)
   >     > {
   >     >     fd[i] = accept(lfd, (struct sockaddr*)&client, &addr_len);
   >     >     if(fd[i] > max)	max = fd[i];
   >     > }
   >     > while(1)
   >     > {
   >     >     // 将要检测的文件描述符集合置空，然后对fd数组的文件描述符设置关注
   >     >     FD_ZERO(&read_fds);
   >     >     for(int i = 0; i < 5; ++i)
   >     >     {
   >     >         FD_SET(fd[i], &read_fds);
   >     >     }
   >     > }
   >     > // 使用select来检测读文件描述符集合的状态，最后一个参数是超时（>0）
   >     > ret = select(max + 1, &read_fds, NULL, NULL, NULL);
   >     > for(int i = 0; i < 5; ++i)
   >     > {
   >     >     if(FD_ISSET(fd[i], &read_fds))
   >     >     {
   >     >         ret = recv(fd[i], buff, sizeof(buff) - 1, 0);
   >     >     }
   >     > }
   >     > ```
   >     >
   >     > select的底层流程：
   >     >
   >     > 1. **用户态 → 内核态**
   >     >
   >     > - 你调用 `select`，把 `fd_set` 传给内核。
   >     > - 内核会把这份 **bitmap（位图）** 从用户空间拷贝到内核空间。
   >     >
   >     > 2. **注册等待**
   >     >
   >     > - 内核检查每个 fd（这里是 socket），看它的“等待队列”是否就绪。
   >     > - 如果都没有事件，它会把当前进程放到这些 fd 对应的 **等待队列**里，并让进程进入 **阻塞态（TASK_INTERRUPTIBLE）**。
   >     >
   >     > 3. **等待事件发生**
   >     >
   >     > - 当网卡接收到数据 → 触发中断（softirq）。
   >     > - 驱动程序通过 **DMA** 把数据搬到内核的接收缓冲区。
   >     > - TCP/IP 协议栈处理后，把数据放到 socket 的 **接收队列**。
   >     > - 内核会标记这个 socket 为 **可读**，并唤醒等待在这个 socket 等待队列上的进程。
   >     >
   >     > 4. **解除阻塞**
   >     >
   >     > - 被唤醒的进程从等待队列中移除。
   >     > - 内核重新检查所有关注的 fd，把就绪的 fd 标记到 `fd_set` 里。
   >     > - 把更新后的 `fd_set` 从 **内核空间拷贝回用户空间**。
   >     > - `select` 返回，就绪的 fd 数量作为返回值。
   >     >
   >     > 5. **用户态遍历**
   >     >
   >     > - 你用 `FD_ISSET(fd[i], &read_fds)` 检查哪些 fd 就绪，然后调用 `recv` 读取数据。
   >   
   > - 缺点：
   >   - 待检测集合需要频繁在用户区和内核区间进行拷贝
   >   - 需要线性检测所有的待检测集合
   >   - `FD`为`bitmap`，能够检测的最大文件描述符有上限
   >   - `fd_set`不能重用，每次循环需要重新创建

7. `poll`

   > - 对比`select`：
   >   - 内核对`FD`的检测的方式：都是线性轮询
   >   - 检测的`FD`集合：都会频繁地在用户区和内核区拷贝，随文件描述符数量增加效率降低
   >   - `FD`上限：`poll`没有限制
   >   - 跨平台：`poll`只能在`UNIX`上使用
   >   - 可以注册更多的事件（POLLIN、POLLOUT、POLLERR、POLLHUP）
   >   
   > - `int poll(struct pollfd *fds, nfds_t nfds, int timeout);`
   >   
   > - 执行原理
   >
   >   > ```c
   >   > struct pollfd
   >   > {
   >   >     int fd;			// 文件描述符
   >   >     short events;	// 注册的事件，输入参数
   >   >     short revents;	// 实际发生的事件，由内核填充，输出参数
   >   > }
   >   > int sscfd = socket(PF_INET, SOCK_STREAM, 0);
   >   > bind(sscfd, (struct sockaddr*)&address, sizeof(address));
   >   > listen(sscfd, 4);
   >   > struct pollfd fds[4];
   >   > for(int i = 0; i < 4; ++i)
   >   > {
   >   >     fds[i].fd = accept(sscfd, (struct sockaddr*)&client, &addr_len);
   >   >     fds[i].events = POLLIN;	// 可读事件
   >   > }
   >   > while(1)
   >   > {
   >   >     ret = poll(fds, 4, 4000);
   >   >     for(int i = 0; i < 4; ++i)
   >   >     {
   >   >         if(fds[i].revents & POLLIN)
   >   >         {
   >   >             fds[i].revents = 0;
   >   >             ret = recv(fds[i].fd, buff, sizeof(buff) - 1, 0);
   >   >         }
   >   >     }
   >   > }
   >   > ```
   >   >
   >   > 1. **用户态调用 poll**：
   >   >     传入一个 `pollfd` 结构体数组
   >   >
   >   > 2. **拷贝到内核态**：
   >   >     内核会把这个数组拷贝进来，转化为一个 **链表结构（poll_list）**，因为用户可能传入很多 fd，内核要能分块存储。
   >   >
   >   > 3. **遍历 fd，挂到等待队列**：
   >   >     内核遍历所有 fd，对每个 fd 调用对应驱动/文件的 `poll` 回调函数，并把当前进程加入 fd 对应的 **等待队列**。
   >   >
   >   >    - 如果此时某个 fd 已经就绪，直接标记 `revents`。
   >   >
   >   >    - 如果没有就绪，就阻塞进程。
   >   >
   >   > 4. **事件就绪 / 超时**：
   >   >
   >   >    - 当某个 fd 有数据到达时，会唤醒等待队列里的进程 → poll 被唤醒。
   >   >
   >   >    - 如果超时时间到了，poll 也会返回。
   >   >
   >   > 5. **返回用户态**：
   >   >     poll 把所有 fd 的就绪事件写回 `pollfd.revents`，拷贝回用户态。应用程序再去处理具体的 fd。

8. `epoll`

   > - `eventpoll`
   >
   > - 对比
   >
   >   - `epoll`基于红黑树管理待检测集合
   >   - `epoll`使用回调机制，不使用轮询
   >   - `epoll`在内核区和用户区使用共享内存，省去了内存拷贝
   >   - `epoll`可以直接得到已就绪的`FD`集合，无需再次检测
   >
   > - 三个重要函数
   >
   >   - `int epoll_create(int size);`创建`epoll`实例，通过红黑树管理待检测集合
   >   - `int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);`管理红黑树上的`FD`
   >   - `int epoll_wait(int epfd, struct epoll_event *events, int maxevents, int timeout);`检测`epoll`树是否有就绪的`FD`
   >
   > - `epoll`高效的原因：将管理待检测任务与阻塞线程进程进行解耦
   >
   > - 结构体
   >
   >   - ```c
   >     struct union epoll_data
   >     {
   >         void *ptr;
   >         int fd;
   >         uint32_t u32;
   >         uint64_t u64;
   >     } epoll_data_t;	// 常用其中的 fd, 用于存储待检测的fd的值
   >     struct epoll_event
   >     {
   >         uint32_t events;	// EPOLL_CTL_ADD、MOD、DEL
   >         epoll_data_t data;
   >     };
   >     int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);
   >     ```
   >
   > - 循环调用`epoll_wait`，得到一个就绪列表，通过第二个参数（数组）传出，如果无法全部传出，在下一次`epoll_wait`传出
   >
   > - 工作模式
   >
   >   - 水平触发模式`LT`：默认，非阻塞/阻塞，当`FD`就绪，内核就一直通知使用者
   >   - 边沿触发模式`ET`：非阻塞，当`FD`就绪，只通知一次
   >   
   > - 执行原理
   >
   >   > - 在epoll_ctl中，为每个fd都指定了回调函数，基于回调函数将就绪事件放到就绪队列中
   >   > - 只需要在epoll_ctl时传递一次fd，epoll_wait不需要再次传递fd
   >   > - epoll基于红黑树+双向就绪链表存储事件，没有最大连接数的限制，不存在C10K问题
   >   > - epoll没有采用MMAP零拷贝技术
   >   >
   >   > ```c
   >   > struct epoll_event events[5];
   >   > // 创建epoll实例，返回一个fd，用于管理后续注册的fd
   >   > int epoll_fd = epoll_create(5);
   >   > for(int i = 0; i < 5; ++i)
   >   > {
   >   >     // 使用epoll_ctl将每个客户端socket加入epoll实例，分配一个epitem，关注可读事件
   >   >     struct epoll_event event;
   >   >     event.data.fd = accept(sscfd, (struct sockaddr*)&client, &addr_len);
   >   >     event.events = EPOLLIN;
   >   >     epoll_ctl(epoll_fd, EPOLL_CTL_ADD, event.data.fd, &event);
   >   > }
   >   > while(1)
   >   > {
   >   >     int ret = epoll_wait(epoll_fd, events, 5, 2000);
   >   >     for(int i = 0; i < ret; ++i)
   >   >     {
   >   >         if(events[i].events & EPOLLIN)
   >   >         {
   >   >             recv(sockfd, buf, BUFFER_SIZE - 1, 0);
   >   >         }
   >   >     }
   >   > }
   >   > ```
   >   >
   >   > - 内核数据结构：
   >   >   - 红黑树rb-tree：存储所有关注的fd，可以快速处理fd
   >   >   - 就绪链表ready-list：只存储已就绪的fd，当事件发生时插入链表，无需扫描所有fd
   >   > - 内核执行流程：
   >   >   - 检查就绪链表，如果有fd，直接返回，不遍历红黑树
   >   >   - 如果就绪队列空，将当前进程挂在epoll实例的进程等待队列上，阻塞等待事件
   >   >   - 某个socket可读可写时，内核中断触发，对应fd会被插入到就绪链表上，唤醒等待epoll_wait的进程
   >   >   - epoll_wait拷贝就绪fd信息到用户态的events数组
   >   >   - 用户直接遍历就绪事件数组，只处理真正有事件的fd
   >   > - eventpoll有三个字段：
   >   >   - ready_list：已就绪的文件描述符双链表
   >   >   - rbt：红黑树，管理所有socket连接
   >   >   - wq：等待队列，当某个进程关注的事件未就绪的时候，当前进程的fd和回调函数，当软中断到达时，来找到阻塞进程
   >   > - epitem：
   >   >   - 对应的红黑树节点
   >   >   - 对应的fd
   >   >   - 对应的epoll实例
   >   >   - 对应的进程等待队列

9. `shutdown`和`close`的区别

   > - close关闭文件描述符，socket本身
   >   - 如果是最后一个引用该socket的fd，会释放资源
   >   - 如果基于TCP，则发送FIN给对端
   >   - 如果有未发送的数据，尝试在内核缓冲区发送完再释放
   > - shutdown半关闭socket，可以控制读写方向
   >   - socket依然存在，fd继续可用
   >   - 参数how：SHUT_RD、SHUT_WR、SHUT_RDWR

10. socket是什么

    > - socket是网络通信的断点，是应用层与传输层/网络层的接口，使得不同主机或进程间能够进行数据交换
    > - 应用程序通过socket API发送和接收数据，而内核处理TCP/IP协议细节
    > - socket与fd：
    >   - fd是内核对进程打开资源（文件、管道、设备、socket）的抽象，本质是个整数，用来索引内核的文件表
    >   - socket是特殊的fd，当调用socket时，内核返回一个fd，指向内核内部的socket结构（网络通信端口）
    >   - 通过socket返回的fd，应用程序可以读写网络数据

11. read、write与send、recv

    > - 前者是通用IO，指从fd读写数据，不支持额外flags
    > - 后者是socket专用，从socket读写数据，可指定flags（MSG_PEEK等）

12. 字节序转换

    > 大端：高位字节存储在低地址：0x12345678 → 内存顺序 `[12 34 56 78]`
    >
    > 小端：低位字节存储在低地址：0x12345678 → 内存顺序 `[78 56 34 12]`
    >
    > - 网络协议规定使用大端序（big-Endian），网络字节序，保证不同cpu架构之间统一的多字节传输标准

13. 