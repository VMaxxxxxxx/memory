1. 常用命令

   > - 文件与目录操作
   >
   >   ```shell
   >   ls             # 列出当前目录下的文件和目录，常用参数 -l（长格式）、-a（显示隐藏文件）
   >   # 示例：ls -la    显示所有文件（包括.开头隐藏文件）和详细信息（权限、时间等）
   >
   >   cd             # 改变当前工作目录（change directory）
   >   # 示例：cd /home/user    切换到指定目录
   >   # 示例：cd ..            返回上一级目录
   >   # 示例：cd ~             回到用户主目录
   >
   >   pwd            # 显示当前所在工作目录的绝对路径（print working directory）
   >   # 示例：pwd             通常用于确认当前目录路径
   >
   >   mkdir          # 创建新目录（make directory）
   >   # 示例：mkdir newdir     创建名为 newdir 的目录
   >   # 示例：mkdir -p a/b/c   递归创建多级目录
   >
   >   rmdir          # 删除空目录（remove directory）
   >   # 示例：rmdir emptydir   删除名为 emptydir 的空目录
   >   # 注意：不能删除非空目录，非空建议用 rm -r
   >
   >   rm             # 删除文件或目录（remove）
   >   # 示例：rm file.txt      删除一个文件
   >   # 示例：rm -r dir        递归删除目录及其内容
   >   # 示例：rm -rf dir       强制递归删除，不会提示（危险操作）
   >
   >   cp             # 复制文件或目录（copy）
   >   # 示例：cp a.txt b.txt   复制文件
   >   # 示例：cp -r dir1 dir2  递归复制整个目录
   >   # 常用参数：-r（递归）、-i（提示覆盖）、-u（只复制更新的）
   >
   >   scp            # 远程文件复制（secure copy），基于 SSH
   >   # 示例：scp file.txt user@192.168.1.10:/tmp     将文件复制到远程主机目录
   >   # 示例：scp -r dir user@host:/path              递归复制目录到远程
   >   # 常用于部署、拉取代码、传输数据等场景
   >
   >   mv             # 移动或重命名文件/目录（move）
   >   # 示例：mv old.txt new.txt        文件重命名
   >   # 示例：mv file.txt /tmp/         移动文件到指定目录
   >   # 也可用于替换文件
   >
   >   touch          # 创建空文件或更新文件的修改时间（timestamp）
   >   # 示例：touch new.txt             创建新空文件
   >   # 示例：touch a.txt b.txt         同时创建多个文件
   >   # 常用于临时生成测试文件或更新时间戳
   >   ```
   >
   > - 文件内容查看
   >
   >   ```shell
   >   cat filename              # 从头到尾显示整个文件内容（concatenate）
   >   # 示例：cat a.txt                      查看文本内容
   >   # 示例：cat file1 file2 > file3        合并两个文件内容到新文件
   >   # 注意：不适合查看大文件，无法翻页
   >
   >   tac filename              # 与 cat 相反，从最后一行开始显示（tac = cat 的倒序）
   >   # 示例：tac a.txt                      倒序显示文件内容
   >
   >   more filename             # 分页显示文件内容，适合查看较长文本，空格翻页，q 退出
   >   # 示例：more log.txt                  一页页查看长文件
   >   # 小技巧：/关键词 可搜索，n 跳到下一个匹配
   >
   >   less filename             # 与 more 类似但功能更强，支持前后翻页，/搜索，q 退出
   >   # 示例：less a.txt                   可上下翻动，推荐替代 more 和 cat
   >   # 小技巧：按 G 跳到文件末尾，gg 跳到开头
   >
   >   head filename             # 查看文件开头内容（默认前 10 行）
   >   # 示例：head file.txt               显示前 10 行
   >   # 示例：head -n 20 file.txt         显示前 20 行
   >
   >   tail filename             # 查看文件末尾内容（默认后 10 行）
   >   # 示例：tail file.txt               显示最后 10 行
   >   # 示例：tail -n 20 file.txt         显示最后 20 行
   >   # 示例：tail -f log.txt             实时追踪日志更新（常用于 log 查看）
   >
   >   wc filename               # 统计文件中的行数、单词数、字符数（word count）
   >   # 示例：wc file.txt                显示行、词、字符数量
   >   # 示例：wc -l file.txt             只统计行数（常用于统计行号）
   >   # 示例：wc -w file.txt             只统计单词数
   >
   >   ```
   >
   > - 文件搜索
   >
   >   ```shell
   >   find [路径] [匹配条件] [操作]       # 在目录层级中查找符合条件的文件，功能强大但较慢
   >   # 示例：find . -name "*.txt"         在当前目录及子目录下查找所有 .txt 文件
   >   # 示例：find /etc -type f            查找 /etc 下所有普通文件
   >   # 示例：find / -size +10M            查找大于 10MB 的文件
   >   # 示例：find . -mtime -1             查找最近一天修改过的文件
   >   # 示例：find . -name "*.log" -delete 删除所有 .log 文件（慎用）
   >
   >   # 常用选项：
   >   # -name "*.ext"     根据名称匹配
   >   # -type f/d         f 文件，d 目录
   >   # -size +N/-N       按大小查找，+ 为大于，- 为小于
   >   # -mtime N          按修改时间查找（单位：天）
   >   # -exec 命令 {} \;  对匹配的每个文件执行命令
   >
   >   locate 关键词                   # 在本地数据库中快速查找文件（通过 updatedb 预生成索引）
   >   # 示例：locate passwd           快速查找包含 "passwd" 的路径
   >   # 示例：locate *.conf           快速查找所有 .conf 文件
   >
   >   # 优点：查找速度极快
   >   # 缺点：结果可能不及时（需先运行 updatedb 更新索引）
   >   # 示例：sudo updatedb           更新数据库，确保 locate 数据准确
   >
   >   grep [选项] "关键词" [文件]     # 在文件中查找包含关键词的行（global regular expression print）
   >   # 示例：grep "main" test.cpp    查找 test.cpp 中包含 "main" 的行
   >   # 示例：grep -i "error" log.txt 忽略大小写查找 "error"
   >   # 示例：grep -r "TODO" ./src    递归搜索目录下所有文件中含 TODO 的行
   >   # 示例：ps aux | grep nginx     在进程中查找 nginx 进程
   >
   >   # 常用选项：
   >   # -i     忽略大小写
   >   # -n     显示匹配行的行号
   >   # -r     递归查找
   >   # -v     反向匹配，显示不包含关键词的行
   >   # -E     支持扩展正则
   >   # -c     统计出现次数
   >   ```
   >
   > - 文件权限与所有权
   >
   >   ```shell
   >   chmod [权限] 文件名              # 修改文件/目录的权限（change mode）
   >   # 两种写法：
   >   # 1）数字法（三位八进制表示 rwx 权限）
   >   # 示例：chmod 755 a.out         用户 rwx，组 r-x，其他 r-x（常用于可执行程序）
   >   # 示例：chmod 644 file.txt      用户 rw-，组 r--，其他 r--（常用于文本文件）
   >
   >   # 2）符号法（u/g/o/a 表用户/组/其他/全部，+/-/= 表添加/删除/设定权限）
   >   # 示例：chmod u+x run.sh        给用户添加执行权限
   >   # 示例：chmod go-w file.txt     去掉组和其他用户的写权限
   >   # 示例：chmod a=r myfile        所有人只读
   >
   >   # 权限位解释：
   >   # r = 读(4)，w = 写(2)，x = 执行(1)
   >   # 三位分别代表：所有者、用户组、其他人
   >
   >   chown [用户][:[组]] 文件名        # 修改文件/目录的所有者和所属组（change owner）
   >   # 示例：chown root file.txt     将文件所有者改为 root
   >   # 示例：chown user:group file   修改文件所有者和所属组
   >   # 示例：chown -R user:group dir 递归修改整个目录的属主
   >
   >   # 注意：只有 root 用户或 sudo 才能修改文件属主
   >   # 可用 ls -l 查看文件当前的所有者和所属组
   >
   >   umask                          # 设置默认权限掩码（user file creation mask）
   >   # 示例：umask                  查看当前 umask 值（如 0022）
   >   # 示例：umask 0027            设置新的默认权限掩码
   >
   >   # umask 的作用：控制新创建文件/目录的默认权限（与 chmod 相对）
   >   # 默认权限 = 权限模板 - umask
   >   # 文件默认权限模板为 666（rw-rw-rw-）
   >   # 目录默认权限模板为 777（rwxrwxrwx）
   >   # 示例：umask 0022
   >   # - 新建文件默认权限 = 666 - 022 = 644（rw-r--r--）
   >   # - 新建目录默认权限 = 777 - 022 = 755（rwxr-xr-x）
   >   ```
   >
   > - 压缩与解压
   >
   >   ```shell
   >   tar [选项] 归档文件名 源文件     # 打包归档工具（常与 gzip 联用），不默认压缩
   >   # 示例：tar -cvf a.tar dir/        将目录 dir 打包成 a.tar（不压缩）
   >   # 示例：tar -xvf a.tar             解包 a.tar 文件
   >   # 示例：tar -zcvf a.tar.gz dir/    打包并用 gzip 压缩（-z 表 gzip）
   >   # 示例：tar -zxvf a.tar.gz         解压 .tar.gz 文件
   >   # 示例：tar -jcvf a.tar.bz2 dir/   用 bzip2 压缩（-j）
   >
   >   # 常用参数：
   >   # -c 创建归档（create）
   >   # -x 解包归档（extract）
   >   # -v 显示过程（verbose）
   >   # -f 指定文件名（file）
   >   # -z 使用 gzip（gzip）
   >   # -j 使用 bzip2（bzip）
   >
   >   gzip 文件名                    # 使用 gzip 压缩单个文件（替换原文件，加 .gz 后缀）
   >   # 示例：gzip file.txt            压缩后得到 file.txt.gz，原文件被删除
   >   # 注意：不能压缩多个文件或目录，一般与 tar 联用
   >
   >   gunzip 文件.gz                 # 解压 .gz 文件（gzip 的反操作）
   >   # 示例：gunzip file.txt.gz       解压为 file.txt，默认删除压缩文件
   >
   >   zip 压缩包名.zip 文件/目录      # 创建 .zip 格式压缩包（可压多个文件或目录）
   >   # 示例：zip a.zip file.txt       压缩单个文件
   >   # 示例：zip -r a.zip dir/        递归压缩整个目录
   >
   >   unzip 压缩包.zip                # 解压 .zip 文件
   >   # 示例：unzip a.zip              解压到当前目录
   >   # 示例：unzip -d outdir a.zip    解压到指定目录 outdir
   >
   >   ```
   >
   > - 进程与资源管理
   >
   >   ```shell
   >   ps                      # 显示当前系统中的进程快照（process status）
   >   # 示例：ps                显示当前 shell 的子进程
   >   # 示例：ps -ef            显示所有进程（标准格式）
   >   # 示例：ps aux            显示所有进程（BSD 风格）
   >   # 示例：ps aux | grep nginx   查找 nginx 相关进程
   >
   >   # 常见字段：
   >   # PID 进程ID，PPID 父进程ID，UID 用户名，%CPU CPU占用，%MEM 内存占用，CMD 命令
   >
   >   top                     # 动态显示系统运行中的进程（实时更新）
   >   # 示例：top                默认显示所有进程，实时刷新
   >   # 交互命令：
   >   # - 按 `P` 排序CPU使用率，`M` 排序内存
   >   # - 按 `k` 杀进程（输入 PID）
   >   # - 按 `q` 退出
   >
   >   htop                    # top 的升级版，图形化交互界面（需安装）
   >   # 示例：htop               实时显示进程，可用方向键、空格选择，多选 kill
   >   # 需要安装：sudo apt install htop
   >
   >   kill PID                # 向指定 PID 发送信号（默认 SIGTERM）
   >   # 示例：kill 1234          终止 PID 为 1234 的进程（发送 TERM 信号）
   >   # 示例：kill -9 1234       强制终止（发送 SIGKILL 信号）
   >
   >   # 常见信号：
   >   # -15（SIGTERM）：正常终止，允许清理资源（默认）
   >   # -9（SIGKILL）：强制终止，立即杀死，无法被拦截
   >
   >   killall 进程名           # 根据进程名称杀死所有匹配的进程
   >   # 示例：killall firefox     杀掉所有名为 firefox 的进程
   >   # 注意：不区分大小写，慎用！
   >
   >   nice -n N 命令          # 设置命令运行时的初始优先级（默认是 0，范围 -20 ~ 19，越小越优先）
   >   # 示例：nice -n 10 ./task   以较低优先级运行程序
   >   # 说明：非 root 用户只能设置正 nice 值（降低优先级）
   >
   >   renice -n N -p PID       # 修改已运行进程的优先级（re-nice）
   >   # 示例：renice -n 5 -p 1234 将 PID 为 1234 的 nice 值设为 5
   >   # 注意：只能提升自己进程的 nice 值，降低需要 root 权限
   >
   >   jobs                    # 查看当前 shell 的后台任务列表
   >   # 示例：jobs               列出当前会话中运行/暂停的作业
   >   # 输出：[1]+  Running ./script.sh &
   >
   >   # 配套命令：
   >   # fg %1  将后台任务恢复到前台
   >   # bg %1  将暂停任务继续在后台运行
   >   # ctrl+z 将前台任务挂起（变为后台暂停）
   >
   >   ```
   >
   > - 网络命令
   >
   >   ```shell
   >   ping [目标主机]                  # 检测网络连通性（ICMP 回显请求）
   >   # 示例：ping www.baidu.com       连续发送 ICMP 包检测延迟与丢包
   >   # 示例：ping -c 4 8.8.8.8        发送 4 个包后停止（-c 是 count）
   >   # 常用于排查网络是否通、DNS 是否可解析
   >
   >   ifconfig                         # 查看和配置网络接口（旧工具）
   >   # 示例：ifconfig                  查看当前网卡配置（IP/MAC/状态）
   >   # 示例：ifconfig eth0 down       禁用指定网卡（需要 sudo）
   >   # 注意：现代系统推荐用 ip 命令替代
   >
   >   ip [子命令]                      # 查看/配置网络接口、路由、地址（新一代网络工具）
   >   # 示例：ip addr                  查看 IP 地址（推荐替代 ifconfig）
   >   # 示例：ip link                  查看网络接口状态
   >   # 示例：ip route                 查看路由表
   >   # 示例：ip a                    是 ip addr 的简写
   >
   >   netstat                          # 显示网络连接、监听端口、路由表等（已废弃）
   >   # 示例：netstat -an              显示所有连接（a）和端口（n 不反解析）
   >   # 示例：netstat -tuln            显示监听的 TCP/UDP 端口
   >   # 替代工具：`ss`
   >
   >   ss                               # 比 netstat 更快，显示 socket 信息（socket statistics）
   >   # 示例：ss -tuln                 显示监听端口（TCP/UDP + no resolve）
   >   # 示例：ss -anp                  显示所有连接和进程（包括 PID）
   >   # 常用于查看服务监听端口、排查占用端口的进程
   >
   >   curl [URL]                       # 发送 HTTP 请求，支持 GET/POST 等（轻量级）
   >   # 示例：curl https://example.com      拉取网页内容
   >   # 示例：curl -I https://baidu.com     查看响应头（-I 是 HEAD 请求）
   >   # 示例：curl -d "a=1&b=2" -X POST ... 发送 POST 请求
   >
   >   wget [URL]                       # 从网络下载文件（命令行下载器）
   >   # 示例：wget https://xxx.com/file.zip  下载文件
   >   # 示例：wget -c xxx.zip                断点续传
   >   # 适合下载大文件，不适合接口调试
   >
   >   ssh user@host                   # 远程登录服务器（Secure Shell）
   >   # 示例：ssh root@192.168.1.100       使用用户名远程登录主机
   >   # 示例：ssh -p 2222 user@host        指定端口（默认22）
   >   # 示例：ssh -i id_rsa.pem user@host  使用密钥认证
   >   # 面试常问：ssh 密钥登录 vs 密码登录、安全性、免密配置方式
   >
   >   ```
   >
   > - 软件包管理
   >
   >   ```shell
   >   apt update                     # 更新软件源索引（不安装软件）
   >   # 示例：sudo apt update         从 /etc/apt/sources.list 中拉取最新的包信息
   >   # 用于刷新缓存，让 apt 知道有哪些包是最新版本（必须定期执行）
   >
   >   apt install 包名               # 安装指定软件包及其依赖
   >   # 示例：sudo apt install curl    安装 curl 工具
   >   # 示例：sudo apt install nginx -y 自动确认安装（-y）
   >   # 可一次安装多个包：sudo apt install git vim htop
   >
   >   apt remove 包名                # 删除软件包，但保留配置文件（如 /etc/ 配置）
   >   # 示例：sudo apt remove nginx     删除 nginx 但保留配置
   >   # 注意：软件包依赖不会自动清理，配置残留仍存在
   >
   >   apt purge 包名                 # 删除软件包及其配置文件（比 remove 更彻底）
   >   # 示例：sudo apt purge nginx      删除 nginx 及其配置文件
   >
   >   apt autoremove                # 自动删除无用的依赖（孤立包）
   >   # 示例：sudo apt autoremove       清理 remove 后遗留的依赖库
   >
   >   dpkg -i xxx.deb               # 安装本地 .deb 包（Debian 安装包）
   >   # 示例：sudo dpkg -i mypkg.deb    安装本地下载的 .deb 文件
   >   # 注意：不处理依赖项（可能报错），通常后续需要 `apt -f install` 修复
   >
   >   dpkg -r 包名                  # 移除已安装的 .deb 包（不清配置）
   >   # 示例：sudo dpkg -r nginx
   >
   >   dpkg -l                       # 列出已安装的软件包
   >   # 示例：dpkg -l | grep nginx       查看是否安装 nginx
   >
   >   dpkg -s 包名                  # 查看指定包的状态信息
   >   # 示例：dpkg -s curl              查看 curl 的安装状态和版本
   >
   >   dpkg -L 包名                  # 查看某个已安装包包含的文件路径列表
   >   # 示例：dpkg -L bash              查看 bash 安装了哪些文件
   >
   >   ```
   >
   > - 磁盘管理
   >
   >   ```shell
   >   df                          # 显示文件系统磁盘空间使用情况（disk free）
   >   # 示例：df -h                 以人类可读格式显示磁盘使用（单位自动转换）
   >   # 示例：df -T                 显示文件系统类型
   >   # 常用于查看磁盘剩余容量和挂载点
   >
   >   du [路径]                   # 估算文件或目录占用的磁盘空间（disk usage）
   >   # 示例：du -sh /var/log       显示 /var/log 总大小（-s 汇总，-h 人类可读）
   >   # 示例：du -h --max-depth=1   显示当前目录下一级文件夹大小
   >   # 用于查找大文件/目录占用空间
   >
   >   mount [设备] [挂载点]      # 挂载设备到指定目录（mount filesystem）
   >   # 示例：mount /dev/sdb1 /mnt  将 /dev/sdb1 挂载到 /mnt 目录
   >   # 示例：mount                查看当前所有挂载点和文件系统
   >   # 需要 root 权限，一般自动挂载由系统管理
   >
   >   umount [挂载点或设备]       # 卸载已挂载的文件系统
   >   # 示例：umount /mnt           卸载 /mnt 挂载点
   >   # 示例：umount /dev/sdb1      卸载设备
   >   # 注意：卸载前确保目录没有被使用（文件打开或当前目录）
   >
   >   lsblk                      # 列出块设备信息（列出硬盘分区和挂载点）
   >   # 示例：lsblk                 显示系统所有块设备、大小、挂载点
   >   # 示例：lsblk -f              显示文件系统类型和 UUID
   >   # 方便查看磁盘分区结构和挂载状态
   >
   >   ```
   >
   > - 其他
   >
   >   ```shell
   >   man 命令名                    # 查看命令的帮助文档（manual）
   >   # 示例：man ls                 查看 ls 的用法、参数
   >   # 快捷键：q 退出，/搜索，n 下一个匹配，h 帮助
   >   # 面试常问：如何在不知道用法时查询命令帮助？答：man 或 --help
   >
   >   alias 名='命令'               # 设置命令别名（简化常用命令）
   >   # 示例：alias ll='ls -alF'     设置 ll 为带详细信息的 ls
   >   # 示例：alias grep='grep --color=auto'
   >   # 查看所有别名：alias
   >   # 永久生效：添加到 ~/.bashrc 后执行 source ~/.bashrc
   >
   >   history                      # 查看历史命令记录（带编号）
   >   # 示例：history | grep ssh     搜索历史中执行过的 ssh 命令
   >   # 快捷键：↑↓ 查历史，`!n` 执行第 n 条命令
   >   # 示例：!123                    执行编号为 123 的命令
   >
   >   date                         # 查看或设置当前系统时间（数据校正）
   >   # 示例：date                    显示当前日期和时间
   >   # 示例：date "+%Y-%m-%d %H:%M"  自定义格式显示
   >   # 示例：date -s "2025-08-01"    设置系统日期（需要 root）
   >
   >   cal                          # 显示日历（calendar）
   >   # 示例：cal                     显示当前月的日历
   >   # 示例：cal 2025                显示 2025 年的日历
   >   # 示例：cal 8 2025              显示 2025 年 8 月的日历
   >
   >   echo [内容]                  # 输出文本到终端，可用于变量显示、调试
   >   # 示例：echo "Hello World"      输出字符串
   >   # 示例：echo $HOME              输出变量内容
   >   # 示例：echo -e "a\nb"          启用转义字符（-e）
   >
   >   read 变量名                  # 从终端读取用户输入（阻塞）
   >   # 示例：read name               读取用户输入并赋值给变量 name
   >   # 示例：read -p "请输入姓名：" name   带提示信息
   >   # 示例：read -s password        隐藏输入内容（密码）
   >
   >   source 文件名                # 在当前 shell 中执行脚本文件内容（不新开子 shell）
   >   # 示例：source ~/.bashrc        应用配置变更（别名、变量等）
   >   # 示例：. filename              与 source 等效（点号加空格）
   >
   >   export 变量=值               # 设置环境变量并导出到子进程
   >   # 示例：export PATH=$PATH:/opt/bin    添加环境变量路径
   >   # 示例：export MYVAR=123              设置变量并可用于子 shell
   >   # 用于 shell 脚本传参或配置全局变量
   >
   >   ```
   >
   > - 文本处理三剑客
   >
   >   ```shell
   >   grep	# 文本搜索过滤工具（只能匹配显示，不能修改），查找匹配的行，支持正则
   >   grep [option] 'pattern' file
   >   # =======================================================================================
   >   grep "main" test.cpp        # 查找包含 main 的行
   >   grep -n "TODO" *.cpp        # 显示行号
   >   grep -r "init" ./src        # 递归查找
   >   grep -v "error" log.txt     # 排除匹配（不含 error 的行）
   >   # =======================================================================================
   >   
   >   awk		# 文本分析工具（只能匹配，不能修改），以行为单位读取文本，并按字段（列）进行处理，适合格式化输出、统计、批量替换等，格式化输出
   >   awk 'pattern { action }' file
   >   # =======================================================================================
   >   awk '{print $1}' file.txt         # 打印第一列
   >   awk -F ':' '{print $1, $3}' /etc/passwd  # 自定义分隔符为冒号，打印第1和第3列
   >   awk '$3 > 50 {print $1}' data.txt # 第三列大于50时打印第一列
   >   awk 'BEGIN{sum=0} {sum+=$2} END{print sum}' file.txt  # 求第二列总和
   >   # =======================================================================================
   >   
   >   sed		# 文本行编辑器（可以匹配，可以修改），流编辑器，查找、替换、删除行、批量改动
   >   sed [option] 'operation' file
   >   # =======================================================================================
   >   sed 's/cat/dog/' file.txt         # 每行的第一个 cat 替换成 dog
   >   sed 's/cat/dog/g' file.txt        # 每行的所有 cat 全部替换成 dog
   >   sed '/DEBUG/d' log.txt            # 删除包含 DEBUG 的行
   >   sed '2a\hello' file.txt           # 在第 2 行后添加 hello
   >   sed -i 's/foo/bar/g' file.txt     # 原地修改（注意 -i）
   >   # =======================================================================================
   >   ```
   >   
   > - 模块处理
   >
   >   ```shell
   >   lsmod                     # 查看当前系统加载的内核模块（list modules）
   >   # 示例输出：
   >   # Module                  Size  Used by
   >   # e1000e                249856  0
   >   # i2c_algo_bit           16384  1 e1000e
   >       
   >   insmod mydriver.ko       # 插入（加载）一个内核模块（insert module）
   >   # 要求提供的是已编译好的 .ko 文件
   >   # 示例：sudo insmod mydriver.ko
   >   # 没有输出，成功后用 lsmod 可看到模块已加载
   >       
   >   rmmod 模块名             # 卸载一个内核模块（remove module）
   >   # 示例：sudo rmmod mydriver
   >   # 没有输出，卸载后 lsmod 中将不再出现该模块
   >       
   >   modprobe 模块名          # 自动处理依赖关系加载模块（推荐替代 insmod）
   >   # 示例：sudo modprobe snd_hda_intel
   >   # 会自动加载 snd_hda_intel 及其依赖模块
   >       
   >   modinfo 模块名或.ko文件   # 查看模块的详细信息（作者、描述、许可等）
   >   # 示例：modinfo mydriver.ko
   >   # 输出：
   >   # filename:       mydriver.ko
   >   # license:        GPL
   >   # author:         your_name
   >   # description:    A test kernel module
   >       
   >   dmesg                    # 查看内核日志缓冲区（debug message）
   >   # 示例：dmesg | tail -20
   >   # 常用于查看模块加载/卸载打印信息，如：
   >   # [12345.678901] mydriver: module loaded
   >   # [12347.234123] mydriver: device registered
   >       
   >   uname -r                # 查看当前运行的内核版本（用于匹配模块）
   >   # 示例：uname -r
   >   # 输出：
   >   # 5.15.0-86-generic
   >       
   >   depmod                  # 生成模块依赖关系索引（通常安装模块后执行）
   >   # 示例：sudo depmod
   >   # 自动扫描 /lib/modules/$(uname -r)/ 目录生成 modules.dep
   >       
   >   ```
   >

