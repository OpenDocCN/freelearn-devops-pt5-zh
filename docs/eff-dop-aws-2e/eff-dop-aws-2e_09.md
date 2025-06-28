# 第九章：评估

# 第一章： 云计算与 DevOps 革命

1.  DevOps 是一个框架和方法论，旨在为开发人员和运维团队之间的协作采纳合适的文化。

1.  DevOps – IaC 代表 **DevOps – 基础设施即代码**，我们应该将我们的垂直基础设施视为代码来管理和处理，这有助于实现可重复、可扩展和可管理的基础设施。

1.  DevOps 文化的关键特征

    +   版本控制所有内容

    +   自动化测试

    +   自动化配置

    +   配置管理

    +   自动化部署

    +   测量

    +   适应虚拟化（公有/私有云）

1.  云中的三种主要服务模型：

    +   **基础设施即服务** (**IaaS**)

    +   **平台即服务** (**PaaS**)

    +   **软件即服务** (**SaaS**)

1.  AWS 是当前最大的公有云服务平台。AWS 提供多种服务，从计算、存储到机器学习和分析，所有这些服务都具有高度的可扩展性和可靠性。使用 AWS 最重要的部分是 *按需付费模式*。你无需投资硬件，直接部署服务并按使用时长付费。当你关闭并移除服务时，不会产生费用——这点非常棒。

# 第二章： 部署你的第一个 Web 应用程序

