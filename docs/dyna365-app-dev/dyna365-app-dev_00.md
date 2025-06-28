# 前言

本书将向您介绍新的设计工具的组成部分，例如用于业务流程的 SiteMap、App 模块和 Visual Designer。深入学习后，您将了解如何利用 Dynamics 365 中 PowerApps 的功能开发自定义的 SaaS（软件即服务）应用程序。您将学习如何使用 Microsoft Flow 自动化业务流程，然后我们将探索 Web API，这是 Dynamics 365 CRM 中最重要的平台更新。您还将学习如何在自定义应用程序中实现 Web API，编写一个 Azure 感知插件，设计并集成云感知解决方案。最后，本书将介绍如何配置服务，使用新发布的功能，如可编辑网格、数据导出服务、LinkedIn 集成、关系洞察和实时辅助。

# 本书适用对象

本书面向有经验的开发人员，他们希望构建业务解决方案软件，并且对 Microsoft Dynamics 365 应用程序开发（尤其是 CRM 领域）不熟悉。

# 本书内容概述

第一章，*自定义应用程序导航*，探索了 Site Map 设计器，这是 Dynamics 365 CRM 中引入的一个新的基于 Web 的工具，允许自定义人员快速定义应用程序中的导航。之前，用户需要导出 SiteMap XML 并手动在 XML 编辑器中更新，或者使用某些第三方工具。内置的 Site Map 设计器使得编辑应用程序的站点地图变得更加容易*。*

第二章，*使用 App 模块设计器设计应用程序*，介绍了 App 模块设计器，它使得为用户添加特定应用程序组件变得更加容易。基本上，一个应用程序是相关实体、仪表板和业务流程流的集合，经过优化，使最终用户只能看到与他们相关的 Dynamics 365 CRM 组件。

第三章，*使用 Visual Process Designer 定义流程*，解释了 Visual Process Designer，它为 Dynamics 365 CRM 的业务流程流带来了拖放设计功能。Microsoft Dynamics 365 CRM 中的业务流程流工具旨在帮助引导用户通过系统中的业务流程。

第四章，*使用业务规则设计器定义业务规则*，带您了解业务规则，这是一种在 Dynamics 365 CRM 中引入的新界面。它经过了完整的用户界面改进，从逐步添加操作改进为拖放操作的方式。

第五章，*创建自定义业务应用程序*，解释了 PowerApps，它提供了模板来构建自定义的 SaaS（软件即服务）应用程序。Microsoft PowerApps 允许企业各层级的用户创建可用的移动应用程序。

第六章，*使用 Microsoft Flow 自动化业务流程*，将引导你创建你最喜欢的应用和服务之间的自动化工作流，从而减少工作量，提高效率。它是一个基于云的工具，普通用户无需开发人员的帮助即可轻松使用。自动化工作流被称为流（flows）。要创建一个流，用户指定在特定事件发生时应执行的操作。

第七章，*使用 Web API 开发应用程序*，涵盖了 WebAPI，它是 Dynamics 365 CRM 中最重要的平台更新之一。它替代了 OData 和最终的基于 SOAP 的服务。它基于 OAuth v2.0 和 Open Data Protocol (OData) v4.0 标准，这两种技术都已广泛应用，并且是平台无关的。因此，它可以从不同平台上的不同类型应用程序中进行调用。

第八章，*在 Dynamics 365 中利用 Azure 扩展*，解释了 Azure 扩展，它将消息请求数据发布到 Microsoft Azure Service Bus 上任何正在监听的应用程序。这为 CRM 与其他业务应用程序之间的集成提供了无限的可能性，无论这些应用程序是基于云还是本地部署。

第九章，*在应用程序中使用可编辑网格*，探讨了可编辑网格，它是 Microsoft Dynamics 365 CRM 中最受欢迎的功能之一。它提供了丰富的主网格和子网格（Web 和移动应用）的内联编辑功能，使得用户可以减少点击操作，直接进行操作，而无需导航到主记录。

第十章，*配置 Microsoft Cognitive Services 与 Dynamics 365*，解释了认知服务的配置，这使得人工智能能够被纳入并与 Dynamics 365 CRM 集成，特别是用于产品推荐和建议知识文章。推荐服务和文本分析服务连接可以在 Dynamics 365 CRM 内轻松配置。

第十一章，*通过学习路径培训用户*，介绍了学习路径，它允许用户编写自定义的应用内帮助体验，这些体验可能是特定于 CRM 解决方案的。它促进了 Dynamics 365 CRM 实施的学习和用户采纳。

第十二章，*Dynamics 365 中的其他新功能*，简要描述了 Dynamics 365 CRM 中一些在前面章节未涉及的其他新功能。

