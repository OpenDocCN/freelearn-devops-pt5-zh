# 第七章：容器故障排除

有时，像我们在第四章中设置的监控和日志系统，*监控 Docker 主机和容器*，可能还不足够。理想情况下，我们应该建立一种可扩展的方式来排查 Docker 部署中的问题。然而，有时我们别无选择，只能登录到 Docker 主机并查看 Docker 容器本身。

本章将涵盖以下主题：

+   使用 `docker exec` 检查容器

+   从 Docker 外部进行调试

+   其他调试套件

# 检查容器

在故障排除服务器时，传统的调试方法是登录并随便查看机器。使用 Docker 时，这种典型的工作流程被分为两个步骤：第一步是使用标准的远程访问工具（如 `ssh`）登录到 Docker 主机，第二步是通过 `docker exec` 进入所需的运行容器的进程命名空间。这在作为最后手段调试应用程序内部发生了什么时非常有用。

本章的大部分内容将涉及故障排除和调试运行 HAProxy 的 Docker 容器。为准备该容器，创建一个名为 `haproxy.cfg` 的 HAProxy 配置文件，内容如下：

```
defaults
  mode http
  timeout connect 5000ms
  timeout client 50000ms
  timeout server 50000ms

frontend stats
  bind 127.0.0.1:80
  stats enable

listen http-in
  bind *:80
  server server1 www.debian.org:80
```

接下来，使用官方的 HAProxy Docker 镜像（`haproxy:1.5.14`），我们将运行该容器并使用我们之前创建的配置。请在 Docker 主机中运行以下命令来启动 HAProxy 并应用我们准备好的配置：

```
dockerhost$ docker run -d -p 80:80 --name haproxy \
 -v `pwd`/haproxy.cfg:/usr/local/etc/haproxy/haproxy.cfg \
 haproxy:1.5.14

```

现在，我们可以开始检查容器并调试它。一个好的初步示例是确认 HAProxy 容器是否正在监听端口 `80`。`ss` 程序可以打印出大多数 Linux 发行版中可用的套接字统计信息，例如我们的 Debian Docker 主机。我们可以运行以下命令来显示 Docker 容器内监听套接字的统计信息：

```
dockerhost$ docker exec haproxy /bin/ss -l
State   Recv-Q Send-Q   Local Address:Port  Peer Address:Port
LISTEN  0      128                  *:http             *:*
LISTEN  0      128          127.0.0.1:http             *:*

```

由于 `ss` 默认包含在 `haproxy:1.5.14` 的 `debian:jessie` 父容器中，这种使用 `docker exec` 的方法才有效。我们不能使用类似的未默认安装的工具，比如 `netstat`。输入相应的 `netstat` 命令将显示以下错误：

```
dockerhost$ docker exec haproxy /usr/bin/netstat -an
dockerhost$ echo $?
255

```

让我们通过查看 Docker 引擎服务的日志来调查发生了什么。输入以下命令显示 `netstat` 程序在我们的容器内不存在：

```
dockerhost$ journalctl -u docker.service –o cat
...
time="..." level=info msg="POST /v1.20/containers/haproxy/exec"
time="..." level=info msg="POST /v1.20/exec/c64fcf22b5c4.../start"
time="..." level=warning msg="signal: killed"
time="..." level=error msg="Error running command in existing...:"
 " [8] System error: exec: \"/usr/bin/netstat\":"
 " stat /usr/bin/netstat: no such file or directory"
time="..." level=error msg="Handler for POST /exec/{n.../start..."
time="..." level=error msg="HTTP Error" err="Cannot run exec c..."
2015/11/18 17:58:12 http: response.WriteHeader on hijacked conn...
2015/11/18 17:58:12 http: response.Write on hijacked connect...
time="..." level=info msg="GET /v1.20/exec/c64fcf22b5c47be8278..."
...

```

查找 `netstat` 是否已安装在系统中的另一种方式是交互式进入我们的容器。`docker exec` 命令具有 `-it` 标志，我们可以使用它来启动一个交互式的 shell 会话来进行调试。输入以下命令，使用 `bash` shell 进入容器：

