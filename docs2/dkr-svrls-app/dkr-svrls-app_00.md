# 前言

容器技术今天已经非常成熟。Docker 作为帮助普及容器的软件包，现在已经被成千上万的开发人员作为日常 DevOps 工具使用。我可以说，Docker 容器引擎现在已经变成了一款无聊的软件。对于一个基础设施级的软件包而言，“无聊”意味着高质量和稳定性。我每天都在使用它，而我知道你们也将它作为工具链的一部分。但我们不再对 Docker 的新版本发布感到兴奋。就像我们对 Linux 内核发布的感受一样。带着这样的感觉，我认为容器的黄金时代已经在不久前结束。

Docker 的崛起发生在 2013 年。它的“文艺复兴”时期是在 2014 到 2016 年之间。2016 年，Docker Swarm 和 Kubernetes 之间的许多编排引擎竞争达到了巅峰。Swarm2K 项目曾是我一生中难得的经历。Docker 后来在 2017 年宣布也将支持 Kubernetes。这场竞赛也就此结束。

几天前，即 2018 年 3 月，就在本书即将出版之际，Docker 的创始人 Solomon Hykes 离开了 Docker Inc.。Docker 这家公司一直在缓慢而强烈地从初创公司向企业业务转型。这对我们意味着什么？企业意味着稳定，而初创公司意味着冒险。让我们迈向新的冒险——容器之后的*无服务器时代*。

本书讨论的主题是无服务器。它是容器和微服务之后的自然进化，方式各异。首先，Docker 容器成为了函数的部署单元，成为*函数即服务*（FaaS）架构中的一个基本工作单元。其次，微服务架构正在逐步演变成 FaaS 架构。FaaS 实际上可以部署在本地或云端。当整个 FaaS 堆栈由云服务提供商管理时，它就变成了完全的无服务器架构。

但是，在这之间会有一些东西。那就是*混合无服务器 FaaS 架构*。这种架构是我希望读者在本书中发现并享受的核心思想。它是我们在成本、自行管理服务器以及服务器控制程度之间取得平衡的一个点。

本书详细介绍了三个主要的 Docker FaaS 平台，分别是*OpenFaas*、*OpenWhisk* 和*Fn Project*。这些项目都处于早期阶段，并且正在积极成熟。因此，对于读者和我来说，这是一个很好的机会，可以一起学习并迎接这一新的浪潮。让我们一起努力。

# 本书适合的人群

如果你是一个开发人员、Docker 工程师、DevOps 工程师，或者任何对在无服务器环境中使用 Docker 感兴趣的相关人员，那么本书适合你。

如果你是本科生或研究生，本书同样适合你，用来加强你在*无服务器和云计算*领域的知识。

# 本书内容

第一章，*无服务器与 Docker*，介绍了无服务器和 Docker。我们将在本章中找到它们之间的关系。我们还将学习通过研究多个 FaaS 平台架构总结出的常见架构。到本章结束时，我们将学会如何在所有三个 FaaS 平台——OpenFaaS、The Fn Project 和 OpenWhisk 上实现“Hello World”。

第二章，*Docker 与 Swarm 集群*，回顾了容器技术、命名空间和 cgroups。然后，我们将介绍 Docker，包括如何安装它，如何使用其基本命令，并理解它的构建、分发和运行工作流。进一步地，我们将回顾 Docker 内置的编排引擎——Docker Swarm。我们将学习如何设置集群并了解 Docker Swarm 的内部工作原理。接着，我们将学习如何设置 Docker 网络、将其附加到容器，并了解如何在 Docker Swarm 中扩展服务。

第三章，*无服务器框架*，讨论了无服务器框架，包括 AWS Lambda、Google Cloud Functions、Azure Functions 和 IBM Cloud Functions 等平台。我们将在本章结束时介绍一个与 FaaS 平台无关的框架——无服务器框架。

第四章，*Docker 上的 OpenFaaS*，解释了如何使用 OpenFaaS。我们将探索它的架构和组件。接着，我们将学习如何使用其提供的工具和模板来准备、构建和部署函数，如何在 Swarm 之上准备其集群，如何使用其用户界面，以及 OpenFaaS 如何利用 Docker 多阶段构建。我们还将讨论如何使用 Prometheus 来监控 FaaS 平台。

