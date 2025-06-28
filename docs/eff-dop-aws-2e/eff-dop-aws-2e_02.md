# 第二章：部署你的第一个 Web 应用程序

在上一章中，我们介绍了云计算的一般概念、它的好处以及什么是 DevOps 思想。AWS 提供了许多服务，所有服务都可以通过 Web 界面、命令行界面、各种 SDK 和 API 轻松访问。在本章中，我们将利用 Web 界面和命令行界面来创建和配置账户，并创建一个 Web 服务器来托管一个简单的 Hello World 应用程序，所有操作都将在几分钟内完成。

在本章中，我们将讨论以下主题：

+   创建和配置你的账户

+   启动你的第一个 Web 服务器

# 技术要求

本章中使用的技术和服务如下：

+   AWS 管理控制台

+   AWS 计算服务

+   AWS IAM

+   AWS CLI 设置

+   Web 应用程序的 JavaScript

+   GitHub 上的现成代码

代码的 GitHub 链接如下：

+   [`raw.githubusercontent.com/yogeshraheja/Effective-DevOps-with-AWS/master/Chapter02/helloworld.js`](https://raw.githubusercontent.com/yogeshraheja/Effective-DevOps-with-AWS/master/Chapter02/helloworld.js)

+   [`raw.githubusercontent.com/yogeshraheja/Effective-DevOps-with-AWS/master/Chapter02/helloworld.conf`](https://raw.githubusercontent.com/yogeshraheja/Effective-DevOps-with-AWS/master/Chapter02/helloworld.conf)

# 创建和配置你的账户

如果你还没有注册 AWS，现在是时候注册了。

# 注册

这一步当然是相当简单且不言自明的。为了注册（如果你还没有注册），在浏览器中打开[`portal.aws.amazon.com`](https://portal.aws.amazon.com)，点击“创建一个新的 AWS 账户”按钮，并按照步骤进行操作。你将需要一个电子邮件地址和信用卡信息。

这个过程的两个例外如下：

+   如果你打算在中国部署服务器，那么你需要在 AWS 中国区域创建账户，网址为[`www.amazonaws.cn/`](https://www.amazonaws.cn/)。

+   AWS 为美国联邦、州和地方政府机构的特定监管需求提供了一项名为**GovCloud**的特殊服务。要注册此服务，请访问以下链接：[`aws.amazon.com/govcloud-us/contact/`](https://aws.amazon.com/govcloud-us/contact/)。

在本书中，我们将使用位于弗吉尼亚州北部的服务器，因此你需要通过标准注册流程进行注册。

亚马逊为新用户提供了一项免费计划，旨在帮助你免费体验 AWS 服务。亚马逊为大多数服务提供免费的信用额度。随着时间的推移，该优惠可能会有所变动，因此本书不会详细介绍此优惠的具体内容，但详细信息可以在[`aws.amazon.com/free/`](https://aws.amazon.com/free/)找到。

完成注册流程后，你将进入 AWS 管理控制台的登录页面。由于 Amazon 现在有很多服务，这个页面可能让人有些不知所措，但你会很快习惯的。如果你喜欢使用书签，那么这个页面绝对是一个首选：

![](img/55489223-a325-4c9b-be97-233e2f7d9d08.png)

你刚刚创建的账户被称为 **root** 账户。此账户将始终拥有对所有资源的完全访问权限。因此，确保将你的密码保存在安全的地方。最佳做法是仅使用 root 账户通过 IAM 服务创建初始用户，我们将在稍后发现这个服务。此外，强烈建议切换到 **多因素认证**（**MFA**）并使用身份服务 **IAM** 管理用户账户，因此请设置一个相对复杂的密码。

# 启用 root 账户的 MFA

为了避免任何问题，我们在完成注册后需要做的第一件事是启用 MFA。如果你以前没有听说过或看过 MFA，它是一种安全系统，要求通过来自不同类别凭证的多种身份验证方法。这些方法用于验证用户身份以便登录。实际上，一旦启用，你将需要在注册时设置的密码才能登录。此外，你还需要另一个来自不同来源的代码。第二个来源可以是通过物理设备（如 SafeNet IDProve，链接：[`amzn.to/2u4K1rR`](http://amzn.to/2u4K1rR)）提供，或者通过手机上的短信，或者通过安装在智能手机上的应用程序。我们将使用第三种选项——安装在智能手机上的应用程序，这完全免费：

1.  前往你的应用商店、Google Play 商店或应用市场，安装一个名为 **Google Authenticator** 的应用（或者其他等效的应用，如 **Authy**）。

1.  在 AWS 管理控制台中，打开右上角的“我的安全凭证”页面：

![](img/3c115402-abc2-4abe-b9cb-a918dee220b0.png)

1.  如果系统提示你创建并使用 AWS **身份与访问管理**（**IAM**）用户，点击“继续到安全凭证”按钮。我们将在第三章中探索 IAM 系统，*将基础设施视为代码*。在页面上展开“多因素认证”（MFA）部分。

1.  选择虚拟 MFA 并按照说明将 Google 验证与 root 账户同步（注意，扫描二维码选项是最简单的配对设备方式）。

从现在开始，你将需要你的密码以及 MFA 应用中显示的令牌，才能作为 root 用户登录 AWS 控制台。

有两个关于管理密码和 MFA 的通用建议如下：

+   有许多优秀的应用程序可以管理密码，例如**1Password**，可以在[`agilebits.com/onepassword`](https://agilebits.com/onepassword)找到，或者**Dashlane**，可以在[`www.dashlane.com`](https://www.dashlane.com/)找到。

+   对于 MFA，你还可以尝试使用**Authy**，它可以在[`www.authy.com`](https://www.authy.com/)找到。它的工作原理类似于 Google Authenticator，但也有一个集中式服务器，允许它跨多个设备（包括桌面应用程序）工作，因此如果你丢失了手机，你也不会失去对 AWS 的访问权限。

正如我们之前所看到的，根账户的使用应该限制到最小。因此，为了创建虚拟服务器、配置服务等，我们将依赖 IAM 服务，这将让我们对每个用户的权限进行细粒度控制。

# 在 IAM 中创建新用户

在本节中，我们将为需要访问 AWS 的不同个人创建和配置账户。目前，我们将保持简单，仅为自己创建一个账户，如下所示：

1.  在 AWS 控制台中导航到 IAM 菜单([`console.aws.amazon.com/iam/`](https://console.aws.amazon.com/iam/))，或者转到 AWS 控制台页面左上角的“服务”下拉菜单，搜索`IAM`：

![](img/cee7de26-a00f-483e-bc71-f348dd82a5bb.png)

1.  从导航窗格中选择“用户”选项。

1.  通过点击“添加用户”按钮创建一个新用户，并确保勾选“编程访问”选项，以为该用户生成访问密钥 ID 和秘密访问密钥。

1.  选择默认选项并创建用户。别忘了下载凭证。

1.  返回到“用户”菜单，点击你的用户名以访问详细页面。

1.  在“权限”标签下，点击“添加权限”按钮，选择“直接附加现有策略”选项。点击“AdministratorAccess”以向我们新创建的用户提供对 AWS 服务和资源的完全访问权限。

1.  选中“AdministratorAccess”选项旁边的复选框，以向我们新创建的用户提供对 AWS 服务和资源的完全访问权限。你将看到如下的页面：

![](img/38edc83d-43d9-4eab-91d6-e97173d81dd8.png)

我们需要做的最后一件事是为此账户设置密码并启用 MFA。可以按照以下步骤完成：

1.  点击“安全凭证”标签。

1.  现在点击“控制台密码”选项，为新创建的用户启用密码。设置你选择的密码并点击“应用”按钮。

1.  在你完成设置密码后，点击“分配 MFA 设备”选项。

1.  选择“虚拟 MFA 设备”选项，并按照剩余的说明操作，以便在你新创建的账户中启用 MFA。你将看到一条信息，说明“MFA 设备已成功关联到你的账户”，如下所示：

![](img/2711ccdd-92c9-4bd2-ba8f-bbdfdd202ee6.png)

1.  此时，你已经准备好开始使用新创建的用户账户。需要注意的是，使用 IAM 用户账户登录与根账户不同，主要的区别是你使用不同的 URL 登录。

1.  转到 [`console.aws.amazon.com/iam/home#home`](https://console.aws.amazon.com/iam/home#home) 或在 IAM 菜单中点击仪表盘。

1.  你将在 IAM 用户登录链接下看到你的独特登录 URL。你也可以自定义该链接。将这个新 URL 保存到书签中，从现在开始，使用这个链接登录 AWS 控制台。

1.  从根账户退出。

1.  重新登录，但这次请使用你的 IAM 用户账户，地址是 `https://AWS-account-ID` 或 [alias.signin.aws.amazon.com/console](https://us-east-1.signin.aws.amazon.com/oauth?SignatureVersion=4&X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAJMOATPLHVSJ563XQ&X-Amz-Date=2018-08-27T10%3A03%3A52.789Z&X-Amz-Signature=bd1d9ecc1e645513ee09a2133776098ba8a82903c49d994f62fe4cb04a5d92d9&X-Amz-SignedHeaders=host&client_id=arn%3Aaws%3Aiam%3A%3A015428540659%3Auser%2Fhomepage&redirect_uri=https%3A%2F%2Fconsole.aws.amazon.com%2Fconsole%2Fhome%3Fstate%3DhashArgs%2523%26isauthcode%3Dtrue&response_type=code&state=hashArgs%23)。

不要分享你的访问密钥和秘密密钥。通过这些步骤，我们强制要求使用 MFA 来访问 AWS 控制台。现在，我们需要两个因素（密码和 MFA 令牌）才能访问控制台。也就是说，我们还创建了一个访问密钥，它的安全性较低。任何拥有秘密密钥和访问密钥（这两者都包含在 `credentials.csv` 中）的人将拥有 AWS 账户的完全管理权限。请确保永远不要在网上共享这些凭证。在 第八章，*加固你的 AWS 环境的安全性*，我们将进行一些更改，以更好地保护这个密钥并要求使用 MFA 才能获得管理员权限。

配置账户的下一步是配置我们的计算机，以便通过命令行界面与 AWS 进行交互。

# 安装和配置命令行界面（CLI）

使用 Amazon 的网页界面通常是探索新服务的好方法。问题是，当你想要快速操作、创建更多可重复的步骤或生成良好的文档时，执行简单命令会更加高效。Amazon 提供了一个非常棒且易于使用的 CLI 工具。该工具是用 Python 编写的，因此具有跨平台特性（Windows、Mac 和 Linux）。

我们将在笔记本电脑/桌面电脑上安装该工具，以便通过 bash 命令与 AWS 交互。Linux 和 macOS X 原生支持 bash。如果你使用其中一种操作系统，可以跳过下一节。在 Windows 上，我们首先需要安装名为 **Windows Subsystem for Linux**（**WSL**）的功能，它将使我们能够运行与 Ubuntu Linux 上非常相似的 Bash 命令。

# 安装 WSL（仅限 Windows）

如今，Linux 和 macOS X 是开发人员使用的最主流操作系统之一。Windows 最近与 Canonical（Linux 发行版之一的背后公司）建立了合作关系，支持 Bash 以及大多数常见的 Linux 包。通过在 Windows 上安装此工具，我们将能够更高效地与我们的服务器交互，这些服务器也运行 Linux：

1.  点击开始按钮，搜索 `设置`，然后打开设置应用：

![](img/693e26db-782b-4fac-b619-8505f41b5e68.jpg)

1.  这将引导您到以下窗口，在其中搜索 `Windows 更新设置`。打开 Windows 更新设置菜单：

![](img/f4451a3e-1a0d-4cfa-9bcc-cf68caa4adb4.png)

1.  在 Windows 更新设置的左侧菜单中，点击“开发人员”子菜单并启用“开发者模式”选项。

1.  一旦开启开发者模式，在左侧菜单的搜索框中搜索 `控制面板` 选项：

![](img/c8b255a1-221b-4987-9dae-b44b30839af8.png)

1.  在控制面板的仪表板中，选择“类别”视图选项，然后点击“程序”选项。接着，在“程序和功能”下，点击“启用或关闭 Windows 功能”选项：

![](img/2e3f5df9-c6ce-405e-9cca-5faf1d2f95c6.png)

1.  在此菜单中，找到名为“Windows 子系统 for Linux (Beta)”的功能，并点击“确定”按钮：

![](img/0d5ef3e7-cf36-4a98-a6a1-d054b26781e4.png)

这将安装该功能并要求您重新启动计算机。

1.  返回 Windows 后，再次点击开始按钮，搜索 `bash`，并启动 Bash on Ubuntu on the Windows 应用：

![](img/9feb5a78-ec22-4f50-9db9-c8ee5e2deb6b.png)

1.  完成初始化步骤后，您将能够像在 Linux 上一样使用 Windows 上的 Bash。

从此时起，使用 Bash 应用程序运行书中列出的命令。

# 安装 AWS CLI 包

AWS CLI 工具是用 Python 编写的。虽然有多种方式来安装它，但我们将使用 **PyPA**，Python 包管理器，来安装此工具。

要安装 PyPA，取决于您的操作系统，您需要运行以下命令：

+   在 Windows 上：

```
$ sudo apt install python-pip
```

+   在 macOS X 上：

```
$ sudo easy_install pip
```

+   在基于 Debian 的 Linux 发行版上：

```
$ sudo apt-get install python-pip python-dev build-essential  
```

+   在基于 Red Hat/CentOS 的 Linux 发行版上：

```
    $ sudo yum -y install python-pip  
```

安装 PyPA 后，您将能够使用 `pip` 命令。

最后，要使用 `pip` 命令安装 AWS CLI，您只需运行以下命令：

```
$ sudo pip install --upgrade --user awscli 
```

如果提示您升级 `pip` 版本到最新的可用版本，执行 `pip install --upgrade pip`。

我们已经展示了来自 CentOS 基于 Linux 发行版的所有输出，但该过程同样适用于所有提到的支持平台。

# 配置 AWS CLI

为此，您需要从在*创建 IAM 中的新用户*部分的第 4 步下载的文件中提取 AWS 访问密钥 ID 和秘密访问密钥：

```
$ more credentials.csv
User Name,Access Key Id,Secret Access Key "yogeshraheja", AKIAII55DTLEV3X4ETAQ, mL2dEC8/ryuZ7fu6UI6kOm7PTlfROCZpai07Gy6T 
```

我们将运行以下命令来配置我们的 AWS 账户：

```
$ aws configure
AWS Access Key ID [None]: AKIAII55DTLEV3X4ETAQ
AWS Secret Access Key [None]: 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
Default region name [None]: us-east-1
Default output format [None]: 
```

此时，我们准备开始使用 CLI。我们可以通过列出用户账户来快速验证一切是否正常，如下所示：

```
$ aws iam list-users
{
    "Users": [
        {
            "UserName": "yogeshraheja",
            "PasswordLastUsed": "2018-08-07T09:57:53Z",
            "CreateDate": "2018-08-07T04:56:03Z",
            "UserId": "AIDAIN22VCQLK43UVWLMK",
            "Path": "/",
            "Arn": "arn:aws:iam::094507990803:user/yogeshraheja"
        }
    ]
}
```

AWS aws-shell

亚马逊有一个第二个命令行工具叫做`aws-shell`。这个工具比经典的`awscli`命令更具交互性，因为它提供了开箱即用的自动补全功能，并且有一个分屏视图，可以让你在输入命令时访问文档。如果你是一个新的 AWS 用户，不妨试试看（`pip install aws-shell`）。

# 创建我们的第一个 Web 服务器

现在我们已经设置好了环境，终于可以启动我们的第一个 EC2 实例了。有几种方法可以做到这一点。由于我们刚刚安装并配置了`awscli`，并且希望查看有效的基础设施管理方法，我们将演示如何使用 CLI 来完成这项工作。

启动虚拟服务器需要提前准备一些信息。我们将使用`aws ec2 run-instances`命令，但需要提供以下信息：

+   一个 AMI ID

+   一个实例类型

+   一个安全组

+   一个 SSH 密钥对

# 亚马逊机器镜像（AMI）

AMI 是一个包含多个内容的包，其中包括操作系统的根文件系统（例如，Linux、UNIX 或 Windows），以及启动系统所需的其他软件。为了找到合适的 AMI，我们将使用`aws ec2 describe-images`命令。默认情况下，`describe-images`命令会列出所有可用的公共 AMI，目前已经超过了 300 万个。为了充分利用这个命令，重要的是结合过滤选项，仅包括我们想要使用的 AMI。在我们的情况下，我们将使用以下方法来过滤我们的 AMI：

+   我们希望名称为 Amazon Linux AMI，这表示 AWS 官方支持的 Linux 发行版。Amazon Linux 基于 Red Hat/CentOS，但包括一些额外的包，以便与其他 AWS 服务的集成更加方便。你可以在[`amzn.to/2uFT13F`](http://amzn.to/2uFT13F)了解更多关于 AWS Linux 的信息。

+   我们想使用`x84_64`位版本的 Linux，以匹配我们将要使用的架构。

+   虚拟化类型应该是 HVM，代表硬件虚拟机。这是最新且性能最强的虚拟化类型。

+   我们需要 GP2 支持，这样我们就可以使用最新一代的实例，而这些实例没有*实例存储*，意味着支持我们实例的服务器将与存储我们数据的服务器不同。

此外，我们将按年龄对输出进行排序，并只查看最近发布的 AMI：

```
$ aws ec2 describe-images --filters "Name=description,Values=Amazon Linux AMI * x86_64 HVM GP2" --query 'Images[*].[CreationDate, Description, ImageId]' --output text | sort -k 1 | tail
```

运行前面命令的输出如下所示：

![](img/b5850390-3f22-4650-ba95-ec90894f10a7.png)

如你所见，当前最新的 AMI ID 是`ami-cfe4b2b0`。当你执行相同的命令时，这可能会有所不同，因为亚马逊供应商定期更新他们的操作系统。

当使用`aws cli --query`选项时，输出对于某些命令非常重要。以前面的例子为例，如果我们只关心某些信息子集，我们可以使用`--query`选项来过滤我们想要的信息。该选项使用**JMESPath**查询语言。

# 实例类型

在本节中，我们将选择用于虚拟服务器的虚拟硬件。AWS 提供了一些选项，最好在他们的文档中进行详细描述：[`aws.amazon.com/ec2/instance-types/`](https://aws.amazon.com/ec2/instance-types/)。我们将在第六章中更详细地讨论实例类型，*扩展您的基础设施*。

现在，我们将选择`t2.micro`实例类型，因为它符合 AWS 的免费使用套餐条件。

# 安全组

安全组的工作方式有点像防火墙。所有 EC2 实例都有一组安全组分配给它们，每个安全组包含允许流入（**入站**）和/或流出（**出站**）流量的规则。

在这个练习中，我们将创建一个运行在`tcp/3000`端口上的小型 Web 应用程序。此外，我们希望能够通过 SSH 连接到实例，因此我们还需要允许到`tcp/22`端口的入站流量。我们将通过以下步骤创建一个简单的安全组来实现这一点：

1.  首先，我们需要找出我们的默认**虚拟私有云**（**VPC**）ID。尽管处于云环境中，物理资源由所有 AWS 客户共享，但安全性仍然受到严格强调。AWS 使用 VPC 的概念来分割他们的虚拟基础设施。你可以把它想象成一个带有自己网络的虚拟数据中心。保护我们的 EC2 实例的安全组与子网相关联，子网又与 VPC 提供的网络相关联：

![](img/e89e10dd-525c-4dd0-8d7c-28cf71e18dd4.png)

要识别我们的 VPC ID，我们可以运行以下命令：

```
    $ aws ec2 describe-vpcs 
    {
        "Vpcs": [
            {
                "VpcId": "vpc-4cddce2a",
                "InstanceTenancy": "default",
                "CidrBlockAssociationSet": [
                    {
                        "AssociationId": "vpc-cidr-assoc-3c313154",
                        "CidrBlock": "172.31.0.0/16",
                        "CidrBlockState": {
                            "State": "associated"
                        }
                    }
                ],
                "State": "available",
                "DhcpOptionsId": "dopt-c0be5fa6",
                "CidrBlock": "172.31.0.0/16",
                "IsDefault": true
            }
        ]
    } 
```

1.  现在我们知道了 VPC ID（你的将会不同），我们可以按照以下步骤创建我们的新安全组：

```
    $ aws ec2 create-security-group \
        --group-name HelloWorld \
        --description "Hello World Demo" \
        --vpc-id vpc-4cddce2a 
    {
        "GroupId": "sg-01864b4c"
    }
```

1.  默认情况下，安全组允许实例的所有出站流量。我们只需打开 SSH（`tcp/22`）和`tcp/3000`的入站流量。然后，我们需要输入以下内容：

```
    $ aws ec2 authorize-security-group-ingress \
        --group-name HelloWorld \
        --protocol tcp \
        --port 22 \
        --cidr 0.0.0.0/0

    $ aws ec2 authorize-security-group-ingress \
        --group-name HelloWorld \
        --protocol tcp \
        --port 3000 \
        --cidr 0.0.0.0/0  
```

1.  我们现在可以使用以下代码验证所做的更改，因为之前的命令并不详细：

```
    $ aws ec2 describe-security-groups \
        --group-names HelloWorld \
        --output text 
    SECURITYGROUPS  Hello World Demo    sg-01864b4c     HelloWorld      
    094507990803    vpc-4cddce2a
    IPPERMISSIONS   22      tcp     22
    IPRANGES        0.0.0.0/0
    IPPERMISSIONS   3000    tcp     3000
    IPRANGES        0.0.0.0/0
    IPPERMISSIONSEGRESS     -1
    IPRANGES        0.0.0.0/0 
```

如预期那样，我们已经开放了适当端口的流量。如果您知道如何找到您的公共 IP 地址，您可以通过将`0.0.0.0/0`替换为您的 IP/32 来改进 SSH 规则，以便只有您可以尝试通过 SSH 连接到该 EC2 实例。

使用 aws cli --output 选项

大多数命令默认会返回 JSON 输出。AWS 全局可用的选项有一定数量。您可以在本章中看到它们的使用。第一个选项是`--output [json | text | table]`：

![](img/f8f066db-fef2-47a2-a091-1c8768870275.png)

# 生成您的 SSH 密钥

默认情况下，Amazon EC2 使用 SSH 密钥对为你提供 SSH 访问权限。你可以在 EC2 中生成密钥对并下载私钥，也可以使用第三方工具（如 OpenSSL）生成密钥，并在 EC2 中导入公钥。我们将使用第一种方法来创建 EC2 SSH 密钥。

在此，确保为你新生成的私有（`.pem`）密钥文件设置只读权限：

```
    $ aws ec2 create-key-pair --key-name EffectiveDevOpsAWS --query   
    'KeyMaterial' --output text > ~/.ssh/EffectiveDevOpsAWS.pem

    $ aws ec2 describe-key-pairs --key-name EffectiveDevOpsAWS
    {
        "KeyPairs": [
            {
                "KeyName": "EffectiveDevOpsAWS",
                "KeyFingerprint": 
          "27:83:5d:9b:4c:88:f6:15:c7:39:df:23:4f:29:21:3b:3d:49:e6:af"

            }
        ]
    }

    $ cat ~/.ssh/EffectiveDevOpsAWS.pem
 -----BEGIN RSA PRIVATE KEY-----
MIIEpAIBAAKCAQEAiZLtUMnO2OKnHvTJOiIP26fThdsU0YRdlKI60in85x9aFZXSrZsKwOhWPpMtnUMJKeGvVQut+gJ1I1PNNjPqS2Dy60jH55hntUhr/ArpaL2ISDX4BgRAP1jcukBqS6+pL+mTp6OUNTToUt7LvAZoeo+10SYbzHF1ZMQLLs96fCMNvnbJdUCa904dJjJs7t/G2ou9RiNMRx8midrWcmmuGKOb1s6FgrxJ5OAMYegeccFVfGOjqPk3f+6QTPOTMNgNQ8ANKOMA9YtcIca/75QGUPifusTqUT4Fqtv3rbUYPvacAnYL9eCthtn1XMG7Oo/mR5MrU60wib2QcPipmrGNbwIDAQABAoIBABSyqkmxUxGGaCZcJbo9Ta16fnRxFZzAEWQ/VCIydv4+1UrSE7RS0zdavT8E3aP/Ze2LKtncu/wVSpJaFVHGVcWpfGKxvIG3iELZ9oUhDyTW/x3+IKanFRNyxyKudk+UyhuPRMu/7JhksV9mbbiILkfiPzSMSzpjB4p1hEkypfbvBnrbB+sRycx+jK5l209rNDukkJVvyFCnqPiH0wmvKRqHTNOMGWmM6CPOU+VpuMX+dIlrSeId7j6hqMjA0rGncnxYi035v2zicvIsEKHZ9MZCnkiRb3kJ9PhueTwwUQmoBYfV5E+1Wu34UmdsmALQEX3xniaR6xf9iWhQ2Nh8LaECgYEAzXHOZDPAUzXitO735KBUaiBp9NMv2gzE862Yf2rmDkFM4Y5RE3DKHrKfeOkrYqlG11On0m44GHBk/g4eqqIEaBjVp6i/Lk74tpQU6Kn1HT3w9lbXEFsCWjYZnev5oHP6PdedtRYNzZsCSNUdlw0kOG5WZZJ4E7mPZyrvK5pq+rMCgYEAq22KT0nD3d59V+LVVZfMzJuUBDeJeD139mmVbzAq9u5Hr4MkurmcIj8Q6jJIQaiC8XC1gBVEl08ZN2oY1+CBE+Gesi7mGOQ2ovDmoTfYRgScKKHv7WwR+N5/N7o26x+ZaoeaBe43Vjp6twaTpKkBOIuT50tvb25v9+UVMpGKcFUC
gYEAoOFjJ3KjREYpT1jnROEM2cKiVrdefJmNTel+RyF2IGmgg+1Hrjqf/OQSH8QwVmWK9SosfIwVX4X8gDqcZzDS1JXGEjIB7IipGYjiysP1D74myTF93u/16qD89H8LD0xjBTSo6lrn2j9tzY0eS+Bdodc9zvKhF4kzNC4Z9wJIjiMCgYAOtqstXP5zt5n4hh6bZxkL4rqUlhO1f0khnDRYQ8EcSp1agh4P7Mhq5BDWmRQ8lnMOuAbMBIdLmV1ntTKGrN1HUJEnaAEV19icqaKR6dIlSFYC4stODH2KZ8ZxiQkXqzGmxBbDNYwIWaKYvPbFJkBVkx1Rt9bLsKXpl/72xSkltQKBgQCYEjUVp4dPzZL1CFryOwV72PMMX3FjOflTgAWr8TJBq/OLujzgwYsTy6cdD3AqnMQ2BlU7Gk4mmDZCVVsMqHFbIHEa5Y4e5qIQhamedl3IgmnMpdyuDYaT/Uh4tw0JxIJabqm+sQZv4s1Otgh00JlGrgFs+0D39Fy8qszqr6J04w==
-----END RSA PRIVATE KEY----- 
    $ chmod 400 ~/.ssh/EffectiveDevOpsAWS.pem
```

# 启动 EC2 实例

我们现在已经拥有了启动实例所需的所有信息。让我们最后按如下方式启动它：

```
$ aws ec2 run-instances \
 --instance-type t2.micro \
 --key-name EffectiveDevOpsAWS \
 --security-group-ids sg-01864b4c \
 --image-id ami-cfe4b2b0
{
    "Instances": [
        {
            "Monitoring": {
                "State": "disabled"
            },
            "PublicDnsName": "",
            "StateReason": {
                "Message": "pending",
                "Code": "pending"
            },
            "State": {
                "Code": 0,
                "Name": "pending"
            },
            "EbsOptimized": false,
            "LaunchTime": "2018-08-08T06:38:43.000Z",
            "PrivateIpAddress": "172.31.22.52",
            "ProductCodes": [],
            "VpcId": "vpc-4cddce2a",
            "CpuOptions": {
                "CoreCount": 1,
                "ThreadsPerCore": 1
            },
            "StateTransitionReason": "",
            "InstanceId": "i-057e8deb1a4c3f35d",
            "ImageId": "ami-cfe4b2b0",
            "PrivateDnsName": "ip-172-31-22-52.ec2.internal",
            "KeyName": "EffectiveDevOpsAWS",
            "SecurityGroups": [
                {
                    "GroupName": "HelloWorld",
                    "GroupId": "sg-01864b4c"
                }
            ],
            "ClientToken": "",
            "SubnetId": "subnet-6fdd7927",
            "InstanceType": "t2.micro",
            "NetworkInterfaces": [
                {
                    "Status": "in-use",
                    "MacAddress": "0a:d0:b9:db:7b:38",
                    "SourceDestCheck": true,
                    "VpcId": "vpc-4cddce2a",
                    "Description": "",
                    "NetworkInterfaceId": "eni-001aaa6b5c7f92b9f",
                    "PrivateIpAddresses": [
                        {
                            "PrivateDnsName": "ip-172-31-22-
                             52.ec2.internal",
                            "Primary": true,
                            "PrivateIpAddress": "172.31.22.52"
                        }
                    ],
                    "PrivateDnsName": "ip-172-31-22-52.ec2.internal",
                    "Attachment": {
                        "Status": "attaching",
                        "DeviceIndex": 0,
                        "DeleteOnTermination": true,
                        "AttachmentId": "eni-attach-0428b549373b9f864",
                        "AttachTime": "2018-08-08T06:38:43.000Z"
                    },
                    "Groups": [
                        {
                            "GroupName": "HelloWorld",
                            "GroupId": "sg-01864b4c"
                        }
                    ],
                    "Ipv6Addresses": [],
                    "OwnerId": "094507990803",
                    "SubnetId": "subnet-6fdd7927",
                    "PrivateIpAddress": "172.31.22.52"
                }
            ],
            "SourceDestCheck": true,
            "Placement": {
                "Tenancy": "default",
                "GroupName": "",
                "AvailabilityZone": "us-east-1c"
            },
            "Hypervisor": "xen",
            "BlockDeviceMappings": [],
            "Architecture": "x86_64",
            "RootDeviceType": "ebs",
            "RootDeviceName": "/dev/xvda",
            "VirtualizationType": "hvm",
            "AmiLaunchIndex": 0
        }
    ],
    "ReservationId": "r-09a637b7a3be11d8b",
    "Groups": [],
    "OwnerId": "094507990803"
}
```

你可以跟踪实例创建的进度。为此，获取`aws ec2 run-instances`命令输出中的实例 ID，并运行以下命令：

```
$ aws ec2 describe-instance-status --instance-ids i-057e8deb1a4c3f35d
{
    "InstanceStatuses": [
        {
            "InstanceId": "i-057e8deb1a4c3f35d",
            "InstanceState": {
                "Code": 16,
                "Name": "running"
            },
            "AvailabilityZone": "us-east-1c",
            "SystemStatus": {
                "Status": "initializing",
                "Details": [
                    {
                        "Status": "initializing",
                        "Name": "reachability"
                    }
                ]
            },
            "InstanceStatus": {
                "Status": "initializing",
                "Details": [
                    {
                        "Status": "initializing",
                        "Name": "reachability"
                    }
                ]
            }
        }
    ]
}
```

一旦`SystemStatus`下的状态从`initializing`变为`ok`，实例就会准备好：

```
$ aws ec2 describe-instance-status --instance-ids i-057e8deb1a4c3f35d --output text| grep -i SystemStatus
SYSTEMSTATUS ok
```

# 使用 SSH 连接到 EC2 实例

本章的主要目标是创建一个简单的`Hello World`网页应用。由于我们从一个普通操作系统开始，我们需要连接到主机，进行必要的更改，将我们的标准服务器转变为网页服务器。为了通过 SSH 访问我们的实例，我们需要找到正在运行的实例的 DNS 名称，方法如下：

```
$ aws ec2 describe-instances \
 --instance-ids i-057e8deb1a4c3f35d \
 --query "Reservations[*].Instances[*].PublicDnsName"
[
    [
        "ec2-34-201-101-26.compute-1.amazonaws.com"
    ]
]      
```

我们现在有了实例的公共 DNS 名称和私钥，可以用来 SSH 连接到它。最后需要了解的是，对于我们在选择 Amazon Linux 的 AMI 时所选的操作系统，默认用户帐户名为`ec2-user`：

```
$ ssh -i ~/.ssh/EffectiveDevOpsAWS.pem ec2-user@ ec2-34-201-101-26.compute-1.amazonaws.com

The authenticity of host 'ec2-34-201-101-26.compute-1.amazonaws.com (172.31.22.52)' can't be established.

ECDSA key fingerprint is SHA256:V4kdXmwb5ckyU3hw/E7wkWqbnzX5DQR5zwP1xJXezPU.

ECDSA key fingerprint is MD5:25:49:46:75:85:f1:9d:f5:c0:44:f2:31:cd:e7:55:9f.

Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'ec2-34-201-101-26.compute-1.amazonaws.com,172.31.22.52' (ECDSA) to the list of known hosts.

       __| __|_ )
       _| ( / Amazon Linux AMI
      ___|\___|___|

https://aws.amazon.com/amazon-linux-ami/2018.03-release-notes/

1 package(s) needed for security, out of 2 available

Run "sudo yum update" to apply all updates.
[ec2-user@ip-172-31-22-52 ~]$
```

如果遇到任何问题，请在 SSH 命令中添加`-vvv`选项进行故障排除。

# 创建一个简单的 Hello World 网页应用

现在我们已经连接到 EC2 实例，准备开始在它上面进行实验。在本书中，我们将专注于 AWS 在技术公司中最常见的用例：托管应用程序。在编程语言方面，我们将使用 JavaScript，这是 GitHub 上最受欢迎的语言之一。也就是说，这个应用程序更多的是为了提供支持，演示如何使用 DevOps 原则最好地使用 AWS。了解 JavaScript 的任何知识都不是理解本书的必要条件：

![](img/74869af1-18ea-46ef-ba9c-fe358d404429.png)

JavaScript 在本书中提供的一些主要优势包括：

+   它非常容易写和读，甚至对于初学者来说

+   它不需要编译

+   它可以在服务器端运行，得益于 Node.js（[`nodejs.org`](https://nodejs.org)）

+   它是 AWS 官方支持的，因此 AWS 的 JavaScript SDK 是一个*一流的成员*

在本章的其余部分，所有命令和代码都将在我们的实例上通过 SSH 运行。

# 安装 Node.js

我们需要做的第一件事是安装 Node.js。Amazon Linux 基于**Red Hat Enterprise Linux**（**RHEL**），并使用`yum`工具来管理和安装软件包。操作系统自带了**Enterprise Linux 的额外软件包**（**EPEL**）预配置。正如我们所预期的，Node.js 在 EPEL 中：

```
[ec2-user@ip-172-31-22-52 ~]$ sudo yum install --enablerepo=epel -y nodejs 
[ec2-user@ip-172-31-22-52 ~]$ node -v
v0.10.48
```

这肯定是一个旧版本的节点，但它对我们需要的功能已经足够了。

# 运行 Node.js Hello World 应用程序

现在，节点已经安装好，我们可以创建一个简单的 Hello World 应用程序。下面是创建此应用程序的代码：

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

随意将其复制到文件中。或者，如果你想节省时间，可以从 GitHub 上下载：

```
[ec2-user@ip-172-31-22-52 ~]$ 
wget https://raw.githubusercontent.com/yogeshraheja/Effective-DevOps-with-AWS/master/Chapter02/helloworld.js -O /home/ec2-user/helloworld.js
--2018-08-19 13:06:42-- https://raw.githubusercontent.com/yogeshraheja/Effective-DevOps-with-AWS/master/Chapter02/helloworld.js
Resolving raw.githubusercontent.com (raw.githubusercontent.com)... 151.101.200.133
Connecting to raw.githubusercontent.com (raw.githubusercontent.com)|151.101.200.133|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 384 [text/plain]
Saving to: ‘/home/ec2-user/helloworld.js’

/home/ec2-user/helloworld.js 100%[=====================================================================================>] 384 --.-KB/s in 0s

2018-08-19 13:06:42 (37.9 MB/s) - ‘/home/ec2-user/helloworld.js’ saved [384/384] 

[ec2-user@ip-172-31-22-52 ~]$  
```

为了运行 Hello World 应用程序，我们现在只需运行以下代码：

```
[ec2-user@ip-172-31-22-52 ~]$ node helloworld.js  
Server running 
```

如果一切顺利，你现在应该能够在浏览器中打开以下链接：`http://your-public-dns-name:3000`。或者在我的例子中，可以在这里找到：`http://ec2-34-201-101-26.compute-1.amazonaws.com:3000`。然后，你将能够看到如下结果：

![](img/cf05de32-d99b-47e8-ac17-ffaf0d1003e9.png)

我们现在将通过在终端窗口中按 *Ctrl* + *C* 停止 Hello World 网络应用程序的执行。

# 使用 upstart 将我们的简单代码转化为服务

由于我们在终端手动启动了节点应用程序，关闭 SSH 连接或按 *Ctrl* + *C* 键会停止节点进程，因此我们的 Hello World 应用程序将不再工作。与标准的 Red Hat 基于发行版不同，Amazon Linux 带有一个名为 **upstart** 的系统。

这个工具非常易于使用，并提供了一些传统 **System-V 启动** 脚本没有的额外功能，比如在进程意外死亡时重新生成进程。要添加 upstart 配置，你需要在 EC2 实例的`/etc/init`目录下创建一个文件。

以下是将其插入`/etc/init/helloworld.conf`的代码：

```
description "Hello world Daemon"

# Start when the system is ready to do networking. Start on started elastic-network-interfaces

# Stop when the system is on its way down. Stop on shutdown

respawn script
exec su --session-command="/usr/bin/node /home/ec2-user/helloworld.js" ec2-user
end script
```

*为什么要在弹性网络接口上启动？* 如果你熟悉 AWS 之外的 upstart，可能曾使用过 `start on run level [345]`。在 AWS 中，问题在于你的网络来自 **弹性网络接口** (**ENI**)，如果应用程序在此服务之前启动，它可能无法正确连接到网络。

```
[ec2-user@ip-172-31-22-52 ~]$ 
sudo wget https://raw.githubusercontent.com/yogeshraheja/Effective-DevOps-with-AWS/master/Chapter02/helloworld.conf -O /etc/init/helloworld.conf
--2018-08-19 13:09:39-- https://raw.githubusercontent.com/yogeshraheja/Effective-DevOps-with-AWS/master/Chapter02/helloworld.conf
Resolving raw.githubusercontent.com (raw.githubusercontent.com)... 151.101.200.133
Connecting to raw.githubusercontent.com (raw.githubusercontent.com)|151.101.200.133|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 301 [text/plain]
Saving to: ‘/etc/init/helloworld.conf’

/etc/init/helloworld.conf 100%[=====================================================================================>] 301 --.-KB/s in 0s

2018-08-19 13:09:39 (54.0 MB/s) - ‘/etc/init/helloworld.conf’ saved [301/301]

[ec2-user@ip-172-31-22-52 ~]$
```

我们现在可以简单地启动我们的应用程序，如下所示：

```
[ec2-user@ip-172-31-22-52 ~]$ sudo start helloworld
helloworld start/running, process 2872
[ec2-user@ip-172-31-22-52 ~]$
```

正如预期的那样，`http://your-public-dns-name:3000` 仍然有效，这一次我们可以安全地关闭 SSH 连接。

# 终止我们的 EC2 实例

和大多数 Hello World 示例一样，一旦`helloworld`消息显示出来，目标就达成了。现在是时候考虑关闭我们的服务器了。由于我们在 AWS 中只为实际使用的资源付费，因此释放像这样的不必要的资源是一种使 AWS 非常具有成本效益的好策略。

我们可以使用`stop`命令干净地关闭 Hello World 服务。然后，我们可以退出虚拟服务器并终止实例，如下所示：

```
[ec2-user@ip-172-31-22-52 ~]$ sudo stop helloworld 
helloworld stop/waiting 
[ec2-user@ip-172-31-22-52 ~]$ ec2-metadata --instance-id 
instance-id: i-057e8deb1a4c3f35d 
[ec2-user@ip-172-31-22-52 ~]$ exit 
logout 
$ aws ec2 terminate-instances --instance-ids i-057e8deb1a4c3f35d 
{ 
    "TerminatingInstances": [ 
        { 
            "InstanceId": "i-057e8deb1a4c3f35d", 
            "CurrentState": { 
                "Code": 32, 
                "Name": "shutting-down" 
            }, 
            "PreviousState": { 
                "Code": 16, 
                "Name": "running" 
            } 
        } 
    ] 
} 
```

# 总结

本章是对 AWS 及其最著名的服务 EC2 的简短介绍。在注册 AWS 后，我们配置了环境，以便通过命令行界面创建虚拟服务器。接下来，我们选择了第一个 AMI，创建了第一个安全组，并生成了 SSH 密钥，这些密钥将在整本书中重复使用。在启动 EC2 实例后，我们手动部署了一个简单的 Node.js 应用程序来显示 Hello World。

虽然得益于 AWS CLI，该过程并不特别繁琐，但仍然需要经历许多步骤，而且这些步骤并不容易重复。我们还在没有任何自动化或验证的情况下部署了应用程序。此外，检查应用程序是否运行的唯一方法是手动检查端点。在接下来的章节中，我们将重新审视创建和管理网页应用程序与基础设施的过程，但这次我们将遵循 DevOps 原则，并结合其最佳实践。

在第三章《将基础设施视为代码》中，我们将讨论我们遇到的第一个问题：如何通过自动化来管理我们的基础设施。为此，我们将编写代码来管理我们的基础设施。

# 问题

请回答以下问题：

1.  如何创建一个免费层的 AWS 账户？

1.  如何使用 AWS 控制台门户创建你的第一个 AWS 云实例？

1.  如何使用 AWS CLI 工具创建你的第一个 AWS 云实例？

1.  如何在新创建的 AWS 实例上部署一个简单的 Hello World 网页应用程序？

1.  如何销毁你创建的 AWS 实例以完成此练习？

# 进一步阅读

请参考以下链接，获取关于 AWS 和 AWS CLI 的更多信息：

+   **AWS 免费层**：[`aws.amazon.com/free/`](https://aws.amazon.com/free/)

+   **AWS 命令行界面**：[`aws.amazon.com/cli/`](https://aws.amazon.com/cli/)
