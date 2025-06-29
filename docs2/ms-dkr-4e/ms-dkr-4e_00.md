# 序言

在现代应用架构和部署方面，Docker 是一个颠覆性的工具。它已经成长为推动创新的关键力量，不仅在系统管理领域，而且在 Web 开发等领域产生了深远影响。但如何确保你跟得上 Docker 推动的创新呢？如何确保你充分利用它的潜力？

本书将向你展示如何使用 Docker，它不仅展示了如何更有效地使用 Docker，还帮助你重新思考和重新构想 Docker 所能实现的可能性。

你将学习基本的主题，如构建、管理和存储镜像，以及最佳实践，然后再深入探讨 Docker 安全性。你将了解如何以新颖创新的方式扩展和集成 Docker。Docker Compose、Docker Swarm 和 Kubernetes 将帮助你以高效的方式管理容器。

本书结束时，你将全面而详细地理解 Docker 的功能，并了解它如何无缝融入你的本地工作流程、高可用的公共云平台以及其他工具中。

# 本书适合谁阅读

如果你是 IT 专业人士，并且认识到 Docker 在从系统管理到 Web 开发各个领域创新中的重要性，但不确定如何充分利用它，本书适合你。

# 本书的内容

*第一章**, Docker 概述*，讨论了 Docker 的起源以及它对开发人员、运维人员和企业的意义。

*第二章**, 构建容器镜像*，介绍了你可以用来构建自己容器镜像的各种方法。

*第三章**, 存储和分发镜像*，介绍了在我们了解如何构建镜像之后，如何共享和分发镜像。

*第四章**, 管理容器*，深入探讨了如何管理容器。

*第五章**, Docker Compose*，介绍了 Docker Compose——一个允许我们共享包含多个容器的应用程序的工具。

*第六章**, Docker Machine、Vagrant 和 Multipass*，介绍了 Docker Machine 和其他工具，它们使你能够在各种平台上启动和管理 Docker 主机。

*第七章**, 从 Linux 到 Windows 容器*，解释了传统上容器是基于 Linux 的工具。通过与 Docker 合作，微软现在引入了 Windows 容器。在这一章中，我们将探讨这两种类型容器之间的差异。

*第八章**, 使用 Docker Swarm 集群*, 本章讨论了到目前为止我们一直针对单个 Docker 主机的情况。Docker Swarm 是 Docker 提供的一种集群技术，允许你在多个主机上运行容器。

*第九章**, Portainer – 一个 Docker 的图形界面*, 解释了我们与 Docker 的大部分交互都是通过命令行进行的。在这里，我们将介绍 Portainer，这是一个允许你通过 Web 界面管理 Docker 资源的工具。

*第十章**, 在公共云中运行 Docker*, 本章我们将探讨如何在公共云服务中运行容器的不同方式。

*第十一章**, Docker 和 Kubernetes*, 本章介绍了 Kubernetes。像 Docker Swarm 一样，你可以使用 Kubernetes 创建和管理集群，运行基于容器的应用程序。

*第十二章**, 探索其他 Kubernetes 选项*, 本章在本地使用 Docker 运行 Kubernetes 后，探讨了在本地机器上使用 Kubernetes 的其他选项。

*第十三章**, 在公共云中运行 Kubernetes*, 本章探讨了来自“四大”云服务提供商的各种 Kubernetes 服务：Azure、Google Cloud、Amazon Web Services 和 DigitalOcean。

*第十四章**, Docker 安全性*, 探讨 Docker 安全性。本章将涵盖从 Docker 主机到如何启动镜像、镜像的来源以及镜像内容的各个方面。

*第十五章**, Docker 工作流*, 本章开始将所有部分整合起来，以便你能够在生产环境中开始使用 Docker，并且能够自如地操作。

*第十六章**, Docker 下一步*, 本章不仅探讨了你如何为 Docker 作出贡献，还讨论了为支持基于容器的应用程序和部署而兴起的更大生态系统。

# 为了充分利用本书

为了充分利用本书，你需要一台能够运行 Docker 的机器。该机器应至少具备 8 GB 的内存和 30 GB 的可用硬盘空间，且搭载 Intel i3 或更高版本的处理器，运行以下操作系统之一：

+   macOS High Sierra 或更高版本

+   Windows 10 专业版

+   Ubuntu 18.04 或更高版本