2. 标准的启动程序

   > 在嵌入式Linux系统中，一个标准的文件服务程序启动脚本通常会放在`/etc/init.d/`或`/etc/rc.d/init.d/`目录下，用于在系统启动时自动运行某个守护程序（如`NFS`、`Samba`或者其他自定义的程序）
   >
   > `# /etc/init.d/S00MyApp`
   >
   > ```shell
   > #!/bin/sh
   > DAEMON_PATH="/usr/bin"		# 可执行文件路径
   > DAEMON="eGTouch"		   # 可执行文件名
   > DAEMON_OPTS="-d"		# 传给eGTouch的参数 -d 表示后台运行
   > 
   > NAME=eGTouch	# 服务名
   > PIDFILE=/var/run/$NAME.pid	 # PID文件位置
   > SCRIPTNAME=/etc/init.d/$NAME	# 脚本路径
   > # 标准库提供函数（log_daemon_msg打印服务启动提示，log_end_msg $?打印执行结果），. 相当于source
   > # 为什么用库函数，是因为更格式化，更结构化，有return code
   > . /lib/lsb/init-functions
   > 
   > do_start(){
   > 	log_daemon_msg "Starting $NAME"
   > 	# 以守护进程形式启动/停止服务，--start启动进程，--background在后台运行，
   > 	# --pidfile指定pid文件位置，--make-pidfile自动创建pid文件，--exec指定要运行的可执行文件路径
   > 	# -- 分隔符，后面是传递给目标程序的参数
   > 	start-stop-daemon --start --background --pidfile $PIDFILE --make-pidfile\
   > 		--exec $DAEMON_PATH/$DAEMON -- $DAEMON_OPTS
   > 	log_end_msg $?
   > }
   > do_stop(){
   > 	log_daemon_msg "Stopping $NAME"
   > 	# 用于结束进程，默认找PIDFILE中的PID，强制终止，最多重试5次
   > 	start-stop-daemon --stop --pidfile $PIDFILE --retry 5
   > 	log_end_msg $?
   > }
   > do_restart(){
   > 	do_stop
   > 	do_start
   > }
   > case "$1"in
   > start)
   > 	do_start
   > 	;;
   > stop)
   > 	do_stop
   > 	;;
   > restart|force-reload)
   > 	do_restart
   > 	;;
   > status)
   > 	status_of_proc "$DAEMON_PATH/$DAEMON" "$NAME" && exit 0 || exit $?
   > 	;;
   > *)
   > 	echo "Usage: $SCRIPTNAME {start|stop|restart|status}"
   > 	exit 1
   > 	;;
   > esac
   > exit 0
   > ```

