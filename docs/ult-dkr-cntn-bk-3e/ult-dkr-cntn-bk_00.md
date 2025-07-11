# 前言

在今天这个快节奏的世界里，开发人员面临着不断的压力，需要快速高效地构建、修改、测试和部署高度分布的应用程序。运维工程师需要一个一致的部署策略，以应对日益增长的应用程序组合，而利益相关者希望保持低成本。Docker 容器，结合 Kubernetes 等容器编排工具，为这些挑战提供了强有力的解决方案。

Docker 容器简化了构建、交付和运行高度分布式应用程序的过程。它们加强了 CI/CD 流水线，并允许公司在 Kubernetes 等单一部署平台上进行标准化。容器化应用程序更加安全，并且可以在任何能够运行容器的平台上运行，无论是在本地还是云端。借助 Docker 容器，开发人员、运维工程师和利益相关者能够实现他们的目标，并保持领先地位。

# 本书适合谁

本书旨在为任何希望了解 Docker 及其功能的人提供指导。无论你是系统管理员、运维工程师、DevOps 工程师、开发人员还是业务相关人员，本书将带领你从零开始学习 Docker 的基础。

通过清晰的解释和实际示例，你将全面了解这项技术的所有功能，最终使你能够在云中部署和运行高度分布的应用程序。如果你希望提升自己的技能并利用 Docker 的强大功能，那么本书就是为你而写的。

# 本书涵盖的内容

*第一章*，*什么是容器，为什么我应该使用它们？* 重点介绍了软件供应链及其中的摩擦。然后，它将容器作为减少这种摩擦并在其上提供企业级安全性的手段。在这一章中，我们还将探讨容器及其周围生态系统的构建方式。我们特别指出了上游开源组件（Moby）与 Docker 和其他厂商下游产品之间的区别。

*第二章*，*设置工作环境*，详细讨论了如何为开发人员、DevOps 和运维人员设置一个理想的环境，以便在使用 Docker 容器时能够高效工作。

*第三章*，*掌握容器*，教你如何启动、停止和删除容器。本章还会教你如何检查容器，获取容器中的额外元数据。此外，它还解释了如何运行额外的进程以及如何附加到已经运行的容器中的主进程。最后，本章介绍了容器的内部工作原理，包括 Linux 命名空间和组等内容。

*第四章*，*创建和管理容器镜像*，介绍了创建容器镜像的不同方法，容器镜像作为容器的模板。它介绍了镜像的内部结构以及如何构建镜像。本章还展示了如何将现有的遗留应用程序“迁移并部署”到容器中运行。

*第五章*，*数据卷和配置*，讨论了可以被容器中运行的有状态组件使用的数据卷。本章还展示了如何为容器内部运行的应用程序定义个别环境变量，以及如何使用包含完整配置设置的文件。

*第六章*，*容器中运行代码的调试*，介绍了常用的技术，这些技术使你能够在容器中运行时演化、修改、调试和测试代码。掌握这些技术后，你将能够享受无摩擦的开发过程，使容器中运行的应用程序的开发过程类似于本地开发应用程序时的体验。

*第七章*，*容器中运行应用程序的测试*，讨论了在容器中运行的应用程序和应用服务的测试。你将了解各种测试类型，并理解如何在使用容器时，最优化地实施和执行这些测试。本章解释了所有测试如何在开发者的机器上本地运行，或者作为完全自动化的 CI/CD 流水线的单独质量关卡执行。

*第八章*，*通过 Docker 技巧和窍门提高生产力*，展示了一些在容器化复杂分布式应用程序时有用的各种技巧、窍门和概念，或者在使用 Docker 自动化复杂任务时的技巧。你还将学习如何利用容器将整个开发环境运行在容器中。

*第九章*，*了解分布式应用架构*，介绍了分布式应用架构的概念，并讨论了运行分布式应用程序所需的各种模式和最佳实践。最后，它讨论了在生产环境中运行这种应用程序时需要满足的附加要求。

