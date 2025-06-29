模拟考试题目和最终备注

本章提供了一些最终备注和模拟题。我们将简要总结考试规范，考试如何呈现给你，以及哪些主题比其他主题更为相关。

本书准备的模拟题与考试中将遇到的题目类似。请仔细阅读它们，因为其中一些是多项选择题。多项选择题将在考试中出现，你应该知道如何作答。

通过这些问题，你将对考试的格式、你将遇到的题目类型以及哪些主题更为相关有一个清晰的了解。

# Docker 认证工程师考试详情

撰写本书时，考试基于 Docker Enterprise 平台。请参考 Docker 官网，[`success.docker.com/certification`](https://success.docker.com/certification)，以获取最新的信息。

Docker 认证工程师考试将验证你在 Docker Enterprise 平台上的专业技能，通常需要至少 6 到 12 个月的使用经验。本书通过实验帮助你理解平台的概念和用法，教会你这些必备技能。

你可以在线支付并参加此考试，但它仅提供英文版本。尽管 Docker 网站显示结果会立即交付，但有时结果可能需要 24 到 48 小时才能交付。

考试包含 55 道题目，本书所涵盖的主题具有不同的权重。我们建议你使用 Docker Enterprise 平台的 30 天免费试用，因为会有很多关于 Docker Enterprise 组件和管理的考试问题。还建议你熟练掌握 Docker 的命令行操作和选项，包括如何获取和筛选有关 Docker 资源的信息。这些是撰写本书时的主题权重：

+   **编排**：25%

+   **镜像创建、管理和注册**：20%

+   **安装与配置**：15%

+   **网络**：15%

+   **安全性**：15%

+   **存储与卷**：10%

这可以帮助你了解每个主题的重要性和问题分布。在接下来的章节中，我们将提供一些模拟题，帮助你为考试做准备。这些题目模拟真实考试。注意，许多题目有多个正确答案，你应该选择所有正确答案。所有这些问题的答案可以在本书的*评估*章节找到。

# 模拟考试题目

1.  如何限制提供给容器的 CPU 数量？

a) 使用`--cap-add CPU`。

b) 使用`--cpuset-cpus`。

c) 使用`--cpus`。

d) 无法指定 CPU 的数量；我们必须使用`--cpu-shares`并定义 CPU 切片。

1.  如何限制容器可用的内存量？

a) 无法限制容器可用的内存量。

b) 使用`--cap-drop MEM`。

c) 使用`--memory`。

d) 使用`--memory-reservation`。

1.  应该导出哪些环境变量才能开始使用受信环境与 Docker 客户端？

a) `export DOCKER_TRUSTED_ENVIRONMENT=1`

b) `export DOCKER_CONTENT_TRUST=1`

c) `export DOCKER_TRUST=1`

d) `export DOCKER_TRUSTED=1`

1.  如何增加运行一个实例的服务的副本数量（标记所有正确答案）？

a) 这对于全局服务不可行。

b) 通过使用 `docker service update --replicas <NUMBER_OF_REPLICAS> <SERVICE>` 更新副本数量。

c) 可以通过使用 `docker service scale <SERVICE>=<NUMBER_OF_REPLICAS>` 增加副本数量。

d) 我们可以使用 `docker service scale up` 创建一个新的副本。

1.  如果我们指定 `node.role!=worker` 限制，全局服务在节点上运行多少副本？

a) 工作节点和管理节点将运行一个副本。

b) 只有工作节点将运行一个副本。

c) 只有管理节点将运行一个副本。

d) 没有节点将运行任何副本。

1.  如何停止当前执行三个副本的 web 服务器服务的所有副本？

a) 使用 `docker service stop webserver`。

b) 使用 `docker rm service webserver`。

c) 使用 `docker service update --replicas 0 webserver`。

d) 以上答案均不正确。

1.  如果我们通过 `-P` 在端口 `8080` 上发布服务，哪些节点将暴露端口 `8080`？

a) 没有节点将暴露端口 `8080` 上的服务。

b) 所有节点将发布端口 `8080`。

c) 我们应该使用特权容器来暴露端口 `8080`。

