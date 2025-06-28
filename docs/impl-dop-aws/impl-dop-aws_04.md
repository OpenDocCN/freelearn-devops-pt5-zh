# 第四章。通过持续集成更快地构建、测试和发布

本章的重点将是快速迭代的价值：根据博伊德定律，快速优于高质量迭代（你可能记得在第一章提到的 OODA 原则，*什么是 DevOps，以及你是否应该关心？*）。

迭代是指软件开发周期，从代码编写、发布（提交到版本控制）、编译（如果需要）、测试，最后部署的过程。

**持续集成**（**CI**）定义了开发人员应该采用的常规和必要的工具，以使迭代尽可能快速。

让我们从人因开始：

+   使用版本控制（例如 Git）

+   提交较小的更改，更频繁

+   首先在本地测试

+   进行同伴代码审查

+   在问题解决之前暂停其他团队活动

然后添加一些自动化（一个 CI 服务器）：

+   监控版本控制的变化（例如 Git 提交）

+   拉取已更改的代码

+   编译并运行测试

    +   成功时，构建工件

    +   失败时，通知团队并暂停管道

+   重复

提交较小的更改有助于更早发现问题，并更容易解决它们；开发者更频繁地收到关于他们工作的反馈，这增加了他们对自己代码状态的信心。

在本地进行测试，尽可能地这样做，能大大减少由于 CI 管道在小问题上卡壳而带来的团队分心。

代码审查在许多方面都是有益的。它们消除不良的编码习惯，因为同伴会确保代码符合约定的标准。它们增加了可见性；同伴能够更多地接触到他人的工作。它们帮助发现机器可能错过的错误。

**丰田方式**教我们在发现问题时要*停线*。在 CI 方面，这意味着在错误发生时暂停管道，集中资源修复这些错误。刚开始时，这看起来可能像是减少生产力并放慢整个过程的明显方式，但一次又一次地证明，最初的开销最终是值得的。通过这种方式，你可以将技术债务保持在最低限度；在开发过程中不断改进代码，防止问题积累并在后期重新浮现。现在是时候重申之前提到的**本地测试**的观点了。你可能不想因为一些微不足道的错误打扰同事，这些错误本可以在提交之前轻松发现。

当你成功建立了这个团队纪律（最困难的部分）之后，是时候通过设置 CI 管道加入一些自动化的味道了。

CI 服务器不知疲倦地监控你的代码仓库，并通过执行一组任务对变化作出反应。我相信显而易见的是，这为工程师节省了大量时间和精力，更不用说他们避免了处理这种单调工作的麻烦。

一个管道，比如在 Jenkins 中，通常由多个阶段组成：每个阶段代表拉取最新代码、在其上运行构建任务、执行测试然后构建产物，且每个阶段的运行都依赖于前一个阶段成功完成。

这通常描述了工程师的习惯与一些工具如何大大改善软件开发周期。持续集成帮助我们更好地协作、编写更好的代码、更频繁地发布，并更快地获取反馈。

用户希望快速发布新功能，开发人员希望看到自己工作的成果——每个人都能从中受益。

我们已经讨论了理论，现在让我们将注意力集中在本章的标题上。我们将使用之前学到的 Terraform 和 Salt 技能，在 AWS 上部署一个包含 Jenkins（v2）CI 服务器的 CI 环境。

