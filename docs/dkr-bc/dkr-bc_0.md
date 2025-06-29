# 前言

很少有技术能够像 Docker 一样，在整个行业中被广泛采用。自 2013 年 3 月首次公开发布以来，Docker 不仅赢得了像你我这样的终端用户的支持，还得到了 Amazon、Microsoft 和 Google 等行业领导者的支持。

Docker 当前在其官网上使用以下句子来描述为什么你会想使用它：

> Docker 提供了一个集成的技术套件，使开发和 IT 运维团队能够在任何地方构建、分发和运行分布式应用程序。

尽管 Docker 的描述听起来很简单，但多年来，开发和 IT 运维团队的终极目标就是拥有一个工具，能够确保应用程序在整个生命周期的各个阶段（从开发到生产）始终如一地工作。

你将学习如何在你选择的操作系统上安装 Docker。你会发现，一旦 Docker 安装完成，无论你使用的是哪个操作系统，运行容器时都会得到相同的结果。

然后，我们将扩展我们的 Docker 安装到公共云中，你将了解到无论你将 Docker 主机部署到哪里，体验始终是一致且简单的。

到最后一章时，你应该对如何将 Docker 集成到你的日常工作流程中以及接下来容器的使用步骤有一个概念。

# 本书内容概览

第一章，*本地安装 Docker*，讲解了如何在 macOS、Windows 10 和 Linux 桌面上安装核心 Docker 引擎及其支持工具，以便为接下来的章节做好准备。

第二章，*使用 Docker 启动应用程序*，使用我们在上一章中安装的 Docker 来启动容器。在这一章结束时，我们将手动启动一个 WordPress 安装，并使用 Docker Compose 定义你的多容器应用程序。我们还将看看如何将你自己的镜像发布到 Docker Hub。

第三章，*云中的 Docker*，解释了如何从本地安装的 Docker 迁移到公共云。在这里，我们将研究如何在各种公共云上启动 Docker 主机，并将我们的应用程序部署到这些主机上。

第四章，*Docker Swarm*，继续使用公共云；但我们将不再仅仅操作单个孤立的 Docker 主机，而是部署并配置一个 Docker Swarm 集群。

第五章，*Docker 插件*，讨论了描述 Docker 时使用的短语，其中包括 *内置电池，但可拆卸*。在这一章中，我们将研究第三方插件，这些插件通过添加持久存储和多主机网络来扩展 Docker 的核心功能。

第六章, *故障排除与监控*，现在我们在本地、远程以及集群中都已经运行了容器，那么可能出现什么问题呢？在这一章中，我们将探讨你可能遇到的一些问题。同时，我们将学习如何使用第一方和第三方工具来部署工具，以获取如 CPU、内存和硬盘利用率等来自容器的度量信息。

第七章, *综合应用*，强调你现在应该已经对 Docker 有了较好的理解，了解它是如何工作的，以及一些可能的使用场景。在这一章中，我们将探讨如何与同事共享容器体验以及接下来应该采取的步骤。

# 本书所需的条件

你需要在以下平台上安装并配置 Docker17.03（CE）：

+   Windows 10 专业版

+   macOS Sierra

+   Ubuntu 16.04 LTS 桌面版

此外，你还应能访问如 Digital Ocean、Amazon Web Service 或 Microsoft Azure 等公共云平台。

# 本书适用对象

本书面向开发人员、IT 专业人士和 DevOps 工程师，帮助他们在不花费大量时间学习的情况下，通过实践掌握 Docker 的知识和技能。如果你一直在为抽出时间掌握 Docker 容器和日常 Docker 任务而感到困扰，那么你来对地方了！

# 约定

在本书中，你会看到许多文本样式，它们用于区分不同类型的信息。以下是这些样式的一些示例及其含义说明。

文本中的代码词汇、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟网址、用户输入和 Twitter 账号都按如下方式显示：“我们可以通过使用`include`指令来包含其他内容。”

一段代码的格式如下：

```
docker container run -d \
    --name mysql \
    -e MYSQL_ROOT_PASSWORD=wordpress \
    -e MYSQL_DATABASE=wordpress \
    mysql
```