d) 我们必须使用 `--network=host` 来发布 30000 以下的端口。

1.  我们应该遵循什么步骤来移除集群中的领导节点？

a) 通过执行 `docker node update --availability=drain <LEADER_NODE>` 确保所有任务在其他节点上运行。

b) 通过在节点上执行 `docker swarm leave`，将节点从集群中移除作为领导者。

c) 将主节点降级为工作节点，然后在该节点上执行 `docker swarm leave`。

d) 一旦节点离开集群，我们可以通过在任何可用的管理节点上执行 `docker node rm <OLD_LEADER_NODE>` 来完全移除该节点。

1.  我们在哪里指定 DevOps 用户只能使用管理员组签名的镜像来运行容器？

a) 在 **Universal Control Plane** (**UCP**) 的 RBAC 中，我们允许 DevOps 用户运行他们的镜像。

b) 在 DTR 的镜像仓库中，我们为 DevOps 用户添加镜像拉取权限。

c) 镜像访问应该在 **Docker Trusted Registry** (**DTR**) 上配置，且 DevOps 用户至少应能够读取该仓库。在 UCP 上，我们仅允许集群中管理员组签名的镜像，并为 DevOps 用户的私有仓库至少添加调度器访问权限。

d) 这在 Docker 企业版中不可行。

1.  访问使用自签名证书的安全仓库中存储的镜像需要执行什么步骤？

a) 我们可以在 Docker 引擎的 `daemon.json` 文件中将我们的仓库配置为“不安全”。

b) 我们应该禁用内容信任以允许从不安全的仓库拉取镜像。

c) 最佳选项是信任自签名证书。我们将把 DTR 创建的 **证书颁发机构**（**CA**）添加到我们系统的受信任 CA 列表中。

d) 我们无法使用自签名证书，因此我们总是需要企业签名证书。

1.  *用户 A* 执行了 `docker service scale --replicas 5 webserver`，而 *用户 B* 执行了 `docker service update --replicas 3 webserver`。在两者执行之后，将会运行多少个副本？

a) 这些命令都无法执行。

b) `webserver` 服务将运行三个副本。

c) `webserver` 服务将运行五个副本。

d) `webserver` 服务将运行八个副本。

1.  哪一行命令会创建名为 `DATA` 的卷？

a) `docker volume create --driver local DATA`。

b) `docker create --volume DATA`。

c) 卷必须在容器执行时创建。

d) 选项中没有有效的选项。

1.  如何确保在运行容器时使用软限制来保证最低的内存可用？

a) 我们无法确保为容器提供最低的内存。

b) 使用 `--memory`。

c) 这必须在您的操作系统上进行配置。

d) 我们使用 `--memory-reservation`。

1.  关于 Swarm 网络，哪个说法是正确的？

a) 所有覆盖网络默认都是加密的。

b) 控制平面节点使用双向 TLS 加密来保护流量。

c) 内部 DNS 可以被外部消费并暴露其服务。

d) 以上所有选项都正确。

1.  哪个概念将请求路由到已部署服务上运行的容器？

a) `ingress` 覆盖网络用于默认情况下通过轮询终端路由请求到不同服务的后端。

b) `docker_gwbridge` 网络用于与不同主机上的容器进行通信。

c) `ingress` 覆盖网络用于默认情况下通过服务的虚拟 IP 路由请求到不同服务的后端。

d) 我们必须使用主机网络来路由请求到容器。

1.  关于签名镜像，以下哪些句子是正确的？

a) 镜像签名确保了镜像的所有权。

b) 可以使用 `docker trust sign <IMAGE>` 来签名镜像。

c) 如果我们在每个命令上设置 Docker 客户端使用 Docker 内容信任，则所有镜像将被签名。

d) 镜像签名基于以下密钥：所有者密钥、仓库密钥、快照密钥和时间戳。

1.  如果我们有一个集群，其中节点本地有 `myapplication` 镜像，但镜像哈希不同，会发生什么？

a) 如果我们没有指定镜像哈希值，每个节点将使用自己的镜像运行任务。

