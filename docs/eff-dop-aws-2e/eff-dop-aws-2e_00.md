# 前言

DevOps 运动已经改变了现代科技公司工作的方式。Amazon Web Services（AWS）一直处于云计算革命的前沿，也是 DevOps 运动的关键推动者，创造了大量的托管服务，帮助你实施 DevOps 原则。

本书将帮助你了解最成功的科技初创公司如何在 AWS 上启动和扩展他们的服务，并学习如何做到这一点。本书解释了如何将基础设施视为代码，这意味着你可以像控制软件一样轻松地将资源上线和下线。你还将构建一个持续集成和持续部署的管道，以保持你的应用程序始终最新。

一旦你掌握了这些，你将学习如何扩展应用程序，以便在流量激增时也能为用户提供最佳性能，利用最新的技术，如容器。此外，你还将深入了解监控和告警，确保用户在使用你的服务时拥有最佳体验。在最后的章节中，你将介绍 AWS 内置工具，如 CodeDeploy 和 CloudFormation，这些工具被许多 AWS 管理员用于执行 DevOps。

到本书结尾时，你将学会如何使用最新和最重要的 AWS 工具来确保平台和数据的安全。

# 本书适合谁阅读

如果你是开发人员、DevOps 工程师，或者你在一个希望构建并使用 AWS 进行软件基础设施的团队中工作，那么这本书适合你。为了充分理解本书内容，需要具备基本的计算机科学知识。

# 本书涵盖内容

第一章，*云与 DevOps 革命*，本章将为任何人打下 DevOps 和云计算旅程的基础。深入理解 DevOps 文化、DevOps 术语和 AWS 生态系统，为未来章节的路线图做准备。

第二章，*部署你的第一个 Web 应用程序*，本章将展示最简单形式的 AWS 基础设施配置，并介绍一些最佳的 AWS 身份验证实践。我们将创建一个简单的 Web 应用程序，了解如何在 AWS 上托管该应用程序的最简单方式，最后将终止实例。整个过程将通过 AWS CLI 工具实现，并将在后续章节中进行自动化，帮助理解如何使用不同的 AWS 和其他著名服务与产品自动化手动任务。

第三章，*将基础设施视为代码*，本章将重点介绍使用 AWS 原生工具 CloudFormation 进行自动化资源配置，以及创建 CloudFormation 模板的技巧。然后，我们将介绍一个配置管理系统，使用 Ansible 自动化应用部署。

第四章，*使用 Terraform 进行基础设施即代码*，本章将重点介绍 Terraform 的基本原理。我们将使用 Terraform 模板为 AWS 实例配置资源，然后扩展 Terraform 的功能，通过另一个 Terraform 模板来部署应用程序。最后，我们将通过将 Terraform 与 Ansible 自动化结合来理解 AWS 的资源配置。

第五章，*增加持续集成和持续部署*，本章将重点介绍如何使用 AWS DevOps 服务和自动化测试框架构建 CI/CD 管道。我们将使用多个工具来准备技术框架，如版本控制、持续集成、自动化测试工具、AWS 原生 DevOps 工具以及基础设施自动化工具，以帮助我们理解如何通过“快速失败”和“频繁失败”来构建一个健壮的生产环境。

第六章，*扩展您的基础设施*，本章将介绍其他有用的、具有成本效益的 AWS 服务，用于构建具有性能导向的可扩展 AWS 基础设施。AWS 服务，如 Elastic Cache、CloudFront、SQS、Kinesis 等，将用于构建我们的应用框架。

第七章，*在 AWS 中运行容器*，本章将介绍市场上最为小众的技术之一——Docker。我们将从理解容器开始，学习使用 Docker 的所有基础概念。在这里，我们将使用 ECS 构建 AWS 容器环境，并为我们的应用构建一个完整的 ECS 框架。最后，我们将使用 AWS DevOps 工具集构建一个完整的 CI/CD 管道来部署 AWS ECS 服务。

第八章，*强化 AWS 环境的安全性*，本章将重点介绍如何使用 AWS 审计服务、AWS IAM 服务来管理并根据角色提供有限访问权限、加强 AWS VPC 模型，最后保护环境免受勒索软件和其他漏洞的攻击。

