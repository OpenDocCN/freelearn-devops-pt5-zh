# 序言

**DevOps**和**AWS**是近年来在科技行业中逐渐增长的两个关键主题，且有充分的理由。

DevOps 正在逐渐成为*事实上的*方法论或框架，并被各类规模的组织采纳。它使技术团队能够更高效地工作，并通过加速开发人员与最终用户之间的反馈循环，使工作更加有意义。团队成员通过增强的协作，享有更愉快、更高效的工作环境。

本书将首先探讨*DevOps*背后的哲学，然后继续通过一些实际示例介绍其最流行的原则。

*AWS*如今几乎成为了*云计算*的代名词，凭借其 31%的市场份额位居行业榜首。从 2006 年开始，*Amazon Web Services*已经发展成一个庞大、独立、复杂的*云*生态系统。它以惊人的速度推出新服务。*AWS*的产品类别从原始计算和数据库资源到存储、分析、*AI*、游戏开发、移动服务和*物联网*（*IoT*）解决方案等应有尽有。

我们将使用*AWS*作为平台应用*DevOps*技术。在接下来的章节中，我们将看到*AWS*的便利性和弹性如何极大地补充了*DevOps*在系统管理和应用开发中的创新方法。

# 本书内容

第一章，*什么是 DevOps，您需要关心吗？*，介绍了*DevOps*哲学。

第二章，*开始将您的基础设施视为代码*，提供了如何使用*Terraform*或*CloudFormation*将基础设施作为代码进行部署的示例。

第三章，*将您的基础设施纳入配置管理*，展示了如何使用*SaltStack*配置*EC2*实例。

第四章，*通过持续集成加速构建、测试和发布*，描述了如何使用*Jenkins CI*服务器设置*CI*工作流的过程。

第五章，*随时准备通过持续交付进行部署*，展示了如何扩展*CI*管道，以使用*Packer*和*Serverspec*生成可部署的*EC2 AMI*。

第六章，*持续部署 - 完全自动化工作流*，提供了一个完全自动化的工作流，并通过添加*AMI*部署所需的功能来完成*CI/CD*管道。

第七章，*度量、日志收集和监控*，介绍了*Prometheus*、*Logstash*、*Elasticsearch*及相关的*DevOps*工具。

第八章, *优化规模与成本*，提供了如何在规划*AWS*部署时考虑可扩展性和成本效率的建议。

第九章, *保护你的 AWS 环境*，介绍了提高*AWS*部署安全性的最佳实践。

第十章, *AWS 技巧与窍门*，包含了一些适用于从初学者到中级*AWS*用户的实用技巧。

# 本书所需的内容

本书中的实际示例涉及到使用*AWS*资源，因此需要一个*AWS*账户。

示例中使用的客户端工具，如*AWS CLI*和*Terraform*，在大多数常见操作系统（*Linux/Windows/Mac OS*）上均受支持。

# 本书适用对象

本书面向那些管理 AWS 基础设施和环境的系统管理员和开发人员，并计划在组织中实施 DevOps 的人。那些准备获得 AWS 认证 DevOps 工程师认证的人也会觉得本书非常有用。期望具备操作和管理 AWS 环境的先前经验。

# 约定

本书中，你将看到一些文本样式，用于区分不同类型的信息。以下是这些样式的一些示例及其含义解释。

文本中的代码词汇、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 用户名如下所示：“我们需要通过 SSH 连接到节点，并检索存储在`/var/lib/jenkins/secrets/initialAdminPassword`中的管理员密码。”

代码块的设置如下：

```
aws-region = "us-east-1" 
vpc-cidr = "10.0.0.0/16" 
vpc-name = "Terraform" 
aws-availability-zones = "us-east-1b,us-east-1c" 

```

当我们希望引起你对代码块中特定部分的注意时，相关的行或项目会以粗体显示：

```
aws-region = "us-east-1" 
vpc-cidr = "10.0.0.0/16" 
vpc-name = "Terraform" 
aws-availability-zones = "us-east-1b,us-east-1c" 

```

任何命令行输入或输出都按如下方式写出：