b) 为了确保所有节点运行相同的镜像版本，我们需要为远程注册表使用 `--with-registry-auth`。

c) 我们将在使用受信内容的节点上使用签名镜像和 Docker 引擎。

d) 无法确保节点使用正确的镜像版本。

1.  以下哪个关于全局服务的说法是正确的？

a) 全局服务将只在每个节点上运行定义服务的一个副本。

b) 全局服务不会基于弹性提供高可用性。

c) 使节点排空不会移除全局服务。

d) 以上所有句子都是正确的。

1.  以下哪些句子关于复制服务是正确的？

a) 我们需要在服务创建时指定实例的数量，因为之后无法更改。

b) 复制服务是默认的服务模式，如果未指定副本数量，它将运行一个实例。

c) 通过将副本数设置为`0`，可以停止复制服务。

d) 以上所有选项都正确。

1.  创建 Swarm 备份需要哪些步骤？

a) 停止任何管理节点上的 Docker Engine，以确保文件保持静态。

b) 复制`/var/lib/docker/swarm`目录的内容以备份 Swarm。

c) Raft 日志应该与普通文件分开进行备份。

d) 以上所有步骤都是必需的。

1.  你学到了什么关于 Swarm 网络的知识？

a) 覆盖网络使用 UDP VXLAN 隧道进行部署。

b) 默认情况下，所有服务副本都可以通过内部负载均衡器平等访问。

c) 内部 DNS 将允许所有在同一覆盖网络上运行的服务相互访问。

d) 以上所有选项都正确。

1.  我们尝试创建一个五个副本的服务，但它无法工作。我们无法进入协调阶段，因为我们收到以下错误：`1/1: no suitable node (scheduling constraints not satisfied on 5 nodes)`。可能出了什么问题？

a) 服务的镜像不存在。

b) 我们使用的是私有镜像，但没有提供身份验证凭据。

c) 我们使用了约束条件来将服务的任务部署到特定节点，但没有一个节点具有所需的标签。

d) 以上所有选项都不正确。

1.  以下哪项关于锁定 Docker Swarm 集群的说法是正确的？

a) 控制、管理和数据平面是安全的。

b) 解锁`/var/lib/docker/swarm`数据需要密码短语。

c) 执行`systemctl restart docker`将需要锁定密码短语。

d) 如果节点重启，Docker Engine 不会自动重启，因此我们会失去 Docker Swarm 的仲裁权。

1.  以下哪项关于复制服务的说法是正确的？

a) 它们将在每个节点上运行一个实例。

b) 只有复制服务才能使用滚动更新功能进行升级。

c) 它们可以通过 Docker 服务更新命令停止：`--replicas 0 <SERVICENAME>`。

d) 我们将使用 Go 模板以提供唯一资源，例如容器内部的卷或主机名，确保所有副本使用各自的资源。

1.  以下哪些方法可以在 Docker Enterprise 上发布应用程序？

a) 我们可以使用 Interlock。

b) 我们可以使用 Ingress Controller。

c) 我们将发布每个应用程序容器。

d) 我们可以使用`host`模式将应用程序发布为好像它们直接在主机级别运行一样。

1.  以下哪项关于 Kubernetes 与 Docker Enterprise 的集成是正确的？

a) Docker Enterprise 开箱即用提供 Kubernetes。

b) 我们必须选择在集群节点中使用哪个协调器，因为每次只能使用一个。

c) 我们可以以混合模式运行主机，以支持 Kubernetes 和 Docker Swarm 负载，尽管不推荐在生产环境中使用。

d) 我们可以使用常见的 Kubernetes 安装命令来升级 Kubernetes 组件。

1.  `docker image import` 和 `docker image load` 在将镜像上传到 Docker 主机时有什么区别？

a) 这些命令没有区别。

b) 两者导入相同的镜像内容。

c) `docker image import` 只会获取包含二进制文件、库和进程配置的镜像层，但不包含如何启动进程、使用哪些卷、应使用哪些端口等元信息。

d) 我们只能使用 `docker image import` 来创建新镜像。

1.  如何让 `docker build` 避免使用缓存的镜像层？

