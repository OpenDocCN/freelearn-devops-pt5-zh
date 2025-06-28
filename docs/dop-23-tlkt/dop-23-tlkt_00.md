# 前言

如果你读过《*DevOps 工具集系列*》中的其他书籍（[`www.devopstoolkitseries.com/`](http://www.devopstoolkitseries.com/)），你应该知道我非常喜欢容器、调度器和编排工具。《*DevOps 2.0 工具集：通过容器化微服务自动化持续部署流水线*》([`www.amazon.com/dp/B01BJ4V66M`](https://www.amazon.com/dp/B01BJ4V66M)) 最初是关于多种不同 DevOps 工具和实践的概述，其中容器占据重要但并非决定性的位置。与此同时，我完全迷上了 Docker 和 Swarm，于是我决定写《*DevOps 2.1 工具集：Docker Swarm：在 Docker Swarm 集群中构建、测试、部署和监控服务*》([`www.amazon.com/dp/1542468914`](https://www.amazon.com/dp/1542468914))。当我完成它时，我觉得一些高级话题没有得到探讨，值得专门写一本书来讲解。于是，《*DevOps 2.2 工具集：自给自足的 Docker 集群：构建自适应和自愈的 Docker 集群*》([`www.amazon.com/dp/1979347190`](https://www.amazon.com/dp/1979347190)) 应运而生。

所有这些书籍（无论是直接还是间接）都集中在 Docker Swarm 上，我觉得 Kubernetes 值得拥有自己的篇幅。说实话，几年前我对它持消极态度。它对于大多数用例来说太复杂了。仅仅尝试安装它，在经历了几天的挣扎后放弃就足够了。然而，自那时以来，Kubernetes 走过了漫长的道路。尽管它的创始原则没有改变，但随着时间的推移，它变得更加成熟、更加简洁，规模也变得更大了。今天，它是最大的也是最被广泛采用的容器编排平台。一些最知名的软件公司纷纷加入其中。许多初创公司也涌现出新的解决方案。Kubernetes 背后的开源社区是软件开发史上最大的社区之一。这个社区充满活力、快速发展，并且对 Kubernetes 的成功有着极大的利益驱动。甚至 Docker 也选择支持它并加入这个社区。Kubernetes 的前景非常光明，任何人都不应忽视它。

如果你已经选择了其他容器调度工具（例如 Docker Swarm、Mesos、Nomad 或其他工具），你可能会想知道是否值得花时间学习 Kubernetes。我认为是值得的。我们应该始终关注不同的解决方案，否则我们就是盲目地被迫选择一个。不管你决定采用 Kubernetes 还是坚持使用其他工具，我认为你应该了解它是如何工作的，以及它提供了什么。这关乎做出明智的决定。

# 概述

本书的目标不是说服你采用 Kubernetes，而是提供其功能的详细概述。我希望你能对 Kubernetes 知识充满信心，然后再决定是否采纳它。除非你已经决定，并且偶然发现这本书来寻找 Kubernetes 的相关指导。

计划是覆盖 Kubernetes 的所有方面，从基础到高级功能。我们不仅会讲解官方项目背后的工具，还会涉及第三方插件。我希望，当你读完这本书时，你能够自称为“Kubernetes 忍者”。我不能说你会了解 Kubernetes 生态系统中的一切，因为这几乎不可能做到，因为它的发展速度远远超过任何一个人能够跟上的速度。我可以说的是，你将非常自信地在生产环境中运行任何规模的 Kubernetes 集群。

像我所有的其他书籍一样，本书也非常实践。书中会有足够的理论，帮助你理解每个主题背后的原理。书中充满了大量示例，因此我需要提醒你一下。如果你打算在公交车上或睡觉前在床上阅读这本书，最好不要购买。你需要坐在电脑前。终端将是你最好的朋友，`kubectl`将是你的爱人。

本书假设你已经对容器，特别是 Docker 感到熟悉。我们不会详细讲解如何构建镜像、什么是容器注册表以及如何编写 Dockerfile。我希望你已经了解这些内容。如果你不太清楚这些，建议先学习一些基础的容器操作，再来阅读这本书。本书的内容是关于在你构建好镜像并将其存储在注册表中之后的事情。

本书的内容是关于大规模运行容器，以及在问题出现时不慌乱。这是关于当前和未来的软件部署与监控。这是关于迎接挑战，并始终走在技术前沿。

最终，你可能会遇到困难，需要帮助。或者你可能想写一个评论或对本书的内容提出意见。请加入 DevOps20 ([`slack.devops20toolkit.com/`](http://slack.devops20toolkit.com/)) Slack 频道，发表你的想法、提问或参与讨论。如果你更倾向于一对一的交流，可以通过 Slack 给我发私信，或者发送电子邮件到 `viktor@farcic.com`。所有我写的书对我来说都非常重要，我希望你能有一个愉快的阅读体验。这个体验的一部分就是可以随时联系我。不要害羞。

请注意，这本书与之前的书籍一样，属于自出版。我认为没有中介介入作者和读者之间是最好的方式。它让我写得更快，更频繁地更新书籍，并与您进行更直接的交流。你的反馈是这个过程的一部分。无论你在书籍只有部分或所有章节完成时购买，都意味着这本书永远不会真正完成。随着时间的推移，它需要更新，以便与技术或流程的变化保持一致。尽可能地，我会保持它的更新，并在合理的时候发布更新。最终，事情可能会发生如此大的变化，以至于更新不再是一个好的选择，那时候就意味着需要一本全新的书。我会继续写，只要你继续支持我**。**

# 下载示例代码文件

你可以从[www.packtpub.com](http://www.packtpub.com)的帐户下载本书的示例代码文件。如果你是在其他地方购买了这本书，你可以访问[www.packtpub.com/support](http://www.packtpub.com/support)，并注册以便直接通过邮件获得文件。

你可以通过以下步骤下载代码文件：

1.  在[www.packtpub.com](http://www.packtpub.com/support)登录或注册。

1.  选择“SUPPORT”标签。

1.  点击“代码下载与勘误”。

1.  在搜索框中输入书名，并按照屏幕上的指示操作。

文件下载后，请确保使用最新版本解压或提取文件夹：

+   Windows 上的 WinRAR/7-Zip

+   Mac 上的 Zipeg/iZip/UnRarX

+   Linux 上的 7-Zip/PeaZip

本书的代码包也托管在 GitHub 上，地址为[`github.com/PacktPublishing/The-DevOps-2.3-Toolkit`](https://github.com/PacktPublishing/The-DevOps-2.3-Toolkit)。我们还在**[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)**上提供了其他代码包，来自我们丰富的书籍和视频目录。快来看看吧！

# 下载彩色图片

我们还提供了一个 PDF 文件，其中包含本书中使用的截图/图表的彩色图片。你可以在此处下载：[`www.packtpub.com/sites/default/files/downloads/TheDevOps2.3Toolkit_ColorImages.pdf`](https://www.packtpub.com/sites/default/files/downloads/TheDevOps2.3Toolkit_ColorImages.pdf)。

# 使用的约定

本书中使用了多种文本约定。

`CodeInText`：表示文本中的代码字、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 用户名。示例："即使我们在容器内执行了`docker`命令，输出仍然清晰地显示了主机上的镜像。"

一段代码如下所示：

```
global: 
  scrape_interval:     15s 

scrape_configs: 
  - job_name: Prometheus 
    metrics_path: /prometheus/metrics 
    static_configs: 
      - targets: 
        - localhost:9090 
```

当我们希望引起你对代码块中特定部分的注意时，相关的行或项目会以粗体显示：

```
global: 
  scrape_interval:     15s 

scrape_configs: 
  - job_name: Prometheus 
    metrics_path: /prometheus/metrics 
    static_configs: 
      - targets: 
        - localhost:9090 
```

任何命令行输入或输出如下所示：

```
docker container exec -it $ID \
 curl node-exporter:9100/metrics

```

**粗体**：表示新术语、重要词汇或屏幕上显示的内容。例如，菜单或对话框中的词汇会以这种形式出现在文本中。以下是一个例子：“请输入`test`在项目名称字段中，选择`Pipeline`作为类型，然后点击确认按钮。”

警告或重要提示如下所示。

小贴士和技巧如下所示。

# 联系我们

我们总是欢迎读者的反馈。

**一般反馈**：请通过电子邮件发送至`feedback@packtpub.com`，并在邮件主题中提及书名。如果您对本书的任何内容有疑问，请发送邮件至`questions@packtpub.com`。

**勘误**：尽管我们已尽力确保内容的准确性，但错误难免会发生。如果您发现本书中的错误，我们非常感激您能够向我们报告。请访问[www.packtpub.com/submit-errata](http://www.packtpub.com/submit-errata)，选择您的书籍，点击“勘误提交表格”链接并填写相关信息。

**盗版**：如果您在互联网上发现我们作品的任何非法复制品，请提供相关的地址或网站名称。请通过`copyright@packtpub.com`与我们联系，并附上材料链接。

**如果您有兴趣成为作者**：如果您在某个领域具有专业知识，并且有意写书或为书籍贡献内容，请访问[authors.packtpub.com](http://authors.packtpub.com/)。
