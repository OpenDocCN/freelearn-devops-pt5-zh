# 前言

《DevOps 与 Docker》概述了容器化的强大功能以及这一创新对开发团队和日常运营的影响。我们还将了解什么是真正的 DevOps，涉及的原则，以及如何通过实施 Docker 工作流促进产品健康。Docker 是一个开源容器化工具，它简化了产品交付的流程，缩短了从商业构思到实施交付的时间。

本书将提供以下知识：

+   Docker 和 DevOps，以及它们为何以及如何集成

+   容器是什么，如何创建和管理它们

+   使用 Docker 扩展交付管道和多次部署

+   容器化应用的编排与交付

第一章*，镜像与容器*，展示了 Docker 如何改进 DevOps 工作流，并介绍了本书中将使用的基本 Docker 终端命令。我们将学习 Dockerfile 语法，以便构建镜像。我们将从镜像中运行容器，然后对镜像和 Docker Hub 进行版本管理，并将 Docker 镜像部署到 Docker Hub。

第二章*，应用容器管理*，介绍了 docker-compose 工具，并概述了多容器应用的配置。我们将管理多个容器并分发应用程序包。最后，我们将使用 docker-compose 进行网络配置。

第三章*，编排与交付*，为我们提供了 Docker Swarm 的概述。接着，我们将使用 Docker 引擎创建一个 Swarm，并管理 Swarm 中的服务和应用。最后，我们将根据实际应用场景测试服务的扩展和缩放。

# 硬件

本书要求以下最低硬件配置：

+   处理器：1.8 GHz 或更高（Core 2 Duo 及以上）

+   内存：最低 2GB RAM

+   硬盘：最低 10 GB

+   稳定的互联网连接（用于拉取和推送镜像）

# 软件

+   操作系统：Windows 8 或更高版本

+   浏览器：Google Chrome 或 Mozilla Firefox（安装了最新更新）

+   安装 Docker

# 本书适合的人群

本书适合希望采用 Docker 工作流来实现应用的一致性、速度和隔离性的开发人员、系统架构师、初级和中级站点可靠性工程师。我们将深入讲解 Docker，因此您需要具备关于 UNIX 概念（如 ssh、端口和日志）的基础知识。

# 约定

在本书中，您将看到多种文本样式，用于区分不同类型的信息。以下是这些样式的一些示例以及它们的含义解释。

文本中的代码词、数据库表名、文件夹名称、文件名、文件扩展名、路径名、虚拟网址、用户输入和 Twitter 账号将如下所示： "一旦创建了一个新的目录，访问该目录并创建一个名为 `run.js` 的文件。"

任何命令行输入或输出都如下所示：

```
docker pull
```

**新术语** 和 **重要词汇** 以粗体显示。您在屏幕上看到的单词，例如在菜单或对话框中，通常会以这样的形式出现在文本中：“点击下一步按钮将您带到下一个屏幕。”

### 注意

警告或重要说明会以框的形式出现，如下所示。

### 小贴士

小贴士和技巧如下所示。

# 读者反馈

我们欢迎读者的反馈。请告诉我们您对本书的看法——您喜欢或不喜欢的内容。读者反馈对我们非常重要，因为它帮助我们开发出您能真正从中受益的图书。

要发送一般反馈，只需通过电子邮件 `<feedback@packtpub.com>` 与我们联系，并在邮件主题中注明书名。

如果您对某个话题有专业知识，并且有兴趣写书或为书籍做贡献，请查看我们的作者指南，地址为 [www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

既然您已经成为 Packt 图书的骄傲拥有者，我们为您提供了一些帮助，帮助您充分利用您的购买。

# 下载示例代码

您可以从您的账户下载本书的示例代码文件，链接为 [`github.com/TrainingByPackt/Beginning-DevOps-with-Docker`](https://github.com/TrainingByPackt/Beginning-DevOps-with-Docker)。如果您在其他地方购买了本书，可以访问 [`www.packtpub.com/support`](http://www.packtpub.com/support) 并注册，以便将文件直接通过电子邮件发送给您。

您可以按照以下步骤下载代码文件：

1.  使用您的电子邮件地址和密码登录或注册我们的网站。

1.  将鼠标指针悬停在顶部的 **SUPPORT** 标签上。

1.  点击 **代码下载** 和 **勘误表**。

1.  在 **搜索** 框中输入书名。

1.  选择您想要下载代码文件的书籍。

1.  从下拉菜单中选择您购买此书的地点。

1.  点击 **代码下载**。

您还可以通过点击 Packt 出版社网站上书籍网页中的**代码文件**按钮下载代码文件。您可以通过在**搜索**框中输入书名来访问此页面。请注意，您需要登录到您的 Packt 账户。

下载文件后，请确保使用以下最新版本的工具解压或提取文件夹：

+   WinRAR / 7-Zip for Windows

+   Zipeg / iZip / UnRarX for Mac

+   7-Zip / PeaZip for Linux

本书的代码包也托管在 GitHub 上，地址为 [`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)。我们还有其他来自我们丰富书籍和视频目录的代码包，您可以在 [`github.com/PacktPublishing/`](https://github.com/PacktPublishing/) 上查看它们！

# 安装

在开始本课程之前，我们将安装 Docker。您可以在这里找到安装步骤：

请访问下面的 Docker Toolbox 页面：[`docs.docker.com/toolbox/toolbox_install_windows/`](https://docs.docker.com/toolbox/toolbox_install_windows/)。

1.  点击安装程序链接进行下载。

1.  双击安装程序以安装 Docker Toolbox。

1.  按“下一步”接受所有默认设置，然后进行安装。

# 勘误表

尽管我们已经尽最大努力确保内容的准确性，但错误还是可能发生。如果您在我们的书籍中发现错误——可能是文本或代码中的错误——我们将非常感激您能报告给我们。通过这样做，您可以帮助其他读者避免困惑，并帮助我们改进后续版本。如果您发现任何勘误，请通过访问[`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)进行报告，选择您的书籍，点击“勘误提交表单”链接，并填写勘误的详细信息。您的勘误一经验证，将被接受并上传至我们的网站，或添加到该书籍的**勘误表**部分中的现有勘误列表。

要查看先前提交的勘误表，请访问[`www.packtpub.com/books/content/support`](https://www.packtpub.com/books/content/support)，并在搜索框中输入书名。所需信息将显示在**勘误表**部分下。

# 盗版

互联网盗版问题普遍存在于所有媒体上。在 Packt，我们非常重视保护我们的版权和许可证。如果您在互联网上发现我们作品的任何非法复制品，请立即提供该位置地址或网站名称，以便我们采取措施。

如发现疑似盗版资料，请通过`<copyright@packtpub.com>`联系我们，提供相关链接。

我们感谢您的帮助，保护我们的作者以及我们为您提供有价值内容的能力。

# 问题

如果您对本书的任何部分有疑问，可以通过`<questions@packtpub.com>`联系我们，我们将尽力解决问题。
