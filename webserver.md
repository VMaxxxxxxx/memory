> - 程序启动时，会切换到指定日志输出目录`/usr/local/log`，然后以固定的端口`10001`实例化`TCPServer`，启动`TCP`服务
>   - 实例化`TCPserver`：设置字段（服务的主端口`port`、线程池工作线程数`threadNum`、**实例化主事件循环**对象`mainLoop`、**实例化线程池对象**`threadPool`（传递`mainLoop`和`threadNum`）），设置初始化监听`setListen`
>     - 初始化监听：`socket`创建用于监听的`fd`、设置端口复用`setsockopt`、`bind`绑定`ip`和`port`、**设置监听`listen`**
>   - 启动`run`：**启动线程池**，创建一个用于**连接请求检测任务**的`channel`（将`lfd`的读事件绑定到回调函数`acceptConn`上），往`mainLoop`中添加这个任务（`ChannelElem::type/channel`），**启动主`Reactor`模型**`mainLoop`
>     - 启动线程池：**实例化`ThreadNum`个工作线程**`WorkThread`，然后将子线程添加到线程池`vector`中，**启动子线程**`run`
>     - 往`mainLoop`中添加一个任务（任务类型：监听`m_lfd`，完成连接建立）（加锁、创建新任务节点、放入任务队列、解锁、由**当前子线程**去处理任务队列或**主线程唤醒对应子线程**去处理任务队列（根据`channel_map`将对应的`channel`的`fd`使用`dispatcher`的`IO`多路复用去处理`ADD、REMOVE、MODIFY`））
> - 