第五章，*Fn 项目*，探索了另一个 FaaS 平台。类似于[第四章](https://cdp.packtpub.com/docker_for_serverless_applications/wp-admin/post.php?post=25&action=edit#post_51)，*Docker 上的 OpenFaaS*，我们将从其架构和组件开始，然后通过一组 CLI 命令来构建、打包和部署函数到 Fn 平台。本章后续部分，我们将学习如何使用其内置 UI 来监控该平台。此外，我们还将使用一个熟悉的工具来帮助分析其日志。

第六章，*Docker 上的 OpenWhisk*，讨论了 OpenWhisk，这是本书中的第三个也是最后一个 FaaS 平台。我们将了解它的概念和架构。

第七章，*操作 FaaS 集群*，讨论了使用 Docker Swarm 准备和操作生产级 FaaS 集群的几种技术。我们将讨论如何用另一种易于使用的容器网络插件替代整个网络层。我们还将展示如何实现新的路由网格机制，以避免当前入口实现中的漏洞。此外，我们将讨论一些高级话题，如 *分布式追踪* 及其实现方法。我们甚至会涵盖通过使用竞价实例来降低成本的概念，并展示如何在这个动态基础设施上实现 Swarm。

第八章，*将它们汇总起来*，解释了如何实现一个异构的 FaaS 系统，结合所有三个 FaaS 平台在一个强大的产品级 Swarm 集群中无缝运行。我们将展示一个基于移动端的银行转账用例，同时包括一个遗留包装器、一个移动后端 WebHook，以及使用 FaaS 的流数据处理。这里的附加亮点是，我们还为用例添加了区块链，展示它们的互操作性。

第九章，*无服务器的未来*，通过先进的概念和研究原型实现来总结本书，这些内容超越了当前的无服务器和 FaaS 技术。

# 如何充分利用本书

读者应该了解 Linux 和 Docker 命令的基础知识。虽然这不是强制性的，但如果读者对网络协议有基本了解并且对云计算概念有所熟悉，将会是一个很大的加分项。

尽管可以使用 MacBook 或 Windows 操作系统的 PC 来运行本书中的示例，但强烈建议读者使用 Ubuntu Linux 16.04 及以上版本。使用 MacBook 或 Windows 的读者可以通过虚拟机上的 Linux 或云实例来运行示例。

# 下载示例代码文件

您可以从您在 [www.packtpub.com](http://www.packtpub.com) 的账户下载本书的示例代码文件。如果您是在其他地方购买的本书，您可以访问 [www.packtpub.com/support](http://www.packtpub.com/support) 并注册，文件将直接通过电子邮件发送给您。

您可以通过以下步骤下载代码文件：

1.  请在 [www.packtpub.com](http://www.packtpub.com/support) 登录或注册。

1.  选择 SUPPORT 选项卡。

1.  点击 Code Downloads & Errata。

1.  在搜索框中输入书名，并按照屏幕上的指示操作。

下载完成后，请确保使用最新版本的工具解压或提取文件夹：

+   Windows 用 WinRAR/7-Zip

+   Mac 用 Zipeg/iZip/UnRarX

+   7-Zip/PeaZip for Linux

本书的代码包也托管在 GitHub 上，地址是 [`github.com/PacktPublishing/Docker-for-Serverless-Applications`](https://github.com/PacktPublishing/Docker-for-Serverless-Applications)。如果代码有更新，GitHub 上的现有代码库也会进行更新。

我们的丰富书籍和视频目录中还有其他代码包，您可以访问 **[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)**。快来看看吧！

# 使用的约定

本书中使用了多种文本约定。

`文本中的代码`: 表示文本中的代码词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟网址、用户输入和 Twitter 用户名。示例：“我们将尝试使用`echoit`函数输出`hello world`，并使用 OpenFaaS。”

代码块如下所示：

```
FROM ubuntu
RUN apt-get update && apt-get install -y nginx
EXPOSE 80
ENTRYPOINT ["nginx", "-g", "daemon off;"]
```

任何命令行输入或输出都按如下方式书写：

```
$ curl -sSL https://get.docker.com | sudo sh
$ docker swarm init --advertise-addr=eth0
```

**粗体**: 表示新术语、重要词汇或屏幕上显示的词汇。例如，菜单或对话框中的词语会以这种方式显示。示例：“以下截图展示了浏览器运行 OpenFaaS Portal。”

警告或重要说明以这种方式呈现。

提示和技巧以这种方式呈现。

# 联系我们

我们始终欢迎读者的反馈。

**一般反馈**: 发送电子邮件至 `feedback@packtpub.com`，并在邮件主题中注明书名。如果您对本书的任何部分有疑问，请通过 `questions@packtpub.com` 联系我们。

**勘误表**: 尽管我们已尽力确保内容的准确性，但难免会出现错误。如果您在本书中发现错误，我们将非常感激您向我们报告。请访问 [www.packtpub.com/submit-errata](http://www.packtpub.com/submit-errata)，选择您的书籍，点击“勘误表提交”链接，并填写相关信息。

**盗版**: 如果您在互联网上发现我们作品的非法复制品，无论形式如何，我们将非常感激您提供其位置地址或网站名称。请通过 `copyright@packtpub.com` 联系我们，并附上相关材料的链接。

**如果你有兴趣成为作者**: 如果你在某个主题方面有专长，且有意撰写或参与书籍的编写，请访问 [authors.packtpub.com](http://authors.packtpub.com/)。

# 评论

请留下评论。阅读并使用本书后，不妨在您购买书籍的网站上留下评论。潜在读者可以通过您的无偏见评价做出购买决策，我们可以了解您对我们的产品的看法，作者也能看到您对其书籍的反馈。谢谢！

有关 Packt 的更多信息，请访问 [packtpub.com](https://www.packtpub.com/)。
