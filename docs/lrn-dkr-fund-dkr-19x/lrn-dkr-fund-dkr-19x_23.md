# 第二十章：评估

# 第一章

以下是本章提出问题的一些示例答案：

1.  正确答案是**D**和**E**。

1.  Docker 容器对 IT 的意义就像运输行业中的集装箱对运输业的意义一样。它定义了如何打包货物的标准。在这里，货物就是开发人员编写的应用程序。供应商（在这种情况下是开发人员）负责将货物打包进容器并确保一切按预期装配好。一旦货物被打包进容器，它就可以被运输。由于这是标准集装箱，运输商可以标准化他们的运输方式，如卡车、火车或船只。运输商并不关心容器里装的是什么。此外，从一种运输方式到另一种运输方式（例如从火车到船）的装卸过程可以高度标准化。这极大提高了运输效率。与此类似，IT 中的运维工程师可以将开发人员构建的软件容器运送到生产系统并在那儿以高度标准化的方式运行，而无需担心容器中装的是什么。它将按预期工作。

1.  容器之所以能改变游戏规则的原因如下：

    +   容器是自包含的，因此如果它们在一个系统上运行，它们就能在任何能运行容器的地方运行。

    +   容器可以在本地和云端运行，也可以在混合环境中运行。这对今天的典型企业非常重要，因为它允许从本地环境到云端的平滑过渡。

    +   容器镜像由最了解的人——开发人员来构建或打包。

    +   容器镜像是不可变的，这对于良好的发布管理非常重要。

    +   容器是基于封装（使用 Linux 命名空间和 cgroups）、机密、内容信任和镜像漏洞扫描的安全软件供应链的推动者。

1.  任何给定的容器可以在任何容器可以运行的地方运行，原因如下：

+   +   容器是自包含的黑盒子。它们不仅封装了一个应用程序，还封装了它的所有依赖项，如库、框架、配置数据、证书等。

    +   容器基于广泛接受的标准，如 OCI。

1.  答案是**B**。容器对现代应用程序以及将传统应用程序容器化都非常有用。企业在进行后者时能获得巨大的收益。据报道，维护传统应用程序的成本可以节省 50%以上。此类传统应用程序的新版本发布之间的时间可减少多达 90%。这些数据已由实际企业客户公开报告。

1.  50%以上。

1.  容器基于 Linux 命名空间（网络、进程、用户等）和 cgroups（控制组）。

# 第二章

以下是本章提出问题的一些示例答案：

1.  `docker-machine`可以用于以下场景：

    1.  在多个提供商（如 VirtualBox、Hyper-V、AWS、MS Azure 或 Google Compute Engine）上创建一个虚拟机，该虚拟机将作为 Docker 主机。

    1.  启动、停止或终止先前生成的虚拟机。

    1.  用这个工具创建的本地或远程 Docker 主机虚拟机中使用 SSH。

    1.  重新生成用于安全使用 Docker 主机虚拟机的证书。

1.  A. 正确。是的，使用 Docker for Windows，您可以开发并运行 Linux 容器。使用本书未讨论的此版本的 Docker for Desktop，您也可以开发并运行原生 Windows 容器。对于 macOS 版本，您只能开发和运行 Linux 容器。

1.  脚本用于自动化过程，从而避免人为错误。构建、测试、共享和运行 Docker 容器是应该始终自动化的任务，以提高其可靠性和可重复性。

1.  以下 Linux 发行版已通过认证，可以运行 Docker：RedHat Linux (RHEL)、CentOS、Oracle Linux、Ubuntu 等。

1.  以下 Windows 操作系统已通过认证，可以运行 Docker：Windows 10 专业版、Windows Server 2016 和 Windows Server 2019。

# 第三章

以下是本章中提出问题的一些示例答案：

1.  Docker 容器的可能状态如下：

    +   `created`：一个已创建但未启动的容器

    +   `restarting`：一个正在重启过程中的容器

    +   `running`：一个当前正在运行的容器

    +   `paused`：一个其进程已被暂停的容器

    +   `exited`：一个已运行并完成的容器

    +   `dead`：一个 Docker 引擎尝试但未能停止的容器