a) Docker 将始终使用缓存的信息。无法避免使用镜像缓存。

b) 默认情况下，镜像缓存是禁用的，因此我们需要使用 `--use-caching` 来确保启用缓存，因为它会加速构建过程。

c) 为了避免镜像缓存，我们可以使用 `--no-cache`。这样，构建将不会使用任何先前保存的层。

d) 以上所有句子都是错误的。

1.  如何从一个仓库中下载所有的镜像？

a) 不可能。我们需要列出所有镜像及其标签，并逐一下载。

b) 每次执行 `docker image pull` 时，我们都会下载所有的镜像及其层，无论是否打算使用它们。

c) 我们可以使用 `docker image pull --all-tags` 来检索所有与仓库相关的镜像。

d) 以上所有句子都不正确。

1.  如何根据特定镜像筛选运行中的容器？

a) 没有这个选项。我们使用 Linux `grep` 命令过滤容器的特定基础镜像。

b) 我们将使用 `ancestor` 键列出所有使用特定镜像的运行容器。

c) 我们将使用 `image` 键列出所有使用特定镜像的运行容器。

d) 以上所有句子都不正确。

1.  如何将本地构建的镜像推送到远程注册表？

a) 我们需要知道注册表的 **完全限定域名** (**FQDN**) 或其 IP 地址。

b) 我们将镜像标记为注册表的 FQDN 或 IP、用户名或组名，以及镜像存储的仓库。

c) 如果注册表使用 TLS/SSL 证书，我们可以将其 CA 加载到系统中以信任它们，或者我们可以使用 `insecure-registries` 键来配置它。

d) 以上所有句子都正确。

1.  哪个选项会将已创建的 `DATA` 卷绑定到容器内部的 `/data` 目录？

a) `-v DATA:/data`

b) `--mount type=volume,source=DATA,target=/data`

c) `--mount DATA:/data`

d) `--volume type=volume,source=DATA,target=/data`

1.  如何将一个 Web 服务器容器暴露在主机的 `80` 端口（`nginx:alpine` 镜像暴露了 `80` 端口）？

a) `docker container run --cap-add NET_ADMIN -p 80:80 -d nginx:alpine`

b) `docker container run --net=host -d nginx:alpine`

c) `docker container run -P nginx:alpine`

d) `docker container run -d -P 80:80 nginx:alpine`

1.  哪些密钥在签署镜像时需要密码短语来解锁？

a) 时间戳

b) 目标

c) 快照

d) 根用户

1.  什么是 Docker 捆绑包，里面包含哪些内容？

a) Docker 包含客户端二进制文件和管理员配置的捆绑包。

b) 所有用户都有自己的 Docker 捆绑包，并且其中包括用户所需的所有环境文件。

c) Docker 捆绑包只包含环境脚本，我们将向管理员请求证书。

d) 用户的 Docker 捆绑包包括使用 CaaS 平台所需的所有环境文件和证书。

1.  如果我们必须部署一个具有七个管理节点和分布式高可用性的集群，最佳的节点分布是什么？

a) 一个数据中心有四个管理节点，另一个数据中心有三个管理节点。

b) 一个数据中心有两个管理节点，第二个数据中心有两个管理节点，第三个数据中心有三个管理节点。

c) 所有管理节点应该位于同一个数据中心。

d) 我们无法通过 7 个节点来管理分布式可用性；至少需要 9 个节点。

1.  哪个概念负责管理 Docker Swarm 服务的外部到内部负载均衡？

a) 路由器网状结构

b) 入口控制器

c) `nodePort`

d) `clusterIP`

1.  `COPY` 和 `ADD` Dockerfile 指令之间有什么区别？

a) `COPY` 以只读模式添加文件。

b) `COPY` 可以用于从外部服务下载文件。

c) `ADD` 可以与打包和压缩的文件一起使用，并会在层的根文件系统中解压缩。

d) `ADD` 和 `COPY` 完全相同，但 `ADD` 是更新的指令。

1.  如何使用相同的 `docker-compose.yaml` 文件部署两个应用程序？

a) 我们无法使用相同的 `docker-compose.yaml` 文件部署两个应用程序。

