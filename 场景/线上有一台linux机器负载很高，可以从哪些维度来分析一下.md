
### 一、**CPU 维度**
- 查看负载情况：
    - `uptime` / `top` / `w`：查看 load average 是否持续高（load 一般不超过 CPU 核心数的 70% 为正常）
- 查看 CPU 占用：
    - `top` / `htop`：找出占用 CPU 高的进程
    - `ps -eo pid,ppid,cmd,%mem,%cpu --sort=-%cpu | head`：查看 CPU 占比高的进程
- 查看上下文切换：
    - `vmstat 1`：关注 `cs`（context switch）是否异常


### 二、**内存维度**
- 查看内存使用情况：
    - `free -m`：查看是否内存占满
    - `top`：查看是否有大量 swap 使用
- 检查是否存在内存泄漏：
    - `ps aux --sort=-%mem`：定位内存占用高的进程
    - 使用工具如 `valgrind` 或 `heaptrack`（适用于 C/C++）


### **三、磁盘 I/O 维度**
- 查看磁盘 I/O 是否是瓶颈：
    - `iostat -x 1`：观察 `%util`（I/O 利用率）是否过高
    - `iotop`：查看哪些进程在频繁读写磁盘
    - `df -h` / `du -sh *`：查看磁盘是否已满

### **四、网络维度**
- 查看网络带宽是否打满：
    - `iftop` / `nload`：查看网络流量情况
    - `ss -tunap`：查看网络连接是否异常增多
- 检查是否有 DDOS 攻击或异常请求：
    - `netstat -anp`：查看连接状态，是否有大量 `SYN_RECV`
    - 使用 `tcpdump` 抓包分析异常流量


### 五、**进程与服务维度**
- 是否有进程僵死、死循环、线程爆炸：
    - `ps`, `top`, `htop`：查看是否有可疑进程
- 应用日志排查：
    - 查看 `/var/log/` 目录日志
    - 检查应用自身日志是否报错或异常请求量激增


### 六、**内核参数和系统调用维度**
- `dmesg`：查看系统或内核是否有错误
- `strace -p <pid>`：查看进程在做什么系统调用
- `sar`：查看历史负载（安装 `sysstat` 工具）


### 七、**容器 / 虚拟化环境**
如果是运行在 Docker 或虚拟机中：
- 检查容器资源限制是否过低（CPU、memory limit）
- 查看宿主机资源使用情况