1.  如果你没有 AWS 云账户，访问 [www.aws.amazon.com](https://aws.amazon.com/) 并创建一个免费套餐账户。按照[`aws.amazon.com/`](https://aws.amazon.com/)上的逐步指南操作。你需要提供信用卡或借记卡信息才能创建 AWS 账户。

1.  访问 [console.aws.amazon.com](https://us-east-1.signin.aws.amazon.com/oauth?SignatureVersion=4&X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAJMOATPLHVSJ563XQ&X-Amz-Date=2018-08-27T09%3A42%3A05.017Z&X-Amz-Signature=9a2851741438a5ac794ebce02b2f9dac6adf96b92ec4e336cc5a35322ede9064&X-Amz-SignedHeaders=host&client_id=arn%3Aaws%3Aiam%3A%3A015428540659%3Auser%2Fhomepage&redirect_uri=https%3A%2F%2Fconsole.aws.amazon.com%2Fconsole%2Fhome%3Fstate%3DhashArgs%2523%26isauthcode%3Dtrue&response_type=code&state=hashArgs%23) 并选择 AWS 计算服务来创建你的第一个 EC2 实例。在控制台上点击 启动实例 按钮，并按照步骤选择 AMI、实例类型（在此选择免费套餐），接着选择实例详情、存储详情、标签和安全组。在此练习中，你可以选择默认选项，因为我们的目标仅仅是熟悉控制台门户，以便通过 DevOps 实践实现自动化。

1.  按照本章中 *创建我们的第一个 Web 服务器* 部分提供的逐步指南，使用 AWS CLI 创建你的第一个 AWS 实例。

1.  跟随本章中 *创建一个简单的 Hello World Web 应用程序* 部分的步骤。你可以从以下链接下载应用程序的示例代码：

    +   [`raw.githubusercontent.com/yogeshraheja/Effective-DevOps-with-AWS/master/Chapter02/helloworld.js`](https://raw.githubusercontent.com/yogeshraheja/Effective-DevOps-with-AWS/master/Chapter02/helloworld.js)

    +   [`raw.githubusercontent.com/yogeshraheja/Effective-DevOps-with-AWS/master/Chapter02/helloworld.conf`](https://raw.githubusercontent.com/yogeshraheja/Effective-DevOps-with-AWS/master/Chapter02/helloworld.conf)

1.  使用`ec2-metadata --instance-id`查找你的 AWS 实例 ID，然后通过修改你的实例 ID 执行以下命令：`aws ec2 terminate-instances --instance-ids <YOUR AWS INSTANCE ID>`。

# 第三章：将基础设施视为代码

1.  IaC 代表基础设施即代码（Infrastructure as Code）。这是一个将你的基础设施对象（例如 EC2 实例、VPC 网络、子网、负载均衡器、存储、应用部署和编排）作为基础设施代码进行处理的过程。IaC 允许在非常短的时间内对整个环境中的基础设施垂直进行更改、复制和回滚。

1.  打开 CloudFormation 模板，访问[`console.aws.amazon.com/cloudformation`](https://console.aws.amazon.com/cloudformation)，然后点击“创建堆栈”按钮。接下来，使用位于[`raw.githubusercontent.com/yogeshraheja/Effective-DevOps-with-AWS/master/Chapter03/EffectiveDevOpsTemplates/helloworld-cf-template-part-1.py`](https://raw.githubusercontent.com/yogeshraheja/Effective-DevOps-with-AWS/master/Chapter03/EffectiveDevOpsTemplates/helloworld-cf-template-part-1.py)的 Python 文件，创建一个`helloworld-cf.template`模板文件。完成后，将模板上传至 Amazon S3。为你的堆栈提供一个名称，然后添加一个 SSH 密钥对及其他可以默认的附加信息。现在审查信息并点击“创建”。当模板创建完成后，点击“输出”标签，点击 Weburl 链接，它将带你到应用程序主页。

提示：通过将脚本的输出保存到`python helloworld-cf-template.py > helloworld-cf.template`文件中生成 CloudFormation 模板。

1.  市场上有多个 SCM（源代码管理）产品，包括 GitLab、BitBucket、GitHub，甚至公共云提供的 SCM 服务。在这里，我们将使用最流行的 SCM 服务之一：GitHub。请在[`github.com`](https://github.com)上创建一个免费的 GitHub 账户。完成后，登录到你的 GitHub 账户，创建第一个名为`helloworld`的公共仓库。

1.  为您的支持平台安装 Git 包，并使用 `git clone <github repository URL>` 克隆之前创建的 GitHub 仓库，您可以在 GitHub 控制台中找到该仓库的 URL。现在，将您的 `helloworld-cf.template` 文件复制到仓库中，并进行 `git add` 和 `git commit` 操作。现在，您可以将本地仓库文件推送到您的 GitHub 账户。为此，执行 `git push` 推送已提交的文件，并通过检查 GitHub 仓库确认。

1.  Ansible 是一款简单、强大且易于学习的配置管理工具，广泛应用于系统/云工程师和 DevOps 工程师，用于自动化处理他们日常的重复性任务。Ansible 的安装非常简单，采用无代理模式运行。

    在 Ansible 中，模块是创建 Ansible 代码文件（YAML 格式）的基本构件。这些文件被称为 Ansible Playbooks（Ansible 剧本）。多个 Ansible 剧本按定义良好的目录结构组织，在 Ansible 中称为 `roles`，其中 roles 是存放 Ansible 代码的目录结构，包括 Ansible 剧本、变量、静态/动态文件等。Ansible 中还包含许多其他对象，包括 Ansible Vault、Ansible Galaxy，以及一个名为 **Ansible Tower** 的 Ansible 图形界面。您可以通过访问 [`docs.ansible.com`](https://docs.ansible.com/) 进一步探索这些对象。

# 第四章：使用 Terraform 实现基础设施即代码

1.  Terraform 是一款高级基础设施工具，主要用于安全、高效地构建、修改和版本控制基础设施。Terraform 不是配置管理工具，因为它专注于基础设施层，并允许诸如 Puppet、Chef、Ansible 和 Salt 等工具执行应用程序部署和编排。

1.  HashiCorp 并未为操作系统提供原生包。Terraform 以单个二进制文件的形式分发，打包在一个 ZIP 压缩包中，可以从[`www.terraform.io/downloads.html`](https://www.terraform.io/downloads.html)下载。下载后，解压 `.zip` 文件并将其放置到 `/usr/bin Linux` 二进制路径下。完成此操作后，运行 `terraform -v` 来确认安装的 Terraform 版本。

1.  为了使用 Terraform 配置 AWS 实例，您需要通过在 `.tf` 文件中创建 `provider` 块来初始化 AWS 提供程序。然后运行 `terraform init`。初始化成功后，您需要继续开发一个 Terraform 模板，并使用 `resources`。在这种情况下，您需要使用 `aws_instance` 资源类型及其适当的属性。完成此操作后，进行验证并规划，最后应用您的 Terraform 模板来创建您的第一个 AWS 实例。

1.  为了使用 Ansible 配置 Terraform，你需要使用一个 **provider** 来初始化平台；使用 **resources** 来创建与平台相关的服务；最后使用 **provisioner** 来与创建的服务建立连接，安装 Ansible，并运行 `ansible-pull` 在系统上执行 Ansible 代码。你可以参考以下链接获取一个示例 Terraform 模板：[`raw.githubusercontent.com/yogeshraheja/EffectiveDevOpsTerraform/master/fourthproject/helloworldansiblepull.tf`](https://raw.githubusercontent.com/yogeshraheja/EffectiveDevOpsTerraform/master/fourthproject/helloworldansiblepull.tf)。

# 第五章：添加持续集成和持续部署

1.  CI、CD 和持续交付这些术语可以定义如下：

    +   **持续集成**：CI 流水线将使我们能够自动和持续地测试提议的代码更改。这将节省开发人员和 QA 的时间，他们不再需要进行那么多手动测试。它还使得代码更改的集成变得更加容易。

    +   **持续部署**：在 CD 中，你大大加速了 DevOps 提供的反馈回路过程。以高速将新代码发布到生产环境可以让你收集真实的客户指标，这往往会暴露出新的和意想不到的问题。

    +   **持续交付**：为了构建我们的持续交付流水线，我们首先将为生产环境创建一个 CloudFormation 堆栈。然后，我们将在 CodeDeploy 中添加一个新的部署组，这将使我们能够将代码部署到新的 CloudFormation 堆栈中。最后，我们将升级流水线，加入审批流程，以便将代码部署到生产环境，并包括生产部署阶段本身。

1.  Jenkins 是最广泛使用的集成工具之一，用于运行我们的 CI 流水线。经过超过 10 年的开发，Jenkins 长期以来一直是实践持续集成的领先开源解决方案。Jenkins 以其丰富的插件生态系统而闻名，并经历了一个重大的新版本发布（Jenkins 2.x），该版本将焦点放在了一些非常 DevOps 相关的功能上，包括能够创建本地交付流水线，并进行检查和版本控制。它还提供了与源代码控制系统（如 GitHub）更好的集成。

1.  为了实现我们的持续部署流水线，我们将关注两个新的 AWS 服务——CodePipeline 和 CodeDeploy：

    +   **CodePipeline** 让我们创建我们的部署流水线。我们将告诉它像之前一样从 GitHub 获取我们的代码，并将其发送到 Jenkins 进行 CI 测试。然而，Jenkins 不再仅仅返回结果到 GitHub，我们将会在 AWS CodeDeploy 的帮助下将代码部署到我们的 EC2 实例。

    +   **CodeDeploy**是一项服务，让我们可以将代码正确地部署到 EC2 实例上。通过添加一定数量的配置文件和脚本，我们可以使用 CodeDeploy 可靠地部署和测试代码。得益于 CodeDeploy，我们无需担心部署顺序中的任何复杂逻辑。它与 EC2 紧密集成，知道如何在多个实例之间执行滚动更新，并且如果需要，还能进行回滚操作。

更多详细信息，请参考本章的*构建持续部署流水线*部分。

# 第六章：扩展你的基础设施

1.  不，这并不总是最佳选择，因为多层应用意味着需要管理更多的组件。如果你的应用在单体架构下运行良好，且你能接受短暂的停机时间，并且流量不会随着时间增加，那么你可以考虑让它继续按现状运行。

1.  在本书使用的多层架构方法中，所有软件都在一个 ZIP 文件中，而在微服务和无服务器架构中，软件被拆分为多个部分。例如，在一个电子商务软件（用于向用户展示内容的服务）中，管理后端以添加新产品的部分是一个服务，而管理支付的部分是另一个服务，等等。

1.  如果你不熟悉该服务，可能会觉得有些困难。然而，AWS 提供了大量文档和视频资源。此外，在本书中，我们展示了如何使用一组基本服务在**多层架构**中打破经典的单体架构方法。

1.  对于 NLB，这个说法是正确的，但如果你使用 ALB 或 CLB，必须先预热。如果流量每五分钟增长超过 50%，你也需要这样做。

1.  使用证书管理器是免费的，除非你需要**请求一个私有证书**，经典的 SSL 证书每年也可能需要 500 美元。

1.  每个 AWS 区域都由可用区（AZ）组织，每个可用区都是一个独立的数据中心。因此，某一个可用区发生问题的情况很少发生，但同一时刻在多个可用区发生问题的概率较低。每个子网只能属于一个可用区，因此将每个组件至少放置在两个，或者最好放置在三个可用区中，会更加方便。

# 第七章：在 AWS 中运行容器

1.  Docker 是一个容器平台，用于构建、分发和运行容器化应用程序。Docker 引擎的四个重要组件如下：

+   +   **容器**：读写模板

    +   **图像**：只读模板

    +   **网络**：容器的虚拟网络

    +   **卷**：容器的持久存储

1.  Docker CE 可以在多种平台上安装，包括 Linux、Windows 和 MacOS。请参考[`docs.docker.com/install/`](https://docs.docker.com/install/)，这是官方的 Docker 链接，点击选择你的平台，然后按照说明安装并配置 Docker CE 的最新版本。

通过运行`docker --version`命令，确认已安装的 Docker CE 版本。

1.  使用 Dockerfile [`github.com/yogeshraheja/helloworld/blob/master/Dockerfile`](https://github.com/yogeshraheja/helloworld/blob/master/Dockerfile) 并使用`docker build`命令创建镜像。这个新创建的镜像是用于 Hello World 应用程序的镜像。通过使用`docker run -d -p 3000:3000 <image-name>`暴露外部端口来创建容器。完成后，使用`curl`或通过浏览器访问公共 IP 和端口`3000`来检查并确认 Web 服务器的输出。

1.  使用您的凭证登录到 AWS 账户，并从服务选项卡中选择 ECS 服务。在这里，您会看到创建 Amazon ECS 集群和 Amazon ECR 仓库的选项。此时，点击“仓库”并创建您的第一个 ECR 仓库。屏幕上还会显示一些您可以用于对 ECR 执行操作的命令。类似地，点击“集群”选项卡，然后在 ECS 屏幕上点击“创建集群”。在这里，选择 Windows、Linux 或仅网络的集群选项，点击“下一步”，并填写您的选择的详细信息。这些信息包括集群名称、配置模型、EC2 实例类型、实例数量等。完成该过程后，点击“创建”。几分钟后，您的 ECS 集群将准备就绪。在本章中，我们通过 CloudFormation 展示了这一过程。如果您有兴趣使用相同的流程设置 ECS 集群，可以按照本章*创建 ECS 集群*部分提供的步骤操作。

# 第八章：加强您的 AWS 环境的安全性

1.  在开始构建基础设施之前，强烈建议您*锁定*根账户（即与注册邮箱绑定的账户）。然后，创建具有必要权限的 IAM 用户和组，并为根账户和 IAM 用户启用多因素认证（MFA），而不仅仅是用户名和密码。

1.  您应该启用 CloudTrail 以注册 IAM 用户和角色的操作，并启用 VPC 流日志以监控和记录网络流量。

1.  不是的；另外还有 WAF，一个在 TCP/IP 协议第 7 层工作的应用程序防火墙。

1.  您必须遵循一些最佳实践来配置应用程序，将应用程序暴露给互联网的表面降到最低，并实现水平扩展。同时，也有 WAF 速率规则帮助限制恶意的 DDoS 攻击。

1.  理论上，您可以这么做，但将它们分布在私有和公共子网之间更为方便，以仅将必要的资源暴露到互联网。其他任何资源都应该保持私密。此外，最佳做法是将您的应用程序的各部分分布在多个可用区中。这意味着实际上是使用多个数据中心。基于这些原因，并且由于一个子网只能位于一个可用区中，您必须使用多个子网。