1.  我们可以使用 `docker container ls`（或者旧的、简短的版本 `docker ps`）列出当前在 Docker 主机上运行的所有容器。请注意，这不会列出已停止的容器，若要查看这些容器需要使用额外的参数 `--all`（或 `-a`）。

1.  要列出所有容器的 ID，无论是运行中的还是已停止的，我们可以使用 `docker container ls -a -q`，其中 `-q` 表示仅输出容器 ID。

# 第四章

以下是本章中提出问题的一些示例答案：

1.  `Dockerfile` 可能如下所示：

```
FROM ubuntu:19.04
RUN apt-get update && \
    apt-get install -y iputils-ping
CMD ping 127.0.0.1
```

请注意，在 Ubuntu 中，`ping` 工具是 `iputils-ping` 包的一部分。使用 `docker image build -t my-pinger` 构建名为 `pinger` 的镜像——例如。

2. `Dockerfile` 可能如下所示：

```
FROM alpine:latest
RUN apk update && \
    apk add curl
```

使用 `docker image build -t my-alpine:1.0` 构建镜像。

3. 一个 Go 应用的 `Dockerfile` 可能如下所示：

```
FROM golang:alpine
WORKDIR /app
ADD . /app
RUN cd /app && go build -o goapp
ENTRYPOINT ./goapp
```

您可以在 `~/fod/ch04/answer03` 文件夹中找到完整的解决方案。

4. 一个 Docker 镜像具有以下特点：

1. 它是不可变的。

2. 它由一到多个层组成。

3. 它包含了运行打包应用所需的文件和文件夹。

5. **C.** 首先，您需要登录 Docker Hub；然后，使用用户名正确标记您的镜像；最后，推送镜像。

# 第五章

以下是本章中提出问题的一些示例答案：

玩转卷的最简单方法是使用 Docker Toolbox，因为在直接使用 Docker for Desktop 时，卷被存储在 Docker for Desktop 透明使用的（稍微隐蔽的）Linux 虚拟机中。

因此，我们建议执行以下操作：

```
$ docker-machine create --driver virtualbox volume-test
$ docker-machine ssh volume-test
```

既然你已经进入名为 `volume-test` 的 Linux 虚拟机，你可以进行以下练习：

1.  要创建一个命名卷，请执行以下命令：

```
$ docker volume create my-products
```

1.  执行以下命令：

```
$ docker container run -it --rm \
 -v my-products:/data:ro \
 alpine /bin/sh
```

1.  要获取主机上卷的路径，请使用以下命令：

```
$ docker volume inspect my-products | grep Mountpoint
```

这（如果你使用的是 `docker-machine` 和 VirtualBox）应当产生如下结果：

```
"Mountpoint": "/mnt/sda1/var/lib/docker/volumes/myproducts/_data"
```

现在执行以下命令：

```
$ sudo su
$ cd /mnt/sda1/var/lib/docker/volumes/my-products/_data
$ echo "Hello world" > sample.txt
$ exit
```

1.  执行以下命令：

```
$ docker run -it --rm -v my-products:/data:ro alpine /bin/sh
/ # cd /data
/data # cat sample.txt
```

在另一个终端中，执行此命令：

```
$ docker run -it --rm -v my-products:/app-data alpine /bin/sh
/ # cd /app-data
/app-data # echo "Hello other container" > hello.txt
/app-data # exit
```

1.  执行如下命令：

```
$ docker container run -it --rm \
 -v $HOME/my-project:/app/data \
 alpine /bin/sh
```

1.  退出两个容器，然后返回主机，执行此命令：

```
$ docker volume prune
```

1.  答案是 B。每个容器都是一个沙箱，因此拥有自己独立的环境。

1.  收集所有环境变量及其对应的值到一个配置文件中，然后通过 `--env-file` 命令行参数将其提供给容器，在 `docker run` 命令中使用，示例如下：

```
$ docker container run --rm -it \
 --env-file ./development.config \
 alpine sh -c "export"
```

# 第六章

以下是本章问题的示例答案：

1.  可能的答案：a) 将源代码挂载到容器中；b) 使用工具，在检测到代码更改时自动重新启动容器中的应用；c) 配置容器进行远程调试。

1.  你可以将包含源代码的文件夹从主机挂载到容器中。

