# 前言

在过去的几年里，Docker 已成为最令人兴奋的新技术之一。无论是企业公司还是初创公司，都纷纷采纳了这一工具。

多个第一方和第三方工具已被开发用于扩展核心 Docker 功能。本书将指导你安装、配置和使用这些工具，并帮助你了解哪些工具最适合某项任务。

# 本书内容简介

第一章，*扩展 Docker 入门*，讨论了 Docker 及其解决的一些问题。我们还将讨论如何通过扩展核心 Docker 引擎来获得更多功能。

第二章，*引入第一方工具*，介绍了 Docker 提供的工具，用于与核心 Docker 引擎协同工作。这些工具包括 Docker Toolbox、Docker Compose、Docker Machine 和 Docker Swarm。

第三章，*存储插件*，介绍了 Docker 插件。我们将从 Docker 自带的默认存储插件开始，接着介绍三个第三方插件。

第四章，*网络插件*，解释了如何在多个 Docker 主机间扩展容器网络，无论是本地部署还是在公共云中。

第五章，*构建你自己的插件*，介绍了如何最佳地编写自己的 Docker 存储或网络插件。

第六章，*扩展你的基础设施*，介绍了如何使用一些成熟的 DevOps 工具来部署和管理你的 Docker 主机和容器。

第七章，*调度器分析*，讨论了如何部署 Kubernetes、Amazon ECS 和 Rancher，跟随前面的章节内容。

第八章，*安全性、挑战与结论*，帮助解释从何处部署 Docker 镜像的安全隐患，并回顾前面章节中讨论过的各种工具及其最佳应用场景。

# 本书所需内容

你需要一台能够运行 VirtualBox（[`www.virtualbox.org/`](https://www.virtualbox.org/)）的 OS X 或 Windows 笔记本电脑或桌面电脑，并且能够访问 Amazon Web Service 和 DigitalOcean 账户，且拥有启动资源的权限。

# 本书适合谁阅读

本书面向开发人员和系统管理员，他们觉得基础的 Docker 安装已经无法满足需求，想通过扩展核心 Docker 引擎的功能来进一步优化配置，以应对不断变化的业务需求。

# 约定

在本书中，您将看到多种文本样式，用以区分不同种类的信息。以下是这些样式的几个示例以及它们的含义说明。

文本中的代码词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 账号等，均以如下方式呈现：“安装完成后，您应该能够通过运行 Docker `hello-world` 容器来检查是否一切正常。”

代码块如下所示：

```
### Dockerfile
FROM php:5.6-apache
MAINTAINER Russ McKendrick <russ@mckendrick.io>
ADD index.php /var/www/html/index.php
```

当我们希望引起您对代码块中特定部分的注意时，相关的行或项目会以粗体显示：

```
version: '2'
services:
  wordpress:
    container_name: "my-wordpress-app"
    image: wordpress
    ports:
      - "80:80"
    environment:
 - "WORDPRESS_DB_HOST=mysql.weave.local:3306"
      - "WORDPRESS_DB_PASSWORD=password"
      - "constraint:node==chapter04-01"
```

任何命令行输入或输出都会按如下方式写出：

```
curl -sSL https://get.docker.com/ | sh

```

**新术语**和**重要单词**以粗体显示。您在屏幕上看到的单词，例如在菜单或对话框中，都会像这样出现在文本中：“要继续安装的下一步，请点击**继续**。”

### 注意

警告或重要提示将以如上所示的框呈现。

### 提示

提示和技巧将以这样的方式呈现。

# 读者反馈

我们始终欢迎读者反馈。请告诉我们您对本书的看法——喜欢或不喜欢的地方。读者反馈对我们非常重要，它帮助我们开发出您真正能从中受益的书籍。

要向我们发送一般反馈，只需通过电子邮件 `<feedback@packtpub.com>` 联系我们，并在邮件主题中提到书籍标题。

如果您在某个主题上拥有专长，并且有兴趣撰写或贡献书籍，请参考我们的作者指南，网址为 [www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

现在，您已经成为一本 Packt 书籍的自豪拥有者，我们为您提供了许多帮助您最大化购买价值的资源。

## 下载示例代码

您可以从 [`www.packtpub.com`](http://www.packtpub.com) 账户中下载本书的示例代码文件。如果您是从其他地方购买的此书，您可以访问 [`www.packtpub.com/support`](http://www.packtpub.com/support) 并注册，文件会直接通过电子邮件发送给您。

您可以通过以下步骤下载代码文件：

1.  使用您的电子邮件地址和密码登录或注册我们的网站。

1.  将鼠标指针悬停在顶部的**支持**标签上。

1.  点击**代码下载和勘误**。

1.  在**搜索**框中输入书名。

1.  选择您要下载代码文件的书籍。

1.  从下拉菜单中选择您购买此书的来源。

1.  点击**代码下载**。

您还可以通过点击书籍网页上的**代码文件**按钮来下载代码文件。您可以通过在**搜索**框中输入书名来访问此页面。请注意，您需要登录您的 Packt 账户。

文件下载完成后，请确保使用最新版本的工具解压或提取文件夹：

+   WinRAR / 7-Zip for Windows

+   Zipeg / iZip / UnRarX for Mac

+   适用于 Linux 的 7-Zip / PeaZip

本书的代码包也托管在 GitHub 上，链接为 [`github.com/PacktPublishing/ExtendingDocker`](https://github.com/PacktPublishing/ExtendingDocker)。我们还提供了其他来自我们丰富图书和视频目录的代码包，您可以在 [`github.com/PacktPublishing/`](https://github.com/PacktPublishing/) 查看。快去看看吧！

## 勘误

尽管我们已尽力确保内容的准确性，但错误有时还是会发生。如果您在我们的书籍中发现错误——可能是文本或代码中的错误——我们将非常感激您能报告此问题。这样，您不仅可以帮助其他读者避免困扰，还能帮助我们改进本书的后续版本。如果您发现任何勘误，请通过访问 [`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)，选择您的书籍，点击 **勘误提交表单** 链接，输入勘误的详细信息。经验证后，您的勘误提交将被接受，并上传到我们的网站或添加到该书籍的勘误列表中。

要查看之前提交的勘误， 请访问 [`www.packtpub.com/books/content/support`](https://www.packtpub.com/books/content/support)，并在搜索框中输入书名。相关信息将显示在 **勘误** 部分。

## 盗版

网络上的版权材料盗版问题是各类媒体的持续问题。在 Packt，我们非常重视保护我们的版权和许可证。如果您在互联网上发现我们作品的任何非法复制版本，请立即提供其位置地址或网站名称，以便我们采取措施。

如果您发现涉嫌侵权的内容，请通过`<copyright@packtpub.com>`联系我们，并附上涉嫌盗版材料的链接。

感谢您帮助保护我们的作者以及我们提供有价值内容的能力。

## 问题

如果您在本书的任何方面遇到问题，可以通过`<questions@packtpub.com>`联系我们，我们将尽力解决问题。
