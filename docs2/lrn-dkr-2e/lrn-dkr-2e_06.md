# 第七章：调试容器

调试一直是软件工程领域中的一项艺术性工作。各种软件构建模块，无论是单独的还是集成的，都需要经过软件开发和测试专业人员的深入和决定性检查，以确保生成的软件应用程序的安全性和可靠性。由于 Docker 容器被认为是下一代关键任务软件工作负载的关键运行时环境，因此容器、开发人员和构建人员进行系统性、审慎的容器验证和验证显得尤为重要。

本章专门为那些掌握准确且相关信息的技术人员编写，帮助他们仔细调试容器内运行的应用程序及容器本身。在本章中，我们还将探讨容器内进程隔离的理论方面。Docker 容器在主机机器上作为用户级进程运行，通常具有操作系统提供的相同隔离级别。随着最新 Docker 版本的发布，许多调试工具已可用，可高效地用于调试应用程序。我们还将介绍主要的 Docker 调试工具，如 `docker exec`、`stats`、`ps`、`top`、`events` 和 `logs`。当前版本的 Docker 使用 Go 编写，并利用 Linux 内核的多个功能来提供其功能。

本章将涵盖的主题列表如下：

+   Docker 容器的进程级隔离

+   调试 `Dockerfile`

+   调试容器化应用程序

本章中的所有命令都在 Ubuntu 环境中测试，如果在本地 Mac 环境中运行，结果可能会有所不同。

在主机机器上安装 Docker 引擎后，可以使用 `-D` 调试选项启动 Docker 守护进程：

```
$ docker -D login
Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one.
Username (vinoddandy):   

```

此`-D`调试标志可以启用 Docker 配置文件（`/etc/default/docker`）中的调试模式：

```
DOCKER_OPTS="-D"  

```

保存并关闭配置文件后，重新启动 Docker 守护进程。

## Docker 容器的进程级隔离

在虚拟化范式中，虚拟机管理程序（hypervisor）模拟计算资源，并提供一个虚拟化环境（称为虚拟机 VM），在其上安装操作系统和应用程序。而在容器范式中，单一的系统（裸金属或虚拟机）实际上被划分为多个独立的服务，能够同时运行且不会相互干扰。这些服务必须相互隔离，以防止它们争用资源或发生依赖冲突（也叫依赖地狱）。Docker 容器技术通过利用 Linux 内核的构件（如命名空间和 cgroups），特别是命名空间，实现了进程级别的隔离。Linux 内核提供了以下五种强大的命名空间工具，用于隔离全球系统资源。它们是用于隔离**进程间通信**（**IPC**）资源的**IPC**命名空间：

+   **network**: 该命名空间用于隔离网络资源，如网络设备、网络堆栈和端口号

+   **mount**: 该命名空间隔离了文件系统的挂载点

+   **PID**: 该命名空间隔离了进程标识符（PID）

+   **user**: 该命名空间用于隔离用户 ID 和组 ID

+   **UTS**: 该命名空间用于隔离主机名和 NIS 域名

当我们需要调试容器内运行的服务时，这些命名空间增加了额外的复杂性，关于这点，你将在下一节中学到更多细节。

在这一节中，我们将通过一系列实践示例来讨论 Docker 引擎如何利用 Linux 命名空间提供进程级别的隔离，其中一个示例如下所示：

1.  首先，使用`docker run`子命令启动一个交互模式的 Ubuntu 容器，如下所示：

```
 $ sudo docker run -it --rm ubuntu /bin/bash
 root@93f5d72c2f21:/#

```

1.  继续查找前述`93f5d72c2f21`容器的进程 ID，可以通过在另一个终端使用`docker inspect`子命令来完成：

```
 $ sudo docker inspect \
 --format "{{ .State.Pid }}" 93f5d72c2f21
 2543

```

显然，从上面的输出中，容器`93f5d72c2f21`的进程 ID 是`2543`。