1.  如果你不能轻松通过单元测试或集成测试覆盖某些场景，并且如果在主机上运行应用程序时无法重现观察到的行为，或者无法直接在主机上运行应用程序（由于缺乏必要的语言或框架），那么容器化的测试可以帮助解决这些问题。

1.  一旦应用程序在生产环境中运行，作为开发人员，我们就不能轻易访问它。如果应用程序表现出异常行为或甚至崩溃，日志通常是我们唯一可以用来帮助我们重现情况并找出根本原因的来源。

# 第七章

以下是本章问题的示例答案：

1.  优缺点：

    +   优点：我们不需要在主机上安装任务所需的特定 Shell、工具或语言。

    +   优点：我们可以在任何 Docker 主机上运行，从树莓派到大型主机，唯一的要求是主机能运行容器。

    +   优点：成功运行后，工具会在容器被移除时从主机上被清除，不会留下任何痕迹。

    +   缺点：我们需要在主机上安装 Docker。

    +   缺点：用户需要具备基本的 Docker 容器知识。

    +   缺点：与本地使用工具相比，使用该工具的方式稍显间接。

1.  在容器中运行测试具有以下优点：

    +   它们在开发者机器上运行和在测试或 CI 系统中运行一样顺畅。

    +   每次测试运行时，保持相同的初始条件会更容易。

    +   所有开发人员使用相同的环境设置，例如库和框架的版本。

1.  在这里，我们期望看到一个图示，展示一个开发人员写代码并提交代码，例如到 GitHub。然后我们希望看到一个自动化服务器，如 Jenkins 或 TeamCity，图中显示它定期轮询 GitHub 是否有更新，或者是 GitHub 触发自动化服务器（通过 HTTP 回调）创建新的构建。图中还应显示自动化服务器运行所有测试，以验证构建的产物，如果所有测试通过，自动将应用程序或服务部署到集成系统中，在那里再次进行测试，例如进行一些冒烟测试。再次，如果这些测试通过，自动化服务器应该要求人工批准是否将其部署到生产环境中（这等同于持续交付），或者自动化服务器应自动将其部署到生产环境中（持续部署）。

# 第八章

这里是本章提出问题的一些示例答案：

1.  你可能在一个资源或功能有限的工作站上工作，或者你的工作站可能被公司锁定，禁止安装任何未获官方批准的软件。有时，你可能需要使用公司尚未批准的语言或框架进行概念验证或实验（但如果概念验证成功，未来可能会被批准）。

1.  当容器化应用程序需要自动化某些与容器相关的任务时，将 Docker 套接字绑定到容器中是推荐的方式。这可以是像 Jenkins 这样的自动化服务器应用程序，通常用于构建、测试和部署 Docker 镜像。

1.  大多数商业应用程序在执行任务时不需要根级别的授权。从安全角度来看，因此强烈建议以最小权限运行此类应用程序，确保它们仅拥有完成任务所需的最低权限。任何不必要的提升权限可能会被黑客在恶意攻击中利用。通过以非 root 用户身份运行应用程序，你可以使潜在的黑客更难以入侵系统。

1.  卷包含数据，而数据的生命周期通常需要远远超过容器或应用程序的生命周期。数据通常是任务关键的，需要安全存储数天、数月，甚至数年。当你删除一个卷时，你会不可逆地删除与其相关的数据。因此，删除卷时一定要确认你知道自己在做什么。

# 第九章

这里是本章提出问题的一些示例答案：

1.  在分布式应用架构中，软件和基础设施的每一部分都需要在生产环境中具有冗余，以确保应用程序的持续运行时间是至关重要的。高度分布式的应用由多个部分组成，随着部分数量的增加，某个部分发生故障或异常行为的可能性也在增加。可以保证，给定足够的时间，每个部分最终都会失败。为了避免应用程序的停机，我们需要对每个部分进行冗余，无论是服务器、网络交换机，还是在容器中集群节点上运行的服务。

1.  在高度分布式、可扩展和容错的系统中，应用程序的单个服务可能会由于扩展需求或组件故障而发生变化。因此，我们不能将不同的服务硬编码在一起。服务 A 需要访问服务 B 时，不应该知道服务 B 的 IP 地址等细节。它应该依赖于外部提供者来获取这些信息。DNS 就是这种位置服务提供者。服务 A 只需要告诉它要与服务 B 通信，DNS 服务将会处理具体的细节。

