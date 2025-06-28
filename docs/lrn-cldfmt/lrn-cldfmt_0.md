# 前言

随着亚马逊 Web 服务（AWS）基础设施逐步向云端迁移，高效管理与云相关的任务仍然是系统管理员面临的一大挑战。CloudFormation 是一种为高效管理 AWS 上的基础设施相关服务而开发的语言，其特点有助于确保 AWS 资源部署过程的安全性。*学习 CloudFormation* 作为一本基础指南，将开启你与 CloudFormation 的学习之旅。我们将向你介绍基础设施即代码（IaC）和实现自动化与基础设施管理所需的 AWS 服务的基本概念。接着，我们将深入探讨 CloudFormation 映射、条件、限制、输出和 EC2 等概念。在最后几章中，你将使用 CloudFormation 模板管理整个 AWS 基础设施。

本书结束时，你将能够使用 CloudFormation 实现基础设施即代码（IaC）。

# 本书适用对象

*学习 CloudFormation* 适用于云工程师、系统管理员、云架构师或任何从事云开发或云管理领域的利益相关者。需要具备 AWS 的基础知识。

# 本书内容

第一章，*介绍 AWS CloudFormation*，将介绍 AWS CloudFormation，并解释如何开始使用 AWS CloudFormation。

第二章，*构建你的第一个 AWS CloudFormation 项目*，将解释如何开始构建第一个 AWS CloudFormation 项目。该项目将通过一步步的实践方法来实施。

第三章，*开发 AWS CloudFormation 模板*，将探讨如何开发 AWS CloudFormation 模板。读者将学习如何使用 JSON 和 YAML 创建 AWS CloudFormation 模板。还将介绍如何使用 AWS CloudFormation Designer 在 GUI 模式下开发 AWS CloudFormation 模板。

第四章，*AWS CloudFormation StackSets*，将探讨 AWS CloudFormation StackSets 的基础知识。在本章结束时，读者将了解如何构建和管理 AWS CloudFormation StackSets。

第五章，*使用 AWS CloudFormation 构建 Lambda 函数*，将带领读者了解如何使用 AWS CloudFormation 部署 Lambda 函数的过程。

第六章，*AWS CloudFormation 安全性*，将讨论如何保护使用 AWS CloudFormation 部署的资源。

*第七章*，*AWS CloudFormation 的生产基础设施管理与测试*，将解释如何使用 AWS CloudFormation 管理和部署测试及生产基础设施。本章不在书中，但可以通过以下链接下载：[`www.packtpub.com/sites/default/files/downloads/Managing_and_Testing_Production_Infrastructure_for_AWS_CloudFormation.pdf`](https://www.packtpub.com/sites/default/files/downloads/Managing_and_Testing_Production_Infrastructure_for_AWS_CloudFormation.pdf)。

# 最大化本书的使用效果

由于实际示例涉及使用 AWS，因此需要拥有一个 AWS 账户。

# 下载示例代码文件

你可以从你在 [www.packtpub.com](http://www.packtpub.com) 的账户下载本书的示例代码文件。如果你在其他地方购买了本书，你可以访问 [www.packtpub.com/support](http://www.packtpub.com/support) 并注册，将文件直接发送到你的邮箱。

你可以通过以下步骤下载代码文件：

1.  登录或注册 [www.packtpub.com](http://www.packtpub.com/support)。

1.  选择 SUPPORT 标签。

1.  点击代码下载与勘误。

1.  在搜索框中输入书名，并按照屏幕上的指示操作。

文件下载后，请确保使用最新版本的工具解压或提取文件夹：

+   Windows 的 WinRAR/7-Zip

+   Mac 的 Zipeg/iZip/UnRarX

+   Linux 的 7-Zip/PeaZip

本书的代码包也托管在 GitHub 上，网址为 [`github.com/PacktPublishing/Learn-CloudFormation`](https://github.com/PacktPublishing/Learn-CloudFormation)。如果代码有更新，它将在现有的 GitHub 仓库中进行更新。

我们还有其他来自我们丰富的书籍和视频目录中的代码包可供下载，网址为 **[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)**。快去看看吧！

# 下载彩色图像

我们还提供了一份 PDF 文件，包含了本书中使用的截图/图表的彩色图像。你可以在这里下载：[`www.packtpub.com/sites/default/files/downloads/LearnCloudFormation_ColorImages.pdf`](https://www.packtpub.com/sites/default/files/downloads/LearnCloudFormation_ColorImages.pdf)。

# 使用的约定

本书中使用了若干文本约定。

`CodeInText`：表示文本中的代码词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 用户名。例如：“我们可以使用 CloudFormation 模板中的 `Parameters` 属性。”

代码块的设置方式如下：

```
exports.handler = (event, context, callback) => {
    var data = event['msg'];
    callback(null, 'Received: ' + data);
};
```

当我们希望引起你注意代码块的特定部分时，相关行或项目会以粗体显示：

```
{
 "Type" : "AWS::S3::Bucket",
 "Properties" : {
 "AccessControl" : String,
 "AccelerateConfiguration" : AccelerateConfiguration,
 "AnalyticsConfigurations" : [ AnalyticsConfiguration, ... ],
 "BucketEncryption" : BucketEncryption,
 "BucketName" : String,
}
```

任何命令行输入或输出均按如下方式书写：

```
$ aws cloudformation create-stack --stack-name my-simple-stack 
 --template-body file://home/user/templates/simple-s3.json 
 --parameters YourBucketName=my-simple-s3
```

**粗体**：表示一个新术语、一个重要的词或你在屏幕上看到的词。例如，菜单或对话框中的词汇将在文本中这样呈现。以下是一个示例：“在管理控制台中，点击左侧菜单中的 Roles。”

警告或重要提示会像这样出现。

小贴士和技巧会像这样出现。

# 联系我们

我们总是欢迎读者的反馈。

**一般反馈**：请通过电子邮件`feedback@packtpub.com`联系我们，并在邮件主题中注明书名。如果你对本书的任何方面有疑问，请通过`questions@packtpub.com`与我们联系。

**勘误**：虽然我们已经尽最大努力确保内容的准确性，但错误仍然可能发生。如果你在本书中发现了错误，我们将非常感激你报告给我们。请访问[www.packtpub.com/submit-errata](http://www.packtpub.com/submit-errata)，选择你的书籍，点击勘误提交表单链接，并输入详细信息。

**盗版**：如果你在互联网上遇到任何形式的我们作品的非法复制，我们将非常感激你提供该位置地址或网站名称。请通过`copyright@packtpub.com`与我们联系，并提供相关材料的链接。

**如果你有兴趣成为作者**：如果你在某个领域有专业知识，并且有兴趣撰写或参与书籍的编写，请访问[authors.packtpub.com](http://authors.packtpub.com/)。

# 评价

请留下评价。当你阅读并使用了这本书后，为什么不在你购买书籍的网站上留下评价呢？潜在的读者可以通过你的公正意见做出购买决定，我们 Packt 团队也能了解你对我们产品的看法，而我们的作者也能看到你对他们书籍的反馈。谢谢！

有关 Packt 的更多信息，请访问[packtpub.com](https://www.packtpub.com/)。
