*第十六章*

# Docker 的下一步

你已经到了本书的最后一章，并且坚持到了最后！在本章中，我们将讨论 Moby 项目，以及你如何为 Docker 和社区做出贡献。最后，我们将简要回顾云原生计算基金会。

本章将涵盖以下主题：

+   Moby 项目

+   为 Docker 做贡献

+   云原生计算基金会

# Moby 项目

2017 年 DockerCon 大会上发布了 Moby 项目。当这个项目宣布时，我有一些问题，向同事们询问这个项目是什么，因为乍一看，Docker 似乎发布了另一个容器系统。

我的回答如下：

Moby 项目是 Docker 公司创建的开源框架，它允许 Docker 及任何愿意为该项目做出贡献的人，组装容器系统，“无需重新发明轮子”。

可以把这个框架看作是由几十个组件组成的一组构建模块，允许你自己组装定制的容器平台。Moby 项目用于创建 Docker 的开源社区版本和商业支持的 Docker Enterprise 版本。

对于任何询问类似项目的例子的人，我会解释红帽是如何处理 Red Hat Enterprise Linux 的：它结合了一个前沿版本、一个稳定的开源发布和一个企业支持版本。

可以把它看作是红帽在 Red Hat Enterprise Linux 中的做法。你有 Fedora，这是红帽操作系统开发者用于引入新软件包和功能的前沿版本开发平台，同时也会去除旧的、过时的组件。

通常，Fedora 的功能版本领先于 Red Hat Enterprise Linux 一两年，而后者是基于 Fedora 项目工作的商业长期支持发布；除了这个版本，还有社区支持版本——CentOS。

你可能会想，*为什么这个项目只在本书的最后提到？* 这个项目并不是为最终用户设计的。它是为了软件工程师、集成商和那些希望在基于容器的系统中进行实验、发明和构建的爱好者而设的——这不一定是 Docker。

在撰写本文时，由于 Docker 公司发生了变化，包括 Docker Enterprise 的出售给 Mirantis 以及 Docker 将注意力重新集中到开发者身上，关于将 Moby 代码重新归入 Docker 的讨论也在进行中。因此，在你阅读本文时，Moby 项目可能会发生变化，我不会进一步深入介绍它；相反，我建议你收藏以下页面，以便跟踪项目的发展动态：