b) Docker Compose 可以使用项目来部署两个应用程序，确保应用程序在不同的卷和端口上运行。

c) 我们可以使用环境变量来固定资源，以避免资源使用冲突。

d) 避免应用组件冲突的唯一选项是将应用程序部署到不同的集群中。

1.  部署 DTR 需要什么？

a) Docker Hub 上的 Docker Enterprise 许可证和适当的仓库 URL

b) Docker Enterprise Engine 和 Docker UCP

c) Docker Engine、DTR 许可证和 Docker Content Trust。

d) 所有前述选项

1.  如何在 Docker Engine 上启用调试？

a) 通过使用 `-D` 参数启动 Docker 守护进程来启用调试。

b) 通过在 `config.json` 文件中将 `debug` 键设置为 `true` 来启用调试。

c) 通过在 `daemon.json` 中启用实验功能。

d) 无前述选项

1.  如何只列出由 `alpine:3.10` 镜像创建的容器？

a) `docker container ls image=alpine:3.10` b) `docker ps --format ancestor=alpine:3.10` c) `docker container ls --filter ancestor=alpine:3.10` d) `docker container ls --filter image=alpine:3.10`

1.  以下关于特权容器哪个是正确的？

a) 资源限制将被避免（CPU、内存和磁盘 I/O）。

b) 它们总是以 root 用户身份运行容器。

c) 这些容器将以所有可用的能力运行。

d) 它们使用主机的内核命名空间运行。

1.  以下关于 Swarm 加入令牌哪个是正确的？

a) 一旦创建，我们必须将其存储在安全的地方，因为它们是不可恢复的。

b) 我们可以通过 `docker swarm join-token recreate` 生成新的令牌，以便为新节点获取新的值，如果我们丢失了它们。

c) 我们可以通过 `docker swarm join-token` 在需要时恢复它们。

d) 一旦重新生成，加入令牌将在所有节点上自动更新。

1.  用于验证 DTR 和 UCP 节点健康状态的端点有哪些？

a) DTR 提供 `/_ping`、`/nginx_status` 和 `/api/v0/meta/cluster_status`。

b) DTR 和 UCP 提供 `/status`。

c) DTR 和 UCP 提供 `/_ping`。

d) DTR 提供 `/status`，UCP 提供 `/_ping`。

1.  哪个命令可以帮助我们查看并恢复因“悬挂镜像”和死容器而丢失的空间？

a) `docker system rm` b) `docker system prune` c) `docker image rm --filter="dangling"` d) `docker container rm -a`

1.  哪个基本组合创建的命令行最终将在容器内运行？

a) `ENTRYPOINT` 会设置要启动的脚本或二进制文件，而 `CMD` 仅在 `ENTRYPOINT` 未定义时使用。

b) `CMD` 总是覆盖 `ENTRYPOINT` 定义。

c) 使用 `ENTRYPOINT` 来定义要启动的脚本或二进制文件，并将 `RUN` 作为参数。

d) 仅当 `ENTRYPOINT` 使用 `exec` 格式配置时，`CMD` 才会将参数添加到定义的 `ENTRYPOINT` 中。

1.  以下关于密钥哪个是正确的？

a) 它们只会在管理节点上可用，因此带有密钥的工作负载必须运行在这些节点上。

b) 它们是短暂的，并部署在内存文件系统上。

c) 即使是管理员，它们也是加密的，因此不能从控制平面恢复。

d) 如果我们需要更改密钥，我们需要创建一个新的密钥并使用这个新的密钥更新服务。

1.  如何确保在生产环境中部署特定镜像？

a) 通过使用镜像的哈希值来部署容器。

b) 签名镜像将确保其标签和来源。

c) 指定正确的标签足以确保其内容。

d) 通过使用 `docker image history` 来查看生成镜像时使用的命令。

1.  以下关于容器隔离的句子哪些是正确的？

a) 主机的硬件资源，如内存和 CPU，是通过 cgroups 授权的。

b) 为了确保容器的限制，我们需要使用操作系统安全模块。

c) 只有 root 用户被允许部署无限资源的容器。