1.  熔断器是一种避免级联故障的机制，适用于分布式应用中出现故障或异常行为的组件。类似于电路中的熔断器，软件驱动的熔断器切断客户端与故障服务之间的通信。如果调用的服务发生故障，熔断器将直接将错误报告给客户端组件。这为系统提供了从故障中恢复或修复的机会。

1.  单体应用比多服务应用更容易管理，因为它由一个单一的部署包组成。另一方面，单体应用更难以扩展以应对需求的增加。在分布式应用中，每个服务可以单独扩展，并且每个服务可以运行在优化的基础设施上，而单体应用则需要在能够支持所有或大部分功能的基础设施上运行。维护和更新单体应用比多服务应用困难得多，后者每个服务都可以独立更新和部署。单体应用通常是一个庞大、复杂且紧密耦合的代码堆栈。即使是小的修改也可能会带来意想不到的副作用。而（微）服务则是自包含、简单的组件，像黑盒一样工作。依赖的服务对服务的内部工作一无所知，因此也不依赖于它。

1.  蓝绿部署是一种软件部署方式，允许在零停机时间的情况下部署应用或应用服务的新版本。例如，如果服务 A 需要更新到新版本，那么当前运行的版本被称为蓝色。新版本的服务被部署到生产环境中，但尚未与其余应用连接。这个新版本被称为绿色。一旦部署成功并且冒烟测试表明它准备就绪，路由器会被重新配置，将流量从蓝色切换到绿色。观察绿色的表现一段时间，如果一切正常，蓝色将被停用。另一方面，如果绿色出现问题，路由器可以简单地切换回蓝色，绿色可以修复并重新部署。

# 第十章

以下是本章提出问题的部分示例答案：

1.  三个核心元素是**沙盒**、**端点**和**网络**。

1.  执行此命令：

```
$ docker network create --driver bridge frontend
```

1.  执行此命令：

```

$ docker container run -d --name n1 \
 --network frontend -p 8080:80 nginx:alpine
$ docker container run -d --name n2 \
 --network frontend -p 8081:80 nginx:alpine
```

测试两个 NGINX 实例是否都在运行：

```
$ curl -4 localhost:8080
$ curl -4 localhost:8081
```

在这两种情况下，你都应该看到 NGINX 的欢迎页面。

1.  要获取所有附加容器的 IP 地址，请运行此命令：

```
$ docker network inspect frontend | grep IPv4Address
```

你应该会看到类似以下的内容：

```
"IPv4Address": "172.18.0.2/16",
"IPv4Address": "172.18.0.3/16",
```

要获取网络使用的子网，请使用以下命令（例如）：

```
$ docker network inspect frontend | grep subnet
```

你应该收到类似以下的内容（来自前面的示例）：

```
"Subnet": "172.18.0.0/16",
```

1.  `host`网络允许我们在主机的网络命名空间中运行容器。

1.  仅在调试目的或构建系统级工具时使用此网络。切勿将`host`网络用于运行生产环境的应用容器！

1.  `none`网络基本上是指容器未连接到任何网络。它应仅用于不需要与其他容器通信且不需要从外部访问的容器。

1.  例如，`none`网络可用于运行批处理进程的容器，这些进程仅需要访问本地资源，例如通过主机挂载卷访问的文件。

1.  Traefik 可用于提供第 7 层或应用层路由。如果你想将某个功能从单体应用中拆分并且有明确的 API 定义，这将特别有用。在这种情况下，你需要将某些 HTTP 调用重新路由到新容器/服务。这只是可能的使用场景之一，但也是最重要的一个。另一个场景是将 Traefik 用作负载均衡器。

# 第十一章

以下是本章提出问题的部分示例答案：

1.  以下代码可用于以分离模式或守护进程模式运行应用：

```
$ docker-compose up -d
```

1.  执行以下命令以显示运行中的服务的详细信息：

```
$ docker-compose ps
```

这应该会产生如下输出：

```
Name               Command               State  Ports
-------------------------------------------------------------------
mycontent_nginx_1  nginx -g daemon off;  Up     0.0.0.0:3000->80/tcp
```

