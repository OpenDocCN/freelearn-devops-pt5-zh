# 前言

移动 DevOps 是移动应用开发中持续集成和持续交付的未来，也是当今快节奏开发文化中的一个非常重要的需求。尽管 DevOps 已被大多数快速发展的开发团队实施并采纳，但移动 DevOps 仍未被大多数移动开发领域广泛使用。它能够改善集成和交付，并提供更强大的反馈机制和早期缺陷捕捉工具。

移动 DevOps 具有自身的实施挑战，而随着数百万设备上存在各种移动平台，以及不同的屏幕比例，使用能够简化物理设备测试和交付给客户的工具变得越来越重要，同时为开发者提供快速反馈机制。

在本书中，我们将使用 Xamarin 探索移动应用开发的基础知识和工具。Xamarin 是微软推出的一个跨平台移动应用开发框架，可以使用共享的代码库和设计来创建 iOS、Android 和 Windows 应用程序。除了 Xamarin，我们还将使用微软工具集中的其他工具，如 Xamarin Test Cloud 和 Visual Studio Team Services，深入研究移动 DevOps 的不同阶段。

使用 Xamarin 的主要动机是它能够开发跨平台应用程序，并且与微软的其他广泛使用的工具在应用程序开发周期的不同阶段具有极好的集成性。

本书结束时，你不仅应该熟悉移动 DevOps 和移动应用开发，而且应该能够在你的新应用或现有的移动应用项目中，使用现有的流行工具，实施、配置和排除移动 DevOps 生命周期中的每一个步骤。

# 本书读者对象

本书主要面向移动应用开发者、DevOps 工程师和愿意将 DevOps 应用于其移动应用开发生命周期的小团队。已经使用 Visual Studio 和/或 C# 作为编程语言的开发者，鼓励使用本书开始跨平台移动应用开发，并理解持续交付和持续集成的工作原理。

如果你已经是 Xamarin 开发者，那么本书将帮助你通过在项目中实施移动 DevOps 来简化快速开发、持续测试和频繁交付管理过程。

# 本书内容

第一章，*简介*，将带你进入 DevOps 和移动 DevOps 的世界，并解释它们之间的差异。本章还将描述在将 DevOps 应用于移动开发时，你可能会遇到的各种挑战。

第二章，*使用代码仓库系统*，探讨了代码仓库系统，并讨论了各种版本控制工具。本章主要关注 Git，深入探讨了源代码版本控制的细节。

第三章，*使用 Xamarin 进行跨平台移动应用开发*，介绍了 Xamarin 和跨平台移动应用开发。本章还解释了在 Windows 机器上设置 Xamarin 和 Visual Studio 的步骤。

第四章，*使用 Xamarin 编写您的第一个 Android 应用程序*，解释了 Android 应用程序的基本概念。内容还描述了使用 Xamarin 创建 Android 应用程序项目的步骤，并讨论了如何在移动设备上部署应用程序的 UI。

第五章，*使用 Xamarin 实现自动化测试*，讨论了在 DevOps 循环中自动化测试的重要性，并深入讲解了为 Xamarin.Android 应用项目编写自动化测试用例的方法。此外，您还将学习如何设置 Xamarin Test Cloud 并在其上运行 Android 应用的自动化测试。

第六章，*使用 Xamarin 配置 TeamCity 进行 CI/CD*，介绍了持续集成的概念，并讨论了可用于持续集成的各种工具。本章让您深入了解如何使用 TeamCity 进行持续集成，详细解释了使用 TeamCity 作为 CI 工具时涉及的各种配置和设置步骤。

第七章，*使用 Visual Studio Team Services 进行 Android 的 CI/CD*，讲解了如何使用 Visual Studio Team Service 实现持续集成和持续交付。内容涵盖了从在 Visual Studio 中创建帐户到配置和排队构建的连续构建过程的各个步骤。

第八章，*在 AWS 上部署应用程序*，描述了如何将应用程序部署和迁移到云端。本章详细解释了从创建实例到创建 ELB 并使用 Terraform、AWS CLI 和 LightSail 配置终节点的各种步骤。

第九章，*监控和优化应用程序*，带您深入了解不同级别的监控，从 API 级别监控开始，逐步介绍如何使用 Android 监控工具进行监控。内容还包括针对 Test Cloud 的监控步骤。

第十章，*调试应用程序*，解释了在 Xamarin 中排查故障是一个常见问题，并涵盖了各种部署生命周期。内容包括调试 Xamarin 代码、故障排除 Android 模拟器、调试 Mono 类库，最后是调试 Git 连接。

第十一章，*案例研究*，详细讲解了移动 DevOps 的整个过程，从移动应用开发和集成到通过两个案例研究进行的持续测试和部署。

# 为了最大限度地利用本书