3. 正则表达式

   > ###### 1. 元字符（基础）
   >
   > | 符号    | 含义                          |
   > | ------- | ----------------------------- |
   > | `.`     | 匹配除换行外的任意单个字符    |
   > | `^`     | 匹配行首                      |
   > | `$`     | 匹配行尾                      |
   > | `*`     | 匹配前一个元素 **零次或多次** |
   > | `+`     | 匹配前一个元素 **一次或多次** |
   > | `?`     | 匹配前一个元素 **零次或一次** |
   > | `{n}`   | 恰好匹配 n 次                 |
   > | `{n,}`  | 至少匹配 n 次                 |
   > | `{n,m}` | 匹配 n 到 m 次                |
   >
   > ---
   >
   > ###### 2. 字符类
   > | 表达式   | 含义                                |
   > | -------- | ----------------------------------- |
   > | `[abc]`  | 匹配 a、b 或 c 中任意一个字符       |
   > | `[^abc]` | 匹配 **不** 是 a、b 或 c 的任意字符 |
   > | `[a-z]`  | 匹配小写字母 a 到 z                 |
   > | `[A-Z]`  | 匹配大写字母 A 到 Z                 |
   > | `[0-9]`  | 匹配数字 0 到 9                     |
   > | `\d`     | 数字，等价 `[0-9]`                  |
   > | `\D`     | 非数字，等价 `[^0-9]`               |
   > | `\w`     | 单词字符（字母、数字、下划线）      |
   > | `\W`     | 非单词字符                          |
   > | `\s`     | 空白字符（空格、Tab、换行）         |
   > | `\S`     | 非空白字符                          |
   >
   > ---
   >
   > ###### 3. 分组与引用
   > | 表达式        | 含义                             |
   > | ------------- | -------------------------------- |
   > | `(abc)`       | 捕获分组，匹配 "abc"             |
   > | `(?:abc)`     | 非捕获分组（仅分组，不保存引用） |
   > | `\1` `\2` ... | 引用前面第 1、2... 个捕获组      |
   >
   > ---
   >
   > ###### 4. 常用转义
   > | 表达式 | 含义              |
   > | ------ | ----------------- |
   > | `\.`   | 匹配字符 `.` 本身 |
   > | `\\`   | 匹配反斜杠 `\`    |
   > | `\[`   | 匹配 `[`          |
   > | `\]`   | 匹配 `]`          |
   > | `\(`   | 匹配 `(`          |
   > | `\)`   | 匹配 `)`          |
   > | `\/`   | 匹配 `/`          |
   >
   > ---
   >
   > ###### 5. 常见示例
   > | 正则              | 说明                           |
   > | ----------------- | ------------------------------ |
   > | `^abc`            | **匹配以 "abc" 开头的行**      |
   > | `abc$`            | **匹配以 "abc" 结尾的行**      |
   > | `^[0-9]+$`        | 匹配**整行**都是数字           |
   > | `^[A-Za-z0-9_]+$` | 匹配**整行**是字母数字下划线   |
   > | `\[[^]]*\]`       | **匹配一对中括号及其内部内容** |
   > | `\([0-9]{3}\)`    | 匹配形如 `(123)` 的内容        |
   > | `\bword\b`        | 匹配完整单词 "word"            |
   >
   > ---
   >
   > ###### 6. sed 常用替换格式
   >
   > ```shell
   > sed 's/正则/替换内容/' file # 仅替换匹配的第一个
   > sed 's/正则/替换内容/g' file # 替换匹配的所有
   > sed -i 's/正则/替换内容/' file # 直接修改文件
   > sed -i '/^pattern/s/old/new/' file # 仅修改匹配行
   > ```

4. `QT`环境变量配置文件

   > ```sh
   > #!/bin/sh
   > ## tslib
   > #  找出/devices中设备名及下方五行内容，然后找出以Handlers=开头的行，使用sed删除开头的字符提取出剩下的
   > DEVICE_NAME="eGalaxTouch Virtual Device for Single"
   > EVENT_DEVICE=$(grep -A 5 "$DEVICE_NAME" /proc/bus/input/devices | grep "Handlers=" | sed 's/H: Handlers=//')
   > if [ -n "$EVENT_DEVICE" ]; then
   > 	TOUCHSCREEN=/dev/input/$EVENT_DEVICE
   > 	echo $TOUCHSCREEN
   > else
   > 	echo "eGalaxTouch Device not found."
   > 	exit 1
   > fi
   > 
   > export TSLIB_ROOT=/usr/local/tslib
   > export TSLIB_TSDEVICE=$TOUCHSCREEN
   > export TSLIB_TSEVENTTYPE=input
   > export TSLIB_CONSOLEDEVICE=none
   > export TSLIB_CALIBFILE=/etc/pointercal
   > export TSLIB_CONFFILE=$TSLIB_ROOT/etc/ts.conf
   > export TSLIB_PLUGINDIR=$TSLIB_ROOT/lib/ts
   > export TSLIB_FBDEVICE=/dev/fb0
   > export PATH=$TSLIB_ROOT/bin:$PATH
   > export LD_LIBRARY_PATH=$TSLIB_ROOT/lib:$LD_LIBRARY_PATH
   > 
   > ## 每次启动都读取内核启动参数，判断是否已经校验，否则进行校验（判断是否存在校准文件，存在也说明校准过）
   > CALI=`grep "calibrate=N" /proc/cmdline`
   > if [ -n "$CALI" ]; then
   > echo "Jump calibrating step now!"
   > else
   > if [ -f /etc/pointercal ]; then
   >    echo "calibrated!"
   > else
   >    /usr/local/tslib/bin/ts_calibrate
   >    sync
   > fi
   > fi
   > 
   > FB_SIZE=$(cat /sys/class/graphics/fb0/virtual_size)
   > 
   > ## Qt 5.6.3
   > export QTDIR=/opt/Qt5.6.3-embedded/
   > export PATH=$QTDIR/bin:$PATH
   > export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$QTDIR/lib:/GJMill/plugins/display:/GJMill/plugins/craft
   > #export LD_LIBRARY_PATH=$QTDIR/lib:$LD_LIBRARY_PATH
   > export QT_PLUGIN_PATH=$QTDIR/plugins
   > export QT_QPA_FONTDIR=$QTDIR/lib/fonts
   > #export QT_FONTPATH=$QTDIR/lib/fonts
   > 
   > #  linuxfb 
   > export QT_QPA_PLATFORM=linuxfb
   > export QT_QPA_FB_TSLIB=1
   > export QT_QPA_EVDEV_TOUCHSCREEN=$TOUCHSCREEN
   > export QT_QPA_EVDEV_MOUSE=/dev/input/event7
   > 
   > if [ $FB_SIZE = "800,600" ]; then
   > export QT_QPA_FB_SIZE=800x600
   > elif [ $FB_SIZE = "1920,720" ]; then
   > export QT_QPA_FB_SIZE=1280x720
   > elif [ $FB_SIZE = "1024,768" ]; then
   > export QT_QPA_FB_SIZE=1024x768
   > elif [ $FB_SIZE = "1280,800" ]; then
   > export QT_QPA_FB_SIZE=1280x800
   > elif [ $FB_SIZE = "800,480" ]; then
   > export QT_QPA_FB_SIZE=800x480
   > fi
   > 
   > case "$color" in
   >     red)
   >         echo "red"
   >         ;;
   >     blue)
   >         echo "blue"
   >         ;;
   >     *)
   >         echo "none"
   >         ;;
   > esac
   > ```
   >

5. 

