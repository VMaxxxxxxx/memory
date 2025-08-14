- **应用编程教科书式精要：文件`IO` / `Socket` / 进程-线程 / `IPC`**

- 文件`IO`
  1. 定义与语义
    - 标准`IO`：`stdio`族，用户态缓冲；适合文本/少量`IO`。
    - 系统`IO`：`open/read/write/close`直面内核；配合`O_NONBLOCK/O_DIRECT`等标志。
    - 阻塞 vs 非阻塞：是否在数据未就绪时挂起；非阻塞返回`EAGAIN/EWOULDBLOCK`。
    - 同步 vs 异步：调用是否等待完成；异步可用`io_uring/AIO`等。
    - 持久化：`write`进入页缓存；`fsync/fdatasync/O_SYNC`控制落盘边界；`rename`原子替换。
  2. 关键接口（按时序）
    - `open(path, flags, mode)`：`O_CREAT/O_TRUNC/O_APPEND/O_CLOEXEC/O_NONBLOCK`。
    - `read/write`：可能短读/短写；`EINTR`需重试；非阻塞配合多路复用。
    - `lseek/pread/pwrite`：偏移管理；`pread/pwrite`不改文件偏移，利于并发。
    - `readv/writev`聚散`IO`；`sendfile/splice`零拷贝；`mmap/munmap/msync`内存映射。
    - `close`：注意`fd`被重用，悬挂句柄导致隐性错误。
  3. 正确性与性能
    - 健壮读写：循环直至完成或致命错误；封装`read_n/write_n`。
    - 刷盘策略：关键元数据前置`fsync`；事务式“写→`fsync`→`rename`”。
    - 缓冲一致：避免`stdio`与`syscall`混用；必要时`setvbuf`。
    - 大文件/洞文件：`lseek`制造空洞；注意备份校验工具行为。
  4. 面试要点
    - 短读/短写、`EINTR`处理；`O_APPEND`原子性；`mmap`与`read/write`取舍；`fd`继承与`FD_CLOEXEC`。

- `Socket`编程
  1. 模型与语义
    - `TCP`：面向连接，字节流，无边界；可靠、有序、拥塞控制。
    - `UDP`：无连接，报文有边界；可能丢包/乱序；适合实时/简单请求答复。
  2. 典型时序
    - 服务端：`socket`→`setsockopt(SO_REUSEADDR[/SO_REUSEPORT])`→`bind`→`listen`→`accept`→`IO`→`close`。
    - 客户端：`socket`→`connect`→`IO`→`close`。
  3. 关键选项与细节
    - 地址与字节序：`sockaddr_in/6`，`htons/htonl/ntohs/ntohl`。
    - 常用选项：`SO_REUSEADDR/REUSEPORT`、`SO_KEEPALIVE`、`TCP_NODELAY`、`SO_RCVBUF/SO_SNDBUF`。
    - 半关闭：`shutdown(RD/WR/RDWR)`实现只读/只写关闭。
  4. 非阻塞与多路复用
    - 非阻塞`connect`：等待可写后用`getsockopt(SO_ERROR)`判定成功。
    - `select/poll`复杂度`O(n)`；`epoll`更可扩展：`LT`安全、`ET`需“读空/写空”到`EAGAIN`。
  5. `TCP`/`UDP`考点
    - 三握四挥，`TIME_WAIT`（主动关闭，防旧包/序列安全）、`CLOSE_WAIT`（被动关闭未`close`）。
    - 粘包/拆包：应用层定界（定长/分隔符/长度前缀/TLV）。
    - `UDP`分片与`MTU`：必要时应用层分片与重传；`connect(UDP)`可固定对端、简化接口。
  6. 面试要点
    - 惊群与`SO_REUSEPORT`；优雅关闭流程；`send`成功但未达的排查路径；`Nagle`与延迟确认对时延的影响。

- 进程与线程模型
  1. 进程
    - 创建/执行：`fork/execve`；写时复制`COW`；`wait/waitpid`回收避免僵尸，孤儿由`init`接管。
    - 资源：地址空间、`fd`表复制但共享底层对象（引用计数）。
  2. 线程
    - 线程共享进程资源；创建/终止：`pthread_create/join/detach`；`TLS`与栈。
  3. 同步与并发
    - 互斥/自旋/读写锁/条件变量/信号量；原子与内存序（`acquire-release`构建`happens-before`）。
    - 典型问题：死锁/活锁/饥饿/伪共享/`ABA`；`while`等待条件变量防虚假唤醒。
  4. 模型与调度
    - `1:1`/`N:1`/`M:N`；线程池、Reactor/Proactor、进程+共享内存；CPU亲和/NUMA。
  5. 信号与中断
    - `sigaction/sigprocmask`；异步信号安全函数；对`IO`的`EINTR`影响。
  6. 面试要点
    - 守护进程与`setsid`；锁选择：自旋 vs 互斥；可见性与伪共享隔离（缓存行对齐）。

- `IPC`机制
  1. 管道/`FIFO`：半双工、字节流、内核缓冲有限；父子或跨进程（`FIFO`）。
  2. 消息队列：具消息边界、可优先级；受内核队列限制；适合控制面。
  3. 共享内存：最高吞吐；需锁/原子确保一致；典型为环形缓冲、生产者-消费者。
  4. 信号量：二值/计数，用于互斥或限流；注意死锁与优先级反转。
  5. 本地域套接字：低开销；支持`fd`传递；常见于同机服务化。
  6. 其它：`eventfd/timerfd/signalfd`便于接入`epoll`；`memfd`匿名共享文件。
  7. 选择指引：跨主机→`TCP/UDP`；同机高吞吐→共享内存；控制消息→消息队列；简单点对点→管道。
  8. 面试要点：共享内存一致性与内存屏障；`msync`只针对映射文件并非共享内存同步。

- 面试速答（背诵版）
  1. `TCP`粘包原因：字节流无边界；解法：定长/分隔符/长度前缀/`TLV`。
  2. `TIME_WAIT`：主动关闭方防旧包与序列安全；`CLOSE_WAIT`：被动关闭未`close`。
  3. 健壮读写：循环直至完成；处理`EINTR/EAGAIN`；必要时超时与重试策略。
  4. `fsync` vs `fdatasync`：后者仅数据及必要元数据；前者更多元数据。
  5. `select/poll/epoll`：伸缩性依次增强；`epoll ET`需读/写至`EAGAIN`。
  6. 优雅关闭：停收新请求→排空写缓冲→`shutdown(WR)`→等待对端`FIN`→`close`。
  7. `FD_CLOEXEC`：防止`exec`后句柄泄漏；与`O_CLOEXEC`在`open`时即设定更安全。


