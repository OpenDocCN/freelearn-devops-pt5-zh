# 序言

Salesforce，凭借其强大的功能和特点，简化了企业在多个领域（如销售、营销和财务）的运作。部署 Salesforce 应用程序是一项复杂的工作，对于管理员和顾问来说，可能会非常繁琐。 本书将帮助你为 Salesforce 实施 DevOps 并探索其功能。你将学习 Salesforce 企业运作中的 DevOps 原则和技术，并了解如何使用 Jenkins 和 Ant 脚本等工具实现持续集成和持续交付。你还将学习如何使用 Force.com 迁移工具和 Git 在 Salesforce 中实现版本控制。

# 本书适合谁阅读

如果你是一个 Salesforce 开发人员、顾问或经理，想要了解 DevOps 工具并为小型和大型 Salesforce 项目建立流水线，那么本书适合你。

# 本书涵盖的内容

第一章，*Salesforce 开发与交付过程*，概述了传统的 Salesforce 开发过程，包括使用的环境以及如何使用 Eclipse 和 Force.com IDE 设置环境。我们还将讨论沙箱和沙箱的类型。

第二章，*将 DevOps 应用于 Salesforce 应用程序*，讨论了 Salesforce 项目中 DevOps 的需求，以及在处理大型 Salesforce 项目的开发和部署时可能面临的挑战。

第三章，*Salesforce 中的部署*，展示了如何将 Salesforce 代码从一个沙箱部署到另一个沙箱，从一个沙箱部署到生产环境，及从一个组织部署到另一个组织。我们将了解不同类型的代码部署，并根据项目类型了解如何使用它们。

第四章，*Force.com 迁移工具介绍*，讨论了 Force.com 迁移工具以及如何在你的环境中设置该工具。我们还将看到如何使用 Ant 迁移工具将元数据部署到开发或测试沙箱。

第五章，*版本控制*，帮助你理解源代码版本控制系统及其类型。我们将主要关注分布式 Git 版本控制。我们还将学习如何在 Salesforce 项目中使用 Git，并将 Salesforce 元数据保存到 Git 中。

第六章，*持续集成*，展示了如何自动备份 Salesforce 元数据并使用 Jenkins 将代码推送到 Git 仓库。我们还将学习如何设置自己的 Jenkins 服务器，并配置它从 Salesforce 沙箱中检索元数据。

第七章，*持续测试*，讲述了代码质量和持续测试。我们将讨论自动化测试中使用的工具，如 Selenium 和 Qualitia。我们还会展示如何在一个示例 Salesforce 应用中使用 Selenium 进行记录和回放的测试用例。

第八章，*跟踪应用程序变化及将 DevOps 应用到 Salesforce 的投资回报率*，讨论了 Bugzilla 的基础知识以及如何在测试人员或用户报告问题时进行跟踪。我们还将学习如何提高生产力并衡量 ROI。

# 为了充分利用本书

要遵循本书中的指示，您需要一个安装了以下软件的 Windows 系统：

+   Java

+   Eclipse

+   Git

+   Jenkins

+   ANT

+   PMD

# 下载彩色图片

我们还提供了一份包含本书中截图/图表的彩色图片的 PDF 文件。您可以在这里下载：[`www.packtpub.com/sites/default/files/downloads/9781788833349_ColorImages.pdf`](https://www.packtpub.com/sites/default/files/downloads/9781788833349_ColorImages.pdf)。

# 使用的约定

本书中使用了多种文本约定。

`CodeInText`：表示文本中的代码词汇、数据库表名、文件夹名称、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 账号名。以下是一个例子：“在命令面板中输入 `apex stat`。”

一段代码设置如下：

```
<?xml version="1.0" encoding="UTF-8"?>
<Package >
    <version>42.0</version>
</Package> 
```

当我们希望您关注代码块的特定部分时，相关的行或项目会以粗体显示：

```
<?xml version="1.0" encoding="UTF-8"?>
<Package >
    <version>42.0</version>
</Package> 
```

所有命令行输入或输出如下所示：

```
$pmd -d "Source Path" -R apex-ruleset -language apex -f CSV > "Destination Ptah\ReportName.csv"
```

**粗体**：表示一个新术语、重要的词汇或您在屏幕上看到的文字。例如，菜单或对话框中的文字会以这种方式显示。以下是一个例子：“输入项目名称和组织设置详情以建立连接。”

警告或重要提示如下所示。

提示和技巧如下所示。

# 联系我们

我们非常欢迎读者的反馈。

**一般反馈**：通过电子邮件 `customercare@packtpub.com` 并在邮件主题中注明书名。如果您对本书的任何内容有疑问，请通过电子邮件与我们联系，地址为 `customercare@packtpub.com`。

**勘误**：尽管我们已经尽一切努力确保内容的准确性，但错误还是会发生。如果您在本书中发现错误，我们将非常感谢您向我们报告。请访问 [www.packt.com/submit-errata](http://packt.com/submit-errata)，选择您的书籍，点击“勘误提交表单”链接，并输入详细信息。

**盗版**：如果您在互联网上发现任何非法复制的作品，我们将非常感谢您提供该作品的地址或网站名称。请通过 `copyright@packt.com` 与我们联系，并提供该资料的链接。

**如果你有兴趣成为作者**：如果你在某个领域有专业知识，并且有兴趣编写或贡献一本书，请访问 [authors.packtpub.com](http://authors.packtpub.com/)。

# 评价

请留下评价。阅读并使用本书后，为什么不在你购买该书的网站上留下评价呢？潜在读者可以看到并参考你的公正意见来做出购买决策，我们在 Packt 可以了解你对我们产品的看法，而我们的作者也能看到你对他们书籍的反馈。谢谢！

欲了解更多关于 Packt 的信息，请访问 [packt.com](http://packt.com)。