+   该项目的官方网站是[`mobyproject.org/`](https://mobyproject.org/)

+   Moby 项目的 GitHub 页面：[`github.com/moby/`](https://github.com/moby/)

+   Moby 项目的 Twitter 账号，是一个不错的新闻来源和教程链接：[`twitter.com/moby/`](https://twitter.com/moby/)

+   关于将 Moby 移回 Docker 的讨论：[`github.com/moby/moby/issues/40222`](https://github.com/moby/moby/issues/40222) 和 [`www.theregister.com/2019/11/22/moby_docker_naming/`](https://www.theregister.com/2019/11/22/moby_docker_naming/)

# 为 Docker 做贡献

所以，您想为 Docker 做贡献吗？您是否有一个很棒的想法，想在 Docker 或其某个组件中实现？让我们为您提供所需的信息和工具。如果您不是程序员，也有其他方式可以贡献。Docker 拥有庞大的用户群，您可以通过帮助支持其他用户的服务来做出贡献。

让我们来了解一下如何做到这一点。

## 为代码做贡献

您为 Docker 做出贡献的最大方式之一是通过帮助 Moby 代码的开发，因为这是 Docker 所依赖的上游项目。

由于 Moby 是开源的，您可以将代码下载到本地机器上，开发新功能并将它们以拉取请求的形式提交回 Moby。然后，它们会定期进行审查，如果他们认为您的贡献应当成为服务的一部分，他们将批准您的拉取请求。这对于知道自己写的内容被接受是一种非常谦虚的体验。

首先，您需要了解如何为贡献做准备：对于 Docker（[`github.com/docker/`](https://github.com/docker/)）和 Moby 项目（[`github.com/moby/`](https://github.com/moby/)）来说，这几乎涵盖了所有内容。

但是我们该如何开始准备贡献呢？最好的起点是按照官方 Moby 文档中提供的指南，您可以在 [`github.com/moby/moby/blob/master/CONTRIBUTING.md`](https://github.com/moby/moby/blob/master/CONTRIBUTING.md) 找到它。

正如您可能已经猜到的，您并不需要太多来建立开发环境，因为大部分开发是在容器内进行的。例如，除了拥有一个 GitHub 账户之外，Moby 列出了以下三款软件作为最低要求：

+   Git：[`git-scm.com/`](https://git-scm.com/)

+   Make：[`www.gnu.org/software/make/`](https://www.gnu.org/software/make/)

+   Docker：如果你已经看到这里，应该不需要链接了。

您可以在 [`github.com/moby/moby/blob/master/docs/contributing/software-required.md`](https://github.com/moby/moby/blob/master/docs/contributing/software-required.md) 查找关于如何为 Mac 和 Linux 准备 Docker 开发环境的更多细节，关于 Windows 的请访问 [`github.com/moby/moby/blob/master/docs/contributing/software-req-win.md`](https://github.com/moby/moby/blob/master/docs/contributing/software-req-win.md)。

要成为一个成功的开源项目，必须有一些社区指南。我建议阅读[这个优秀的快速入门指南](https://github.com/moby/moby/blob/master/CONTRIBUTING.md#moby-community-guidelines)，以及[更详细的贡献工作流程文档](https://github.com/moby/moby/blob/master/docs/contributing/who-written-for.md)。

## 提供 Docker 支持

你还可以通过其他方式为 Docker 做出贡献，超越对 Docker 代码或功能的贡献。你可以利用你获得的知识帮助他人在支持渠道中解决问题。社区非常开放，总有人愿意提供帮助。

当我遇到问题时，发现自己在困惑时，我觉得这很有帮助。能得到帮助的同时，也能帮助他人，这是一种良好的互动。这也是一个极好的地方，可以收集供你使用的想法。你可以看到其他人在根据他们的设置提出了哪些问题，这可能激发你想到一些你可以在自己的环境中使用的想法。

你还可以关注 GitHub 上关于服务的 issues。这些可能是功能请求及 Docker 如何实现它们，也可能是服务使用过程中出现的问题。你可以帮助测试其他人遇到的问题，看看是否能够重现问题，或者找到解决他们问题的可能方案。

Docker 拥有一个非常活跃的社区，可以在[`www.docker.com/docker-community`](https://www.docker.com/docker-community)找到；在这里，你不仅能看到最新的社区新闻和活动，还能在他们的 Slack 频道中与 Docker 用户和开发者进行交流。在撰写本书时，已有超过 80 个频道，涵盖了各种主题，比如 Docker for Mac、Docker for Windows、Alpine Linux、Swarm、存储和网络等，任何时刻都有数百名活跃用户。

最后，还有 Docker 论坛，可以在[`forums.docker.com/`](https://forums.docker.com/)找到。如果你想搜索主题、问题或关键字，这里是一个很好的资源。

Docker 社区由行为准则管理，涵盖了其员工和社区整体应如何行动。该准则是开源的，并且采用创作共用署名 3.0 许可证，内容如下：

我们致力于为每个人提供一个无骚扰的体验，我们不容忍任何形式的骚扰行为。我们要求你考虑他人，保持专业和尊重，尊重所有其他参与者的行为。本守则及相关程序也适用于发生在社区活动范围外的不可接受行为，无论是在所有社区场所——线上和线下——还是在所有一对一的交流中，任何可能对社区成员的安全与福祉产生不利影响的行为。Docker, Inc（DockerCon，聚会，用户组）组织的活动或在 Docker, Inc 设施举办的活动的展览者、演讲者、赞助商、工作人员及所有其他参与者都必须遵守这些社区指南和行为守则。

多样性和包容性使 Docker 社区更强大。我们鼓励来自各种背景的参与，并希望明确表明我们的立场。

我们的目标是为每个人维持一个安全、有帮助和友好的 Docker 社区，无论其经验、性别认同与表达、性取向、残障、外貌、体型、种族、民族、年龄、宗教、国籍或其他受适用法律保护的类别如何。

完整的行为守则可以在[`github.com/docker/code-of-conduct/`](https://github.com/docker/code-of-conduct/)找到。

## 其他贡献

还有其他方式可以为 Docker 做出贡献。你可以做一些事情，比如在你的机构中推广该服务并收集兴趣。你可以通过自己组织的通信渠道开始这项沟通，无论是电子邮件分发列表、群体讨论、IT 圆桌会议还是定期安排的会议。

你也可以在组织内安排聚会，以促进成员之间的交流。这些聚会旨在不仅包括你的组织，还包括所在城市或镇的成员，从而实现更广泛的沟通和服务推广。

你可以访问[`www.docker.com/community/meetup-groups/`](https://www.docker.com/community/meetup-groups/)搜索你所在地区是否已经有聚会。

# 云原生计算基金会

我们在*第十一章*中简要讨论了云原生计算基金会，*Docker 和 Kubernetes*。云原生计算基金会（简称 CNCF）成立的目的是为那些允许你管理容器和微服务架构的项目提供一个中立的、非厂商特定的家园。

其会员包括 Docker、亚马逊 Web 服务、谷歌云、微软 Azure、红帽、甲骨文、VMWare 和 Digital Ocean 等。2020 年 6 月，Linux 基金会报告称 CNCF 有 452 个成员。这些成员不仅贡献项目，还贡献工程时间、代码和资源。

## 毕业项目

在撰写本书时，已经有十个毕业的项目，其中一些我们在之前的章节中已经讨论过。已讨论的两个项目也是基金会维护的十个项目中最著名的，分别如下：

+   **Kubernetes** ([`kubernetes.io`](https://kubernetes.io))：这是第一个捐赠给基金会的项目。正如我们之前提到的，它最初由 Google 开发，现在基金会成员和开源社区的贡献者已经超过 2300 名。

+   **Prometheus** ([`prometheus.io`](https://prometheus.io))：该项目由 SoundCloud 捐赠给基金会。正如我们在 *第十五章*，*Docker 工作流* 中所看到的，它是一个实时监控和告警系统，依赖强大的时间序列数据库引擎。

还有以下内容：

+   **Envoy** ([`www.envoyproxy.io/`](https://www.envoyproxy.io/))：最初在 Lyft 内部创建并被 Apple、Netflix 和 Google 等公司使用，Envoy 是一个高度优化的服务网格，提供负载均衡、追踪以及跨环境的数据库和网络活动的可观察性。

+   **CoreDNS** ([`coredns.io/`](https://coredns.io/))：这是一个小巧、灵活、可扩展且高度优化的 DNS 服务器，采用 Go 语言编写，并从底层设计上旨在运行在可以支持成千上万个容器的基础设施中。

+   **Containerd** ([`containerd.io/`](https://containerd.io/))：我们在 *第一章*，*Docker 概述* 中简要提到过 Containerd，作为 Docker 正在开发的开源项目之一，我们在 *第十二章*，*发现更多 Kubernetes 选项* 中也使用了它。它是一个标准的容器运行时，允许开发者在其平台或应用程序中嵌入一个可以管理 Docker 和 OCI 兼容镜像的运行时。

+   **Fluentd** ([`www.fluentd.org/`](https://www.fluentd.org/))：该工具允许你从大量来源收集日志数据，并将日志数据路由到多个日志管理、数据库、归档和告警系统中，如 Elastic Search、AWS S3、MySQL、SQL Server、Hadoop、Zabbix 和 DataDog 等。

+   **Jaeger** ([`www.jaegertracing.io/`](https://www.jaegertracing.io/))：这是一个完全分布式的追踪系统，最初由 Uber 开发，用于监控其广泛的微服务环境。现在，像 Red Hat 这样的公司也在使用它，拥有现代化的用户界面，并原生支持 OpenTracing 以及多种后端存储引擎。它被设计为与其他 CNCF 项目（如 Kubernetes 和 [Prometheus](https://vitess.io/)）集成。

+   [**Vite**](https://vitess.io/) ([`vitess.io/`](https://vitess.io/))：自 2011 年起，Vite 一直是 YouTube MySQL 数据库基础设施的核心组成部分。它是一个集群系统，通过分片水平扩展 MySQL。

+   **Helm** ([`helm.sh/`](https://helm.sh/))：为 Kubernetes 构建，Helm 是一个包管理器，允许用户将他们的 Kubernetes 应用程序打包成易于分发的格式，并迅速成为标准。

要毕业，一个项目必须完成以下要求：

+   采用了 CNCF 行为规范，该规范类似于 Docker 发布的规范。完整的行为规范可以在[`github.com/cncf/foundation/blob/master/code-of-conduct.md`](https://github.com/cncf/foundation/blob/master/code-of-conduct.md)找到。

+   获得了**Linux 基金会**（**LF**）**核心基础设施倡议**（**CII**）最佳实践徽章，证明该项目正在使用一套已建立的最佳实践进行开发——完整的标准可以在[`github.com/coreinfrastructure/best-practices-badge/blob/master/doc/criteria.md`](https://github.com/coreinfrastructure/best-practices-badge/blob/master/doc/criteria.md)找到。

+   至少获得两个组织的贡献者参与该项目。

+   通过`GOVERNANCE.md`和`OWNERS.md`文件在项目的代码库中公开定义了提交者流程和项目治理结构。

+   在`ADOPTERS.md`文件或项目网站上的徽标中公开列出项目的采用者。

+   获得**技术监督委员会**（**TOC**）的超级多数投票。你可以在[`github.com/cncf/toc`](https://github.com/cncf/toc)了解更多关于该委员会的信息。

还有一种项目状态是目前大多数项目所处的状态。

## 孵化中的项目

处于孵化阶段的项目最终应该获得毕业状态。以下项目都已经完成以下要求：

+   演示该项目至少被三个独立的最终用户使用（不是项目发起人）。

+   获得了足够数量的贡献者，包括内部和外部的。

+   展示了增长和良好的成熟度。

TOC 积极参与与项目合作，确保活动水平足够满足前述标准，因为指标可能因项目而异。

当前的项目列表如下：

+   **OpenTracing** ([`opentracing.io/`](https://opentracing.io/))：这是目前在 CNCF 框架下的两个追踪项目之一，另一个是 Jaeger。它不是一个应用程序，而是作为一组库和 API 下载并使用，让你能够将行为追踪和监控集成到基于微服务的应用程序中。

+   **gRPC** ([`grpc.io`](https://grpc.io)): 与 Kubernetes 类似，gRPC 是由 Google 捐赠给 CNCF 的。它是一个开源、可扩展且优化性能的 RPC 框架，已经在 Netflix、Cisco 和 Juniper Networks 等公司投入生产使用。

+   **CNI** ([`github.com/containernetworking`](https://github.com/containernetworking)): **CNI**（**容器网络接口**）并不是一个你可以直接下载使用的工具。它是一个网络接口标准，旨在嵌入到容器运行时中，例如 Kubernetes 和 Mesos。拥有统一的接口和 API 集合，可以通过第三方插件和扩展更一致地支持这些运行时中的高级网络功能。

+   **公证人** ([`github.com/theupdateframework/notary`](https://github.com/theupdateframework/notary)): 该项目最初由 Docker 编写，是 TUF 的实现，接下来我们将介绍 TUF。它的设计旨在为开发者提供一个加密工具，使其能够签署容器镜像，并提供一个机制来验证容器镜像和内容的来源。

+   **TUF** ([`theupdateframework.github.io`](https://theupdateframework.github.io)): **更新框架**（**TUF**）是一个标准，通过使用加密密钥，允许软件产品在安装和更新过程中自我保护。它由纽约大学工程学院开发。

+   **NATS** (https://nats.io): 这是一个为运行微服务或支持物联网设备的架构环境设计的消息传递系统。

+   **Linkerd** ([`linkerd.io`](https://linkerd.io)): 由 Twitter 构建，Linkerd 是一个服务网格，旨在扩展并处理每秒数万个安全请求。

+   **Rook** ([`rook.io`](https://rook.io)): 该项目专注于提供一个编排层，用于在 Kubernetes 上管理 Ceph、Red Hat 的分布式存储系统等。

+   **Harbor** ([`goharbor.io/`](https://goharbor.io/)): 这是一个开源镜像注册中心，专注于安全性和访问控制，内置镜像扫描、RBAC 控制和镜像签名。最初由 VMWare 开发。

+   **etcd** ([`etcd.io/`](https://etcd.io/)): 这是一个简单的分布式键值存储系统，提供 REST API，旨在低延迟操作，并使用 raft 一致性算法。[raft 一致性算法](https://www.openpolicyagent.org/)

+   [**Open Policy A**](https://www.openpolicyagent.org/)**gent** ([`www.openpolicyagent.org/`](https://www.openpolicyagent.org/)): 这是一个统一的工具集和框架，用于管理你的云原生堆栈中的策略。

+   **cri-o** ([`cri-o.io/`](https://cri-o.io/)): 这是为 Kubernetes 构建的轻量级容器运行时；它允许你运行任何符合 OCI 标准的容器。

+   **Cloudevents** ([`cloudevents.io/`](https://cloudevents.io/)): 这是一个用于以通用方式描述事件数据的规范。它还提供了 Go、JavaScript、Java、C#、Ruby 和 Python 的 SDK，使你能够轻松地将该规范引入到自己的项目中。

+   **Falco** ([`falco.org/`](https://falco.org/)): Falco 提供一个云原生的运行时安全引擎，可以在执行过程中解析来自内核的 Linux 系统调用。

+   **Dragonfly** ([`d7y.io/en-us/`](https://d7y.io/en-us/)): 这是一个基于 P2P 的开源镜像和文件分发系统。

我们在本书的各个章节中使用了一些这些项目，随着你开始解决诸如容器路由和在环境中监控应用程序等问题，其他项目也肯定会引起你的兴趣。

## CNCF 景观

CNCF 提供了一个交互式地图，展示了他们及其成员管理的所有项目，可以在[`landscape.cncf.io/`](https://landscape.cncf.io/)找到。一个关键的要点如下：

**你正在查看 1,403 张卡片，总共有 2,263,137 颗星，市值为 16.98 万亿美元，资金为 660.5 亿美元**。

虽然我相信你会同意这些都是一些非常令人印象深刻的数字，但这有什么意义呢？多亏了 CNCF 的工作，我们有了像 Kubernetes 这样的项目，它们为跨多个云基础设施提供商以及本地和裸金属服务提供了一个标准化的工具集、API 和方法——为你创建和部署自己的高可用、可扩展、性能强大的容器和微服务应用程序提供了构建模块。

# 摘要

我希望本章能给你提供一些关于容器旅程中下一步行动的思路。我发现，虽然使用这些服务很简单，但通过成为大型、友好且热情的开发者和其他用户社区的一部分，你能从中获得更多，这些社区就像你一样，围绕着各种软件和项目蓬勃发展。

这种社区感和协作精神通过云原生计算基金会的成立得到了进一步加强。这个基金会汇集了许多大型企业，这些企业直到几年前还不会考虑与那些曾被视为竞争对手的大型企业在大规模项目中公开合作。

# 评估

# *第一章*，Docker 概述

这是本章中提出的问题的一些示例答案：

1.  Docker Hub: https:[//hub.docker.com/](http://hub.docker.com/)

1.  `$ docker image pull nginx`

1.  Moby 项目

1.  Mirantis 公司

1.  `$ docker container help`

# *第二章*，构建容器镜像

这是本章中提出的问题的一些示例答案：

1.  错误；它用于向镜像添加元数据。

1.  你可以将 `CMD` 追加到 `ENTRYPOINT`，但不能反过来。

1.  正确。

1.  快照化一个故障容器，以便你能在 Docker 主机之外查看它。

1.  `EXPOSE` 指令暴露容器上的端口，但不会映射主机上的端口。

# *第三章*，存储与分发镜像

这里是本章提出的问题的示例答案：

1.  正确。

1.  这允许你在上游 Docker 镜像更新时自动更新你的 Docker 镜像。

1.  是的，它们是（如本章中的示例所示）。

1.  正确；如果你使用命令行登录，你已登录到 Docker for Mac 和 Docker for Windows。

1.  你将通过名称删除它们，而不是使用镜像 ID。

1.  端口 `5000`。

# *第四章*，管理容器

这里是本章提出的问题的示例答案：

1.  `-a` 或 `--all`。

1.  错误；应当是反过来的。

1.  当你按下 *Ctrl + C* 时，你会回到终端；然而，保持容器活动的进程依然在运行，因为我们已经从进程中分离，而不是终止它。

1.  错误；它会在指定的容器内生成一个新的进程。

1.  你可以使用`--network-alias [别名名称]`标志。

1.  运行 `docker volume inspect [卷名称]` 会给你该卷的信息。

# *第五章*，Docker Compose

这里是本章提出的问题的示例答案：

1.  YAML，或称为 YAML 不是标记语言。

1.  `restart` 标志与 `--restart` 标志相同。

1.  错误；你可以使用 Docker Compose 在运行时构建镜像。

1.  默认情况下，Docker Compose 使用存储 Docker Compose 文件的文件夹名称。

1.  你使用 `-d` 标志来启动容器的分离模式。

1.  使用 `docker-compose config` 命令可以暴露出 Docker Compose 文件中的语法错误。

1.  Docker 应用将你的 Docker Compose 文件打包成一个小的 Docker 镜像，可以通过 Docker Hub 或其他注册表分享，Docker 应用的命令行工具可以根据镜像中包含的数据渲染出有效的 Docker Compose 文件。

# *第六章*，Docker Machine，Vagrant 和 Multipass

这里是本章提出的问题的示例答案：

1.  使用了 `--driver` 标志。

1.  错误；它会给你命令。相反，你需要运行 `eval $(docker-machine env my-host)`。

1.  错误；需要先安装 Docker。

1.  Packer。

1.  Docker 守护进程配置不再被认为是最佳实践。

# *第七章*，从 Linux 迁移到 Windows 容器

这里是本章提出的问题的示例答案：

1.  你可以使用 Hyper-V 隔离来在最小化的虚拟化管理程序中运行你的容器。

1.  命令是 `docker inspect -f “{{ .NetworkSettings.Networks.nat.IPAddress }}” [CONTAINER NAME]`。

1.  错误；管理 Windows 容器时，所需运行的 Docker 命令没有任何区别。

# *第八章*，使用 Docker Swarm 进行集群管理

以下是本章中提出问题的一些示例答案：

1.  错误；独立的 Docker Swarm 已不再支持，也不被认为是最佳实践。

1.  你需要 Docker Swarm 管理节点的 IP 地址，以及用于认证工作节点的令牌。

1.  你会使用 `docker node ls`。

1.  你会添加 `--pretty` 标志。

1.  你会使用 `docker node promote [node name]`。

1.  你会运行 `docker service scale cluster=[x] [service name]`，其中 `[x]` 是你希望扩展的容器数量。

# *第九章*，Portainer — Docker 的图形用户界面

以下是本章中提出问题的一些示例答案：

1.  路径是 `/var/run/docker.sock`。

1.  端口是 `9000`。

1.  错误；应用程序有自己的定义。当运行 Docker Swarm 时，你可以使用 Docker Compose 启动堆栈。

1.  正确；所有的统计数据都是实时显示的。

# *第十章*，在公共云中运行 Docker

以下是本章中提出问题的一些示例答案：

1.  一个 Azure Web 应用

1.  一个 EC2 实例

1.  错误

1.  Knative

1.  亚马逊 ECS

# *第十一章*，Docker 和 Kubernetes

以下是本章中提出问题的一些示例答案：

1.  错误；你始终可以查看 Kubernetes 使用的镜像。

1.  `docker` 和 `kube-system` 命名空间。

1.  你会使用 `kubectl describe --namespace [NAMESPACE] [POD NAME]`。

1.  你会运行 `kubectl create -f [FILENAME OR URL]`。

1.  端口 `8001`。

1.  它曾被称为 Borg。

# *第十二章*，探索其他 Kubernetes 选项

以下是本章中提出问题的一些示例答案：

1.  错误

1.  MiniKube、Kind 和 K3d

1.  MicroK8s 和 K3s

# *第十三章*，在公共云中运行 Kubernetes

以下是本章中提出问题的一些示例答案：

1.  `kubectl create namespace sock-shop`

1.  `kubectl -n sock-shop describe services front-end-lb`

1.  `eksctl`

# *第十四章*，Docker 安全

以下是本章中提出问题的一些示例答案：

1.  你会添加 `--read-only` 标志；或者，如果你想将一个卷设置为只读，可以添加 `:ro`。

1.  在理想的世界里，每个容器只运行一个进程。

1.  通过运行 Docker Bench Security 应用程序。

1.  Docker 的套接字文件，可以在 `/var/run/docker.sock` 找到；另外，如果你的主机系统运行的是 Systemd，文件路径为 `/usr/lib/systemd`。

1.  错误；Quay 扫描的是公共和私有镜像。

# *第十五章*，Docker 工作流

这里是本章问题的一些示范答案：

1.  Nginx（web）容器提供网站服务；WordPress（WordPress）容器运行传递给 Nginx 容器的代码。

1.  `wp` 容器运行一个单一的进程，该进程一旦运行就会存在。

1.  cAdvisor 仅保留 5 分钟的指标数据。

1.  你可以使用 `docker-compose down --volumes --rmi all`。