```
dockerhost$ docker exec -it haproxy /bin/bash
root@b397ffb9df13:/# 

```

现在我们处于标准的 shell 环境中，可以使用容器内所有标准的 Linux 工具进行调试。我们将在下一节介绍一些这些命令。现在，让我们来看一下为什么 `netstat` 在容器内无法使用，具体如下：

```
root@b397ffb9df13:/# netstat
bash: netstat: command not found
root@b397ffb9df13:/# /usr/bin/netstat -an
bash: /usr/bin/netstat: No such file or directory 

```

如我们所见，`bash` 告诉我们，经过更互动的调试过程后，我们已经确认没有安装 `netstat`。

我们可以通过在容器内安装它来提供一个快速的解决方案，类似于在普通的 Debian 环境中所做的那样。虽然我们仍然在容器内，我们将输入以下命令来安装 `netstat`：

```
root@b397ffb9df13:/# apt-get update
root@b397ffb9df13:/# apt-get install -y net-tools

```

现在，我们可以成功运行 `netstat`，如下所示：

```
root@b397ffb9df13:/# netstat -an
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address   Foreign Address      State
tcp        0      0 0.0.0.0:80      0.0.0.0:*            LISTEN
tcp        0      0 127.0.0.1:80    0.0.0.0:*            LISTEN
Active UNIX domain sockets (servers and established)
Proto RefCnt Flags       Type       State         I-Node   Path

```

这种临时容器调试方法不推荐使用！下一次我们在设计 Docker 基础架构时，应该使用适当的工具和监控系统。下次让我们改进在第四章，*监控 Docker 主机和容器*，中构建的初始版本！以下是这种临时方法的一些局限性：

1.  当我们停止并重新创建容器时，安装的 `netstat` 包将不再可用。这是因为原始的 HAProxy Docker 镜像根本就不包含它。临时安装包来运行容器违背了 Docker 的主要特性，即启用不可变基础设施。

1.  如果我们想将所有调试工具打包到 Docker 镜像中，镜像的大小将相应增加。这意味着我们的部署将变得更大，变得更慢。记住，在第二章，*优化 Docker 镜像*，中我们优化了容器的大小以减小其体积。

1.  对于仅包含所需二进制文件的最小容器，我们现在几乎是“盲目”的。连 `bash` shell 都不可用！我们无法进入容器，看看以下命令：

    ```
    dockerhost$ docker exec -it minimal_image /bin/bashdockerhost$ echo $?
    255

    ```

总结来说，`docker exec` 是一个强大的工具，可以让我们进入容器并通过运行各种命令进行调试。结合 `-it` 参数，我们可以获得一个交互式 shell 来进行更深层次的调试。由于假设容器内的所有工具都可以使用，这种方法有一定的局限性。

### 注意

