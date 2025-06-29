# 第四章：与容器共享数据

“一次只做一件事，并且做到最好”，这已经成为**信息技术**（**IT**）行业一个成功的座右铭，并且已经流行了很长时间。这一广泛使用的准则也非常适合构建和暴露 Docker 容器，并且被推荐作为获得 Docker 启发的容器化范式原本设想的好处的最佳实践之一。这意味着，我们必须将一个单一应用程序及其直接依赖和库写入一个 Docker 容器中，以确保容器的独立性、自给自足、横向可扩展性和机动性。让我们看看为什么容器如此重要：

+   **容器的时间性**：容器通常随着应用程序的生命周期而存在，反之亦然。然而，这对应用程序数据有一些负面影响。应用程序自然会经历各种变化，以适应业务和技术的变化，即使在生产环境中也是如此。还有其他原因，如应用程序故障、版本更替和应用程序维护，使得软件应用程序需要不断地更新和升级。在通用计算模型的情况下，即使应用程序因某种原因停止运行，与该应用程序相关的持久化数据仍然可以保存在文件系统中。然而，在容器范式下，应用程序的升级通常是通过系统地创建一个新的容器，其中包含更新版本的应用程序，并简单地丢弃旧的容器来完成的。同样，当应用程序发生故障时，需要启动一个新容器，并且必须丢弃旧的容器。总的来说，容器通常具有时间性。

+   **业务连续性的需求**：在容器环境中，完整的执行环境，包括其数据文件，通常被打包并封装在容器内。由于任何原因，当容器被丢弃时，应用程序的数据文件也会随容器一起消失。然而，为了提供不中断和不中断的服务，这些应用程序数据文件必须保存在容器外部，并在需要时传递到容器内，以确保业务连续性。这意味着容器的弹性和可靠性需要得到保证。此外，一些应用程序数据文件，如日志文件，需要在容器外部收集并访问，以进行各种后续分析。Docker 技术通过一种名为数据卷的新构件非常创新地解决了这个文件持久性问题。

Docker 技术提供了三种不同的持久存储方式：

+   第一个推荐的方法是使用通过 Docker 的卷管理创建的卷。

+   第二种方法是将 Docker 主机的目录挂载到容器内的指定位置。

+   另一种选择是使用数据专用容器。数据专用容器是一种特别设计的容器，用于与一个或多个容器共享数据。

本章将涵盖以下主题：

+   数据卷

+   共享主机数据

+   容器间共享数据

+   可避免的常见陷阱

## 数据卷

数据卷是 Docker 环境中数据共享的基础构建块。在深入了解数据共享的细节之前，必须先理解数据卷的概念。到目前为止，我们在镜像或容器中创建的所有文件都是联合文件系统的一部分。容器的联合文件系统随着容器的消失而消失。换句话说，当容器被删除时，它的文件系统也会被自动删除。然而，企业级应用程序必须持久化数据，容器的文件系统无法满足这种需求。

然而，Docker 生态系统通过数据卷概念优雅地解决了这个问题。数据卷本质上是 Docker 主机文件系统的一部分，它会被挂载到容器内。你也可以通过可插拔的卷驱动程序使用其他高级文件系统，如 Flocker 和 GlusterFS，作为数据卷。由于数据卷不是容器文件系统的一部分，它具有独立于容器的生命周期。

可以通过`Dockerfile`中的`VOLUME`指令将数据卷写入 Docker 镜像中。同时，也可以在启动容器时使用`docker run`子命令的`-v`选项指定数据卷。以下示例详细演示了`Dockerfile`中`VOLUME`指令的应用：

1.  创建一个非常简单的`Dockerfile`，包含基础镜像（`ubuntu:16.04`）和数据卷（`/MountPointDemo`）的指令：

```
      FROM ubuntu:16.04 
      VOLUME /MountPointDemo 

```

1.  使用`docker build`子命令，以`mount-point-demo`为名称构建镜像：

```
      $ sudo docker build -t mount-point-demo .

```

1.  构建完镜像后，让我们使用`docker inspect`子命令快速检查镜像中的数据卷：

```
 $ sudo docker inspect mount-point-demo
 [
 {
 "Id": "sha256:<64 bit hex id>",
 "RepoTags": [
 "mount-point-demo:latest"
 ],
 ... TRUNCATED OUTPUT ... 
 "Volumes": {
 "/MountPointDemo": {}
 },
 ... TRUNCATED OUTPUT ...

```

显然，在前面的输出中，数据卷已经被记录在镜像本身中。

1.  现在，让我们使用`docker run`子命令从之前创建的镜像启动一个交互式容器，如下所示：

```
      $ sudo docker run --rm -it mount-point-demo

```

从容器提示符下，让我们使用`ls -ld`命令检查数据卷的存在：

```
 root@8d22f73b5b46:/# ls -ld /MountPointDemo
 drwxr-xr-x 2 root root 4096 Nov 18 19:22 
 /MountPointDemo

```