*第十章*，*使用单主机网络*，介绍了 Docker 容器的网络模型及其在单主机上的实现形式——桥接网络。章节还介绍了**软件定义网络**（**SDNs**）的概念，以及它们如何用于保护容器化应用程序。它还涵盖了如何打开容器端口对外公开，从而使容器化的组件能够从外部世界访问。最后，它介绍了 Traefik，一个反向代理，用于在容器之间启用复杂的 HTTP 应用层路由。

*第十一章*，*使用 Docker Compose 管理容器*，介绍了由多个服务组成的应用程序的概念，每个服务都运行在一个容器中，并解释了 Docker Compose 如何通过声明性方法帮助我们轻松构建、运行和扩展这样的应用程序。

*第十二章*，*传输日志和监控容器*，展示了如何收集容器日志并将其发送到中央位置，在那里聚合的日志可以被解析以提取有用的信息。你还将学习如何为应用程序添加监控，使其暴露出度量数据，以及如何将这些度量数据抓取并再次发送到中央位置。最后，本章教你如何将收集到的度量数据转换为图形化的仪表板，用于监控容器化的应用程序。

*第十三章*，*介绍容器编排*，详细阐述了容器编排工具的概念。它解释了为什么需要编排工具，以及它们的概念性工作原理。本章还将概述最流行的编排工具，并列出它们各自的优缺点。

*第十四章*，*介绍 Docker Swarm*，介绍了 Docker 的本地编排工具 SwarmKit。它详细阐述了 SwarmKit 用于在本地或云环境中部署和运行分布式、弹性、稳健且高可用的应用程序的所有概念和对象。

*第十五章*，*在 Docker Swarm 上部署和运行分布式应用程序*，介绍了路由网格，并演示了如何将由多个服务组成的第一个应用程序部署到 Swarm 上。

*第十六章*，*介绍 Kubernetes*，介绍了目前最流行的容器编排工具 Kubernetes。它介绍了核心的 Kubernetes 对象，这些对象用于在集群中定义和运行分布式、弹性、稳健且高可用的应用程序。最后，它介绍了 minikube，作为本地部署 Kubernetes 应用程序的一种方式，并且涵盖了 Kubernetes 与 Docker Desktop 的集成。

*第十七章*，*使用 Kubernetes 部署、更新和保护应用程序*，教你如何将应用程序部署、更新和扩展到 Kubernetes 集群。它还展示了如何使用活性和就绪探针来为应用程序服务提供支持，以便 Kubernetes 进行健康检查和可用性检查。此外，本章还解释了如何实现零停机部署，从而实现不中断的更新和回滚关键任务应用程序。最后，它介绍了 Kubernetes Secrets，作为配置服务和保护敏感数据的一种方式。

*第十八章*，*在云中运行容器化应用程序*，概述了在云中运行容器化应用程序的几种最流行的方式。讨论了微软 Azure、亚马逊 AWS 和谷歌云引擎上的完全托管服务。我们将分别在每个云平台上创建一个托管的 Kubernetes 集群，并将一个简单的分布式应用程序部署到这些集群中。我们还将比较这三种服务的设置和使用的简便性。

*第十九章*，*监控和故障排除生产环境中的应用程序*，涵盖了用于监控和仪表化单个服务或整个分布式应用程序的不同技术，应用程序运行在 Kubernetes 集群中。你将了解基于关键指标的告警概念。本章还展示了如何在不更改集群或集群节点的情况下，故障排除生产环境中运行的应用程序服务。

# 为了最大化本书的学习效果

| **本书中涵盖的软件/硬件** | **操作系统要求** |
| --- | --- |
| Docker v23.x | Windows、macOS 或 Linux |
| Docker Desktop |  |
| Kubernetes |  |
| Docker SwarmKit |  |