第九章，*监控与警报*，本章将重点介绍如何使用 AWS CloudWatch 服务为您的 AWS 环境构建监控框架。我们将使用一些著名的仪表盘工具来可视化日志。最后，将使用 AWS SNS 服务创建通知框架，以便在 AWS 环境出现问题时通知用户并采取纠正措施。有关此章节的详细信息，请参考[`www.packtpub.com/sites/default/files/downloads/Monitoring_and_Alerting.pdf`](https://www.packtpub.com/sites/default/files/downloads/Monitoring_and_Alerting.pdf)。

# 如何充分利用本书

本书所需的软件如下：

+   AWS 管理控制台

+   AWS 计算服务

+   AWS IAM

+   AWS CLI 设置

+   用于 Web 应用程序的 JavaScript

# 下载示例代码文件

你可以从你的账户在 [www.packt.com](http://www.packt.com) 下载本书的示例代码文件。如果你是从其他地方购买的这本书，你可以访问 [www.packt.com/support](http://www.packt.com/support) 并注册，文件会直接通过邮件发送给你。

你可以按照以下步骤下载代码文件：

1.  登录或注册于 [www.packt.com](http://www.packt.com)。

1.  选择“SUPPORT”标签。

1.  点击“Code Downloads & Errata”。

1.  在搜索框中输入书名，并按照屏幕上的指示操作。

下载文件后，请确保使用以下最新版本的工具解压或提取文件夹：

+   WinRAR/7-Zip for Windows

+   Zipeg/iZip/UnRarX for Mac

+   7-Zip/PeaZip for Linux

本书的代码包也托管在 GitHub 上，地址为 [`github.com/PacktPublishing/Effective-DevOps-with-AWS-Second-Edition`](https://github.com/PacktPublishing/Effective-DevOps-with-AWS-Second-Edition)。如果代码有更新，将会在现有的 GitHub 仓库中更新。

我们还提供了来自我们丰富图书和视频目录中的其他代码包，地址为 **[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)**。快来看看吧！

这些代码也可以在以下仓库中找到：

+   [`github.com/yogeshraheja/Effective-DevOps-with-AWS`](https://github.com/yogeshraheja/Effective-DevOps-with-AWS)

+   [`github.com/giuseppeborgese/effective_devops_with_aws__second_edition`](https://github.com/giuseppeborgese/effective_devops_with_aws__second_edition)

# 下载彩色图像

我们还提供了一份 PDF 文件，里面有本书中使用的截图/图表的彩色图像。你可以在这里下载：[`www.packtpub.com/sites/default/files/downloads/9781789539974_ColorImages.pdf`](https://www.packtpub.com/sites/default/files/downloads/9781789539974_ColorImages.pdf)。

# 使用的约定

本书中使用了若干文本约定。

`CodeInText`：表示文本中的代码词汇、数据库表名、文件夹名称、文件名、文件扩展名、路径名、虚拟网址、用户输入以及 Twitter 用户名。以下是一个示例：“点击 start 按钮并搜索 `settings` 选项。”

代码块的设置方式如下：

```
var http = require("http") http.createServer(function (request, response) {
// Send the HTTP header
// HTTP Status: 200 : OK
// Content Type: text/plain
response.writeHead(200, {'Content-Type': 'text/plain'})
// Send the response body as "Hello World" response.end('Hello World\n')
}).listen(3000)

// Console will print the message console.log('Server running')
```

当我们希望引起你对代码块中特定部分的注意时，相关的行或项目会以粗体显示：

```
$ aws ec2 describe-instance-status --instance-ids i-057e8deb1a4c3f35d --output text| grep -i SystemStatus
SYSTEMSTATUS ok
```

任何命令行输入或输出会按照以下方式书写：

```
$ aws ec2 authorize-security-group-ingress \
   --group-name HelloWorld \
   --protocol tcp \
   --port 3000 \
   --cidr 0.0.0.0/0
```

**粗体**：表示一个新术语、重要单词或在屏幕上看到的单词。例如，菜单或对话框中的单词会以这种形式出现在文本中。以下是一个示例：“在此菜单中，找到名为 Windows Subsystem for Linux (Beta) 的功能。”

警告或重要说明如下所示。

提示和技巧如下所示。

# 获取联系

我们欢迎读者的反馈。

**一般反馈**：如果你对本书的任何方面有疑问，请在邮件主题中提到书名，并通过`customercare@packtpub.com`联系我们。

**勘误**：尽管我们已尽力确保内容的准确性，但错误仍然可能发生。如果您在本书中发现任何错误，我们将非常感激您向我们报告。请访问 [www.packt.com/submit-errata](http://www.packt.com/submit-errata)，选择您的书籍，点击“勘误提交表单”链接，并填写相关信息。

**盗版**：如果您在互联网上发现我们作品的任何非法复制品，我们将非常感激您提供位置地址或网站名称。请通过 `copyright@packt.com` 联系我们并提供相关链接。

**如果您有兴趣成为作者**：如果您在某个领域拥有专长并且有兴趣撰写或参与编写一本书，请访问 [authors.packtpub.com](http://authors.packtpub.com/)。

# 评价

请留下评价。阅读并使用本书后，为什么不在您购买本书的网站上留下评价呢？潜在读者可以查看并参考您的公正意见做出购买决策，我们 Packt 也能了解您对我们产品的看法，作者也能看到您对其书籍的反馈。感谢您！

有关 Packt 的更多信息，请访问 [packt.com](http://www.packt.com/)。