1.  获取了容器的进程 ID 后，我们继续查看容器关联的进程在 Docker 主机中是如何呈现的，可以使用`ps`命令：

```
 $ ps -fp 2543
 UID PID PPID C STIME TTY TIME 
 CMD
 root 2543 6810 0 13:46 pts/7 00:00:00 
 /bin/bash

```

真的吗？是不是很神奇？我们启动了一个以`/bin/bash`作为命令的容器，结果在 Docker 主机中也有了`/bin/bash`进程。

1.  让我们进一步操作，使用`cat`命令在 Docker 主机上显示`/proc/2543/environ`文件：

```
 $ sudo cat -v /proc/2543/environ
 PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin /bin^@HOSTNAME=93f5d72c2f21^@TERM=xterm^@HOME=/root^@$

```

在前面的输出中，`HOSTNAME=93f5d72c2f21`从其他环境变量中脱颖而出，因为`93f5d72c2f21`既是容器 ID，也是我们之前启动的容器的主机名。

1.  现在，让我们回到终端，我们正在运行交互式容器`93f5d72c2f21`，并使用`ps`命令列出容器内运行的所有进程：

```
 root@93f5d72c2f21:/# ps -ef
 UID PID PPID C STIME TTY TIME CMD
 root 1 0 0 18:46 ? 00:00:00 /bin/bash
 root 15 1 0 19:30 ? 00:00:00 ps -ef

```

令人惊讶，不是吗？在容器内，`/bin/bash` 进程的进程 ID 是 `1`，而在容器外的 Docker 主机上，进程 ID 是 `2543`。此外，**父进程 ID** (**PPID**) 是 `0`（零）。

在 Linux 世界中，每个系统只有一个 PID 为 `1` 和 PPID 为 `0` 的 `root` 进程，它是该系统完整进程树的根。Docker 框架巧妙地利用了 Linux 的 PID 命名空间来启动一个全新的进程树；因此，在容器内运行的进程无法访问 Docker 主机的父进程。然而，Docker 主机可以完全查看由 Docker 引擎创建的子 PID 命名空间。

网络命名空间确保所有容器在主机机器上都有独立的网络接口。此外，每个容器都有自己的回环接口。每个容器使用其自己的网络接口与外部世界通信。你会惊讶地发现，命名空间不仅有自己的路由表，而且还有自己的 iptables、链和规则。本章的作者正在他的主机上运行三个容器。在这里，理应为每个容器配备三个网络接口。让我们运行 `docker ps` 命令：

```
$ sudo docker ps
41668be6e513 docker-apache2:latest "/bin/sh -c 'apachec
069e73d4f63c nginx:latest "nginx -g ' 
871da6a6cf43 ubuntu "/bin/bash"   

```

所以，这里有三个接口，每个容器一个。让我们通过运行以下命令来获取它们的详细信息：

```
$ ifconfig
veth2d99bd3 Link encap:EthernetHWaddr 42:b2:cc:a5:d8:f3
inet6addr: fe80::40b2:ccff:fea5:d8f3/64 Scope:Link
 UP BROADCAST RUNNING MTU:9001 Metric:1
veth422c684 Link encap:EthernetHWaddr 02:84:ab:68:42:bf
inet6addr: fe80::84:abff:fe68:42bf/64 Scope:Link
 UP BROADCAST RUNNING MTU:9001 Metric:1
vethc359aec Link encap:EthernetHWaddr 06:be:35:47:0a:c4
inet6addr: fe80::4be:35ff:fe47:ac4/64 Scope:Link
 UP BROADCAST RUNNING MTU:9001 Metric:1  

```

挂载命名空间确保挂载的文件系统仅对同一命名空间中的进程可访问。容器 A 无法看到容器 B 的挂载点。如果你想查看你的挂载点，需要先使用 `exec` 命令（将在下一部分中描述）登录到容器中，然后转到 `/proc/mounts`：