d) 特权容器将绕过定义的进程能力和执行用户。

1.  我们如何确保镜像内容的不可变性？

a) 通过使用签名镜像。

b) 在 DTR 中定义不可变标签。

c) 通过使用镜像扫描。

d) 镜像不能是不可变的。

1.  以下关于叠加网络的哪些句子是正确的？

a) DTR 部署了一个叠加网络`dtr-ol`，用于路由集群的内部通信。

b) 当没有任务连接到管理节点时，叠加定义的网络只存在于管理节点上。

c) `interlock-extension`连接到服务定义的网络，将请求路由到适当的后端。

d) Docker Swarm 叠加网络是加密的，并使用 VXLAN 部署。

1.  `HEALTHCHECK --start-period=15s CMD curl --fail https://localhost:8080 | exit 1`这行在 Dockerfile 中的作用是什么？

a) 它将每 15 秒执行一次定义的`curl`命令，如果连续三次失败，它将标记容器为不健康。

b) 它将在第一次执行时等待 15 秒，然后 Docker 引擎将每 30 秒运行一次定义的`curl`命令，如果连续三次失败，它将标记容器为不健康。

c) 这一行没有任何作用；健康检查必须为每个容器单独配置。

d) Docker 引擎将每 15 秒运行此探测程序，如果失败，它将重新启动容器。

1.  以下关于 Docker Engine 访问的哪些说法是正确的？

a) 默认情况下，只有 Docker 套接字的所有者才被允许在独立主机上运行容器。

b) 我们可以允许用户运行容器，允许他们访问 Docker Engine 的 Unix 套接字或 API 的 TCP 端口（默认启用）。

c) 任何被允许登录到主机的人也被允许运行容器。

d) 只有 root 用户被允许在主机上运行特权容器。

1.  我们如何修改已部署服务的发布端口？

a) 这是不可能的；我们必须删除服务并重新创建它。

b) 只有在服务使用主机网络（`--net=host`）运行时，我们才能更改端口。

c) 我们使用`docker service update --publish-add <NEW_PORT> --publish-rm <OLD_PORT>`。

d) 以上选项都不正确。

1.  如何确保 Web 服务器 Docker 服务在所有集群节点上运行一个 NGINX 实例？

a) 通过使用`docker service create --type=global --instances=1 --name=webserver --image=nginx:alpine`。

b) 通过使用`docker service create --mode=global --name=webserver nginx:alpine`。c) UCP 可以确保管理员在所有节点上运行任何服务，并勾选`allow run on manager nodes`。

d) `docker service create --name=webserver --image=nginx:alpine`足以在所有节点上执行一个实例。

1.  我们如何设置一个名为`myregistry/myorganization/baseimages`的仓库，让内部用户可以访问，并且镜像由 DevOps 组用户拥有和管理？

a) 我们需要创建一个名为`myorganization`的组织。

b) 我们将创建 `myregistry/myorganization/baseimages` 作为公共仓库。

c) 我们将 DevOps 团队用户配置为 `myregistry/myorganization/baseimages` 仓库的管理员。

d) `myregistry/myorganization/baseimages` 仓库将作为私有仓库为 `myorganization` 用户创建。

1.  我们如何查看名为 `webserver` 的容器发布的端口？

a) 使用 `docker container ls --filter name=webserver`。

b) 使用 `docker container port webserver`。

c) 使用 `docker container inspect webserver --format="{{ .NetworkSettings.Ports }}"`。

d) 以上所有答案都是正确的。

1.  哪个概念负责管理 Kubernetes 的内部负载均衡？

a) Router Mesh

b) Ingress 控制器

c) Interlock

d) `clusterIP`

1.  哪些资源用于将 Pod 与 Kubernetes 定义的卷链接？

a) `persistentVolume`

b) `persistentVolumeClaim`

c) `storageClass`

d) `persistentDataVolume`

1.  部署服务时需要哪些标签？

**a) `com.docker.lb.port`**

b) `com.docker.interlock.port`

c) `com.docker.interlock.hosts`

d) `com.docker.lb.backend`