本书假设你具备中级的 Windows 操作系统知识，具备云计算和应用开发生命周期的基本知识，同时具备面向对象编程语言（如 Java 或 C#）的初学者级别知识。本书将通过移动 DevOps 生命周期的各个阶段，讲解应用开发基础和应用交付的基本概念。如果你有使用 Visual Studio 的经验，并且熟悉 C#编程，那将是一个很大的优势。

安装 Visual Studio 和 Xamarin 的最低要求如下：

+   Windows 要求：Windows 7

+   Android 6.0/API 级别 23

以下是 Android 模拟器的硬件要求：

+   支持 Hyper-V

+   4GB 或更高内存

+   64 位版本的 Windows 操作系统

请注意，由于没有硬件加速时，Android SDK 模拟器的速度非常慢，因此推荐使用英特尔的硬件加速执行管理器（HAXM）来大幅提升 Android 模拟器的性能。

安装必要的 Visual Studio 和 Xamarin.Android 包、Git，并连接到 Xamarin Test Cloud 时，需要联网。

# 下载示例代码文件

你可以通过你的账户在 [www.packtpub.com](http://www.packtpub.com) 下载本书的示例代码文件。如果你是在其他地方购买的本书，可以访问 [www.packtpub.com/support](http://www.packtpub.com/support)，注册后我们会将文件直接发送到你的邮箱。

你可以按照以下步骤下载代码文件：

1.  请登录或注册 [www.packtpub.com](http://www.packtpub.com/support)。

1.  选择“支持”标签。

1.  点击“代码下载与勘误”。

1.  在搜索框中输入书名，并按照屏幕上的指示操作。

下载文件后，请确保使用以下最新版本的工具解压或提取文件夹：

+   WinRAR/7-Zip for Windows

+   Zipeg/iZip/UnRarX for Mac

+   7-Zip/PeaZip for Linux

本书的代码包也托管在 GitHub 上，地址为 [`github.com/PacktPublishing/Mobile-DevOps`](https://github.com/PacktPublishing/Mobile-DevOps)。如果代码有更新，它将在现有的 GitHub 仓库中更新。

我们还提供了其他代码包，这些代码包来自我们丰富的图书和视频目录，可以在**[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)**找到。快来看看吧！

# 下载彩色图像

我们还提供了一个 PDF 文件，包含本书中使用的截图/图表的彩色图像。你可以在这里下载：[`www.packtpub.com/sites/default/files/downloads/MobileDevOps_ColorImages.pdf`](https://www.packtpub.com/sites/default/files/downloads/MobileDevOps_ColorImages.pdf)。

# 使用的约定

本书中使用了多种文本约定。

`CodeInText`：表示文本中的代码词语、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟网址、用户输入和 Twitter 账号。例如：“`MainActivity.cs`文件包含了我们用于处理事件和其他功能的 C#代码，位于主屏幕中。”

一段代码如下所示：

```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Action": "ec2:*",
      "Effect": "Allow",
      "Resource": "*"
    },
```

任何命令行输入或输出都按如下方式书写：

```
$ mkdir terraform
$ cd terraform
$ terraform workspace new MyTestMachine
```

**粗体**：表示一个新术语、一个重要的词语或您在屏幕上看到的词语。例如，菜单或对话框中的词语会以这样的方式显示在文本中。以下是一个例子：“点击 Visual Studio Community 2017 下提供的免费下载。”

警告或重要提示会这样出现。

小贴士和技巧会这样出现。

# 联系我们

我们始终欢迎读者的反馈。

**一般反馈**：请发送电子邮件到`feedback@packtpub.com`，并在邮件主题中注明书名。如果您对本书的任何部分有问题，请通过`questions@packtpub.com`联系我们。

**勘误**：虽然我们已尽最大努力确保内容的准确性，但难免会出现错误。如果您在本书中发现了错误，我们将不胜感激您向我们报告。请访问[www.packtpub.com/submit-errata](http://www.packtpub.com/submit-errata)，选择您的书籍，点击“勘误提交表单”链接，并输入相关信息。

**盗版**：如果您在互联网上发现我们作品的任何非法版本，我们将非常感激您提供相关的网址或网站名称。请通过`copyright@packtpub.com`联系我们，并附上相关材料的链接。

**如果你有兴趣成为作者**：如果您在某个领域有专长并且有意写书或为书籍做贡献，请访问[authors.packtpub.com](http://authors.packtpub.com/)。

# 评价

请留下您的评价。在阅读并使用本书后，为什么不在您购买它的网站上留下评价呢？潜在的读者可以查看并参考您的公正意见来做出购买决策，我们在 Packt 可以了解您对我们产品的看法，而我们的作者也可以看到您对他们书籍的反馈。谢谢！

欲了解更多有关 Packt 的信息，请访问[packtpub.com](https://www.packtpub.com/)。