**如果你正在使用本书的数字版，建议你自己输入代码或从本书的 GitHub 仓库访问代码（下节会提供链接）。这样可以帮助你避免与复制和粘贴代码相关的潜在错误。**

# 下载示例代码文件

你可以从 GitHub 下载本书的示例代码文件，网址为 https://github.com/PacktPublishing/The-Ultimate-Docker-Container-Book/。如果代码有更新，它将会在 GitHub 仓库中更新。

我们还有其他来自我们丰富目录的代码包，涵盖了书籍和视频内容，网址：https://github.com/PacktPublishing/。快来看看吧！

# 使用的约定

本书中使用了多种文本约定。

`文中的代码`：表示文本中的代码词汇、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟网址、用户输入和 Twitter 用户名。例如：“一旦 Chocolatey 安装完成，可以通过 `choco` `--version` 命令来测试它。”

代码块设置如下：

```
while :do
    curl -s http://jservice.io/api/random | jq '.[0].question'
    sleep 5
done
```

当我们希望您特别关注代码块中的某一部分时，相关的行或项目会以粗体显示：

```
…secrets: demo-secret: "<<demo-secret-value>>" 
other-secret: "<<other-secret-value>>" 
yet-another-secret: "<<yet-another-secret-value>>" 
…
```

任何命令行输入或输出都如下所示：

```
$ docker version $ docker container run hello-world
```

**粗体**：表示一个新术语、一个重要的词或屏幕上显示的词。例如，菜单或对话框中的词会以**粗体**显示。举个例子：“从菜单中选择**仪表盘**。”

提示或重要说明

显示如下。

# 联系我们

我们始终欢迎读者的反馈。

**一般反馈**：如果您对本书的任何部分有疑问，请通过 email us at customercare@packtpub.com，并在邮件主题中提及书名。

**勘误表**：虽然我们已经尽力确保内容的准确性，但错误还是会发生。如果您在本书中发现错误，我们将非常感激您报告给我们。请访问 www.packtpub.com/support/errata 填写表格。

**盗版**：如果您在互联网上发现我们作品的任何非法副本，我们将非常感激您提供其所在地址或网站名称。请通过 copyright@packt.com 联系我们，并附上相关链接。

**如果您有兴趣成为作者**：如果您在某个领域拥有专业知识，并且有兴趣撰写或为书籍做贡献，请访问 authors.packtpub.com。

# 分享您的想法

一旦您阅读了*《终极 Docker 容器书》*，我们非常希望听到您的想法！请点击这里直接访问亚马逊的书籍评价页面并分享您的反馈。

您的评价对我们以及技术社区非常重要，它将帮助我们确保提供优质的内容。

# 下载本书的免费 PDF 版

感谢您购买本书！

您喜欢在旅途中阅读，但无法随身携带纸质书籍吗？您的电子书购买是否与您选择的设备不兼容？

不用担心，现在每本 Packt 图书，您都可以免费获得该书的 DRM-free PDF 版本。

在任何地方、任何设备上阅读。搜索、复制并将您最喜欢的技术书籍中的代码直接粘贴到您的应用程序中。

福利不仅仅是这样，您还可以独享折扣、新闻通讯以及每天将免费精彩内容送到您的邮箱。

按照以下简单步骤，获取这些福利：

1.  扫描二维码或访问以下链接

![](img/B19199_QR_Free_PDF.jpg)

https://packt.link/free-ebook/9781804613986

1.  提交您的购买证明

1.  就这样！我们会直接通过电子邮件将您的免费 PDF 和其他福利发送给您。

# 第一部分：简介

*第一部分*的目标是向您介绍容器的概念，并解释为什么它们在软件行业中如此有用。您还将了解如何准备您的工作环境，以便使用 Docker。

本部分包括以下章节：

+   *第一章*，*什么是容器以及为什么我应该使用它们？*

+   *第二章*, *设置工作环境*
