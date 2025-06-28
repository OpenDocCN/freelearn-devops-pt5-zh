# 前言

欢迎来到*Azure 云管理实战*。本书旨在帮助你入门 Azure，并指导你踏上云计算之旅。我们将从云计算的概念开始，帮助你理解云与本地基础设施的区别。基础的**基础设施即服务**（**IaaS**）和**平台即服务**（**PaaS**）服务将会被讲解，帮助你理解如何利用云服务。接下来，我们将讨论迁移和混合云，解释如何将你的服务迁移到云端，并如何将其与本地资源结合。还将讲解身份和安全，以帮助你保障和保护云资源。最后，我们将讨论一些最佳实践和基础设施即代码（Infrastructure-as-Code），以帮助你管理和监控资源。

本书将通过真实的案例和逐步的指导，帮助你理解云计算的原则，并在你的环境中应用这些知识。

# 本书的目标读者

本书面向 IT 专业人员、系统工程师、管理员、DevOps 从业者以及任何想要了解 Azure 和云计算概念的人。你应该具备基本的云计算知识，并具备中级的网络和服务器管理知识。

# 为了充分利用本书

建议具备基本的云计算知识。为了更好地理解本地基础设施与云基础设施之间的关键差异，建议具备中级的服务器和网络管理知识。本书将使用以下工具：

+   Windows Server 2016

+   MS SQL Server 2016

+   Hyper-V

+   Active Directory

+   PowerShell

+   Microsoft Azure

+   Azure PowerShell

+   Azure CLI

# 下载示例代码文件

你可以从[www.packt.com](http://www.packt.com)的账户中下载本书的示例代码文件。如果你在其他地方购买了本书，可以访问[www.packt.com/support](http://www.packt.com/support)，注册后将文件直接发送到你的邮箱。

你可以通过以下步骤下载代码文件：

1.  在[www.packt.com](http://www.packt.com)登录或注册。

1.  选择 SUPPORT 标签。

1.  点击“代码下载与勘误”。

1.  在搜索框中输入书籍名称，并按照屏幕上的指示操作。

文件下载后，请确保使用以下最新版本的工具解压或提取文件夹：

+   适用于 Windows 的 WinRAR/7-Zip

+   适用于 Mac 的 Zipeg/iZip/UnRarX

+   适用于 Linux 的 7-Zip/PeaZip

本书的代码包也托管在 GitHub 上，网址是[`github.com/PacktPublishing/Hands-On-Cloud-Administration-in-Azure`](https://github.com/PacktPublishing/Hands-On-Cloud-Administration-in-Azure)。如果代码有更新，它会在现有的 GitHub 库中进行更新。

我们还有其他来自丰富书籍和视频目录的代码包，访问**[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)**查看！

# 下载彩色图像

我们还提供了一份 PDF 文件，其中包含本书中使用的截图/图表的彩色图像。你可以在这里下载它：`www.packtpub.com/sites/default/files/downloads/9781789134964_ColorImages.pdf`。

# 使用的规范

本书中使用了一些文本规范。

`CodeInText`：表示文本中的代码字、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟网址、用户输入和 Twitter 用户名。这里有一个例子：“基础级 VM 适用于`dev/test`环境，尽管它们的性能与标准级 VM 相似，但仍有一些限制。”

一段代码的格式如下所示：

```
{
  "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "name": {
```

所有的命令行输入或输出格式如下所示：

```
Connect-AzureRmAccount
```

**粗体**：表示新术语、重要词汇或在屏幕上看到的词汇。例如，菜单或对话框中的词汇在文本中会以这种方式出现。这里有一个例子：“要创建一个新的 Azure 虚拟机，我们需要选择‘新建资源’，然后选择‘新建虚拟机’。”

警告或重要事项以这种方式出现。

提示和技巧以这种方式出现。

# 联系我们

我们始终欢迎读者的反馈。

**一般反馈**：如果你对本书的任何部分有疑问，请在邮件主题中提及书名，并通过`customercare@packtpub.com`与我们联系。

**勘误表**：虽然我们已尽力确保内容的准确性，但错误是难免的。如果你在本书中发现了错误，请向我们报告。请访问[www.packt.com/submit-errata](http://www.packt.com/submit-errata)，选择你的书籍，点击勘误提交表单链接，并填写相关信息。

**盗版**：如果你在互联网上发现我们作品的任何非法复制版本，我们将非常感激你能提供该材料的地址或网站名称。请通过`copyright@packt.com`与我们联系，并提供该材料的链接。

**如果你有兴趣成为作者**：如果你在某个领域有专长，并且有兴趣撰写或为书籍做贡献，请访问[authors.packtpub.com](http://authors.packtpub.com/)。

# 书评

请留下评论。一旦你阅读并使用了本书，为什么不在你购买的站点上留下评论呢？潜在读者可以看到并参考你的公正意见来做出购买决策，我们 Packt 可以了解你对我们产品的看法，作者们也能看到你对他们书籍的反馈。谢谢！

如需了解更多关于 Packt 的信息，请访问[packt.com](http://www.packt.com/)。