```
root@871da6a6cf43:/# cat /proc/mounts
rootfs / rootfsrw 0 0/dev/mapper/docker-202:1-149807 871da6a6cf4320f625d5c96cc24f657b7b231fe89774e09fc771b3684bf405fb / ext4 rw,relatime,discard,stripe=16,data=ordered 0 0 proc /procproc rw,nosuid,nodev,noexec,relatime 0 0   

```

让我们运行一个具有挂载点的容器，该挂载点作为 **存储区域网络** (**SAN**) 或 **网络附加存储** (**NAS**) 设备运行，并通过登录到容器来访问它。这个部分作为练习交给你。我在工作中的一个项目中实现了这个功能。

还有其他命名空间可以将这些容器/进程隔离到，其中包括用户、IPC 和 UTS。用户命名空间允许你在命名空间内拥有 root 权限，而不会将该权限授予命名空间外的进程。使用 IPC 命名空间隔离进程时，它将拥有自己的 IPC 资源，例如，System V IPC 和 POSIX 消息。UTS 命名空间隔离了系统的主机名。

Docker 使用 `clone` 系统调用实现了这个命名空间。在主机机器上，你可以检查 Docker 为容器（PID 为 `3728`）创建的命名空间：

```
$ sudo ls /proc/3728/ns/
cgroup ipc mnt netpid user uts  

```

在大多数工业化部署的 Docker 中，人们广泛使用已修补的 Linux 内核来满足特定需求。此外，一些公司修补了他们的内核，以便将任意进程附加到现有的命名空间中，因为他们认为这是部署、控制和编排容器最便捷且最可靠的方式。

### 控制组

Linux 容器依赖于**控制组**（**cgroups**），它不仅跟踪进程组，还暴露 CPU、内存和块 I/O 使用的度量。你可以访问这些度量并获得网络使用度量。Cgroups 是 Linux 容器的另一个重要组成部分。Cgroups 已经存在了一段时间，并且最初在 Linux 内核代码 2.6.24 中合并。它们确保每个 Docker 容器将获得固定的内存、CPU 和磁盘 I/O，因此任何容器都不能在任何情况下使主机机器崩溃。Cgroups 不起到防止一个容器被访问的作用，但它们对于抵御一些**拒绝服务**（**DoS**）攻击是至关重要的。

在 Ubuntu 16.04 中，cgroup 实现位于 `/sys/fs/cgroup` 路径下。Docker 的内存信息可以在 `/sys/fs/cgroup/memory/docker/` 路径下找到。

同样，CPU 的详细信息可以在 `/sys/fs/cgroup/cpu/docker/` 路径下找到。

让我们找出容器（`41668be6e513e845150abd2dd95dd574591912a7fda947f6744a0bfdb5cd9a85`）能够消耗的最大内存限制。

为此，你可以前往 cgroup 内存路径，并检查 `memory.max_usage_in_bytes` 文件：

```
/sys/fs/cgroup/memory/docker/41668be6e513e845150abd2dd95dd574591912a7
fda947f6744a0bfdb5cd9a85

```

执行以下命令查看内容：

```
$ cat memory.max_usage_in_bytes
13824000

```

因此，默认情况下，任何容器最多只能使用 13.18 MB 的内存。同样，CPU 参数可以在以下路径中找到：

```
/sys/fs/cgroup/cpu/docker/41668be6e513e845150abd2dd95dd574591912a7fda
947f6744a0bfdb5cd9a85

```

传统上，Docker 容器内部只运行一个进程。因此，通常你会看到有人分别为 PHP、NGINX 和 MySQL 启动三个容器。然而，这只是一个误解。你也可以将这三个进程都运行在同一个容器内。

Docker 隔离了许多底层主机的方面，允许容器中的应用程序在没有 root 权限的情况下运行。然而，这种隔离性不如虚拟机强大，虚拟机在 Hypervisor 上运行独立的操作系统实例，并且不会与底层操作系统共享内核。将具有不同安全配置文件的应用程序作为容器在同一主机上运行并不是一个好主意，但将不同的应用程序封装为容器化应用程序，避免它们直接在同一主机上运行，仍然有安全性上的好处。