1.  以下命令可用于扩展 Web 服务：

```
$ docker-compose up --scale web=3
```

# 第十二章

以下是本章提出问题的部分示例答案：

1.  一个关键任务的、高可用的应用程序，它实现为一个高度分布的互联应用服务系统，过于复杂，无法手动监控、操作和管理。容器编排工具在这方面提供帮助。它们自动化大多数典型任务，比如协调期望状态，或收集和聚合系统的关键指标。人类无法快速反应使得这样的应用程序具备弹性或自愈能力。需要以容器编排工具形式的 软件支持。

1.  容器编排工具将我们从以下繁琐的日常任务中解放出来：

    +   扩展服务的规模

    +   负载均衡请求

    +   将请求路由到目标

    +   监控服务实例的健康状况

    +   保护分布式应用程序

1.  在这个领域的赢家是 Kubernetes，它是开源的，属于 CNCF。最初由 Google 开发。我们还有 Docker Swarm，它是专有的，由 Docker 开发。AWS 提供了一种名为 ECS 的容器服务，也是专有的，并且与 AWS 生态系统紧密集成。最后，微软提供了 AKS，它与 AWS ECS 有相同的优缺点。

# 第十三章

这里是本章提出问题的一些示例答案：

1.  正确答案如下：

```
$ docker swarm init [--advertise-addr <IP address>]
```

`--advertise-addr` 是可选的，只有当主机有多个 IP 地址时才需要。

1.  在你要移除的工作节点上，执行以下命令：

```
 $ docker swarm leave
```

在一个主节点上，执行命令 `$ docker node rm -f <node ID>`，其中 `<node ID>` 是要移除的工作节点的 ID。

1.  正确答案如下：

```
$ docker network create \
 --driver overlay \
 --attachable \
 front-tier
```

1.  正确答案如下：

```
$ docker service create --name web \
 --network front-tier \
 --replicas 5 \
 -p 3000:80 \
 nginx:alpine
```

1.  正确答案如下：

```
$ docker service update --replicas 3 web
```

# 第十四章

这里是本章提出问题的一些示例答案：

1.  零停机部署意味着分布式应用程序中服务的新版本可以更新为新版本，而不需要应用程序停止工作。通常，使用 Docker SwarmKit 或 Kubernetes（如我们将看到的），这是以滚动方式进行的。一个服务由多个实例组成，且这些实例是分批更新的，这样大部分实例始终在运行。

1.  默认情况下，Docker SwarmKit 使用滚动更新策略来实现零停机部署。

1.  容器是自包含的部署单元。如果部署的新版本服务无法正常工作，我们（或系统）只需要回滚到之前的版本。服务的前一个版本也是以自包含的容器形式部署的。从概念上讲，前进（更新）或后退（回滚）没有区别。一个版本的容器被另一个版本替代。主机本身不受这些更改的影响。

1.  Docker 密钥在静态存储时是加密的。它们只会传输给使用这些密钥的服务和容器。由于群集节点之间的通信使用相互 TLS，因此密钥是加密传输的。密钥永远不会物理存储在工作节点上。

1.  实现此操作的命令如下：

```
$ docker service update --image acme/inventory:2.1 \
 --update-parallelism 2 \
 --update-delay 60s \
 inventory
```

6. 首先，我们需要将旧的密钥从服务中删除，然后需要将新版本的密钥添加进去（无法直接更新密钥）：

```
$ docker service update \
 --secret-rm MYSQL_PASSWORD \
 inventory
$ docker service update \
 --secret-add source=MYSQL_PASSWORD_V2, target=MYSQL_PASSWORD \
 inventory
```

# 第十五章

以下是本章提出的问题的一些示例答案：

1.  Kubernetes 主节点负责管理集群。所有创建对象、重新调度 Pod、管理 ReplicaSets 等的请求都发生在主节点上。主节点不在生产或类生产集群中运行应用程序工作负载。

1.  在每个工作节点上，我们有 kubelet、代理和容器运行时。

1.  答案是 A. **是的**。你无法在 Kubernetes 集群上运行独立容器。Pod 是该集群中部署的原子单元。

