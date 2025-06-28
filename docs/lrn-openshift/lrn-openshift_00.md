# 前言

OpenShift 是一个应用管理平台，利用 Docker 作为隔离的运行时来运行应用程序，并使用 Kubernetes 进行容器编排。它首次发布于 2011 年 5 月 4 日，深受 Google 工程师开发的 Borg 启发，Borg 是一个用于管理数十万个容器的容器编排解决方案。2014 年 9 月，OpenShift 进行了重新设计，Docker 和 Kubernetes 成为其主要构建模块，从那时起，它经历了大量的改进，并拥有了日益壮大的用户和开发者社区。在撰写本文时，OpenShift 的最新版本是 3.9，发布于 2018 年 3 月 28 日，版本 3.10 正在开发中。3.9 版本是 3.7 版本之后的下一个版本，因此，从技术角度看，它也包含了 3.8 版本的预期更改，并代表了其生命周期中的一个重要步骤。

依赖于 Docker，OpenShift 将容器化的真正优势带给了企业，使他们能够快速响应客户日益增长的需求，并通过支持高可用性和多数据中心部署，保持良好的声誉。从商业角度来看，OpenShift 在五年内将投资相关的成本降低了 531%，年均效益为 129 万美元，回报周期为 8 个月——更多详情请参考 [`cdn2.hubspot.net/hubfs/4305976/s3-files/idc-business-value-of-openshift-snapshot.pdf`](https://cdn2.hubspot.net/hubfs/4305976/s3-files/idc-business-value-of-openshift-snapshot.pdf)。

开发者会发现，OpenShift 的自助服务门户易于使用，提供了快速访问所有功能和部署策略的途径，支持未修改的源代码，以及 Docker 镜像和 Dockerfile，允许开发者集中精力进行开发，而无需管理环境。OpenShift 通过依赖 Jenkins 管道和与 SCM 的集成，能够自动化管理发布的各个方面。

在操作方面，OpenShift 提供了自动故障转移和恢复功能，以及高可用性和可扩展性，这意味着运维团队可以将时间用于更高层次的任务。它还可以与由其他厂商（而非 Red Hat）开发的各种 SDN 技术集成。而且，依赖于这些知名技术，使得学习曲线较为平缓。

从安全角度来看，OpenShift 可以与企业身份管理解决方案集成，用于用户管理和角色分配。它能够将应用暴露给企业安全基础设施，以实现细粒度的访问控制和审计，保护应用使用的敏感数据，并管理不同应用之间的访问权限。

本书中的示例基于 OpenShift Origin 3.9 演示，但由于技术上的等效性，它们同样适用于 Red Hat OpenShift 容器平台^(TM)。本书以模块化的方式编写，因此，如果你已经熟悉某些主题，可以随时跳到另一个章节。

# 本书读者群体

本书是为刚接触 OpenShift 的专业人士编写的，但也涉及一些高级主题，如 CI/CD 流水线、高可用性和多数据中心架构。读者不需要具备 Docker、Kubernetes 或 OpenShift 的背景知识，但对基本概念的了解会有所帮助。本书不涉及如何使用 Linux，因此至少需要一年以上的 Linux 使用经验。本书的主要目标并不是理论知识，而是实践经验，因此我们采用了实践导向的方法，在可能的情况下使用虚拟实验室。本书从介绍容器的概念和优势开始，帮助新手迅速掌握基本知识，随后逐步深入，带领读者了解 Kubernetes 和 OpenShift 的基本和高级概念。最后，本书提供了一个关于高可用性多数据中心架构的参考。在我们开始撰写本书之前，我们意识到关于如何在多个数据中心部署 OpenShift 以实现高可用性和容错性的信息非常少。正是由于这一点，我们决定汇聚我们的经验，共同编写这本书。

# 本书内容概述

第一章，*容器与 Docker 概述*，讨论了容器以及如何使用 Docker 构建镜像和运行容器。

第二章，*Kubernetes 概述*，解释了 Kubernetes 如何协调 Docker 容器，并介绍了如何使用其 CLI。

第三章，*CRI-O 概述*，介绍了 CRI-O 作为容器运行时接口，并解释了它与 Kubernetes 的集成。

第四章，*OpenShift 概述*，解释了 OpenShift 作为 PaaS 的角色，并涵盖了其提供的不同版本。

第五章，*构建 OpenShift 实验室*，展示了如何通过多种方法在 OpenShift 上设置你自己的虚拟实验室。

第六章，*OpenShift 安装*，通过使用 Ansible 进行 OpenShift 的高级安装，提供了实际操作经验。

第七章，*管理持久存储*，介绍了 OpenShift 如何为应用程序提供持久存储。

第八章，*核心**OpenShift 概念*，带你了解 OpenShift 背后最重要的概念和资源。

第九章，*高级 OpenShift 概念*，探讨了 OpenShift 的资源，并解释了如何进一步管理它们。

第十章，*OpenShift 中的安全性*，描述了 OpenShift 如何在多个层次上处理安全性。

第十一章，*管理 OpenShift 网络*，探讨了 OpenShift 中虚拟网络的每种网络配置的使用案例。

第十二章，*在 OpenShift 中部署简单应用*，展示了如何在 OpenShift 中部署一个单容器应用。

第十三章，*使用模板部署多层应用*，带你了解如何通过模板部署复杂应用。

第十四章，*从 Dockerfile 构建应用镜像*，解释了如何使用 OpenShift 从 Dockerfile 构建镜像。

第十五章，*从源代码构建 PHP 应用*，解释了如何在 OpenShift 中实现源代码到镜像的构建策略。

第十六章，*从源代码构建多层应用*，展示了如何在 OpenShift 上部署一个多层 PHP 应用。

第十七章，*OpenShift 中的 CI/CD 管道*，介绍了如何使用 Jenkins 和 Jenkinsfile 在 OpenShift 上实现 CI/CD。

第十八章，*OpenShift 高可用架构概述*，展示了如何为你的 OpenShift 集群的各个层级带来高可用性。

第十九章，*OpenShift 高可用设计（单数据中心和多数据中心）*，解释了构建地理分布式 OpenShift 集群所需的条件。

第二十章，*OpenShift 高可用网络设计*，探讨了构建 HA OpenShift 解决方案所需的网络设备和协议。

第二十一章，*OpenShift 3.9 的新特性*，为你提供 OpenShift 3.9 的最新功能的洞察，并解释了为什么你可能想要使用它。

# 要充分利用本书

本书假设您具备 Linux 和开源的实际经验，能够熟练使用命令行界面（CLI），熟悉如 nano 或 vi/vim 等文本编辑器，并了解如何使用 SSH 访问正在运行的机器。由于 OpenShift 只能安装在基于 RHEL 的 Linux 发行版上，具有 RHEL/CentOS 7 的经验会更为合适，而不是基于 Debian 的变种。了解云技术和容器将是加分项，但不是必需的。

为确保顺利的体验，我们建议使用具有足够 RAM 的笔记本或台式电脑，因为这是 OpenShift 最关键的资源。您可以在 GitHub 仓库中找到“软件硬件清单”部分，其中列出了您的学习环境的所有要求。使用低于 8GB RAM 的系统可能会导致在安装 OpenShift 时出现偶尔的失败和整体不稳定，虽然这会提升您的故障排除技能，但也会让人分心。

另一个重要的方面涉及到您环境中的 DNS。一些网络服务提供商，如 Cox（[`www.cox.com`](https://www.cox.com)），会将所有不存在的域名（那些导致上游 DNS 返回 NXDOMAIN 响应的域名）的请求重定向到一个包含合作伙伴搜索结果的自定义网页。通常，这不是问题，但在安装 OpenShift 时，这会导致安装失败。发生这种情况是因为您虚拟机和由 OpenShift 管理的容器的本地 DNS 查找设置包括几个域名，这些域名按照顺序进行联系，直到返回 NXDOMAIN 为止，下一域名只有在前一个返回 NXDOMAIN 后才会被尝试。因此，当您的服务提供商拦截这些请求时，它可能会返回自己的 IP 地址，这会导致 OpenShift 安装程序尝试通过该 IP 检查某个服务的健康状况。正如预期的那样，请求不会得到回应，安装将失败。对于 Cox 来说，这一功能被称为*增强错误结果*，因此我们建议您在账户中选择退出此功能。

# 下载示例代码文件

您可以从您的账户中下载本书的示例代码文件，网址为[www.packtpub.com](http://www.packtpub.com)。如果您是从其他地方购买本书的，您可以访问[www.packtpub.com/support](http://www.packtpub.com/support)，注册后可以将文件直接通过邮件发送给您。

您可以通过以下步骤下载代码文件：

1.  在[www.packtpub.com](http://www.packtpub.com/support)登录或注册。

1.  选择“支持”标签。

1.  点击“代码下载与勘误”。

1.  在搜索框中输入书名并按照屏幕上的指示操作。

文件下载完成后，请确保使用最新版本的以下工具来解压或提取文件夹：

+   Windows 版的 WinRAR/7-Zip

+   Mac 版的 Zipeg/iZip/UnRarX

+   Linux 版的 7-Zip/PeaZip

本书的代码包也托管在 GitHub 上，网址为 [`github.com/PacktPublishing/Learn-OpenShift`](https://github.com/PacktPublishing/Learn-OpenShift)。如果代码有更新，它将在现有的 GitHub 仓库中进行更新。

我们还提供了来自我们丰富书籍和视频目录的其他代码包，网址为 **[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)**。快来查看吧！

# 下载彩色图像

我们还提供了一份包含本书截图/图表的彩色图像 PDF 文件。你可以在此下载：[`www.packtpub.com/sites/default/files/downloads/LearnOpenShift_ColorImages.pdf`](https://www.packtpub.com/sites/default/files/downloads/LearnOpenShift_ColorImages.pdf)。

# 使用的规范

本书中使用了许多文本规范。

`CodeInText`：表示文本中的代码词汇、数据库表名、文件夹名称、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 用户名。例如：“假设模板存储在本地，文件名为 `wordpress.yaml`。”

一段代码的格式如下所示：

```
...
node('nodejs') {
  stage('build') {
    openshiftBuild(buildConfig: 'nodejs-mongodb-example', showBuildLogs: 'true')
  }
```

当我们希望特别强调代码块中的某一部分时，相关的行或项目将用粗体显示：

```
openshiftBuild(buildConfig: 'nodejs-mongodb-example', showBuildLogs: 'true')
}
stage('approval') {
 input "Approve moving to deploy stage?"
}
stage('deploy') {
  openshiftDeploy(deploymentConfig: 'nodejs-mongodb-example')
}
```

任何命令行输入或输出的格式如下所示：

```
$ vagrant ssh
```

**粗体**：表示一个新术语、重要词汇或在屏幕上看到的词汇。例如，菜单或对话框中的词汇将以这种形式出现在文本中。以下是一个示例：“当你点击‘登录’按钮后，页面会显示如下。”

警告或重要提醒以这种形式出现。

小贴士和技巧以这种形式出现。

# 联系我们

我们始终欢迎读者的反馈。

**一般反馈**：请发送电子邮件至 `feedback@packtpub.com`，并在邮件主题中注明书名。如果您对本书的任何方面有疑问，请通过 `questions@packtpub.com` 与我们联系。

**勘误表**：虽然我们已经尽力确保内容的准确性，但难免会出现错误。如果您在本书中发现错误，请帮助我们报告。请访问 [www.packtpub.com/submit-errata](http://www.packtpub.com/submit-errata)，选择您的书籍，点击勘误表提交链接，并输入详细信息。

**盗版**：如果你在互联网上发现我们作品的任何非法复制品，感谢你提供具体的地址或网站名称。请通过电子邮件 `copyright@packtpub.com` 联系我们，并附上相关链接。

**如果你有兴趣成为作者**：如果你在某个领域有专业知识，并且有意撰写或参与书籍的编写，请访问 [authors.packtpub.com](http://authors.packtpub.com/)。

# 评审

请留下评论。在阅读并使用本书后，为什么不在您购买该书的网站上留下评论呢？潜在读者可以看到并参考您的公正意见，从而做出购买决策，我们在 Packt 能了解您对我们产品的看法，作者也能看到您对他们书籍的反馈。谢谢！

有关 Packt 的更多信息，请访问 [packtpub.com](https://www.packtpub.com/)。
