前言

微服务和容器改变了开发人员创建新应用程序的方式。微服务架构使我们能够将应用程序解耦成多个组件，而今天我们有工具可以提供这些组件之间有序且无缝的交互。此外，容器也改变了应用程序的部署工件。我们从二进制文件转变为容器镜像。这种新的开发工作流帮助开发人员更快、更安全地构建应用程序，并确保最终产品能够按预期工作，无论在什么环境下，都无需做大量修改。作为容器部署的应用程序将遵循通用的代码版本控制规则，这有助于我们跟踪组件发布和行为。

本书将介绍微服务和容器，并帮助我们学习这些技术的关键概念。我们将学习容器的工作原理，了解在不同场景下如何实现网络，并探索 Docker Swarm 和 Kubernetes 编排策略及环境。我们还将涵盖所有 Docker 企业版组件和功能，帮助实现生产环境中的容器即服务平台。本书涵盖的所有主题，以及样题和详细答案，将帮助你掌握通过官方 Docker 认证助理考试所需的知识。

# 本书适合人群

本书面向那些希望了解容器技术并准备参加 Docker 认证助理考试的人。书中也为 Docker 企业产品编写，作为 Kubernetes 术语和功能的介绍。

需要具备良好的 Linux 和 Windows 使用知识，一些网络技能将帮助你理解容器网络和使用负载均衡器及代理来提供功能齐全的容器即服务环境。

本书中的实验室集中于 Linux 主机，因为大多数当前的 Docker 企业版组件都部署在 Linux 操作系统上。Windows 主机可以作为 Docker Swarm 和 Kubernetes 集群的一部分，但控制平面使用 Linux 主机进行部署。

# 本书涵盖内容

第一章，*使用 Docker 的现代基础设施和应用程序*，介绍了微服务架构和容器作为现代基础设施的完美匹配，并涵盖了 Docker 引擎的概念。

第二章，*构建 Docker 镜像*，介绍了 Docker 镜像构建过程、命令行工具以及创建良好安全镜像的最佳实践。

第三章，*运行 Docker 容器*，展示了 Docker 如何帮助我们在系统中运行容器，并解释了这些进程如何与 Docker 引擎主机隔离。

第四章，*容器持久性与网络*，解释了如何管理容器生命周期外的数据，以及容器如何与内部和外部资源交互。

第五章，*部署多容器应用程序*，解释了如何部署基于容器的应用程序组件。我们将学习如何使用基础设施即代码文件来管理应用程序的组件。

第六章，*Docker 内容信任简介*，展示了如何在基于容器的环境中提高安全性，确保镜像所有权、不可变性和来源。

第七章，*编排简介*，在深入 Docker Swarm 和 Kubernetes 作为编排工具之前，回顾了编排概念。

第八章，*使用 Docker Swarm 进行编排*，讲解了 Docker Swarm 的特性和实现，解释了如何使用这个编排工具来实现应用程序。

第九章，*使用 Kubernetes 进行编排*，介绍了基本的 Kubernetes 概念，并将其与 Docker Swarm 进行比较，以帮助您为不同的应用或基础设施实施最佳解决方案。

第十章，*Docker Enterprise 平台简介*，介绍了 Docker Enterprise 组件，并解释了 Docker 如何创建一个生产就绪的容器即服务平台。

第十一章，*通用控制平面*，解释了 Docker Enterprise 的控制平面组件。我们将学习如何在生产环境中实现通用控制平面，以及如何管理 Docker Enterprise 平台。

第十二章，*在 Docker Enterprise 中发布应用程序*，回顾了发布应用程序的不同方法，并展示了如何使用 Interlock 和 Ingress Controller 来确保我们的 Docker Swarm 和 Kubernetes 平台的安全。

第十三章，*使用 DTR 实现企业级注册表*，解释了 Docker Enterprise 如何提供一个生产就绪的注册表，用于管理和存储 Docker 镜像。

第十四章，*总结重要概念*，总结了前几章中学到的最重要概念。本章将帮助我们为 Docker Certified Associate 考试做准备。

第十五章，*模拟考试题目与最终备注*，包含了一些模拟的 Docker Certified Associate 考试题目，并解释了考试流程的基础知识。

# 要充分利用本书

为了跟随本书的实验和示例，建议你在计算机上安装 Docker Engine。本书为你提供了一套虚拟环境，使你能够在不修改计算机的情况下运行所有实验。还有许多实验需要部署集群，涉及多个节点。这些实验将部署虚拟机，因此你无需安装大量节点，尽管你可以在自己的基础设施上部署所有实验。