如前所述，数据卷是 Docker 主机文件系统的一部分，它会被挂载，如下所示：

```
 root@8d22f73b5b46:/# mount | grep MountPointDemo
 /dev/xvda2 on /MountPointDemo type ext3 
 (rw,noatime,nobarrier,errors=remount-ro,data=ordered) 

```

1.  在这一节中，我们检查了镜像，以了解镜像中的数据卷声明。现在我们已经启动了容器，让我们使用`docker inspect`子命令并在不同的终端中提供容器 ID 作为参数，检查容器的数据卷。我们之前创建了一些容器，针对这个目的，我们直接从容器的提示符中获取`8d22f73b5b46`容器 ID：

```
 $ sudo docker inspect -f 
 '{{json .Mounts}}' 8d22f73b5b46 
 [
 {
 "Propagation": "",
 "RW": true,
 "Mode": "",
 "Driver": "local",
 "Destination": "/MountPointDemo",
 "Source":
"/var/lib/docker/volumes/720e2a2478e70a7cb49ab7385b8be627d4b6ec52e6bb33063e4144355d59592a/_data",
"Name": "720e2a2478e70a7cb49ab7385b8be627d4b6ec52e6bb33063e4144355d59592a"
 }
 ]

```

显然，在这里，数据卷被映射到 Docker 主机中的一个目录，并且该目录以读写模式挂载。这个目录，也称为卷，是 Docker 引擎在容器启动时自动创建的。从 Docker 1.9 版本开始，卷通过顶级卷管理命令进行管理，我们将在下一节中进一步深入探讨。

到目前为止，我们已经看到了`Dockerfile`中`VOLUME`指令的含义，以及 Docker 如何管理数据卷。像`Dockerfile`中的`VOLUME`指令一样，我们可以使用`docker run`子命令的`-v <容器挂载点路径>`选项，如下所示：

```
$ sudo docker run -v /MountPointDemo -it ubuntu:16.04  

```

启动容器后，我们建议您在新启动的容器中尝试执行`ls -ld /MountPointDemo`和`mount`命令，然后按照第 5 步所示检查容器。

在这里描述的两种情况中，Docker 引擎会自动在`/var/lib/docker/volumes/`目录下创建卷，并将其挂载到容器。当使用`docker rm`子命令删除容器时，Docker 引擎不会删除在容器启动时自动创建的卷。此行为是为了保持容器应用程序在卷文件系统中存储的状态。如果您想删除 Docker 引擎自动创建的卷，可以在删除容器时通过提供`-v`选项给`docker rm`子命令来实现，这适用于已停止的容器：

```
$ sudo docker rm -v 8d22f73b5b46  

```

如果容器仍在运行，则可以通过在先前命令中添加`-f`选项来删除容器以及自动生成的目录：

```
$ sudo docker rm -fv 8d22f73b5b46  

```

我们已经向您介绍了在 Docker 主机中自动生成目录并将其挂载到容器的数据卷的技巧和提示。然而，使用`docker run`子命令的`-v`选项时，可以将用户定义的目录挂载到数据卷。在这种情况下，Docker 引擎不会自动生成任何目录。

系统生成目录存在一个潜在的目录泄漏问题。换句话说，如果忘记删除系统生成的目录，可能会遇到一些不必要的问题。有关更多信息，请阅读本章中的*避免常见陷阱*部分。

## 卷管理命令