1.  在一个 Pod 内运行的所有容器共享相同的 Linux 内核网络命名空间。因此，所有在这些容器中运行的进程可以通过 `localhost` 相互通信，这与直接在主机上运行的进程或应用程序通过 `localhost` 互相通信的方式类似。

1.  `pause` 容器的唯一作用是为在其中运行的容器保留 Pod 的命名空间。

1.  这是一个糟糕的主意，因为一个 Pod 的所有容器是共同定位的，这意味着它们都运行在同一个集群节点上。此外，如果多个容器运行在同一个 Pod 中，它们只能一起扩展或缩减。然而，应用程序的不同组件（即 `web`、`inventory` 和 `db`）通常在可扩展性或资源消耗方面有非常不同的要求。`web` 组件可能需要根据流量进行扩展或缩减，而 `db` 组件则在存储方面有特殊要求，而其他组件没有。如果我们将每个组件都运行在自己的 Pod 中，那么在这方面我们就更具灵活性。

1.  我们需要一个机制来确保在集群中运行多个实例的 Pod，并确保实际运行的 Pod 数量始终与期望数量相符，即使个别 Pod 因网络分区或集群节点故障而崩溃或消失。ReplicaSet 是为任何应用程序服务提供可扩展性和自我修复的机制。

1.  每当我们想在 Kubernetes 集群中更新应用程序服务时，而不造成服务的停机，就需要部署对象。部署对象为 ReplicaSets 添加了滚动更新和回滚功能。

1.  Kubernetes 服务对象用于使应用服务参与服务发现。它们为一组 pod 提供稳定的端点（通常由 ReplicaSet 或部署管理）。Kube 服务是定义逻辑集的抽象，并规定如何访问它们的策略。Kube 服务有四种类型：

    +   **ClusterIP**：在仅能从集群内部访问的 IP 地址上暴露服务；这是一个虚拟 IP（VIP）。

    +   **NodePort**：在每个集群节点上发布一个 30,000 到 32,767 范围内的端口。

    +   **LoadBalancer**：这种类型使用云提供商的负载均衡器（例如 AWS 上的 ELB）将应用服务暴露给外部。

    +   **ExternalName**：当你需要为集群外部服务（例如数据库）定义代理时使用。

# 第十六章

以下是本章中提出问题的一些示例答案：

1.  假设我们在注册表中有一个 Docker 镜像用于两个应用服务：Web API 和 Mongo DB，接下来我们需要做以下操作：

    +   使用 StatefulSet 为 Mongo DB 定义一个部署，我们称之为`db-deployment`。StatefulSet 应该有一个副本（复制 Mongo DB 更为复杂，超出了本书的范围）。

    +   定义一个名为`db`的 Kubernetes 服务，类型为`ClusterIP`，用于`db-deployment`。

    +   定义一个用于 Web API 的部署，我们称之为`web-deployment`。我们将这个服务扩展为三个实例。

    +   定义一个名为`api`的 Kubernetes 服务，类型为`NodePort`，用于`web-deployment`。

    +   如果我们使用机密（secrets），则需要通过 kubectl 直接在集群中定义这些机密。

    +   使用 kubectl 部署应用程序。

1.  要为应用实现第 7 层路由，理想情况下我们使用 IngressController。IngressController 是一个反向代理，例如 Nginx，它有一个 sidecar 监听 Kubernetes 服务器 API 的相关更改，并在检测到此类更改时更新反向代理的配置并重启它。然后，我们需要在集群中定义 Ingress 资源，定义路由规则，例如基于上下文的路由，如`https://example.com/pets to <service name>/<port>`，或者像`api/32001`这样的配对。一旦 Kubernetes 创建或更改了这个 Ingress 对象，IngressController 的 sidecar 就会捕捉到并更新代理的路由配置。

1.  假设这是一个集群内部的库存服务，那么我们接下来做以下操作：

    +   在部署版本 1.0 时，我们定义一个名为`inventory-deployment-blue`的部署，并为 pod 添加标签`color: blue`。

    +   我们为前面的部署部署一个类型为`ClusterIP`的 Kubernetes 服务，命名为 inventory，选择器包含`color: blue`。

    +   当我们准备好部署支付服务的新版本时，我们为服务的 2.0 版本定义一个部署，并将其命名为`inventory-deployment-green`。我们为 pod 添加一个标签`color: green`。

    +   我们现在可以对 "green" 服务进行冒烟测试，当一切正常时，我们可以更新库存服务，使得选择器包含 `color: green`。