Jenkins（参考：[`jenkins.io`](https://jenkins.io)）是一个流行且成熟的开源项目，专注于自动化。它有许多集成，支持各种平台和编程语言。认识 Jenkins：[`wiki.jenkins-ci.org/display/JENKINS/Meet+Jenkins`](https://wiki.jenkins-ci.org/display/JENKINS/Meet+Jenkins)。

我们的 CI 环境部署可以分为三个主要阶段：

1.  准备一个**基础设施即代码**部署：

    +   编写**Terraform**模板以提供 VPC 和 EC2 实例

    +   编写**Salt**状态以在 EC2 实例上安装 Jenkins、NGINX 和其他软件

1.  部署 IaC：

    +   部署 Terraform 模板和 Salt 状态

1.  设置 CI：

    +   配置 Jenkins 管道以实现演示应用程序的持续集成

# 准备 IaC

根据我们的*基础设施即代码*（IaC）原则，这次部署也将主要通过模板来完成。我们将尝试复用上一章的一些 Terraform 和 Salt 代码。

## Terraform 模板

对于这个特定的设置，我们可以简化我们的模板，因为我们只需要 VPC、一些网络配置以及一个 EC2 实例。

让我们浏览一下我们*TF*仓库中的文件：

### 变量

我们需要的几个变量可以分为与 VPC 和 EC2 相关的：

### 注意

请参考：[`github.com/PacktPublishing/Implementing-DevOps-on-AWS/blob/master/5585_04_CodeFiles/Terraform/variables.tf`](https://github.com/PacktPublishing/Implementing-DevOps-on-AWS/blob/master/5585_04_CodeFiles/Terraform/variables.tf)。

**VPC**

```
variable "aws-region" { 
  type = "string" 
  description = "AWS region" 
} 
variable "vpc-cidr" { 
  type = "string" 
  description = "VPC CIDR" 
} 
variable "vpc-name" { 
  type = "string" 
  description = "VPC name" 
} 
variable "aws-availability-zones" { 
  type = "string" 
  description = "AWS zones" 
} 

```

**EC2**

```
variable "jenkins-ami-id" { 
  type="string" 
  description = "EC2 AMI identifier" 
} 
variable "jenkins-instance-type" { 
  type = "string" 
  description = "EC2 instance type" 
} 
variable "jenkins-key-name" { 
  type = "string" 
  description = "EC2 ssh key name" 
} 

```

### 变量（值）

根据基本的变量定义，我们现在提供一些值：

### 注意

请参考：[`github.com/PacktPublishing/Implementing-DevOps-on-AWS/blob/master/5585_04_CodeFiles/Terraform/terraform.tfvars`](https://github.com/PacktPublishing/Implementing-DevOps-on-AWS/blob/master/5585_04_CodeFiles/Terraform/terraform.tfvars)。

**VPC**

我们将把部署保持在美国东部（US East）：

```
aws-region = "us-east-1" 
vpc-cidr = "10.0.0.0/16" 
vpc-name = "Terraform" 
aws-availability-zones = "us-east-1b,us-east-1c" 

```

**EC2**

一个 Nano 实例足以进行测试。请确保引用的密钥对存在：

```
jenkins-ami-id = "ami-6869aa05" 
jenkins-instance-type = "t2.nano" 
jenkins-key-name = "terraform" 

```

### 资源

### 注意

请参阅：[`github.com/PacktPublishing/Implementing-DevOps-on-AWS/blob/master/5585_04_CodeFiles/Terraform/resources.tf`](https://github.com/PacktPublishing/Implementing-DevOps-on-AWS/blob/master/5585_04_CodeFiles/Terraform/resources.tf)

#### 创建 VPC

作为标准（良好）实践，我们将所有资源都创建在 VPC 内：

```
# Set a Provider 
provider "aws" { 
  region = "${var.aws-region}" 
} 

# Create a VPC 
resource "aws_vpc" "terraform-vpc" { 
  cidr_block = "${var.vpc-cidr}" 

  tags { 
    Name = "${var.vpc-name}" 
  } 
} 

```

#### 添加网络组件

我们添加了一个网关、一张路由表，以及一个互联网连接子网，Jenkins 实例将从这里启动：

**IGW**

```
# Create an Internet Gateway 
resource "aws_internet_gateway" "terraform-igw" { 
  vpc_id = "${aws_vpc.terraform-vpc.id}" 
} 

```

**路由表**

```
# Create public route tables 
resource "aws_route_table" "public" { 
  vpc_id = "${aws_vpc.terraform-vpc.id}" 
  route { 
    cidr_block = "0.0.0.0/0" 
    gateway_id = "${aws_internet_gateway.terraform-igw.id}" 
  } 

  tags { 
    Name = "Public" 
  } 
} 

```

**子网**

```
# Create and associate public subnets with a route table 
resource "aws_subnet" "public-1" { 
  vpc_id = "${aws_vpc.terraform-vpc.id}" 
  cidr_block = "${cidrsubnet(var.vpc-cidr, 8, 1)}" 
  availability_zone = "${element(split(",",var.aws-availability-zones), count.index)}" 
  map_public_ip_on_launch = true 

  tags { 
    Name = "Public" 
  } 
} 

resource "aws_route_table_association" "public-1" { 
  subnet_id = "${aws_subnet.public-1.id}" 
  route_table_id = "${aws_route_table.public.id}" 
} 

```

#### 添加 EC2 节点及相关资源

我们的 Jenkins 节点的安全组需要允许 HTTP/S 访问以及 SSH 访问，方便我们在需要时访问命令行：

**安全组**

```
resource "aws_security_group" "jenkins" { 
  name = "jenkins" 
  description = "ec2 instance security group" 
  vpc_id = "${aws_vpc.terraform-vpc.id}" 

  ingress { 
    from_port = "22" 
    to_port = "22" 
    protocol = "tcp" 
    cidr_blocks = ["0.0.0.0/0"] 
  } 

  ingress { 
    from_port = "80" 
    to_port = "80" 
    protocol = "tcp" 
    cidr_blocks = ["0.0.0.0/0"] 
  } 

  ingress { 
    from_port = "443" 
    to_port = "443" 
    protocol = "tcp" 
    cidr_blocks = ["0.0.0.0/0"] 
  } 

  egress { 
    from_port = 0 
    to_port = 0 
    protocol = "-1" 
    cidr_blocks = ["0.0.0.0/0"] 
  } 

} 

```

**IAM 角色**

我们将使用 IAM 角色来授予 Jenkins 访问 AWS 服务的权限：

```
resource "aws_iam_role" "jenkins" { 
    name = "jenkins" 
    path = "/" 
    assume_role_policy = <<EOF 
{ 
  "Version": "2012-10-17", 
  "Statement": [ 
    { 
      "Action": "sts:AssumeRole", 
      "Principal": { 
        "Service": "ec2.amazonaws.com" 
      }, 
      "Effect": "Allow", 
      "Sid": "" 
    } 
  ] 
} 
EOF 
} 

```

**IAM 角色策略**

这个策略将允许 Jenkins 从 CodeCommit 仓库读取，并在 S3 存储桶上执行所有操作（除了删除）：

```
resource "aws_iam_role_policy" "jenkins" { 
    name = "jenkins" 
    role = "${aws_iam_role.jenkins.id}" 
    policy = <<EOF 
{ 
    "Version": "2012-10-17", 
    "Statement": [ 
       { 
            "Effect": "Allow", 
            "Action": [ 
                "codecommit:Get*", 
                "codecommit:GitPull", 
                "codecommit:List*" 
            ], 
            "Resource": "*" 
       }, 
       { 
            "Effect": "Allow", 
            "NotAction": [ 
                "s3:DeleteBucket" 
            ], 
            "Resource": "*" 
        } 
    ] 
} 
EOF 
} 

```

**IAM 配置文件**

```
resource "aws_iam_instance_profile" "jenkins" { 
    name = "jenkins" 
    roles = ["${aws_iam_role.jenkins.name}"] 
} 

```

**EC2 实例**

在这里，我们定义了一个单实例，并且其引导 UserData 脚本：

```
resource "aws_instance" "jenkins" { 
    ami = "${var.jenkins-ami-id}" 
    instance_type = "${var.jenkins-instance-type}" 
    key_name = "${var.jenkins-key-name}" 
    vpc_security_group_ids = ["${aws_security_group.jenkins.id}"] 
    iam_instance_profile = "${aws_iam_instance_profile.jenkins.id}" 
    subnet_id = "${aws_subnet.public-1.id}" 
    tags { Name = "jenkins" } 

```

在这里，我们设置了启动 EC2 实例所需的属性，比如实例类型、使用的 AMI、安全组、子网等等。

接下来，我们添加了引导 shell 脚本，帮助我们安装所需的软件包、检出 Git 仓库并运行 Salt：

```
    user_data = <<EOF 
#!/bin/bash 
set -euf -o pipefail 
exec 1> >(logger -s -t $(basename $0)) 2>&1 
# Install Git and set CodeComit connection settings 
# (required for access via IAM roles) 
yum -y install git 
git config --system credential.helper '!aws codecommit credential-helper $@' 
git config --system credential.UseHttpPath true 
# Clone the Salt repository 
git clone https://git-codecommit.us-east-1.amazonaws.com/v1/repos/salt /srv/salt; chmod 700 /srv/salt 
# Install SaltStack 
yum -y install https://repo.saltstack.com/yum/amazon/salt-amzn-repo-latest-1.ami.noarch.rpm 
yum clean expire-cache; yum -y install salt-minion; chkconfig salt-minion off 
# Put custom minion config in place (for enabling masterless mode) 
cp -r /srv/salt/minion.d /etc/salt/ 
# Trigger a full Salt run 
salt-call state.apply 
EOF 

    lifecycle { create_before_destroy = true } 
} 

```

**弹性 IP**

最后，我们为 Jenkins 配置一个静态 IP：

```
resource "aws_eip" "jenkins" { 
  instance = "${aws_instance.jenkins.id}" 
  vpc      = true 
} 

```

### 输出

### 注意

请参阅：[`github.com/PacktPublishing/Implementing-DevOps-on-AWS/blob/master/5585_04_CodeFiles/Terraform/outputs.tf`](https://github.com/PacktPublishing/Implementing-DevOps-on-AWS/blob/master/5585_04_CodeFiles/Terraform/outputs.tf)

一些有用的输出，用来提供 Jenkins 节点的地址：

```
output "VPC ID" { 
  value = "${aws_vpc.terraform-vpc.id}" 
} 

output "JENKINS EIP" { 
  value = "${aws_eip.jenkins.public_ip}" 
} 

```

这就是我们定义的 VPC 基础设施。现在我们可以开始配置 Salt 和应用栈了。

## SaltStack 代码

你应该还记得上一章我们提到的那个最喜爱的配置管理工具。我们将使用 SaltStack 来为我们配置 EC2 Jenkins 节点。

### 状态

### 注意

请参阅：[`github.com/PacktPublishing/Implementing-DevOps-on-AWS/tree/master/5585_04_CodeFiles/CodeCommit/salt/states`](https://github.com/PacktPublishing/Implementing-DevOps-on-AWS/tree/master/5585_04_CodeFiles/CodeCommit/salt/states)

**top.sls**

我们正在使用一个单一的从属节点（minion），所有的状态都应用于它：

```
base: 
  '*': 
    - users 
    - yum-s3 
    - jenkins 
    - nginx 
    - docker 

```

**users**

我们添加了一个 Linux 用户账户，配置了其 SSH 密钥和 **sudo** 权限：

```
veselin: 
  user.present: 
    - fullname: Veselin Kantsev 
    - uid: {{ salt'pillar.get' }} 
... 

```

**yum-s3**

作为 CI 流水线的一部分，我们将在 S3 中存储 RPM 制品。Cob（参考：[`github.com/henrysher/cob`](https://github.com/henrysher/cob)）是一个 Yum 包管理器插件，它使得可以通过 IAM 角色访问基于 S3 的 RPM 仓库。

我们部署插件、其配置和一个仓库定义（目前禁用）作为管理文件：

```
yum-s3_cob.py: 
  file.managed: 
    - name: /usr/lib/yum-plugins/cob.py 
    - source: salt://yum-s3/files/cob.py 

yum-s3_cob.conf: 
  file.managed: 
    - name: /etc/yum/pluginconf.d/cob.conf 
    - source: salt://yum-s3/files/cob.conf 

yum-s3_s3.repo: 
  file.managed: 
    - name: /etc/yum.repos.d/s3.repo 
    - source: salt://yum-s3/files/s3.repo 

```

**Jenkins**

这里是主角——Jenkins 先生。我们在 CI 流水线中使用 Docker，因此接下来是 `include`。Docker 使我们能够将不同的流水线步骤隔离开来，这让依赖管理变得更加容易，并有助于保持 Jenkins 节点的干净。

```
include: 
  - docker 

```

同时我们确保安装 Java 和其他一些前提条件：

```
jenkins_prereq: 
  pkg.installed: 
    - pkgs: 
      - java-1.7.0-openjdk 
      - gcc 
      - make 
      - createrepo 

```

然后，安装 Jenkins 本身：

```
jenkins: 
  pkg.installed: 
    - sources: 
      - jenkins: http://mirrors.jenkins-ci.org/redhat-stable/jenkins-2.7.1-1.1.noarch.rpm 
    - require: 
      - pkg: jenkins_prereq 
... 

```

**NGINX**

我们将使用 NGINX 作为反向代理和 SSL 终止点。这并不是说 Jenkins 不能单独提供服务，只是将角色分开被认为是更好的实践：

```
include: 
  - jenkins 

nginx: 
  pkg.installed: [] 
... 
{% for FIL in ['crt','key'] %} 
/etc/nginx/ssl/server.{{ FIL }}: 
... 
{% endfor %} 

```

**Docker**

是时候提到 Docker 了，鉴于它如今的（应得的）流行。它非常适合我们的 CI 需求，为可能需要的各种测试和构建提供隔离的环境：

```
docker: 
  pkg.installed: [] 

  service.running: 
    - enable: True 
    - reload: True 

```

### Pillars

### 注意

请参考：[`github.com/PacktPublishing/Implementing-DevOps-on-AWS/tree/master/5585_04_CodeFiles/CodeCommit/salt/pillars`](https://github.com/PacktPublishing/Implementing-DevOps-on-AWS/tree/master/5585_04_CodeFiles/CodeCommit/salt/pillars)。

**top.sls**

我们的独立 minion 获取了所有内容：

```
base: 
  '*': 
    - users 
    - nginx 

```

**users**

设置 Linux 账户的密码哈希和一致的 UID：

```
users: 
  veselin: 
    uid: 5001 
    password: ... 

```

**NGINX**

我们将 SSL 数据存储在此 Pillar 中：

```
nginx: 
  crt: | 
    -----BEGIN CERTIFICATE----- 
    ... 
    -----END CERTIFICATE----- 
  key: | 
    -----BEGIN RSA PRIVATE KEY----- 
    ... 
    -----END RSA PRIVATE KEY----- 

```

### Minion 配置

### 注意

请参考：[`github.com/PacktPublishing/Implementing-DevOps-on-AWS/tree/master/5585_04_CodeFiles/CodeCommit/salt/minion.d`](https://github.com/PacktPublishing/Implementing-DevOps-on-AWS/tree/master/5585_04_CodeFiles/CodeCommit/salt/minion.d)。

**masterless.conf**

我们仍然在使用 Salt 的独立（无主）模式，因此这是我们额外的`minion`配置：

```
file_client: local 
file_roots: 
  base: 
    - /srv/salt/states 
pillar_roots: 
  base: 
    - /srv/salt/pillars 

```

由于前面的所有代码，我们应该能够运行 Terraform 并最终得到一个准备好使用的 Jenkins 服务。

让我们试试看。

# 部署基础设施即代码（IaC）

我们首先创建一个 Terraform EC2 密钥对和一个 Terraform IAM 用户，方法与前几章相同（不要忘记写下访问/密钥 API）。然后，我们授予 IAM 用户对 EC2、IAM、S3 和 CodeCommit 服务的操作权限：

### 注意

请参考：[`github.com/PacktPublishing/Implementing-DevOps-on-AWS/blob/master/5585_04_CodeFiles/Terraform/iam_user_policy.json`](https://github.com/PacktPublishing/Implementing-DevOps-on-AWS/blob/master/5585_04_CodeFiles/Terraform/iam_user_policy.json)。

```
{ 
    "Version": "2012-10-17", 
    "Statement": [ 
      { 
          "Effect": "Allow", 
          "NotAction": [ 
              "codecommit:DeleteRepository" 
          ], 
          "Resource": "*" 
      }, 
      { 
          "Effect": "Allow", 
          "NotAction": [ 
              "s3:DeleteBucket" 
          ], 
          "Resource": "*" 
      }, 
      { 
          "Sid": "Stmt1461764665000", 
          "Effect": "Allow", 
          "Action": [ 
              "ec2:AllocateAddress", 
... 

```

然后我们将一个 SSH 公钥与用户关联（按照上一章的截图）以允许 `codecommit` 仓库访问。

接下来，我们需要使用之前生成的密钥设置我们的 AWS CLI 环境：

```
$ export AWS_ACCESS_KEY_ID='user_access_key'
$ export AWS_SECRET_ACCESS_KEY='user_secret_access_key'
$ export AWS_DEFAULT_REGION='us-east-1'

```

现在我们应该能够使用 CLI 工具并创建我们的 SaltStack 仓库：

```
$ aws codecommit create-repository --repository-name salt 
      --repository-description "SaltStack repo"
{
"repositoryMetadata": {
"repositoryName": "salt",
"cloneUrlSsh": 
    "ssh://git-codecommit.us-east-1.amazonaws.com/v1/repos/salt",
...

```

我们在本地克隆仓库：

```
$ git clone ssh://SSH_KEY_ID@git-codecommit.us-east-
     1.amazonaws.com/v1/repos/salt
Cloning into 'salt'...
warning: You appear to have cloned an empty repository.
Checking connectivity... done.

```

（其中`SSH_KEY_ID`是我们在这里上传公钥后看到的）

最后，您可以复制本章的现成 Salt 代码示例，提交并推送到 `codecommit` 仓库。

### 注意

请参考：[`github.com/PacktPublishing/Implementing-DevOps-on-AWS/tree/master/5585_04_CodeFiles/CodeCommit/salt`](https://github.com/PacktPublishing/Implementing-DevOps-on-AWS/tree/master/5585_04_CodeFiles/CodeCommit/salt)

在 SaltStack 仓库同步后，我们可以继续进行 Terraform 和引导过程。在我们的 TF 模板文件夹中，我们运行熟悉的命令序列：

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
 VPC ID      = vpc-xxxxxx

```

最后，我们得到 Jenkins 节点的 IP 地址，之后需要将其解析为主机名（例如，通过 `nslookup` 命令）。在浏览器中加载该地址，您应该能看到 Jenkins 界面。

# 设置 CI

成功部署 Terraform 后，我们进入服务配置阶段。更具体地说，是 Jenkins 和集成流水线的配置。

## Jenkins 初始化

在 Jenkins 第一次运行时，我们需要完成一个简短的设置过程。首先，我们需要 SSH 登录到节点，并获取存储在 `/var/lib/jenkins/secrets/initialAdminPassword` 中的管理员密码：

![Jenkins 初始化](img/image_04_001.jpg)

我们主要关心的是与建议插件一起包含的流水线插件组：

![Jenkins 初始化](img/image_04_002.jpg)

插件安装完成后，是时候创建第一个用户了：

![Jenkins 初始化](img/image_04_003.jpg)

这样，初始化过程完成，Jenkins 已经准备好使用：

![Jenkins 初始化](img/image_04_004.jpg)

## 编写一个示例应用

在配置 CI 流水线之前，最好先有些集成工作可做。一个简单的 Hello World 类型的 PHP 代码就足够了，所以我诚恳地向所有 PHP 开发者道歉，向您展示我们的示例应用源代码：

### 注意

请参考：[`github.com/PacktPublishing/Implementing-DevOps-on-AWS/tree/master/5585_04_CodeFiles/CodeCommit/demo-app`](https://github.com/PacktPublishing/Implementing-DevOps-on-AWS/tree/master/5585_04_CodeFiles/CodeCommit/demo-app)。

```
src/index.php: 
<?php 

function greet($name) { 
  return "Hello $name!"; 
} 

$full_name = "Bobby D"; 
 greet ($full_name); 

Clapping fades... 
And naturally, a unit test for it: 
tests/indexTest.php: 

<?php 
require_once "src/index.php"; 

class IndexTest extends PHPUnit_Framework_TestCase 
{ 
  public function testGreet() { 
    global $full_name; 
    $expected = "Hello $full_name!"; 
    $actual = greet($full_name); 
    $this->assertEquals($expected, $actual); 
    } 
}
```

在我们的 `demo-app` 文件夹中有一个第三个文件，名字很有意思，叫做 `Jenkinsfile`，我们稍后会讨论它。

现在让我们将代码放入代码仓库：

```
$ aws codecommit create-repository --repository-name demo-app 
      --repository-description "Demo app"
{
"repositoryMetadata": {
"repositoryName": "demo-app",
"cloneUrlSsh": 
    "ssh://git-codecommit.us-east-1.amazonaws.com/v1/repos/demo-app"
...
Then we clone it locally (replace SSH_KEY_ID as before):
$ git clone ssh://SSH_KEY_ID@git-codecommit.us-east-
      1.amazonaws.com/v1/repos/demo-app
...

```

最后，我们将 `demo-app` 代码放入空的仓库，提交并推送所有更改到 codecommit。

## 定义流水线

是时候决定 CI 流水线要为我们做些什么了。这里是一些有用步骤的列表，作为起点：

1.  从 Git 中检出应用程序源代码

1.  通过在 Docker 容器内运行 PHPUnit（在 Jenkins 主机上）来进行测试

1.  通过在 Jenkins 主机上的容器中执行 FPM 来构建应用程序工件

1.  将工件上传到外部存储（例如，Yum 仓库）

转换为 Jenkins 流水线代码：

### 注意

请参考：[`github.com/PacktPublishing/Implementing-DevOps-on-AWS/blob/master/5585_04_CodeFiles/CodeCommit/demo-app/Jenkinsfile`](https://github.com/PacktPublishing/Implementing-DevOps-on-AWS/blob/master/5585_04_CodeFiles/CodeCommit/demo-app/Jenkinsfile)。

```
#!groovy 

node { 

  stage "Checkout Git repo" 
    checkout scm 

  stage "Run tests" 
    sh "docker run -v \$(pwd):/app --rm phpunit/phpunit tests/" 
  stage "Build RPM" 
    sh "[ -d ./rpm ] || mkdir ./rpm" 
    sh "docker run -v \$(pwd)/src:/data/demo-app -v \$(pwd)/rpm:/data/rpm --rm tenzer/fpm fpm -s dir -t rpm -n demo-app -v \$(git rev-parse --short HEAD) --description "Demo PHP app" --directories /var/www/demo-app --package /data/rpm/demo-app-\$(git rev-parse --short HEAD).rpm /data/demo-app=/var/www/" 

  stage "Update YUM repo" 
    sh "[ -d ~/repo/rpm/demo-app/ ] || mkdir -p ~/repo/rpm/demo-app/" 
    sh "mv ./rpm/*.rpm ~/repo/rpm/demo-app/" 
    sh "createrepo ~/repo/" 
    sh "aws s3 sync ~/repo s3://MY_BUCKET_NAME/ --region us-east-1 --delete" 

  stage "Check YUM repo" 
    sh "yum clean all" 
    sh "yum info demo-app-\$(git rev-parse --short HEAD)" 
} 

```

一般来说，定义一个流水线包含一系列任务/阶段的设置。让我们回顾一下之前的每个阶段：

+   我们从 Git 检出`demo-app`代码开始。假设代码库地址与`Jenkinsfile`的地址相同。

+   在下一个阶段，我们利用 Docker 的隔离性，启动一个容器，容器中包含运行 PHPUnit（参考：https://phpunit.de）所需的一切，以对我们的`demo-app`源代码进行测试。如果你想添加更多测试或进一步修改，可以查看`${GIT_URL}/Examples/Chapter-4/CodeCommit/demo-app/`下的`tests/`文件夹。

+   如果测试通过，我们就开始使用一个干净且用户友好的工具叫做 FPM（参考：https://github.com/jordansissel/fpm）在 Docker 容器中构建一个 RPM 工件。我们使用短的`git commit hash`作为我们的 demo-app 的版本标识。

+   我们将 RPM 工件移动到指定的仓库文件夹，使用`createrepo`将其创建为 YUM 仓库，并将所有数据同步到一个 Amazon S3 存储桶。目的是稍后使用这个基于 S3 的 YUM 仓库来部署我们的`demo-app`。

+   最后，作为一个额外的步骤，我们检查一下我们刚刚同步的包是否可以通过 YUM 获取。

我们的流水线现在已定义，但在运行之前，我们需要满足一个（S3）依赖项。我们需要创建一个 S3 存储桶来存储流水线生成的 RPM 工件。然后，我们需要更新 Jenkins 和 Saltstack 代码中的部分内容，添加该 S3 存储桶的地址。

为了与 S3 进行交互，我们将在之前为 Terraform 配置的环境中使用 AWS CLI 工具：

```
$ aws s3 mb s3://MY_BUCKET_NAME

```

存储桶名称由你决定，但请记住，全球 S3 命名空间是共享的，所以名称越唯一越好。

接下来，我们更新我们的流水线定义（`Jenkinsfile`）。查找包含`MY_BUCKET_NAME`的行：

```
sh "aws s3 sync ~/repo s3://MY_BUCKET_NAME/ --region us-east-1 
        --delete"

```

我们还需要更新 SaltStack（再次替换`MY_BUCKET_NAME`）：

### 注意

请参考：[`github.com/PacktPublishing/Implementing-DevOps-on-AWS/blob/master/5585_04_CodeFiles/CodeCommit/salt/states/yum-s3/files/s3.repo`](https://github.com/PacktPublishing/Implementing-DevOps-on-AWS/blob/master/5585_04_CodeFiles/CodeCommit/salt/states/yum-s3/files/s3.repo)。

```
[s3-repo] 
name=S3-repo 
baseurl=https://s3.amazonaws.com/MY_BUCKET_NAME 
enabled=1 
gpgcheck=0 

```

这个`repo`文件将在流水线的最后阶段使用，正如我们稍后将看到的那样。此时，你需要提交并推送这两项更改：将`Jenkinsfile`提交到`demo-app`仓库，将`s3.repo`文件提交到 SaltStack 仓库。然后，你需要通过 SSH 连接到 Jenkins 节点，拉取并应用 Salt 更改。

## 设置流水线

回到 Jenkins 界面。在登录后，我们点击欢迎页面上的`创建新作业`链接：

![设置流水线](img/image_04_005.jpg)

我们选择**Pipeline**作为作业类型，并为其选定一个名称：

![设置流水线](img/image_04_006.jpg)

下一屏幕将带我们进入作业配置的详细信息。顶部我们选择**丢弃旧构建**，以保持 Jenkins 工作空间的简洁。我们设定，只保留此作业最近五次执行的详细信息：

![设置管道](img/image_04_007.jpg)

在**构建触发器**下，我们选择每 5 分钟轮询一次 Git 仓库以检查更改：

![设置管道](img/image_04_008.jpg)

在下方，我们选择**从 SCM 加载管道脚本**，将 SCM 设置为**Git**，并添加我们`demo-app`仓库的 URL（即[`git-codecommit.us-east-1.amazonaws.com/v1/repos/demo-app`](https://git-codecommit.us-east-1.amazonaws.com/v1/repos/demo-app)）以进行轮询：

![设置管道](img/image_04_009.jpg)

无需额外的凭证，因为这些凭证将通过 EC2 IAM 角色自动获取。请注意**脚本路径**，它引用了我们之前提到的 Jenkins 文件。这是一个很棒的新功能，它为我们提供了代码作为管道的功能，详细说明请见这里：[`jenkins.io/doc/pipeline/#loading-pipeline-scripts-from-scm`](https://jenkins.io/doc/pipeline/#loading-pipeline-scripts-from-scm)。

这样我们就可以方便地将应用程序代码和 Jenkins 管道定义一起放在版本控制中。

保存管道作业后，Jenkins 将开始轮询 Git 仓库，并在检测到更改时触发执行（或者你可以点击**立即构建**强制执行一次）。

每次成功构建后，都会将一个 RPM 包上传到我们的 YUM 仓库。继续进行实验，通过修改`demo-app`源代码来破坏构建，使得测试失败。

要进行故障排除，请查看**构建历史**列表，选择失败的作业，并检查其**控制台输出**：

![设置管道](img/image_04_010.jpg)

现在你已经熟悉了我们的示例管道，我鼓励你扩展它：添加更多的阶段，让一些任务并行执行，启用聊天或电子邮件通知，或者链接管道让它们触发彼此。

随着你将更多日常手动操作转化为 Jenkins 作业，你将会体会到实施 CI 服务器的好处。

你可以放心，你的团队成员也会喜欢这个。

### 注意

请记得删除之前示例中使用的任何 AWS 资源（如 VPC、EC2、S3、IAM、CodeCommit 等），以避免不必要的费用。

# 总结

本章中我们研究了如何在 AWS 上启动和配置持续集成环境的示例。

我们利用之前的 Terraform 和 SaltStack 知识来准备 AWS 基础设施。

在 Jenkins CI 的帮助下，我们构建了一个管道，它会获取应用程序源代码，对其进行测试，构建 RPM 包，并将其存储到远程 YUM 仓库中以备后用。

我们的下一个主题将是持续交付，这是持续集成的扩展，它让我们更接近能够有信心地将应用程序部署到生产环境。