1.  我们如何在 Kubernetes 中将服务暴露到外部？

a) 使用 Interlock

b) 使用 Ingress 控制器

c) 使用 `nodePort` 服务

d) 使用 `clusterIP` 资源

1.  我们如何知道系统中容器和卷使用了多少空间？

a) 通过使用 `docker system prune`

b) 通过使用 `docker system df`

c) 使用 `docker container df`

d) 通过使用 `docker volume df`

1.  DBA 团队在 UCP 中应该设置什么角色，以允许他们创建自己的卷？

a) `Scheduler`。

b) `仅查看`.

c) 只有 UCP 管理员可以创建卷和其他集群资源。

d) `Restricted Control` 足以在他们的私有集合中创建卷。

1.  应该使用哪个标志来配置所有可用的 FQDN 用于 UCP？

a) `--san`

b) `--external-name`

c) `--external-url`

d) `--ucp-url`

1.  用户无法推送镜像到我们的 DTR 内部仓库。我们应该验证什么？

a) Docker 登录访问。

b) DTR CA 证书应该被信任。

c) 我们应该验证镜像的仓库是否存在。

d) 显示镜像存在漏洞，且 DTR 的镜像扫描拒绝了用户的镜像。

1.  DTR 部署了哪些内部网络？

a) `docker_gwbridge`

b) `dtr-ol`

c) `ingress`

d) `dtr-internal`

1.  哪些 Kubernetes 资源提供应用程序的弹性？

a) `Deployment`

b) `ReplicaSet`

c) `Pod`

d) `Replicated`

1.  `docker swarm --force-new-cluster` 命令的作用是什么？

a) 它用于在故障情况下恢复集群。它将所有管理节点设置为工作节点，只保留一个管理节点。

b) 此命令将销毁集群，用于移除整个集群。

c) 应该使用 `--force-new-cluster` 停止部署在工作节点上的所有服务。

d) 应用程序的服务不会受到此命令的影响。此命令仅影响控制平面。

1.  关于 Docker Swarm 和 Kubernetes 网络的哪些说法是正确的？

a) 默认情况下，Ingress 覆盖网络将被加密。

b) 双向 TLS 通信确保 Docker Swarm 中控制平面的安全。

c) Kubernetes 网络通过`networkPolicy`资源默认实现隔离。

d) Kubernetes 使用证书来确保用户访问和内部控制平面的安全性。

对于一些模拟问题，可能有多个正确答案。

# 总结

这是我们成为 Docker 认证助理的旅程中的最后一章。我们回顾了前几章所学习的所有内容，并通过一些模拟考试问题进行了复习。如前所述，这些问题中的一些是旧考试中出现的真实问题。

成为 Docker 认证助理的旅程并不容易。我们从头开始学习本书，理解为什么在谈到微服务架构时，容器变得如此受欢迎。接着，我们描述了如何使用 Docker 引擎执行容器，结合各种隔离策略以确保容器内进程之间的安全性。由于容器是基于镜像的，我们学习了如何使用 Docker 工具来构建和维护这些镜像。我们了解了不同的 Docker 对象，并学习了如何使用不同的网络方法、存储卷或安全策略来部署应用程序。我们学习了如何部署微服务应用，其中每个组件都在容器中运行。我们学习了在独立环境和集群分布式环境中运行所有组件的区别。编排器在分布式环境中是关键。它们保持应用程序的健康，并帮助我们更快地开发微服务，提供无中断服务更新应用程序组件的功能。我们学习了 Docker Swarm 和 Kubernetes，因为它们都是 Docker 企业平台的一部分。最后，我们介绍了 Docker 的企业级 CaaS 平台——Docker Enterprise。我们了解了它的所有组件和功能，以及它们如何帮助我们部署应用程序并提高其安全性。

这是本书所涵盖所有主题的简要总结。如果你按照此流程进行学习，你已经准备好参加 Docker 认证助理考试了。本书还应作为命令参考，尽管我们知道技术变化非常迅速。现在你已准备好参加考试，深呼吸并在[`prod.examity.com/docker`](https://prod.examity.com/docker)预约考试。祝你好运！