1.  一些机密信息，应该通过 Kubernetes secrets 提供给服务，包括密码、证书、API 密钥 ID、API 密钥密文和令牌。

1.  秘密值的来源可以是文件或 base64 编码的值。

# 第十七章

以下是本章问题的一些示例答案：

1.  出于性能和安全原因，我们不能在生产系统上进行实时调试。这包括交互式调试或远程调试。然而，应用服务可能由于代码缺陷或其他与基础设施相关的问题（如网络故障或不可用的外部服务）而表现出意外行为。为了快速确定服务行为异常或失败的原因，我们需要尽可能多的日志信息。这些信息应该为我们提供线索，并引导我们找到错误的根本原因。当我们对服务进行监控时，正是为了做到这一点——我们以合理的方式生成尽可能多的信息，形式是日志条目和发布的度量。

1.  Prometheus 是一个用于收集由其他基础设施服务，最重要的是应用服务提供的功能性或非功能性度量的服务。由于 Prometheus 本身周期性地从所有配置的服务中拉取这些度量，服务本身无需担心发送数据。Prometheus 还定义了生产者应呈现度量数据的格式。

1.  要为基于 Node.js 的应用服务进行监控，我们需要执行以下四个步骤：

    1.  向项目中添加 Prometheus 适配器。Prometheus 的维护者推荐使用名为 `siimon/prom-client` 的库。

    1.  在应用启动时配置 Prometheus 客户端。这包括定义度量注册表。

    1.  暴露一个 HTTP GET 端点/度量，在该端点我们返回度量注册表中定义的度量集合。

    1.  最后，我们定义 `counter`、`gauge` 或 `histogram` 类型的自定义度量，并在我们的代码中使用它们；例如，我们每次调用某个端点时，就增加一个 `counter` 类型的度量。

1.  在生产环境中，Kubernetes 集群节点通常只包含一个最小化的操作系统，以尽可能减少攻击面并避免浪费宝贵的资源。因此，我们不能假设通常用于排查应用或进程问题的工具在各个主机上都可用。一个强大且推荐的排查方式是运行一个特殊的工具或排查容器，作为一个临时 pod 的一部分。这个容器可以作为堡垒，从中我们可以调查与有问题服务相关的网络或其他问题。一个已经成功被许多 Docker 工程师在客户现场使用的容器是 `netshoot`。

# 第十八章

以下是本章问题的一些示例答案：

1.  要在 AWS 上安装 UCP，我们按以下步骤操作：

    +   创建一个包含子网和安全组（SG）的虚拟私有云（VPC）。

    +   然后，提供一组 Linux 虚拟机，可能作为自动伸缩组（ASG）的一部分。许多 Linux 发行版是支持的，如 CentOS、RHEL 和 Ubuntu。

    +   接下来，在每个虚拟机上安装 Docker。

    +   最后，选择一台虚拟机并使用 `docker/ucp` 镜像来安装 UCP。

    +   一旦 UCP 安装完成，将其他虚拟机加入集群，作为工作节点或管理节点。

1.  以下是考虑使用托管 Kubernetes 服务的一些原因：

    +   你可能不想安装和管理一个 Kubernetes 集群，或者没有足够的资源来完成这一任务。

    +   你希望专注于为你的业务带来价值的部分，在大多数情况下，这就是应该运行在 Kubernetes 上的应用，而不是 Kubernetes 本身。

    +   你偏向于一种按需付费的成本模型，只为你需要的部分付费。

    +   你的 Kubernetes 集群节点会自动进行补丁更新。

    +   升级 Kubernetes 版本且零停机时间的操作简单直接。

1.  将容器镜像托管在云服务提供商的容器注册表（例如微软 Azure 上的 ACR）上的两个主要原因是：

    +   镜像地理位置靠近你的 Kubernetes 集群，因此延迟和传输网络成本极低。

    +   生产环境或类似生产环境的集群最好与互联网隔离，因此 Kubernetes 集群节点无法直接访问 Docker Hub。
