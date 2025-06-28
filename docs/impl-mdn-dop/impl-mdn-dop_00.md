# 序言

DevOps 是快速高效地部署软件的最新革命。通过一套自动化工具、编排平台和一些流程，公司可以加速其 IT 系统的发布周期，使工程师能够在业务流程中更有效地工作。

# 本书内容涵盖

第一章，*实际世界中的 DevOps*，展示了 DevOps 在当前 IT 公司工程部门中的位置，以及如何调整资源以最大化交付潜力。

第二章，*云数据中心*，比较了不同的云解决方案，用于管理云上的资源（虚拟机、网络、磁盘等）。

第三章，*Docker*，教授关于 Docker 及其内部工作原理的知识，以便更好地理解容器化技术的运作方式。

第四章，*持续集成*，讨论了可以用来执行测试以及其他许多操作的持续集成技术，正如我们将在第八章，*发布管理 - 持续交付*中看到的。

第五章，*基础设施即代码*，展示了如何以可编程的方式描述我们的基础设施，并应用软件开发生命周期的最佳实践来确保其完整性。

第六章，*服务器配置*，展示了如何使用 Ansible 来管理远程服务器的配置，以便简化大量服务器的维护工作，即使我们将重点放在 Kubernetes 上，这也是值得了解的。

第七章，*Docker Swarm 和 Kubernetes - 集群基础设施*，简要介绍了 Docker Swarm，然后引导您关注 Kubernetes，这是最先进的容器编排技术，被世界上最大的公司如 Google 广泛使用。

第八章，*发布管理 - 持续交付*，展示如何在 Google Cloud Platform 上使用 Kubernetes 和 Jenkins 设置持续交付管道。

第九章，*监控*，展示了如何监控我们的软件和服务器，以便能够在潜在故障发生前非常快速地发现并修复（可能）而不会影响我们的客户。

# 您需要准备什么

为了能够跟随本书及其内容，你需要在 Google Cloud Platform 上注册一个试用账户，并安装一个编辑器（我使用的是 Atom，但任何其他编辑器也可以），以及在本地机器上安装 Node.js。你还需要在本地安装 Docker，以便测试不同的示例。我们将在 Kubernetes 示例中使用**Google 容器引擎**（**GKE**），但如果你想在本地玩 Kubernetes，也可以使用 Minikube，尽管你需要一台相当强大的计算机。

# 本书适用对象

本书适合那些希望在 DevOps 领域提升技能的工程师，特别是那些想要精通 Kubernetes 和容器的读者。中级技能的读者最为理想。

# 约定

本书中，你会看到多种文本样式，用于区分不同类型的信息。以下是这些样式的一些示例及其含义的解释。文本中的代码词汇、数据库表名、文件夹名称、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 句柄以如下方式显示：“下一行代码会读取链接并将其分配给`BeautifulSoup`函数。” 一段代码块如下所示：

```
resource "google_compute_address" "my-first-ip" {
 name = "static-ip-address"
}
```

当我们希望引起你对代码块中特定部分的注意时，相关的行或项目会用粗体显示：

```
resource "google_compute_address" "my-first-ip" {
 name = "static-ip-address"
}
```

任何命令行输入或输出都写作如下：

```
docker commit 329b2f9332d5 my-ubuntu
```

**新术语**和**重要词汇**以粗体显示。你在屏幕上看到的词语，例如在菜单或对话框中，文本中会像这样显示：“为了下载新模块，我们将进入 Files | Settings | Project Name | Project Interpreter。”

警告或重要提示以这种方式显示。

提示和技巧以这种方式出现。

# 读者反馈

我们始终欢迎读者的反馈。告诉我们你对本书的看法——你喜欢或不喜欢的部分。读者的反馈对我们很重要，因为它帮助我们开发出你真正能从中受益的书籍。如果你有专业知识并且有兴趣编写或参与编写一本书，请查阅我们的作者指南：[www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

现在你是 Packt 书籍的骄傲拥有者，我们提供了许多帮助你最大限度利用这本书的资源。

# 下载示例代码

你可以从你的帐户中下载本书的示例代码文件，网址为：[`www.packtpub.com`](http://www.packtpub.com)。如果你在其他地方购买了本书，可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)并注册，文件会直接通过电子邮件发送给你。你可以按照以下步骤下载代码文件：

1.  使用你的电子邮件地址和密码登录或注册我们的官网。

1.  将鼠标指针悬停在顶部的 SUPPORT 标签上。

1.  点击代码下载与勘误表。

1.  在搜索框中输入书籍名称。

1.  选择您想要下载代码文件的书籍。

1.  从下拉菜单中选择您购买本书的渠道。

1.  点击“代码下载”。

文件下载完成后，请确保使用最新版本的以下工具解压或解压文件夹：

+   Windows 版的 WinRAR / 7-Zip

+   Mac 版的 Zipeg / iZip / UnRarX

+   Linux 版的 7-Zip / PeaZip

该书的代码包也托管在 GitHub 上，网址为[`github.com/PacktPublishing/Implementing-Modern-DevOps`](https://github.com/PacktPublishing/Implementing-Modern-DevOps)。我们还提供了其他书籍和视频的代码包，您可以在[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)找到它们。快来看看吧！

# 下载本书的彩色图片。

我们还为您提供了包含本书中使用的截图/图表的彩色图片的 PDF 文件。这些彩色图片将帮助您更好地理解输出中的变化。您可以从[`www.packtpub.com/sites/default/files/downloads/ImplementingModernDevOps_ColorImages.pdf`](https://www.packtpub.com/sites/default/files/downloads/ImplementingModernDevOps_ColorImages.pdf)下载该文件。

# 勘误表

尽管我们已尽最大努力确保内容的准确性，但错误仍然会发生。如果您在我们的书籍中发现错误——可能是文本或代码中的错误——我们将非常感激您向我们报告。通过这样做，您可以帮助其他读者避免挫折，并帮助我们改进后续版本。如果您发现任何勘误，请访问[`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)报告，选择您的书籍，点击勘误提交表格链接，填写勘误的详细信息。一旦您的勘误被验证，您的提交将被接受，勘误内容将被上传到我们的网站或添加到该书籍的勘误列表中。

要查看之前提交的勘误，请访问[`www.packtpub.com/books/content/support`](https://www.packtpub.com/books/content/support)，并在搜索框中输入书籍名称。所需的信息将在勘误部分显示。

# 盗版

网络盗版问题在所有媒体上都是一个持续的难题。在 Packt，我们非常重视对版权和许可证的保护。如果您在互联网上发现任何非法的我们作品的副本，请立即提供其位置地址或网站名称，以便我们采取措施。请通过`copyright@packtpub.com`与我们联系，并提供可疑盗版材料的链接。

我们感谢您帮助我们保护作者权益，并确保我们能够为您提供有价值的内容。

# 问题

如果您在本书的任何方面遇到问题，欢迎通过`questions@packtpub.com`联系我们，我们会尽力解决问题。