# 若要充分利用本书

本书假设读者对 Dynamics CRM 有一定基础知识，并希望学习 Dynamics 365 中引入的新特性。然而，未曾接触过之前版本并从零开始学习 Dynamics 365 的读者同样能从中受益。开发人员、自定义人员、管理员和高级用户都能通过学习 Dynamics 365 中引入的最新功能和变化提升他们的技能。

您可以通过 Dynamics 365 的试用实例（[`trials.dynamics.com/`](https://trials.dynamics.com/)）以及 Visual Studio 2017 的免费社区版（[`www.visualstudio.com/vs/community/`](https://www.visualstudio.com/vs/community/)）尝试本书中提到的所有功能和主题。部分内容不适用于 Dynamics 365 的本地版本。

# 下载示例代码文件

您可以通过您的帐户在 [www.packtpub.com](http://www.packtpub.com) 下载本书的示例代码文件。如果您在其他地方购买了本书，可以访问 [www.packtpub.com/support](http://www.packtpub.com/support) 并注册，以便直接将文件发送到您的邮箱。

您可以按照以下步骤下载代码文件：

1.  登录或注册 [www.packtpub.com](http://www.packtpub.com/support)。

1.  选择“支持”标签。

1.  点击“代码下载与勘误”。

1.  在搜索框中输入书名，并按照屏幕上的指示操作。

一旦文件下载完成，请确保使用最新版的工具解压或提取文件夹：

+   Windows 版的 WinRAR/7-Zip

+   Mac 版的 Zipeg/iZip/UnRarX

+   Linux 版的 7-Zip/PeaZip

本书的代码包也托管在 GitHub 上，网址为 [`github.com/PacktPublishing/Dynamics-365-Application-Development`](https://github.com/PacktPublishing/Dynamics-365-Application-Development)。我们还有来自丰富图书和视频目录的其他代码包，访问 **[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)**，快去看看！

# 下载彩色图像

我们还提供了一份 PDF 文件，包含本书中使用的带色彩的截图/图示。您可以在此下载： [`www.packtpub.com/sites/default/files/downloads/Dynamics365ApplicationDevelopment_ColorImages.pdf`](https://www.packtpub.com/sites/default/files/downloads/Dynamics365ApplicationDevelopment_ColorImages.pdf)。

# 使用的约定

本书中使用了一些文本约定。

`CodeInText`：表示文本中的代码词、数据库表名、文件夹名称、文件名、文件扩展名、路径名、虚拟网址、用户输入和 Twitter 句柄。例如：“监听应用程序需要实现 `IServiceEndpointPlugin` 接口的 `Execute` 方法，并使用 `WS2007HttpRelayBinding`，将 `RemoteExecutionContext` 从 Azure Service Bus 传递过去。”

代码块如下所示：

```
   Request: GET [Organization URI] /api/data/v9.0/contacts? 
    $select=firstname&$top=5
    Accept: application/json
    OData-MaxVersion: 4.0
    OData-Version: 4.0
```

**粗体**：表示新术语、重要单词或你在屏幕上看到的单词。例如，菜单或对话框中的词语会以这种方式出现在文本中。示例如下：“完成所有更改后，点击保存实体按钮：”

警告或重要说明如下所示。

提示和技巧如下所示。

# 联系我们

我们始终欢迎读者的反馈。

**一般反馈**：发送电子邮件至`feedback@packtpub.com`，并在邮件主题中提及书名。如果你有任何关于本书的疑问，请通过`questions@packtpub.com`与我们联系。

**勘误表**：虽然我们已尽一切努力确保内容的准确性，但错误仍然会发生。如果你在本书中发现错误，我们将非常感激你向我们报告。请访问[www.packtpub.com/submit-errata](http://www.packtpub.com/submit-errata)，选择你的书籍，点击“勘误提交表单”链接并填写相关详情。

**盗版**：如果你在互联网上发现任何我们作品的非法复制品，无论其形式如何，我们将非常感激你提供其位置地址或网站名称。请通过`copyright@packtpub.com`与我们联系，并附上该材料的链接。

**如果你有兴趣成为作者**：如果你在某个领域有专长，并且有兴趣写书或为书籍做贡献，请访问[authors.packtpub.com](http://authors.packtpub.com/)。

# 评论

请留下评论。当你阅读并使用完本书后，为什么不在购买本书的网站上留下评论呢？潜在读者可以参考并利用你的公正意见做出购买决策，我们 Packt 可以了解你对我们产品的看法，作者也能看到你对他们书籍的反馈。谢谢！

关于 Packt 的更多信息，请访问[packtpub.com](https://www.packtpub.com/)。