提供的虚拟环境要求在你的计算机上安装 Vagrant（[`www.vagrantup.com/`](https://www.vagrantup.com/)）和 VirtualBox（[`www.virtualbox.org/`](https://www.virtualbox.org/)）。Docker 镜像和软件将从互联网下载，因此也需要互联网连接。下表显示了运行本书所有实验所需的计算机资源。完成每个部分或章节的所有实验后，你可以通过销毁环境来释放资源。

| **本书中涵盖的软件/硬件** | **章节** | **运行虚拟环境的操作系统要求** |
| --- | --- | --- |
| Docker 独立平台（Docker Engine） | 1 到 7 | 2 vCPU，4 GB RAM 和 10 GB 磁盘空间。 |
| Docker Swarm 集群平台 | 8 | 4 vCPU，8 GB RAM 和 50 GB 磁盘空间。 |
| Kubernetes 集群平台 | 9 | 4 vCPU，8 GB RAM 和 50 GB 磁盘空间。 |
| Docker Enterprise 平台 | 11、12 和 13 | 8 vCPU，16 GB RAM 和 100 GB 磁盘空间。 |

第 1 到第六章的实验需要一个节点。最低要求 2 vCPU 和 4 GB RAM。第八章和第九章的实验将分别部署 4 个和 3 个虚拟节点，需要更多的本地资源。在这些情况下，你的计算机至少需要 4 vCPU 和 8 GB RAM。Docker 企业实验要求更多资源，因为平台每个虚拟节点的 CPU 和内存要求较大。这些实验将顺利运行，要求至少 8 vCPU 和 16 GB RAM，因为 Vagrant 环境将部署 4 个虚拟节点，每个节点 4 GB RAM。

就磁盘空间而言，你的计算机至少需要有 100 GB 的可用空间来支持最大的环境。

最低要求的 Vagrant 版本为 2.2.8，最低要求的 VirtualBox 版本为 6.0.0。本书中的实验可以在 macOS、Windows 10 和 Linux 上执行。实验在编写本书期间已经在 Ubuntu Linux 18.04 LTS 和 Windows 10 Pro 操作系统上进行过测试。

所有实验都可以在 Docker Swarm、Kubernetes 和 Docker Enterprise 上执行，尽管建议使用虚拟环境来执行所有实验步骤，包括安装过程。

**如果你正在使用本书的数字版本，我们建议你自己输入代码，或者通过 GitHub 仓库访问代码（链接将在下一节提供）。这样做可以帮助你避免因复制粘贴代码而可能出现的错误。**

在参加考试前，确保你理解并能够回答第十五章中的所有问题，*模拟考试问题和最终备注*。本章中的问题与当前 Docker 认证助理考试中的问题非常相似。

## 下载示例代码文件

你可以从你的账户在[www.packt.com](http://www.packt.com)下载本书的示例代码文件。如果你是从其他地方购买的这本书，可以访问[www.packtpub.com/support](https://www.packtpub.com/support)并注册，以便将文件直接通过电子邮件发送给你。

你可以按照以下步骤下载代码文件：

1.  登录或注册到[www.packt.com](http://www.packt.com)。

1.  选择“支持”选项卡。

1.  点击代码下载。

1.  在搜索框中输入书名，并按照屏幕上的指示操作。

下载文件后，请确保使用最新版本的解压缩工具解压或提取文件夹：

+   Windows 下的 WinRAR/7-Zip

+   Mac 下的 Zipeg/iZip/UnRarX

+   Linux 下的 7-Zip/PeaZip

本书的代码包也托管在 GitHub 上，网址为[`github.com/PacktPublishing/Docker-Certified-Associate-DCA-Exam-Guide`](https://github.com/PacktPublishing/Docker-Certified-Associate-DCA-Exam-Guide)。如果代码有更新，将会在现有的 GitHub 仓库中更新。

我们还提供了来自丰富图书和视频目录的其他代码包，网址为**[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)**。快来看看吧！

## 实战中的代码

本书的实战代码视频可以在[`bit.ly/34FSiEp`](https://bit.ly/34FSiEp)观看。

## 下载彩色图像

我们还提供了一份 PDF 文件，包含本书中使用的屏幕截图/图表的彩色图像。你可以在这里下载：[`www.packtpub.com/sites/default/files/downloads/9781839211898_ColorImages.pdf`](http://www.packtpub.com/sites/default/files/downloads/9781839211898_ColorImages.pdf)[.](http://www.packtpub.com/sites/default/files/downloads/9781839211898_ColorImages.pdf)

## 使用的约定

本书中使用了许多文本约定。

`CodeInText`：表示文本中的代码字、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟网址、用户输入和 Twitter 用户名。以下是一个示例：“我们可以配置所需的共享存储来执行`reconfigure`操作。”

代码块设置如下：

```
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: test-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
```

当我们希望引起你对代码块中特定部分的注意时，相关行或项目会设置为粗体：

```
services:
  colors:
    image: codegazers/colors:1.16
    deploy:
      replicas: 3
```

任何命令行输入或输出将如下所示：

```
$ sudo mount -t nfs 10.10.10.11:/data /mnt
$ sudo cp -pR /var/lib/docker/volumes/dtr-registry-c8a9ec361fde/_data/* /mnt/
```

**粗体**：表示新术语、重要词汇或你在屏幕上看到的词汇。例如，菜单或对话框中的词汇会像这样出现在文本中。以下是一个示例：“截图显示了垃圾回收配置页面。”

警告或重要提示将以如下方式显示。

提示和技巧将以如下方式显示。

# 联系我们

我们始终欢迎读者的反馈。

**一般反馈**：如果您对本书的任何方面有疑问，请在邮件主题中提及书名，并将您的问题发送至`customercare@packtpub.com`。

**勘误**：尽管我们已尽力确保内容的准确性，但错误偶尔会发生。如果您发现本书中的错误，我们将感激您能报告给我们。请访问[www.packtpub.com/support/errata](https://www.packtpub.com/support/errata)，选择您的书籍，点击勘误提交表单链接，并输入相关细节。

**盗版**：如果您在互联网上发现任何我们作品的非法复制版，我们将非常感激您提供相关位置地址或网站名称。请通过`copyright@packt.com`与我们联系，并提供该资料的链接。

**如果您有兴趣成为作者**：如果您在某个领域拥有专业知识，并且有兴趣撰写或参与编写书籍，请访问[authors.packtpub.com](http://authors.packtpub.com/)。

## 评价

请留下评论。在您阅读并使用完本书后，不妨在您购买该书的网站上留下评价。潜在读者可以通过您的公正意见做出购买决策，我们 Packt 也能了解您对我们产品的看法，我们的作者也可以看到您对他们书籍的反馈。谢谢！

想了解更多关于 Packt 的信息，请访问[packt.com](http://www.packt.com/)。