从 1.9 版本开始，Docker 引入了一个顶级卷管理命令，以便更有效地管理持久化文件系统。该卷管理命令能够管理 Docker 主机上的数据卷。除此之外，它还帮助我们通过可插拔的卷驱动程序（如 Flocker、GlusterFS 等）扩展 Docker 的持久化能力。你可以在[`docs.docker.com/engine/extend/legacy_plugins/`](https://docs.docker.com/engine/extend/legacy_plugins/)找到支持的插件列表。

`docker volume`命令支持以下四个子命令：

+   `create`：这将创建一个新的卷

+   `inspect`：这将显示一个或多个卷的详细信息

+   `ls`：这将列出 Docker 主机中的卷

+   `rm`：这将删除一个卷

让我们通过一些例子快速探索卷管理命令。你可以使用`docker volume create`子命令来创建一个卷，如下所示：

```
$ sudo docker volume create
50957995c7304e7d398429585d36213bb87781c53550b72a6a27c755c7a99639

```

上述命令将通过自动生成一个 64 位十六进制数字字符串作为卷名称来创建一个卷。然而，使用一个有意义的名称来命名卷会更有效，便于识别。你可以使用`--name`选项通过`docker volume create`子命令来命名卷：

```
$ sudo docker volume create --name example
example  

```

现在，我们已经创建了两个卷，一个有卷名，一个没有卷名，让我们使用`docker volume ls`子命令来显示它们：

```
$ sudo docker volume ls
DRIVER VOLUME NAME
local 50957995c7304e7d398429585d36213bb87781c53550b72a6a27c755c7a99639
local example  

```

列出所有卷后，让我们运行`docker volume inspect`子命令查看我们之前创建的卷的详细信息：

```
$ sudo docker volume inspect example
[
 {
 "Name": "example",
 "Driver": "local",
 "Mountpoint": 
 "/var/lib/docker/volumes/example/_data",
 "Labels": {},
 "Scope": "local"
 }
]

```

`docker volume rm`子命令允许你删除不再需要的卷：

```
$ sudo docker volume rm example
example

```

现在我们已经熟悉了 Docker 卷管理，让我们深入探讨接下来的数据共享部分。

## 共享主机数据

之前，我们描述了如何使用`Dockerfile`中的`VOLUME`指令在 Docker 镜像中创建数据卷。然而，为了确保 Docker 镜像的可移植性，Docker 并没有提供在构建时挂载主机目录或文件的机制。Docker 唯一提供的功能是在容器启动时将主机目录或文件挂载到容器的数据卷中。Docker 通过`docker run`子命令的`-v`选项暴露了主机目录或文件挂载的功能。`-v`选项有五种不同的格式，如下所示：

+   `-v <container mount path>`

+   `-v <host path>:<container mount path>`

+   `-v <host path>:<container mount path>:<read write mode>`

+   `-v <volume name>:<container mount path>`

+   `-v <volume name>:<container mount path>:<read write mode>`

`<host path>`格式是 Docker 主机中的绝对路径，`<container mount path>`是容器文件系统中的绝对路径，`<volume name>`是使用`docker volume create`子命令创建的卷的名称，`<read write mode>`可以是只读（`ro`）或读写（`rw`）模式。第一个`-v <container mount path>`格式在本章的*数据卷*部分已解释过，作为启动容器时创建挂载点的方法。第二和第三个格式使我们能够将 Docker 主机中的文件或目录挂载到容器挂载点。第四和第五个格式使我们能够将使用`docker volume create`子命令创建的卷挂载到容器。

我们希望通过一些例子深入挖掘，进一步了解主机的数据共享。在第一个例子中，我们将演示如何在 Docker 主机和容器之间共享一个目录；在第二个例子中，我们将演示文件共享。

在这里，在第一个例子中，我们从 Docker 主机挂载一个目录到容器，执行一些基本的文件操作，并验证这些操作在 Docker 主机上，具体步骤如下所示：

1.  首先，让我们使用`docker run`子命令的`-v`选项启动一个交互式容器，将 Docker 主机的`/tmp/hostdir`目录挂载到容器的`/MountPoint`目录：

```
      $ sudo docker run -v /tmp/hostdir:/MountPoint \
 -it ubuntu:16.04

```

如果在 Docker 主机上找不到`/tmp/hostdir`，Docker 引擎将会创建该目录。然而，问题是，系统生成的目录无法通过`docker rm`子命令的`-v`选项删除。

1.  成功启动容器后，我们可以使用`ls`命令检查`/MountPoint`的存在：

```
 root@4a018d99c133:/# ls -ld /MountPoint
 drwxr-xr-x 2 root root 4096 Nov 23 18:28 
 /MountPoint

```

1.  现在，我们可以继续使用`mount`命令检查挂载详细信息：

```
 root@4a018d99c133:/# mount | grep MountPoint
 /dev/xvda2 on /MountPoint type ext3 
 (rw,noatime,nobarrier,errors=
 remount-ro,data=ordered)

```

1.  在这里，我们将验证`/MountPoint`，使用`cd`命令切换到`/MountPoint`目录，使用`touch`命令创建一些文件，并使用`ls`命令列出这些文件，具体操作如下脚本所示：

```
 root@4a018d99c133:/# cd /MountPoint/
 root@4a018d99c133:/MountPoint# touch {a,b,c}
 root@4a018d99c133:/MountPoint# ls -l
 total 0
 -rw-r--r-- 1 root root 0 Nov 23 18:39 a
 -rw-r--r-- 1 root root 0 Nov 23 18:39 b
 -rw-r--r-- 1 root root 0 Nov 23 18:39 c

```

1.  使用新的终端，您可能需要验证`/tmp/hostdir` Docker 主机目录中的文件，使用`ls`命令查看，因为我们的容器是在现有终端的交互模式下运行的：

```
 $ sudo ls -l /tmp/hostdir/
 total 0
 -rw-r--r-- 1 root root 0 Nov 23 12:39 a
 -rw-r--r-- 1 root root 0 Nov 23 12:39 b
 -rw-r--r-- 1 root root 0 Nov 23 12:39 c

```

在这里，我们可以看到与第 4 步中相同的文件集。然而，您可能已经注意到文件时间戳的差异。这个时间差是由于 Docker 主机和容器之间的时区差异造成的。

1.  最后，让我们运行`docker inspect`子命令，并将`4a018d99c133`容器 ID 作为参数，查看是否设置了 Docker 主机和容器挂载点之间的目录映射，具体操作如下命令所示：

```
 $ sudo docker inspect \
 --format='{{json .Mounts}}' 4a018d99c133
 [{"Source":"/tmp/hostdir",
 "Destination":"/MountPoint","Mode":"",
 "RW":true,"Propagation":"rprivate"}]

```

显然，在前面的`docker inspect`子命令输出中，Docker 主机的`/tmp/hostdir`目录被挂载到容器的`/MountPoint`挂载点上。

在第二个示例中，我们将从 Docker 主机挂载一个文件到容器，更新容器中的文件，并按照以下步骤从 Docker 主机验证这些操作：

1.  为了将文件从 Docker 主机挂载到容器中，该文件必须先在 Docker 主机上存在。否则，Docker 引擎会创建一个指定名称的新目录，并将其作为目录挂载。我们可以通过使用`touch`命令在 Docker 主机上创建一个文件来开始：

```
      $ touch /tmp/hostfile.txt

```

1.  使用`docker run`子命令的`-v`选项启动一个交互式容器，将 Docker 主机上的`/tmp/hostfile.txt`文件挂载到容器的`/tmp/mntfile.txt`：

```
      $ sudo docker run -v /tmp/hostfile.txt:/mntfile.txt \
 -it ubuntu:16.04

```

1.  成功启动容器后，现在让我们使用`ls`命令检查`/mntfile.txt`的存在：

```
 root@d23a15527eeb:/# ls -l /mntfile.txt
 -rw-rw-r-- 1 1000 1000 0 Nov 23 19:33 /mntfile.txt

```

1.  然后，使用`mount`命令继续检查挂载详细信息：

```
 root@d23a15527eeb:/# mount | grep mntfile
 /dev/xvda2 on /mntfile.txt type ext3 
 (rw,noatime,nobarrier,errors=remount-ro,data=ordered)

```

1.  然后，使用`echo`命令将一些文本更新到`/mntfile.txt`：

```
      root@d23a15527eeb:/# echo "Writing from Container" 
 > mntfile.txt

```

1.  同时，切换到 Docker 主机的另一个终端，使用`cat`命令打印`/tmp/hostfile.txt`的内容：

```
 $ cat /tmp/hostfile.txt
 Writing from Container 

```

1.  最后，使用`docker inspect`子命令，并以`d23a15527eeb`容器 ID 作为参数，查看 Docker 主机与容器挂载点之间的文件映射：

```
 $ sudo docker inspect \
 --format='{{json .Mounts}}' d23a15527eeb
 [{"Source":"/tmp/hostfile.txt", 
 "Destination":"/mntfile.txt",
 "Mode":"","RW":true,"Propagation":"rprivate"}]

```

从前面的输出可以看出，Docker 主机中的`/tmp/hostfile.txt`文件已作为`/mntfile.txt`挂载到容器内。

对于最后一个示例，我们将创建一个 Docker 卷并将命名数据卷挂载到容器中。在这个示例中，我们不会像前两个示例那样进行验证步骤。然而，建议你按照第一个示例中的验证步骤进行操作。

1.  使用`docker volume create`子命令创建一个命名数据卷，如下所示：

```
      $ docker volume create --name namedvol

```

1.  现在，使用`docker run`子命令的`-v`选项启动一个交互式容器，将名为`namedvol`的数据卷挂载到容器的`/MountPoint`：

```
      $ sudo docker run -v namedvol:/MountPoint \
 -it ubuntu:16.04

```

在容器启动过程中，若`namedvol`还未创建，Docker 引擎会创建它。

1.  成功启动容器后，你可以重复第一示例中第 2 到第 6 步的验证步骤，你会发现这个示例也会有相同的输出模式。

### 主机数据共享的实用性

在上一章中，我们在 Docker 容器中启动了一个 HTTP 服务。然而，如果你记得的话，HTTP 服务的日志文件仍然在容器内部，无法直接从 Docker 主机访问。在本节中，我们将逐步阐明从 Docker 主机访问日志文件的过程：

1.  让我们从启动一个 Apache2 HTTP 服务容器开始，通过使用`docker run`子命令的`-v`选项，将 Docker 主机上的`/var/log/myhttpd`目录挂载到容器的`/var/log/apache2`目录。在这个示例中，我们利用了在上一章中构建的`apache2`镜像，执行以下命令：

```
      $ sudo docker run -d -p 80:80 \
 -v /var/log/myhttpd:/var/log/apache2 apache2
9c2f0c0b126f21887efaa35a1432ba7092b69e0c6d523ffd50684e27eeab37ac

```

如果你回忆起第六章中的`Dockerfile`，*在容器中运行服务*，`APACHE_LOG_DIR`环境变量通过`ENV`指令设置为`/var/log/apache2`目录。这将使得 Apache2 HTTP 服务将所有日志信息路由到`/var/log/apache2`数据卷。

1.  一旦容器启动，我们可以切换到 Docker 主机上的`/var/log/myhttpd`目录：

```
      $ cd /var/log/myhttpd

```

1.  或许，这里快速检查一下`/var/log/myhttpd`目录中的文件是合适的：

```
 $ ls -1
 access.log
 error.log
 other_vhosts_access.log

```

在这里，`access.log`文件包含了所有由 Apache2 HTTP 服务器处理的访问请求。`error.log`文件是一个非常重要的日志文件，记录了我们的 HTTP 服务器在处理任何 HTTP 请求时遇到的错误。`other_vhosts_access.log`文件是虚拟主机日志，在我们的案例中，它将始终为空。

1.  我们可以使用带有`-f`选项的`tail`命令来显示`/var/log/myhttpd`目录中所有日志文件的内容：

```
 $ tail -f *.log
 ==> access.log <==

 ==> error.log <==
 AH00558: apache2: Could not reliably determine the 
 server's fully qualified domain name, using 172.17.0.17\. 
 Set the 'ServerName' directive globally to suppress this 
 message
 [Thu Nov 20 17:45:35.619648 2014] [mpm_event:notice] 
 [pid 16:tid 140572055459712] AH00489: Apache/2.4.7 
 (Ubuntu) configured -- resuming normal operations
 [Thu Nov 20 17:45:35.619877 2014] [core:notice] 
 [pid 16:tid 140572055459712] AH00094: Command line: 
 '/usr/sbin/apache2 -D FOREGROUND'
 ==> other_vhosts_access.log <==

```

`tail -f`命令将持续运行，并显示文件的内容，一旦它们被更新。在这里，`access.log`和`other_vhosts_access.log`都为空，而`error.log`文件中有一些错误信息。显然，这些错误日志是由容器内运行的 HTTP 服务生成的。日志随后被存储在 Docker 主机目录中，该目录在容器启动时被挂载。

1.  在继续运行`tail -f *`时，让我们从容器内部的网页浏览器连接到 HTTP 服务，并观察日志文件：

```
 ==> access.log <==
 111.111.172.18 - - [20/Nov/2014:17:53:38 +0000] "GET / 
 HTTP/1.1" 200 3594 "-" "Mozilla/5.0 (Windows NT 6.1; 
 WOW64) 
 AppleWebKit/537.36 (KHTML, like Gecko) Chrome/39.0.2171.65 
 Safari/537.36"
 111.111.172.18 - - [20/Nov/2014:17:53:39 +0000] "GET 
 /icons/ubuntu-logo.png HTTP/1.1" 200 3688 
 "http://111.71.123.110/" "Mozilla/5.0 (Windows NT 6.1; 
 WOW64) 
 AppleWebKit/537.36 (KHTML, like Gecko) Chrome/39.0.2171.65 
 Safari/537.36"
 111.111.172.18 - - [20/Nov/2014:17:54:21 +0000] "GET 
 /favicon.ico HTTP/1.1" 404 504 "-" "Mozilla/5.0 (Windows 
 NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) 
 Chrome/39.0.2171.65 Safari/537.36"

```

HTTP 服务会更新`access.log`文件，我们可以通过`docker run`子命令的`-v`选项从挂载的主机目录进行操作。

## 在容器之间共享数据

在前一节中，你学到了 Docker 引擎如何无缝地实现 Docker 主机和容器之间的数据共享。尽管这是大多数使用案例的非常有效的解决方案，但也有一些使用场景需要在一个或多个容器之间共享数据。为了解决这种情况，Docker 提供的方案是使用`docker run`子命令的`--volume-from`选项，将一个容器的数据卷挂载到其他容器中。

### 仅数据容器

在 Docker 引入顶层卷管理功能之前，数据专用容器是实现数据持久性的推荐方法。了解数据专用容器是值得的，因为你会发现许多基于数据专用容器的实现。数据专用容器的主要职责是保留数据。创建数据专用容器与*数据卷*部分中展示的方法非常相似。此外，容器会显式命名，方便其他容器通过容器名称挂载数据卷。而且，即使数据专用容器处于停止状态，容器的数据卷仍然可以被其他容器访问。数据专用容器可以通过两种方式创建，如下所示：

+   在启动容器时，通过配置数据卷和容器的名称来实现。

+   数据卷也可以在构建镜像时通过`Dockerfile`进行配置，之后容器启动时可以指定容器名称。

在以下示例中，我们通过配置`docker run`子命令的`-v`和`--name`选项来启动数据专用容器，如下所示：

```
$ sudo docker run --name datavol \
 -v /DataMount \
 busybox:latest /bin/true

```

这里，容器是从`busybox`镜像启动的，该镜像因其较小的占用空间而广泛使用。在此，我们选择执行`/bin/true`命令，因为我们不打算在容器中进行任何操作。因此，我们使用`--name`选项将容器命名为`datavol`，并使用`docker run`子命令的`-v`选项创建了一个新的`/DataMount`数据卷。`/bin/true`命令立即以`0`退出状态退出，从而停止容器，并继续保持停止状态。

### 从其他容器挂载数据卷

Docker 引擎提供了一个便捷的接口，用于将一个容器的数据卷挂载（共享）到另一个容器。Docker 通过`docker run`子命令的`--volumes-from`选项提供这个接口。`--volumes-from`选项接受容器名称或容器 ID 作为输入，并自动挂载指定容器上的所有数据卷。Docker 允许你使用`--volumes-from`选项多次挂载多个容器的数据卷。

这是一个实际示例，展示了如何从另一个容器挂载数据卷，并一步步演示数据卷挂载过程：

1.  我们首先启动一个交互式的 Ubuntu 容器，并从上节中启动的数据专用容器（`datavol`）挂载数据卷：

```
      $ sudo docker run -it \
 --volumes-from datavol \
 ubuntu:latest /bin/bash

```

1.  现在，从容器的提示符开始，让我们使用`mount`命令验证数据卷挂载：

```
 root@e09979cacec8:/# mount | grep DataMount
 /dev/xvda2 on /DataMount type ext3 
 (rw,noatime,nobarrier,errors=remount-ro,data=ordered)

```

在这里，我们成功地从`datavol`数据专用容器挂载了数据卷。

1.  接下来，我们需要使用`docker inspect`子命令，从另一个终端检查此容器的数据卷：

```
      $ sudo docker inspect --format='{{json .Mounts}}' 
 e09979cacec8
 [{"Name":
 "7907245e5962ac07b31c6661a4dd9b283722d3e7d0b0fb40a90
 43b2f28365021","Source":
 "/var/lib/docker/volumes 
 /7907245e5962ac07b31c6661a4dd9b283722d3e7d0b0fb40a9043b
 2f28365021/_data","Destination":"
 /DataMount","Driver":"local","Mode":"",
 "RW":true,"Propagation":""}]

```

显然，来自`datavol`数据-only 容器的数据卷被挂载，就像它们直接挂载到此容器一样。

我们可以从另一个容器挂载数据卷，并展示挂载点。通过使用数据卷在容器之间共享数据，我们可以使挂载的数据显示工作，正如这里所示：

1.  让我们重用在前一个示例中启动的容器，并通过向文件中写入一些文本，在`/DataMount`数据卷中创建`/DataMount/testfile`文件，如下所示：

```
      root@e09979cacec8:/# echo \
 "Data Sharing between Container" > \
 /DataMount/testfile 

```

1.  只需启动一个容器，使用`cat`命令显示我们在上一步中写入的文本：

```
      $ sudo docker run --rm \
 --volumes-from datavol \
 busybox:latest cat /DataMount/testfile

```

以下是前述命令的典型输出：

```
      Data Sharing between Container

```

显然，前述`容器之间的数据共享`输出来自我们新创建的容器化`cat`命令，它是我们在步骤 1 中写入`datavol`容器`/DataMount/testfile`的文本。

酷吧？你可以通过共享数据卷无缝地在容器之间共享数据。在这个示例中，我们使用数据-only 容器作为数据共享的基础容器。然而，Docker 允许我们共享任何类型的数据卷，并将数据卷一个接一个地挂载，正如这里所示：

```
$ sudo docker run --name vol1 --volumes-from datavol \
 busybox:latest /bin/true
$ sudo docker run --name vol2 --volumes-from vol1 \
 busybox:latest /bin/true

```

在`vol1`容器中，我们从`datavol`容器挂载了数据卷。然后，在`vol2`容器中，我们从`vol1`容器挂载了数据卷，而该数据卷最终来自`datavol`容器。

### 容器之间数据共享的实用性

在本章前面，你学习了如何从 Docker 主机访问 Apache2 HTTP 服务的日志文件。尽管通过挂载 Docker 主机目录到容器来共享数据非常方便，但后来我们了解到，容器之间可以通过仅使用数据卷来共享数据。因此，在这里，我们通过容器之间共享数据来改变 Apache2 HTTP 服务日志处理的方法。为了在容器之间共享日志文件，我们将根据以下步骤启动以下容器：

1.  首先，创建一个数据-only 容器，将数据卷暴露给其他容器。

1.  然后，一个利用数据-only 容器数据卷的 Apache2 HTTP 服务容器。

1.  一个用于查看由 Apache2 HTTP 服务生成的日志文件的容器。

如果你在 Docker 主机的`80`端口运行任何 HTTP 服务，请为以下示例选择一个未使用的端口。如果没有，请首先停止 HTTP 服务，然后继续示例，以避免端口冲突。

现在，我们将细致地带你一步步完成制作相应镜像并启动容器以查看日志文件的过程：

1.  在这里，我们开始使用 `VOLUME` 指令在 `Dockerfile` 中创建 `/var/log/apache2` 数据卷。这个 `/var/log/apache2` 数据卷直接映射到 `APACHE_LOG_DIR`，这是在第六章中通过 `ENV` 指令设置的环境变量，*在容器中运行服务*：

```
      ####################################################### 
      # Dockerfile to build a LOG Volume for Apache2 Service 
      ####################################################### 
      # Base image is BusyBox 
      FROM busybox:latest 
      # Author: Dr. Peter 
      MAINTAINER Dr. Peter <peterindia@gmail.com> 
      # Create a data volume at /var/log/apache2, which is 
      # same as the log directory PATH set for the apache image 
      VOLUME /var/log/apache2 
      # Execute command true 
      CMD ["/bin/true"] 

```

由于这个 `Dockerfile` 是为了启动仅数据容器而设计的，因此默认执行命令被设置为 `/bin/true`。

1.  我们将继续使用 `docker build` 从前面的 `Dockerfile` 构建一个名为 `apache2log` 的 Docker 镜像，如下所示：

```
 $ sudo docker build -t apache2log .
 Sending build context to Docker daemon 2.56 kB
 Sending build context to Docker daemon
 Step 0 : FROM busybox:latest
 ... TRUNCATED OUTPUT ...

```

1.  使用 `docker run` 子命令从 `apache2log` 镜像启动一个仅数据容器，并使用 `--name` 选项将生成的容器命名为 `log_vol`：

```
      $ sudo docker run --name log_vol apache2log

```

执行前述命令后，容器将在 `/var/log/apache2` 中创建一个数据卷，并将其移动到停止状态。

1.  与此同时，您可以使用 `docker ps` 子命令并加上 `-a` 选项来验证容器的状态：

```
 $ sudo docker ps -a
 CONTAINER ID IMAGE COMMAND 
 CREATED STATUS PORTS 
 NAMES
 40332e5fa0ae apache2log:latest "/bin/true" 
 2 minutes ago Exited (0) 2 minutes ago 
 log_vol

```

根据输出，容器以 `0` 的退出值退出。

1.  使用 `docker run` 子命令启动 Apache2 HTTP 服务。在这里，我们将重用在第六章中构建的 `apache2` 镜像，*在容器中运行服务*。此外，在这个容器中，我们将使用 `--volumes-from` 选项从第 3 步启动的仅数据容器 `log_vol` 挂载 `/var/log/apache2` 数据卷：

```
      $ sudo docker run -d -p 80:80 \
 --volumes-from log_vol \
 apache2
 7dfbf87e341c320a12c1baae14bff2840e64afcd082dda3094e7cb0a0023cf42  

```

在成功启动带有从 `log_vol` 挂载的 `/var/log/apache2` 数据卷的 Apache2 HTTP 服务后，我们可以通过临时容器访问日志文件。

1.  在这里，我们列出了由 Apache2 HTTP 服务存储的文件，使用了一个临时容器。这个临时容器通过挂载来自 `log_vol` 的 `/var/log/apache2` 数据卷来启动，并且使用 `ls` 命令列出了 `/var/log/apache2` 中的文件。此外，`docker run` 子命令的 `--rm` 选项用于在执行完 `ls` 命令后移除容器：

```
 $ sudo docker run --rm \
 --volumes-from log_vol \
 busybox:latest ls -l /var/log/apache2
 total 4
 -rw-r--r-- 1 root root 0 Dec 5 15:27 
 access.log
 -rw-r--r-- 1 root root 461 Dec 5 15:27 
 error.log
 -rw-r--r-- 1 root root 0 Dec 5 15:27 
 other_vhosts_access.log

```

1.  最后，使用 `tail` 命令访问 Apache2 HTTP 服务生成的错误日志，命令如下所示：

```
 $ sudo docker run --rm \
 --volumes-from log_vol \
 ubuntu:16.04 \
 tail /var/log/apache2/error.log
 AH00558: apache2: Could not reliably determine the 
 server's fully qualified domain name, using 172.17.0.24\. 
 Set the 'ServerName' directive globally to suppress this 
 message
 [Fri Dec 05 17:28:12.358034 2014] [mpm_event:notice] 
 [pid 18:tid 140689145714560] AH00489: Apache/2.4.7 
 (Ubuntu) configured -- resuming normal operations
 [Fri Dec 05 17:28:12.358306 2014] [core:notice] 
 [pid 18:tid 140689145714560] AH00094: Command line: 
 '/usr/sbin/apache2 -D FOREGROUND'

```

## 避免常见的陷阱

到目前为止，我们已经讨论了数据卷如何有效地用于在 Docker 主机与容器之间、以及容器之间共享数据。使用数据卷进行数据共享正变得越来越强大且在 Docker 模式中不可或缺。然而，它也带来了一些需要小心识别并消除的陷阱。在本节中，我们尝试列出一些与数据共享相关的常见问题，并提出解决方法。

### 目录泄漏

在*数据卷*章节中，你已了解 Docker 引擎会根据`Dockerfile`中的`VOLUME`指令以及`docker run`子命令的`-v`选项自动创建目录。我们还明白，Docker 引擎不会自动删除这些自动生成的目录，以保持容器中运行的应用程序的状态。我们可以使用`docker rm`子命令的`-v`选项强制 Docker 删除这些目录。手动删除过程面临两大挑战，分别如下：

+   **未删除的目录：** 可能出现某些情况下，你可能有意或无意选择不删除生成的目录，即使在删除容器时。

+   **第三方镜像：** 我们经常利用可能已使用`VOLUME`指令构建的第三方 Docker 镜像。同样，我们也可能拥有自己带有`VOLUME`指令的 Docker 镜像。当我们使用这些 Docker 镜像启动容器时，Docker 引擎会自动生成所需的目录。由于我们无法预知数据卷的创建，因此可能无法使用`docker rm`子命令的`-v`选项删除这些自动生成的目录。

在前述场景中，一旦关联的容器被删除，就没有直接方法来识别已删除容器的目录。以下是一些避免此问题的建议：

+   始终使用`docker inspect`子命令检查 Docker 镜像，并检查镜像中是否包含任何数据卷。

+   始终使用`docker rm`子命令的`-v`选项删除为容器创建的任何数据卷（目录）。即使数据卷被多个容器共享，使用`docker rm`子命令的`-v`选项仍然是安全的，因为与数据卷关联的目录仅会在最后一个共享该数据卷的容器被删除时才会被删除。

+   如果出于某种原因，你选择保留自动生成的目录，你必须保持清晰的记录，以便稍后能够删除它们。

+   实现一个审计框架，用于审计并找出与任何容器无关的目录。

### 数据卷的副作用

如前所述，Docker 通过在构建时使用`VOLUME`指令使我们能够访问 Docker 镜像中的每个数据卷。然而，数据卷不应在构建时用于存储任何数据，否则将导致不良效果。

本节中，我们将通过编写一个`Dockerfile`展示在构建过程中使用数据卷的副作用，并通过构建此`Dockerfile`来展示其影响。

以下是`Dockerfile`的详细信息：

1.  使用 Ubuntu 16.04 作为基础镜像构建镜像：

```
      # Use Ubuntu as the base image 
      FROM ubuntu:16.04 

```

1.  使用`VOLUME`指令创建`/MountPointDemo`数据卷：

```
      VOLUME /MountPointDemo 

```

1.  使用`RUN`指令在`/MountPointDemo`数据卷中创建一个文件：

```
      RUN date > /MountPointDemo/date.txt 

```

1.  使用`RUN`指令显示`/MountPointDemo`数据卷中的文件：

```
      RUN cat /MountPointDemo/date.txt 

```

1.  使用`docker build`子命令从这个`Dockerfile`构建镜像，如下所示：

```
 $ sudo docker build -t testvol .
 Sending build context to Docker daemon 2.56 kB
 Sending build context to Docker daemon
 Step 0 : FROM ubuntu:16.04
 ---> 9bd07e480c5b
 Step 1 : VOLUME /MountPointDemo
 ---> Using cache
 ---> e8b1799d4969
 Step 2 : RUN date > /MountPointDemo/date.txt
 ---> Using cache
 ---> 8267e251a984
 Step 3 : RUN cat /MountPointDemo/date.txt
 ---> Running in a3e40444de2e
 cat: /MountPointDemo/date.txt: No such file or directory
 2014/12/07 11:32:36 The command [/bin/sh -c cat 
 /MountPointDemo/date.txt] returned a non-zero code: 1

```

在`docker build`子命令的前面输出中，你会注意到构建在第 3 步失败，因为它找不到第 2 步中创建的文件。显然，第 2 步中创建的文件在到达第 3 步时消失了。这个不希望发生的效果是由于 Docker 构建镜像的方式造成的。理解 Docker 的镜像构建过程可以揭开这个谜团。

在构建过程中，对于`Dockerfile`中的每一条指令，遵循以下步骤：

1.  通过将`Dockerfile`指令转换为等效的`docker run`子命令来创建一个新容器。

1.  将新创建的容器提交为镜像。

1.  将第 1 步和第 2 步重复，处理新创建的镜像作为第 1 步的基础镜像。

当一个容器被提交时，它会保存容器的文件系统，并故意不保存数据卷的文件系统。因此，存储在数据卷中的任何数据将在此过程中丢失。所以，在构建过程中，绝对不要使用数据卷作为存储。

## 概要

对于企业级分布式应用程序来说，数据是最重要的工具和组成部分，使得其操作和输出独具特色。借助 IT 容器化，整个过程以快速且明亮的方式开始。通过智能利用 Docker 引擎，IT 和商业软件解决方案被高效容器化。然而，最初的动因是需要更快且无误地实现具备应用感知的 Docker 容器，因此数据与容器内的应用紧密耦合。然而，这种紧密关系也带来了实际的风险。如果应用崩溃，数据也会丢失。此外，多个应用可能依赖于相同的数据，因此数据必须在多个应用之间共享。

本章我们讨论了 Docker 引擎在促进 Docker 主机与容器之间以及容器之间无缝共享数据的能力。数据卷被规定为促进日益增长的 Docker 生态系统中各个组成部分之间共享数据的基础构建块。在下一章，我们将解释容器编排的概念，并看看如何通过一些自动化工具简化这一复杂的方面。编排对于实现复合容器是必不可少的。