```
$ terraform validate
$ terraform plan
Refreshing Terraform state prior to plan...
...
Plan: 11 to add, 0 to change, 0 to destroy.
$ terraform apply
aws_iam_role.jenkins: Creating...
...
Apply complete! Resources: 11 added, 0 changed, 0 destroyed.
Outputs:
JENKINS EIP = x.x.x.x
VPC ID = vpc-xxxxxx

```

**新术语**和**重要词汇**以粗体显示。你在屏幕上看到的词语，比如菜单或对话框中的内容，会像这样显示在文本中：“我们选择**Pipeline**作为作业类型，并为其命名。”

### 注意

警告或重要的注释会以框的形式出现，如下所示。

### 提示

提示和技巧如下所示。

# 读者反馈

我们非常欢迎读者的反馈。请告诉我们你对本书的看法——你喜欢什么或不喜欢什么。读者的反馈对我们非常重要，它帮助我们开发出能真正让你受益的书籍。

要向我们发送一般反馈，只需通过电子邮件发送至 feedback@packtpub.com，并在邮件主题中提到书名。

如果你在某个领域拥有专业知识，并且有兴趣写书或为书籍贡献内容，请查看我们的作者指南：[www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

既然你已经是一本 Packt 书籍的骄傲拥有者，我们有很多方式帮助你从购买中获得最大价值。

## 下载示例代码

您可以从您的帐户中下载本书的示例代码文件，网址是[`www.packtpub.com`](http://www.packtpub.com)。如果您在其他地方购买了本书，可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)并注册，让我们将文件通过电子邮件直接发送给您。

您可以通过以下步骤下载代码文件：

1.  使用您的电子邮件地址和密码登录或注册我们的网站。

1.  将鼠标指针悬停在顶部的**支持**标签上。

1.  点击**代码下载和勘误表**。

1.  在**搜索**框中输入书名。

1.  选择您希望下载代码文件的书籍。

1.  从下拉菜单中选择您购买本书的渠道。

1.  点击**代码下载**。

下载文件后，请确保使用最新版本的工具解压或提取文件夹：

+   Windows 版的 WinRAR / 7-Zip

+   Mac 版的 Zipeg / iZip / UnRarX

+   Linux 版的 7-Zip / PeaZip

完整的代码集也可以从以下 GitHub 仓库下载：[`github.com/PacktPublishing/Implementing-DevOps-on-AWS`](https://github.com/PacktPublishing/Implementing-DevOps-on-AWS)。

## 下载本书的彩色图片

我们还为您提供了一个 PDF 文件，其中包含本书中使用的截图/图表的彩色图片。这些彩色图片将帮助您更好地理解输出中的变化。您可以从[`www.packtpub.com/sites/default/files/downloads/ImplementingDevOpsonAWS_ColorImages.pdf`](https://www.packtpub.com/sites/default/files/downloads/ImplementingDevOpsonAWS_ColorImages.pdf)下载该文件。

## 勘误

尽管我们已尽力确保内容的准确性，但错误仍然可能发生。如果您在我们的书中发现错误——可能是文本或代码中的错误——我们将非常感激您能向我们报告。通过这样做，您可以帮助其他读者避免困惑，并帮助我们改进后续版本。如果您发现任何勘误，请访问[`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)，选择您的书籍，点击**勘误提交表单**链接，填写勘误的详细信息。一旦您的勘误被验证，您的提交将被接受，勘误将被上传到我们的网站，或被添加到该书籍勘误列表中的“勘误”部分。

要查看之前提交的勘误，请访问[`www.packtpub.com/books/content/support`](https://www.packtpub.com/books/content/support)，并在搜索框中输入书名。所需信息将出现在**勘误**部分。

## 盗版

互联网版权盗版问题在各类媒体中一直存在。我们在 Packt 非常重视版权和许可证的保护。如果你在互联网上遇到任何非法复制的我们作品的形式，请立即提供该地址或网站名称，以便我们采取措施。

请通过链接将涉嫌盗版的资料发送至 copyright@packtpub.com。

我们感谢你在保护我们的作者及其作品内容方面的帮助。

## 问题

如果你对本书的任何内容有问题，可以通过 questions@packtpub.com 联系我们，我们将尽力解决问题。
