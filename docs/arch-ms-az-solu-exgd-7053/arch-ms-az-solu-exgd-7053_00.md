# 前言

Azure 是一个不断发展的平台，提供适用于不同行业需求的最前沿技术环境。随着新功能和特性的快速推出，保持最新状态变得越来越困难。本书将为您提供 Azure 当前所有功能和能力的完整概述，并且是准备 70-535 考试的完整指南。

本书将涵盖所有考试目标。它将从设计计算基础设施开始，您将学习如何使用虚拟机、Web 应用程序、无服务器和微服务、高性能计算（HPC）以及其他计算密集型应用程序设计解决方案。您将学习如何使用 Azure 虚拟网络设计有效的网络实现，以及如何为混合应用程序设计连接性。您将学习如何使用不同的数据服务、关系数据库存储和 NoSQL 存储设计数据实现。您还将学习如何保持您的解决方案和应用程序的安全性，以及如何在 Azure 中使用不同平台服务，如物联网和人工智能平台与服务。最后，您将了解 Azure 提供的所有监控功能和解决方案。

每一章结束时都会有一个*进一步阅读*部分，这是本书的一个非常重要的部分，因为它将为您提供通过 70-535 考试所需的额外，有时甚至是至关重要的信息。随着考试问题会随时间稍作调整，本书很快会变得过时，进一步阅读部分将为您提供所有更新。

# 本书适合谁阅读

本书面向那些希望通过 70-535 考试：架构微软 Azure 解决方案，并从架构角度拓宽其 Azure 知识的有经验的开发人员和架构师。

# 最大限度地利用本书

本书假设读者已经熟悉网络、安全、数据库、集成、开发和管理应用程序，以及在 Azure 平台上解决方案的基础知识。

# 下载示例代码文件

您可以从您的账户下载本书的示例代码文件，网址是[www.packtpub.com](http://www.packtpub.com)。如果您在其他地方购买了本书，可以访问[www.packtpub.com/support](http://www.packtpub.com/support)并注册，代码文件将直接通过电子邮件发送给您。

您可以通过以下步骤下载代码文件：

1.  登录或注册：[www.packtpub.com](http://www.packtpub.com/support)。

1.  选择 SUPPORT 选项卡。

1.  点击代码下载和勘误表。

1.  在搜索框中输入书名，并按照屏幕上的说明操作。

文件下载后，请确保使用以下最新版本解压或提取文件夹：

+   WinRAR/7-Zip for Windows

+   Zipeg/iZip/UnRarX for Mac

+   7-Zip/PeaZip for Linux

本书的代码包也托管在 GitHub 上，地址为：[`github.com/PacktPublishing/Architecting-Microsoft-Azure-Solutions-Exam-Guide-70-535`](https://github.com/PacktPublishing/Architecting-Microsoft-Azure-Solutions-Exam-Guide-70-535)。如果代码有更新，将会更新到现有的 GitHub 仓库。

我们的丰富书籍和视频目录中还有其他代码包可供下载，您可以访问**[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)**查看！

# 下载彩色图片

我们还提供了一份 PDF 文件，其中包含本书中使用的屏幕截图/图表的彩色图片。您可以在此下载：[`www.packtpub.com/sites/default/files/downloads/ArchitectingMicrosoftAzureSolutionsExamGuide70535_ColorImages.pdf`](https://www.packtpub.com/sites/default/files/downloads/ArchitectingMicrosoftAzureSolutionsExamGuide70535_ColorImages.pdf)。

# 使用的约定

本书中使用了许多文本约定。

`CodeInText`：表示文本中的代码词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 用户名。以下是一个例子：“将下载的`WebStorm-10*.dmg`磁盘映像文件挂载为系统中的另一个磁盘。”

代码块设置如下：

```
namespace PacktPubToDoAPI.Models
{
    public class TodoItem
    {
        public long Id { get; set; }
        public string Name { get; set; }
        public bool IsComplete { get; set; }
    }
} 
```

任何命令行输入或输出都按如下格式书写：

```
Login-AzureRmAccount
```

**粗体**：表示一个新术语、重要的单词，或是您在屏幕上看到的单词。例如，菜单或对话框中的单词会以这种方式显示在文本中。以下是一个例子：“点击 New，在右侧选择一个图像（或者您可以输入一个

搜索框中的图像名称）。

警告或重要提示以这种方式呈现。

提示和技巧以这种方式呈现。

# 联系我们

我们始终欢迎读者的反馈。

**一般反馈**：请通过电子邮件`feedback@packtpub.com`联系我们，并在邮件主题中注明书名。如果您对本书的任何方面有疑问，请通过电子邮件`questions@packtpub.com`与我们联系。

**勘误表**：尽管我们已经尽力确保内容的准确性，但错误仍然可能发生。如果您发现本书中的错误，我们将非常感谢您向我们报告。请访问 [www.packtpub.com/submit-errata](http://www.packtpub.com/submit-errata)，选择您的书籍，点击“勘误提交表格”链接并填写详细信息。

**盗版**：如果您在互联网上遇到我们作品的任何非法复制版本，我们将非常感谢您提供位置地址或网站名称。请通过`copyright@packtpub.com`与我们联系，并附上链接。

**如果您有兴趣成为作者**：如果您在某个主题方面拥有专业知识，并且有兴趣撰写或为书籍做出贡献，请访问 [authors.packtpub.com](http://authors.packtpub.com/)。

# 评价

请留下评论。当您阅读并使用完本书后，为什么不在您购买本书的网站上留下评论呢？潜在读者可以通过您的公正意见做出购买决策，我们在 Packt 也能了解您对我们产品的看法，而我们的作者则能看到您对其书籍的反馈。谢谢！

欲了解更多有关 Packt 的信息，请访问 [packtpub.com](https://www.packtpub.com/)。