当我们希望引起你对代码块中特定部分的注意时，相关的行或项目会以粗体显示：

```
# Install the packages we need to run wp-cli
RUN apt-get update &&\
apt-get install -y sudo less mysql-client &&\
```

任何命令行输入或输出都按如下方式书写：

```
curl -L "https://github.com/docker/compose/releases/download/1.10.0/docker-compose
-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
chmod +x /tmp/docker-compose
sudo cp /tmp/docker-compose /usr/local/bin/docker-compose

```

**新术语** 和 **重要词汇** 会以粗体显示。在屏幕上看到的词汇，例如菜单或对话框中的内容，会按如下方式显示：“点击**下一步**按钮将带你进入下一个屏幕。”

### 注意

警告或重要提示会以框框形式显示，如下所示。

### 小贴士

小贴士和技巧会以如下形式出现。

# 读者反馈

我们始终欢迎读者的反馈。告诉我们你对这本书的看法——你喜欢或不喜欢什么。读者反馈对我们非常重要，它帮助我们开发出真正能让你受益的书籍。

如果要向我们提供一般反馈，请发送电子邮件至 `<feedback@packtpub.com>`，并在邮件主题中提及书名。

如果你在某个主题方面有专业知识，并且有兴趣撰写或参与撰写书籍，请查看我们的作者指南：[www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

现在你已经成为一本 Packt 书籍的骄傲拥有者，我们提供了许多帮助你充分利用购买的资源。

## 下载示例代码

你可以从你的账户中下载所有你购买的 Packt Publishing 书籍的示例代码文件，访问[`www.packtpub.com`](http://www.packtpub.com)。如果你在其他地方购买了本书，可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)并注册以便直接将文件通过电子邮件发送给你。

你可以按照以下步骤下载代码文件：

1.  使用你的电子邮件地址和密码登录或注册我们的官方网站。

1.  将鼠标指针悬停在页面顶部的**支持**标签上。

1.  点击**代码下载与勘误表**。

1.  在**搜索**框中输入书籍名称。

1.  选择你想下载代码文件的书籍。

1.  从下拉菜单中选择你购买本书的地方。

1.  点击**代码下载**。

一旦文件下载完成，请确保使用以下最新版本的工具解压或提取文件夹：

+   WinRAR / 7-Zip for Windows

+   Zipeg / iZip / UnRarX for Mac

+   7-Zip / PeaZip for Linux

本书的代码包也托管在 GitHub 上，地址是[`github.com/PacktPublishing/Docker-Bootcamp`](https://github.com/PacktPublishing/Docker-Bootcamp)。我们还有其他来自丰富书籍和视频目录的代码包，均可在[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)找到。快来看看吧！

## 勘误

虽然我们已尽最大努力确保内容的准确性，但错误仍然可能发生。如果你在我们的书中发现错误——可能是文本或代码中的错误——我们将非常感激你向我们报告。这样，你可以帮助其他读者避免困扰，并帮助我们改进本书的后续版本。如果你发现任何勘误，请通过访问[`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)来报告，选择你的书籍，点击**勘误提交表单**链接，并输入勘误的详细信息。勘误经验证后，我们会接受你的提交，并将勘误上传到我们的网站或添加到该书的勘误列表中。

要查看之前提交的勘误，请访问[`www.packtpub.com/books/content/support`](https://www.packtpub.com/books/content/support)，并在搜索框中输入书籍名称。相关信息将显示在**勘误**部分。

## 盗版

互联网上的版权侵权问题在所有媒体中都持续存在。在 Packt，我们非常重视版权和许可证的保护。如果您在互联网上遇到任何形式的我们作品的非法复制品，请立即提供该位置地址或网站名称，以便我们采取措施解决问题。

请通过 `<copyright@packtpub.com>` 联系我们，并附上涉嫌盗版材料的链接。

我们感谢您的帮助，保护我们的作者以及我们为您带来有价值内容的能力。

## 问题

如果您对本书的任何部分有疑问，您可以通过 `<questions@packtpub.com>` 与我们联系，我们将尽力解决问题。