此外，你需要访问以下一个或多个公共云服务提供商：DigitalOcean、Amazon Web Services、Azure 和 Google Cloud。

**如果你使用的是本书的数字版，建议你亲自输入代码或通过 GitHub 仓库访问代码（链接将在下一节提供）。这样可以避免因复制和粘贴代码而可能导致的错误。**

# 下载示例代码文件

您可以从您的账户下载本书的示例代码文件，网址为 [www.packt.com](http://www.packt.com)。如果您是在其他地方购买的此书，可以访问 [www.packtpub.com/support](http://www.packtpub.com/support) 注册，以便直接将文件发送至您的邮箱。

您可以按照以下步骤下载代码文件：

1.  登录或注册 [www.packt.com](http://www.packt.com)。

1.  选择**支持**标签。

1.  点击**代码下载**。

1.  在**搜索**框中输入书名并按照屏幕上的指示操作。

文件下载完成后，请确保使用最新版本的工具解压文件：

+   Windows 版 WinRAR/7-Zip

+   Mac 版 Zipeg/iZip/UnRarX

+   Linux 版 7-Zip/PeaZip

本书的代码包也托管在 GitHub 上，网址为 [`github.com/PacktPublishing/Mastering-Docker-Fourth-Edition`](https://github.com/PacktPublishing/Mastering-Docker-Fourth-Edition)。如果代码有更新，将会在现有的 GitHub 仓库中更新。

我们还提供了其他代码包，来自我们丰富的书籍和视频目录，网址为 [`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)。快去看看吧！

# 代码示例

本书的《代码示例》视频可以在 https://bit.ly/35aQnry 观看。

# 下载彩色图片

我们还提供了一份包含本书中截图/图表的彩色图片的 PDF 文件，您可以在此下载：[`www.packtpub.com/sites/default/files/downloads/9781839213519_ColorImages.pdf`](http://www.packtpub.com/sites/default/files/downloads/9781839213519_ColorImages.pdf)。

# 使用的约定

本书中使用了多种文本约定。

`文本中的代码`：表示文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 用户名。示例：‘您可以使用以下 `docker inspect` 命令查看容器的标签。’

代码块如下所示：

```
ENTRYPOINT ['nginx']
CMD ['-g', 'daemon off;']
```

当我们希望您注意代码块的某一部分时，相关行或项目会以粗体显示：

```
[default]
exten => s,1,Dial(Zap/1|30)
exten => s,2,Voicemail(u100)
exten => s,102,Voicemail(b100)
exten => i,1,Voicemail(s0)
```

任何命令行输入或输出如下所示：

```
$ curl -sSL https://get.docker.com/ | sh
$ sudo systemctl start docker
```

`Yes` 将启动 Docker 安装程序，显示以下提示。

提示或重要说明

显示效果如下。

# 联系我们

我们非常欢迎读者的反馈。

`customercare@packtpub.com`。

**勘误**：尽管我们已经尽力确保内容的准确性，但错误仍然会发生。如果您在本书中发现错误，欢迎向我们报告。请访问 [www.packtpub.com/support/errata](http://www.packtpub.com/support/errata)，选择您的书籍，点击勘误提交表单链接并输入详细信息。

`copyright@packt.com` 并附有相关资料链接。

**如果您有兴趣成为作者**：如果您在某个领域有专长，并且有兴趣撰写或为书籍做贡献，请访问[authors.packtpub.com](http://authors.packtpub.com)。

# 评价

请留下您的评价。当您阅读并使用本书后，为什么不在您购买该书的网站上留下评价呢？潜在的读者可以看到并参考您的公正意见来做出购买决策，我们在 Packt 也能了解您对我们产品的看法，而我们的作者也可以看到您对他们书籍的反馈。谢谢！

欲了解更多 Packt 信息，请访问[packt.com](http://packt.com)。

# **第一部分**：开始使用 Docker

在本节中，您将学习如何安装、配置并使用 Docker 在本地机器上启动简单和复杂的容器化应用程序。

本节包含以下章节：

*第一章*，*Docker 概述*

*第二章*，*构建容器镜像*

*第三章*，*存储与分发镜像*

*第四章*，*容器管理*

*第五章*，*使用 Docker Compose 启动多个容器*

*第六章*，*使用 Docker Machine、Vagrant 和 Multipass*