### 调试容器化应用程序

计算机程序（软件）有时不能按预期行为执行。这可能是由于代码错误，或是开发、测试和部署系统之间的环境变化导致的。Docker 容器技术通过将所有应用程序依赖项容器化，尽可能消除开发、测试和部署之间的环境问题。然而，仍然可能存在由于代码缺陷或内核行为变化而导致的异常，这需要调试。调试是软件工程领域最复杂的过程之一，在容器化架构中，由于隔离技术的存在，调试变得更加复杂。在本节中，我们将学习一些使用 Docker 原生工具以及外部来源提供的工具来调试容器化应用程序的小技巧。

最初，许多 Docker 社区成员独立开发了自己的调试工具，但后来 Docker 开始支持原生工具，如 `exec`、`top`、`logs` 和 `events`。在本节中，我们将深入探讨以下 Docker 工具：

+   `exec`

+   `ps`

+   `top`

+   `stats`

+   `events`

+   `logs`

+   `attach`

我们还将考虑调试 `Dockerfile`。

## docker exec 命令

`docker exec` 命令为用户提供了必要的帮助，尤其是那些部署自己 Web 服务器或有其他应用程序在后台运行的用户。现在，运行 SSH 守护进程不再需要登录到容器。

1.  首先，创建一个 Docker 容器：

```
 $ sudo docker run --name trainingapp \ 
 training/webapp:latest 
 Unable to find image 
 'training/webapp:latest' locally
 latest: Pulling from training/webapp
 9dd97ef58ce9: Pull complete 
 a4c1b0cb7af7: Pull complete 
 Digest: sha256:06e9c1983bd6d5db5fba376ccd63bfa529e8d02f23d5079b8f74a616308fb11d
 Status: Downloaded newer image for 
 training/webapp:latest

```

1.  接下来，运行 `docker ps -a` 命令以获取容器 ID：

```
      $ sudo docker ps -a
 a245253db38b training/webapp:latest 
 "python app.py"

```

1.  然后，运行 `docker exec` 命令以登录到容器：

```
 $ sudo docker exec -it a245253db38b bash
 root@a245253db38b:/opt/webapp#

```

1.  请注意，`docker exec` 命令只能访问正在运行的容器，因此如果容器停止运行，你需要重启停止的容器才能继续。`docker exec` 命令使用 Docker API 和 CLI 在目标容器中生成一个新进程。所以，如果你在目标容器内运行 `ps -aef` 命令，输出看起来如下：

```
 # ps -aef
 UID PID PPID C STIME TTY TIME 
 CMD
 root 1 0 0 Nov 26 ? 00:00:53 
 python app.py
 root 45 0 0 18:11 ? 00:00:00 
 bash
 root 53 45 0 18:11 ? 00:00:00 
 ps -aef

```

在这里，`python app.y` 是已经在目标容器中运行的应用程序，而 `docker exec` 命令则在容器内启动了 `bash` 进程。如果你运行 `kill -9 pid(45)`，你将自动退出容器。