更多关于 `docker exec` 命令的信息可以在官方文档中找到，[`docs.docker.com/reference/commandline/exec`](https://docs.docker.com/reference/commandline/exec)。

下一节将介绍如何通过外部工具检查正在运行的容器的状态，从而绕过这个限制。我们将简要概述如何使用其中一些工具。

# 从外部调试

尽管 Docker 在容器内隔离了网络、内存、CPU 和存储资源，每个容器仍然需要访问 Docker 主机的操作系统来执行实际命令。我们可以利用这些调用传递到主机操作系统的特性，从外部拦截并调试我们的 Docker 容器。在本节中，我们将介绍一些选定的工具以及如何使用它们与 Docker 容器进行交互。我们可以在 Docker 主机本身或在具有提升权限的同级容器内部执行交互，以查看 Docker 主机的一些组件。

## 跟踪系统调用

**系统调用跟踪器** 是服务器操作中必不可少的工具之一。它是一种工具，能够拦截并跟踪应用程序向操作系统发出的调用。每个操作系统都有其独特的实现。即使我们在 Docker 容器内部运行各种应用程序和进程，最终它们都会作为一系列系统调用进入 Docker 主机的 Linux 操作系统。

在 Linux 系统中，`strace` 程序用于跟踪这些系统调用。`strace` 的拦截和日志记录功能可以用来从外部检查我们的 Docker 容器。容器生命周期内进行的系统调用列表可以提供关于其行为的概览。

要开始使用 `strace`，只需在我们的 Debian Docker 主机中输入以下命令来安装它：

```
dockerhost$ apt-get install strace

```

### 提示

通过向 `docker run` 命令添加 `--pid=host` 选项，我们可以将容器的 PID 命名空间设置为 Docker 主机的 PID 命名空间。这样，我们就能够在 Docker 容器内安装并使用 `strace`，从而检查 Docker 主机上的所有进程。如果我们使用相应的基础镜像，还可以从其他 Linux 发行版（如 CentOS 或 Ubuntu）安装 `strace`。

有关此选项的更多信息，请访问 [`docs.docker.com/engine/reference/run/#pid-settings-pid`](http://docs.docker.com/engine/reference/run/#pid-settings-pid)。

现在我们已经在 Docker 主机中安装了 `strace`，可以使用它来检查我们在上一节中创建的 HAProxy 容器内的系统调用。输入以下命令开始跟踪来自 `haproxy` 容器的系统调用：

```
dockerhost$ PID=`docker inspect -f '{{.State.Pid}}' haproxy`dockerhost$ strace -p $PID
epoll_wait(3, {}, 200, 1000)            = 0
epoll_wait(3, {}, 200, 1000)            = 0
epoll_wait(3, {}, 200, 1000)            = 0
epoll_wait(3, {}, 200, 1000)            = 0
...

```

如你所见，我们的 HAProxy 容器发出了 `epoll_wait()` 调用，等待传入的网络连接。现在，在另一个终端中输入以下命令，向正在运行的容器发出 HTTP 请求：

```
$ curl http://dockerhost

```

现在，让我们回到之前运行的 `strace` 程序。我们可以看到以下几行输出：

```
...
epoll_wait(3, {}, 200, 1000)            = 0
epoll_wait(3, {{EPOLLIN, {u32=5, u64=5}}}, 200, 1000) = 1
accept4(5, {sa_family=AF_INET, sin_port=htons(56470), sin_addr...
setsockopt(6, SOL_TCP, TCP_NODELAY, [1], 4) = 0
accept4(5, 0x7ffc087a6a50, [128], SOCK_NONBLOCK) = -1 EAGAIN (...
recvfrom(6, "GET / HTTP/1.1\r\nUser-Agent: curl"..., 8192, 0, ...
socket(PF_INET, SOCK_STREAM, IPPROTO_TCP) = 7
fcntl(7, F_SETFL, O_RDONLY|O_NONBLOCK)  = 0
setsockopt(7, SOL_TCP, TCP_NODELAY, [1], 4) = 0
connect(7, {sa_family=AF_INET, sin_port=htons(80), sin_addr=in...
epoll_wait(3, {}, 200, 0)               = 0
sendto(7, "GET / HTTP/1.1\r\nUser-Agent: curl"..., 74, MSG_DON...
recvfrom(6, 0x18c488e, 8118, 0, 0, 0)   = -1 EAGAIN (Resource ...
epoll_ctl(3, EPOLL_CTL_ADD, 7, {EPOLLOUT, {u32=7, u64=7}}) = 0...
epoll_ctl(3, EPOLL_CTL_ADD, 6, {EPOLLIN|EPOLLRDHUP, {u32=6, u6...
epoll_wait(3, {{EPOLLOUT, {u32=7, u64=7}}}, 200, 1000) = 1
sendto(7, "GET / HTTP/1.1\r\nUser-Agent: curl"..., 74, MSG_DON...
epoll_ctl(3, EPOLL_CTL_DEL, 7, 6bbc1c)  = 0
epoll_wait(3, {}, 200, 0)               = 0
recvfrom(7, 0x18c88d4, 8192, 0, 0, 0)   = -1 EAGAIN (Resource ...
epoll_ctl(3, EPOLL_CTL_ADD, 7, {EPOLLIN|EPOLLRDHUP, {u32=7, u6...
epoll_wait(3, {{EPOLLIN, {u32=7, u64=7}}}, 200, 1000) = 1
recvfrom(7, "HTTP/1.1 200 OK\r\nDate: Fri, 20 N"..., 8192, 0, ...
epoll_wait(3, {}, 200, 0)               = 0
sendto(6, "HTTP/1.1 200 OK\r\nDate: Fri, 20 N"..., 742, MSG_DO...
epoll_wait(3, {{EPOLLIN|EPOLLRDHUP, {u32=6, u64=6}}}, 200, 100...
recvfrom(6, "", 8192, 0, NULL, NULL)    = 0
shutdown(6, SHUT_WR)                    = 0
close(6)                                = 0
setsockopt(7, SOL_SOCKET, SO_LINGER, {onoff=1, linger=0}, 8) =...
close(7)                                = 0
epoll_wait(3, {}, 200, 1000)            = 0
...

```

我们可以看到，HAProxy 执行了标准的 BSD 样式的套接字系统调用，如 `accept4()`、`socket()` 和 `close()`，用于接受、处理和终止来自 HTTP 客户端的网络连接。最后，它再次回到 `epoll_wait()`，等待下一个连接。同时，请注意，`epoll_wait()` 调用在整个跟踪过程中都有出现，即使 HAProxy 正在处理某个连接。这表明 HAProxy 可以处理并发连接。

跟踪系统调用是调试生产环境系统的一个非常有用的技巧。运维人员有时会接到页面通知，但并没有立即访问源代码的权限。或者，有时我们只会收到编译后的二进制文件（或普通的 Docker 镜像），它们在生产环境中运行，而没有源代码（也没有 `Dockerfile`）。我们从正在运行的应用程序中能获取到的唯一线索，就是捕捉它对 Linux 内核所做的系统调用。

### 注意

`strace` 的网页可以在 [`sourceforge.net/projects/strace/`](http://sourceforge.net/projects/strace/) 上找到。更多信息也可以通过其手册页面访问，方法是输入以下命令：

```
dockerhost$ man 1 strace

```

要获取更全面的 Linux 系统中系统调用的列表，请参考 [`man7.org/linux/man-pages/man2/syscalls.2.html`](http://man7.org/linux/man-pages/man2/syscalls.2.html)。这对于理解 `strace` 输出的各种信息非常有用。

## 分析网络数据包

我们部署的大多数 Docker 容器都涉及提供某种形式的网络服务。在本章的 HAProxy 示例中，我们的容器基本上提供 HTTP 网络流量。无论我们运行什么类型的容器，网络数据包最终都必须通过 Docker 主机出去，以完成我们发送给它的请求。通过捕获和分析这些数据包的内容，我们可以深入了解 Docker 容器的性质。在本节中，我们将使用一个叫做 `tcpdump` 的数据包分析器，查看我们的 Docker 容器接收和发送的网络数据包流量。

要开始使用 `tcpdump`，我们可以在我们的 Debian Docker 主机中输入以下命令来安装它：

```
dockerhost$ apt-get install -y tcpdump

```

### 提示

我们还可以将 Docker 主机的网络接口暴露给容器。通过这种方法，我们可以在容器中安装 `tcpdump`，而不会污染我们的主 Docker 主机与临时调试包。可以通过在 `docker run` 时指定 `--net=host` 标志来实现。这样，我们就可以在 Docker 容器内部使用 `tcpdump` 访问 `docker0` 接口。

使用 `tcpdump` 的示例将特别适用于 VMware Fusion 7.0 的 Vagrant VMware Fusion 提供程序。假设我们有一个 Docker Debian 主机作为 Vagrant VMware Fusion box，运行以下命令来挂起和恢复我们 Docker 主机的虚拟机：

```
$ vagrant suspend
$ vagrant up
$ vagrant ssh
dockerhost$

```

现在我们回到 Docker 主机内部，运行以下命令并注意到，我们再也无法在交互式的 `debian:jessie` 容器内解析 `www.google.com`，如下所示：

```
dockerhost$ docker run -it debian:jessie /bin/bash
root@fce09c8c0e16:/# ping www.google.com
ping: unknown host

```

现在，让我们在一个独立的终端中运行 `tcpdump`。在运行前面的 ping 命令时，我们会注意到从 `tcpdump` 终端输出以下内容：

```
dockerhost$ tcpdump -i docker0
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on docker0, link-type EN10MB (Ethernet), capture size 262144 bytes
22:03:34.512942 ARP, Request who-has 172.17.42.1 tell 172.17.0.7, length 28
22:03:35.512931 ARP, Request who-has 172.17.42.1 tell 172.17.0.7, length 28
22:03:38.520681 ARP, Request who-has 172.17.42.1 tell 172.17.0.7, length 28
22:03:39.520099 ARP, Request who-has 172.17.42.1 tell 172.17.0.7, length 28
22:03:40.520927 ARP, Request who-has 172.17.42.1 tell 172.17.0.7, length 28
22:03:43.527069 ARP, Request who-has 172.17.42.1 tell 172.17.0.7, length 28

```

如我们所见，交互式 `/bin/bash` 容器正在查找 `172.17.42.1`，这通常是附加在 Docker 引擎网络设备 `docker0` 上的 IP 地址。弄清楚这一点后，查看 `docker0`，请输入以下命令：

```
dockerhost$ ip addr show dev docker0
3: docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
 link/ether 02:42:46:66:64:b8 brd ff:ff:ff:ff:ff:ff
 inet6 fe80::42:46ff:fe66:64b8/64 scope link
 valid_lft forever preferred_lft forever

```

现在，我们可以查看问题。`docker0` 设备没有附加 IPv4 地址。不知何故，VMware 恢复 Docker 主机时，会移除 `docker0` 中映射的 IP 地址。幸运的是，解决方案是简单地重启 Docker 引擎，Docker 会自动重新初始化 `docker0` 网络接口。在 Docker 主机中输入以下命令以重启 Docker 引擎：

```
dockerhost$ systemctl restart docker.service

```

现在，当我们运行与之前相同的命令时，我们会看到 IP 地址已附加，如下所示：

```
dockerhost$ ip addr show dev docker0
3: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noque...
 link/ether 02:42:46:66:64:b8 brd ff:ff:ff:ff:ff:ff
 inet 172.17.42.1/16 scope global docker0
 valid_lft forever preferred_lft forever
 inet6 fe80::42:46ff:fe66:64b8/64 scope link
 valid_lft forever preferred_lft forever

```

让我们回到最初显示问题的命令，我们将看到问题已经解决，如下所示：

```
root@fce09c8c0e16:/# ping www.google.com
PING www.google.com (74.125.21.105): 56 data bytes
64 bytes from 74.125.21.105: icmp_seq=0 ttl=127 time=65.553 ms
64 bytes from 74.125.21.105: icmp_seq=1 ttl=127 time=38.270 ms
...

```

### 注意

有关 `tcpdump` 数据包转储器和分析器的更多信息，请访问 [`www.tcpdump.org`](http://www.tcpdump.org)。我们还可以通过输入以下命令访问我们安装它的文档：

```
dockerhost$ man 8 tcpdump

```

## 观察块设备

从我们的 Docker 容器访问的数据通常存储在物理存储设备中，例如硬盘或固态硬盘。在 Docker 的写时复制文件系统下面，是一个随机访问的物理设备。这些硬盘被分组为块设备。这里存储的数据是随机访问的固定大小的数据，称为 *块*。

所以，如果我们的 Docker 容器出现异常的 I/O 行为和性能问题，我们可以使用名为 `blktrace` 的工具来追踪和排查发生在块设备中的问题。该程序会拦截内核生成的所有与块设备交互的事件，这些事件来自进程。在本节中，我们将设置 Docker 主机，以便观察支持我们容器的块设备。

要使用 `blktrace`，我们需要通过安装 `blktrace` 程序来准备我们的 Docker 主机。在 Docker 主机内输入以下命令来安装它：

```
dockerhost$ apt-get install -y blktrace

```

此外，我们需要启用文件系统的调试。我们可以通过在 Docker 主机中输入以下命令来实现：

```
dockerhost$ mount -t debugfs debugfs /sys/kernel/debug

```

准备工作完成后，我们需要弄清楚如何告诉 `blktrace` 该在哪里监听 I/O 事件。要追踪我们容器的 I/O 事件，我们需要知道 Docker 运行时的根目录在哪里。在我们 Docker 主机的默认配置中，运行时指向 `/var/lib/docker`。要找出它属于哪个分区，请输入以下命令：

```
dockerhost$ df -h 
Filesystem      Size  Used Avail Use% Mounted on 
/dev/dm-0       9.0G  7.6G  966M  89% / 
udev             10M     0   10M   0% /dev 
tmpfs            99M   13M   87M  13% /run 
tmpfs           248M   52K  248M   1% /dev/shm 
tmpfs           5.0M     0  5.0M   0% /run/lock 
tmpfs           248M     0  248M   0% /sys/fs/cgroup 
/dev/sda1       236M   34M  190M  15% /boot

```

如前述输出所示，我们的 Docker 主机的 `/var/lib/docker` 目录位于 `/` 分区下。这就是我们将指向 `blktrace` 监听事件的地方。输入以下命令来开始监听此设备的 I/O 事件：

```
dockerhost$ blktrace -d /dev/dm-0 -o dump

```

### 提示

在 `docker run` 中使用 `--privileged` 标志，我们可以在容器内使用 `blktrace`。这样做将允许我们在提升的权限下挂载调试过的文件系统。

有关扩展容器权限的更多信息，请参考 [`docs.docker.com/engine/reference/run/#runtime-privilege-linux-capabilities-and-lxc-configuration`](https://docs.docker.com/engine/reference/run/#runtime-privilege-linux-capabilities-and-lxc-configuration)。

为了创建一个简单的工作负载，在我们的磁盘中生成 I/O 事件，我们将从容器中创建一个空文件，直到 / 分区的可用空间耗尽。输入以下命令生成该工作负载：

```
dockerhost$ docker run -d --name dump  debian:jessie \
 /bin/dd if=/dev/zero of=/root/dump bs=65000

```

根据我们根分区的可用空间，这个命令可能会很快完成。现在，让我们使用以下命令获取我们刚刚运行的容器的 PID：

```
dockerhost$ docker inspect -f '{{.State.Pid}}' dump
11099

```

现在我们知道了生成 I/O 事件的 Docker 容器的 PID，我们可以通过 `blktrace` 程序的配套工具 `blkparse` 来查找该 PID。`blktrace` 程序仅监听 Linux 内核的块 I/O 层的事件，并将结果输出到文件中。`blkparse` 程序是用来查看和分析这些事件的辅助工具。在我们之前生成的工作负载中，我们可以使用以下命令查找与 Docker 容器的 PID 对应的 I/O 事件：

```
dockerhost$ blkparse -i dump.blktrace.0 | grep --color " $PID "
...
254,0    0      730    10.6267 11099  Q   R 13667072 + 24 [exe]
254,0    0      732    10.6293 11099  Q   R 5042728 + 16 [exe]
254,0    0      734    10.6299 11099  Q   R 13900768 + 152 [exe]
254,0    0      736    10.6313 11099  Q  RM 4988776 + 8 [exe]
254,0    0     1090    10.671 11099   C  W 11001856 + 1024 [0]
254,0    0     1091    10.6712 11099  C  W 11002880  +  1024  [0]
254,0    0     1092    10.6712 11099  C  W 11003904  +  1024 [0]
254,0    0     1093    10.6712 11099  C  W 11004928  +  1024 [0]
254,0    0     1094    10.6713 11099  C  W 11006976  +  1024 [0]
254,0    0     1095    10.6714 11099  C  W 11005952  +  1024 [0]
254,0    0     1138    10.6930 11099  C  W 11239424  +  1024 [0]
254,0    0     1139    10.6931 11099  C  W 11240448  +  1024 [0]
...

```

在前面突出显示的输出中，我们可以看到 `/dev/dm-0` 块偏移量的位置为 `11001856`，并且完成了一次 `1024` 字节的数据写入（`W`）。为了进一步探测，我们可以查看该偏移位置所生成的事件。输入以下命令来过滤出该偏移位置：

```
dockerhost$ blkparse -i dump.blktrace.0 | grep 11001856
...
254,0  0  1066  10.667  8207  Q   W 11001856 + 1024 [kworker/u2:2]
254,0  0  1090  10.671 11099  C   W 11001856 + 1024 [0]
...

```

我们可以看到 `kworker` 进程正在将写入（`W`）请求排入队列（`Q`），这意味着写入操作已经被内核排队。40 毫秒后，注册的写入请求完成了 Docker 容器进程的操作。

我们刚刚执行的调试过程只是通过 `blktrace` 跟踪块 I/O 事件可以做到的一小部分。例如，我们还可以更详细地探测 Docker 容器的 I/O 行为，找出应用程序中出现的瓶颈。是否有大量的写入操作？读取操作是否如此频繁，以至于需要缓存？拥有实际的事件数据，而不仅仅是内建的 `docker stats` 命令提供的性能指标，在深入排查问题时非常有帮助。

### 注意

有关 `blkparse` 输出的不同值以及如何在 `blktrace` 中捕获 I/O 事件的更多信息，请参考位于 [`www.cse.unsw.edu.au/~aaronc/iosched/doc/blktrace.html`](http://www.cse.unsw.edu.au/~aaronc/iosched/doc/blktrace.html) 的用户指南。

# 一套故障排除工具

在 Docker 容器内调试应用程序需要一种不同于 Linux 上普通应用程序的方式。然而，实际使用的程序是相同的，因为容器内部的所有调用最终都会传递到 Docker 主机的内核操作系统。通过了解调用如何穿越容器外部，我们可以使用任何其他调试工具来排查问题。

除了标准的 Linux 工具外，还有一些特定于容器的工具，这些工具将前述的标准工具进行了封装，以便更适应容器的使用。以下是其中一些工具：

+   Red Hat 的 `rhel-tools` Docker 镜像是一个包含我们之前讨论的各种工具的巨大容器。其文档页面[`access.redhat.com/documentation/en/red-hat-enterprise-linux-atomic-host/version-7/getting-started-with-containers/#using_the_atomic_tools_container_image`](https://access.redhat.com/documentation/en/red-hat-enterprise-linux-atomic-host/version-7/getting-started-with-containers/#using_the_atomic_tools_container_image)展示了如何以正确的 Docker 权限运行该工具，以确保其正常工作。

+   CoreOS 的`t`oolbox`程序是一个小型脚本工具，它使用 Systemd 的`systemd-nspawn`程序创建一个小型的 Linux 容器。通过复制流行的 Docker 镜像的根文件系统，我们可以安装任何我们需要的工具，而无需将临时调试工具污染 Docker 主机的文件系统。其使用方法已在其网页上进行文档说明，网址为[`coreos.com/os/docs/latest/install-debugging-tools.html`](https://coreos.com/os/docs/latest/install-debugging-tools.html)。

+   `nsenter`程序是一个进入 Linux 控制组进程命名空间的工具。它是`docker exec`程序的前身，并被认为已经不再维护。要了解 `docker exec` 如何诞生的历史，请访问 `nsenter` 程序的项目页面，网址为[`github.com/jpetazzo/nsenter`](https://github.com/jpetazzo/nsenter)。

# 总结

请记住，登录到 Docker 主机并不是一种可扩展的方案。在应用程序层面添加监控工具，除了操作系统本身提供的工具外，能够帮助更快速、更高效地诊断未来可能遇到的问题。记住，没人愿意在凌晨两点起来运行`tcpdump`来调试一个着火的 Docker 容器！

在下一章中，我们将总结并再次审视将基于 Docker 的工作负载推向生产环境所需要的步骤。