如果你是一个热衷的开发者，想要增强 `exec` 功能，你可以参考 [`github.com/chris-rock/docker-exec`](https://github.com/chris-rock/docker-exec)。

建议仅将 `docker exec` 命令用于监控和诊断目的，我个人相信每个容器一个进程的理念，这是广泛强调的最佳实践之一。

## docker ps 命令

`docker ps` 命令在容器内部可用，用于查看进程的状态。这个命令类似于 Linux 环境中的标准 `ps` 命令，而*不是*我们在 Docker 主机上运行的 `docker ps` 命令。

这个命令在 Docker 容器内部运行：

```
root@5562f2f29417:/# ps -s
UID PID PENDING BLOCKED IGNORED CAUGHT STAT TTY TIME COMMAND
0 1 00000000 00010000 00380004 4b817efb Ss 
? 0:00 /bin/bash
0 33 00000000 00000000 00000000 73d3fef9 R+ ? 0:00 ps -s
root@5562f2f29417:/# ps -l
F S UID PID PPID C PRI NI ADDR SZ WCHAN TTY TIME CMD
4 S 0 1 0 0 80 0 - 4541 wait ? 00:00:00 bash
root@5562f2f29417:/# ps -t
PID TTY STAT TIME COMMAND
 1 ? Ss 0:00 /bin/bash
 35 ? R+ 0:00 ps -t
root@5562f2f29417:/# ps -m
PID TTY TIME CMD
 1 ? 00:00:00 bash
 - - 00:00:00 -
 36 ? 00:00:00 ps
 - - 00:00:00 -
root@5562f2f29417:/# ps -a
PID TTY TIME CMD
 37 ? 00:00:00 ps 

```

使用 `ps --help <simple|list|output|threads|misc|all>` 或 `ps --help <s|l|o|t|m|a>` 获取额外的帮助文本。

## docker top 命令

您可以通过以下命令从 Docker 主机机器上运行 `top` 命令：

```
docker top [OPTIONS] CONTAINER [ps OPTIONS]  

```

这将列出容器的运行进程列表，而无需登录到容器中，如下所示：

```
$ sudo docker top a245253db38b
UID PID PPID C
STIME TTY TIME CMD
root 5232 3585 0
Mar22 ? 00:00:53 python app.py 
$ sudo docker top a245253db38b -aef
UID PID PPID C
STIME TTY TIME CMD
root 5232 3585 0
Mar22 ? 00:00:53 python app.py  

```

Docker `top` 命令在 Docker 容器内运行时提供有关 CPU、内存和交换使用情况的信息：

```
root@a245253db38b:/opt/webapp# top
top - 19:35:03 up 25 days, 15:50, 0 users, load average: 0.00, 0.01, 0.05
Tasks: 3 total, 1 running, 2 sleeping, 0 stopped, 0 zombie
%Cpu(s): 0.0%us, 0.0%sy, 0.0%ni, 99.9%id, 0.0%wa, 0.0%hi, 0.0%si, 0.0%st
Mem: 1016292k total, 789812k used, 226480k free, 83280k buffers
Swap: 0k total, 0k used, 0k free, 521972k cached
PID USER PR NI VIRT RES SHR S %CPU %MEM 
TIME+ COMMAND
 1 root 20 0 44780 10m 1280 S 0.0 1.1 0:53.69 python
 62 root 20 0 18040 1944 1492 S 0.0 0.2 0:00.01 bash
 77 root 20 0 17208 1164 948 R 0.0 0.1 0:00.00 top  

```

如果在容器内运行 `top` 命令时出现 `error - TERM environment variable not set` 错误，请执行以下步骤来解决：

运行 `echo $TERM` 命令。您将得到结果 `dumb`。然后执行以下命令：

```
$ export TERM=dumb 

```

这将解决错误。

## docker stats 命令

`docker stats` 命令使您能够从 Docker 主机机器查看容器的内存、CPU 和网络使用情况，如下所示：

```
$ sudo docker stats a245253db38b
CONTAINER CPU % MEM USAGE/LIMIT MEM % NET I/O
a245253db38b 0.02% 16.37 MiB/992.5 MiB 1.65%
3.818 KiB/2.43 KiB  

```

您也可以运行 `stats` 命令来查看多个容器的使用情况：

```
$ sudo docker stats a245253db38b f71b26cee2f1   

```

Docker 提供对容器统计信息的只读访问 *read only* 参数。这简化了容器的 CPU、内存、网络 IO 和块 IO。Docker `stats` 实用程序仅为运行中的容器提供这些资源使用情况详细信息。

## Docker events 命令

Docker 容器将报告以下实时事件：`create`, `destroy`, `die`, `export`, `kill`, `omm`, `pause`, `restart`, `start`, `stop` 和 `unpause`。以下是几个示例，说明如何使用这些命令：

```
$ sudo docker pause a245253db38b
a245253db38b 
$ sudo docker ps -a
a245253db38b training/webapp:latest "python app.py" 
4 days ago Up 4 days (Paused) 0.0.0.0:5000->5000/tcp sad_sammet 
$ sudo docker unpause a245253db38b
a245253db38b 
$ sudo docker ps -a
a245253db38b training/webapp:latest "python app.py" 
4 days ago Up 4 days 0.0.0.0:5000->5000/tcpsad_sammet  

```

Docker 镜像还将报告 untag 和 delete 事件。

多个过滤器的使用将作为 AND 操作处理；例如，

`--filter container= a245253db38b --filter event=start` 将显示容器 `a245253db38b` 的事件，并且事件类型为 `start`。

目前支持的过滤器有 container、event 和 image。

## docker logs 命令

此命令在不登录到容器中的情况下获取容器的日志。它批量检索执行时存在的日志。这些日志是 stdout 和 stderr 的输出。通常用法如 `docker logs [OPTIONS] CONTAINER` 所示。

`-follow` 选项将持续提供输出直到结束，`-t` 将提供时间戳，`--tail= <number of lines>` 将显示容器日志消息的行数：

```
$ sudo docker logs a245253db38b
* Running on http://0.0.0.0:5000/
172.17.42.1 - - [22/Mar/2015 06:04:23] "GET / HTTP/1.1" 200 -
172.17.42.1 - - [24/Mar/2015 13:43:32] "GET / HTTP/1.1" 200 -
$ sudo docker logs -t a245253db38b
2015-03-22T05:03:16.866547111Z * Running on http://0.0.0.0:5000/
2015-03-22T06:04:23.349691099Z 172.17.42.1 - - [22/Mar/2015 06:04:23] "GET / HTTP/1.1" 200 -
2015-03-24T13:43:32.754295010Z 172.17.42.1 - - [24/Mar/2015 13:43:32] "GET / HTTP/1.1" 200 -  

```

我们还在 第二章 中使用了 `docker logs` 实用程序，*处理 Docker 容器* 和 第六章，*在容器中运行服务*，来查看我们容器的日志。

## docker attach 命令

`docker attach` 命令附加正在运行的容器，当您想实时查看 stdout 中的内容时非常有用：

```
$ sudo docker run -d --name=newtest alpine /bin/sh -c "while true; do sleep 2; df -h; done"
Unable to find image 'alpine:latest' locally
latest: Pulling from library/alpine
3690ec4760f9: Pull complete 
Digest: sha256:1354db23ff5478120c980eca1611a51c9f2b88b61f24283ee8200bf9a54f2e5c
1825927d488bef7328a26556cfd72a54adeb3dd7deafb35e317de31e60c25d67
$ sudo docker attach newtest
Filesystem Size Used Available Use% Mounted on
none 7.7G 3.2G 4.1G 44% /
tmpfs 496.2M 0 496.2M 0% /dev
tmpfs 496.2M 0 496.2M 0% /sys/fs/cgroup
/dev/xvda1 7.7G 3.2G 4.1G 44% /etc/resolv.conf
/dev/xvda1 7.7G 3.2G 4.1G 44% /etc/hostname
/dev/xvda1 7.7G 3.2G 4.1G 44% /etc/hosts
shm 64.0M 0 64.0M 0% /dev/shm
tmpfs 496.2M 0 496.2M 0% /proc/sched_debug
Filesystem Size Used Available Use% Mounted on
none 7.7G 3.2G 4.1G 44% /
tmpfs 496.2M 0 496.2M 0% /dev

```

默认情况下，该命令会附加 stdin 并代理信号到远程进程。可以使用选项控制这两种行为。要从进程中分离，请使用默认的*Ctrl* + *C*组合键。

## 调试一个 Dockerfile

有时，创建一个`Dockerfile`并不会一开始就让一切正常工作。`Dockerfile`并不总是能构建镜像，有时能构建镜像，但启动容器时会崩溃。

我们在`Dockerfile`中设置的每个指令都将被构建为一个独立的临时镜像，供其他指令在其基础上构建。以下示例解释了这一点：

1.  使用你喜欢的编辑器创建一个`Dockerfile`：

```
      FROM busybox 
      RUN ls -lh 
      CMD echo Hello world 

```

1.  现在，通过执行以下命令来构建镜像：

```
 $ docker build .
 Sending build context to Docker daemon 2.048 kB
 Step 1 : FROM busybox
 latest: Pulling from library/busybox
 56bec22e3559: Pull complete 
 Digest: sha256:29f5d56d12684887bdfa50dcd29fc31eea4aaf4ad3bec43daf19026a7ce69912
 Status: Downloaded newer image for busybox:latest
 ---> e02e811dd08f
 Step 2 : RUN ls -lh
 ---> Running in 7b47d3c46cfa
 total 36
 drwxr-xr-x 2 root root 12.0K Oct 7 18:18 bin
 dr-xr-xr-x 130 root root 0 Nov 27 01:36 proc
 drwxr-xr-x 2 root root 4.0K Oct 7 18:18 root
 dr-xr-xr-x 13 root root 0 Nov 27 01:36 sys
 drwxrwxrwt 2 root root 4.0K Oct 7 18:18 tmp
 ---> ca5bea5887d6
 Removing intermediate container 7b47d3c46cfa
 Step 3 : CMD echo Hello world
 ---> Running in 490ecc3d10a9
 ---> 490d1c3eb782
 Removing intermediate container 490ecc3d10a9
 Successfully built 490d1c3eb782
 **$**  

```

注意`---> Running in 7b47d3c46cfa`这一行。`7b47d3c46cfa`是一个有效的镜像，可以用来重试失败的指令，并查看发生了什么。

要调试这个镜像，我们需要创建一个容器，然后登录进去分析错误。调试是分析发生了什么的过程，它因情况而异，但通常，我们开始调试的方法是尝试手动使失败的指令正常工作并理解错误。当我使指令正常工作时，通常会退出容器，更新我的`Dockerfile`，然后重复这一过程，直到得到一个有效的结果。

## 总结

在本章中，你已经看到使用 Linux 容器技术（如 LXC，现在是 Libcontainer）进行容器隔离的过程。Libcontainer 是 Docker 在 Go 编程语言中实现的，用于访问内核命名空间和 cgroups。这个命名空间用于进程级别的隔离，而 cgroups 则用于限制运行中容器的资源使用。由于容器直接作为独立进程运行在 Linux 内核之上，**通用可用**（**GA**）的调试工具不足以在容器内部调试容器化进程。Docker 现在为你提供了一整套工具，用于有效地调试容器以及容器内部的进程。`docker exec`命令允许你登录到容器，而无需在容器内运行 SSH 守护进程。本章中你已经看到过每个调试工具的详细内容。

`docker stats`命令提供容器的内存和 CPU 使用信息。`docker events`命令报告事件，如创建、销毁和停止。同样，`docker logs`命令可以在不登录容器的情况下获取容器的日志。

下一步，你可以尝试最新的 Microsoft Visual Studio Docker 工具。它提供了一种一致的方式来开发和验证在 Linux Docker 容器中的应用程序。详情请参考[`docs.microsoft.com/en-us/azure/vs-azure-tools-docker-edit-and-refresh`](https://docs.microsoft.com/en-us/azure/vs-azure-tools-docker-edit-and-refresh)。

如果你想在 IDE（如 Visual Studio Code）中实时调试运行中的 Node.js 应用程序，可以参考这个博客：[`blog.docker.com/2016/07/live-debugging-docker/`](https://blog.docker.com/2016/07/live-debugging-docker/)。

下一章将阐述 Docker 容器可能面临的安全威胁，以及如何通过各种安全方法、自动化工具、最佳实践、关键指南和指标来应对这些威胁。我们还将讨论容器与虚拟机的安全性对比，并探讨 Docker 在第三方安全工具和实践方面的适应性。
