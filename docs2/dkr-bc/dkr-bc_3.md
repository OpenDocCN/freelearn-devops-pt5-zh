# 第三章 Docker 在云中

第三个工具是 Docker 和 Docker Compose，两个我们在 第一章 *安装 Docker* 时安装的工具，就是 Docker Machine；这是一个命令行工具，允许你管理本地和远程的 Docker 主机。

在本章中，我们将通过使用 Docker Machine 在三个公共云驱动程序中配置 Docker 主机，来查看它们的基本使用方法。我们将会在以下环境中启动 Docker 主机：

+   Digital Ocean [`www.digitalocean.com/`](https://www.digitalocean.com/)

+   亚马逊云服务 - [`aws.amazon.com/`](https://aws.amazon.com/)

+   微软 Azure - [`azure.microsoft.com/`](https://azure.microsoft.com/)

所有操作只需一个命令。

# Docker Machine

Docker Machine 可以连接到以下服务，配置 Docker 主机，并配置你的本地 Docker 客户端，使其能够与新启动的远程实例通信：

除了已提到的三家公共云提供商，Docker Machine 还支持：

+   谷歌计算引擎 - [`cloud.google.com/compute/`](https://cloud.google.com/compute/)

+   Rackspace - [`www.rackspace.co.uk/cloud/`](http://www.rackspace.co.uk/cloud/)

+   IBM Softlayer [`www.softlayer.com`](http://www.softlayer.com)

+   Exoscale - [`www.exoscale.ch/`](https://www.exoscale.ch/)

+   VMware vCloud Air - [`vcloud.vmware.com/`](http://vcloud.vmware.com/)

+   它还支持以下自托管平台：

+   OpenStack - [`www.openstack.org/`](https://www.openstack.org/)

+   微软 Hyper-V - [`www.microsoft.com/virtualization/`](http://www.microsoft.com/virtualization/)

+   VMware vSphere - [`www.vmware.com/uk/products/vsphere/`](http://www.vmware.com/uk/products/vsphere/)

此外，它还允许你使用 VirtualBox 本地启动 Docker 主机 - [`www.virtualbox.org/`](https://www.virtualbox.org/)。如果你的本地工作站不符合 Docker for Mac 或 Windows 的最低规格，这将非常有用。

# Digital Ocean 驱动程序

让我们开始在云中创建一些实例。首先，让我们在 Digital Ocean 启动一台机器。

在 Digital Ocean 上启动实例有两个先决条件，第一个是一个 Digital Ocean 账户，第二个是一个 API 令牌。

要注册 Digital Ocean 账户，请访问 [`www.digitalocean.com/`](https://www.digitalocean.com/) 并点击**注册**。注册或登录账户后，你可以通过点击顶部菜单中的**API**链接来生成 API 令牌。

要获取你的令牌，点击**生成新令牌**并按照屏幕上的说明操作。

### 提示

你只有一次机会记录下你的令牌；请确保将其保存在安全的地方，因为任何拥有该令牌的人都可以在你的账户中启动实例。

一旦获得令牌，你可以使用 Docker Machine 启动实例。为此，运行以下命令，确保将示例 API 令牌替换为你自己的令牌：

```
docker-machine create \
 --driver digitalocean \
 --digitalocean-access-token 14760f5bdee403cebb36117c22c83e5ee51188515f493a6c0d281c094c552536 \
 dotest

```

### 注意

请注意，这些示例中使用的令牌已被撤销。

这将在你的 Digital Ocean 账户中启动一个名为**dotest**的实例。

如果你查看 Digital Ocean 控制面板，你现在应该能看到 Docker Machine 创建的实例：

![Digital Ocean 驱动程序](img/B06455_03_03.jpg)

我们也可以通过以下命令确认我们的 Digital Ocean Docker 主机正在运行：

```
docker-machine ls

```

这将返回所有我们正在运行的机器，确认它们的状态、IP 地址、Docker 版本和名称。还有一列显示你的本地客户端与哪个 Docker Machine 管理的 Docker 主机进行通信：

![Digital Ocean 驱动程序](img/B06455_03_04.jpg)

默认情况下，你的本地 Docker 客户端配置为与我们的本地 Docker 安装进行通信；由于我们是通过 Docker for Mac 或 Windows 启动的本地 Docker 安装，或者你在 Linux 上安装了 Docker，Docker Machine 将不会列出它。

让我们修改它，使其能够与 Digital Ocean 实例交互。

为此，你需要更改一些本地环境变量；幸运的是，Docker Machine 提供了一种简单的方法来找出这些变量并进行更改。

要找出这些变量，只需要运行以下命令：

```
docker-machine env dotest

```

这将告诉你需要运行什么命令才能从`default`机器切换到`dotest`；最好的地方是命令本身将结果格式化为可执行的形式，所以如果我们再次运行命令，但这次以可以执行输出的方式：

```
eval $(docker-machine env dotest)

```

或者，如果你在 Windows 上使用 PowerShell 启动了实例，可以使用以下命令：

```
docker-machine env dotest | Invoke-Expression

```

现在，如果你使用 Docker Machine 列出所有主机，你会发现`dotest`环境现在是活动的：

![Digital Ocean 驱动程序](img/B06455_03_05.jpg)

现在我们已经激活了 Digital Ocean 实例，你可以在本地机器上运行`docker container run`命令，它们将在 Digital Ocean 实例上执行；让我们通过运行`hello-world`容器来测试这一点。

运行以下命令：

```
docker container run hello-world

```

你应该会看到镜像下载，并且如果你运行以下命令，`hello-world`容器的输出也会显示：

```
docker container ls –a

```

你应该能看到`hello-world`容器在几秒钟前已经退出。

你可以使用以下命令通过 SSH 连接到 Digital Ocean 实例：

```
docker-machine ssh dotest

```

登录后，运行`docker container ls –a`命令，演示你在本地运行的`docker container run`命令已经在 Digital Ocean 实例上执行。

这个设置的优点是，你不需要频繁通过 SSH 连接到实例。

### 注意

你可能注意到的一件事是，我们告诉 Docker Machine 的只是我们想使用 Digital Ocean 和我们的 API 令牌；我们没有告诉它要在哪个区域启动实例，想要什么配置，甚至是要使用哪个 SSH 密钥。

Docker Machine 有一些合理的默认设置，具体如下：  

+   digitalocean-image = ubuntu-16-04-x64  

+   digitalocean-region = nyc3  

+   digitalocean-size = 512mb  

由于我位于英国，我们来看一下如何更改 Docker Machine 启动的主机的区域和配置。  

首先，我们应该通过运行以下命令移除`dotest`实例：  

```
docker-machine rm dotest

```

这将终止在 NYC3 区域运行的 512 MB 实例。  

### 提示  

终止未使用的实例非常重要，因为它们在每小时运行时都会产生费用；记住，使用 Docker Machine 的一个关键优势是你可以快速启动实例，并尽可能减少交互。  

现在我们已经移除了旧实例，接下来，我们需要为`docker-machine`命令添加一些额外的标志，以便在指定的区域和配置下启动新实例，我们将把新实例命名为`douktest`。更新后的`docker-machine create`命令现在看起来像这样（再次提醒，记得用你自己的 API 令牌替换示例令牌）：  

```
docker-machine create \
 --driver digitalocean \
 --digitalocean-access-token 14760f5bdee403cebb36117c22c83e5ee51188515f493a6c0d281c094c552536\
 --digitalocean-region lon1 \
 --digitalocean-size 1gb \
 douktest

```

你应该会看到与之前类似的命令输出；一旦实例被部署，你可以通过运行以下命令使其生效：  

```
eval $(docker-machine env douktest)

```

当你进入控制面板时，你会注意到实例已经在指定的区域和期望的配置下启动：  

![数字海洋驱动程序](img/B06455_03_07.jpg)  

有关每个区域的详细信息以及每个区域中可用的机器类型，你可以通过运行以下命令查询 Digital Ocean API（同样记得替换 API 令牌）：  

```
curl -X GET -H "Content-Type: application/json" -H "Authorization: Bearer 14760f5bdee403cebb36117c22c83e5ee51188515f493a6c0d281c094c552536" "https://api.digitalocean.com/v2/regions" | python -mjson.tool

```

这将输出每个区域的信息。  

最后，还有一件事；我们仍然没有找到关于 SSH 密钥的信息。每次运行 Docker Machine 时，都会为你启动的实例创建一个新的 SSH 密钥，并将其上传到提供商；每个密钥都会保存在用户主目录的`.docker`文件夹中。例如，可以通过运行以下命令找到`douktest`的密钥：  

```
cd ~/.docker/machine/machines/douktest/

```

在这里，你还会找到用于验证 Docker 代理与实例上的 Docker 安装的证书及其配置：  

![数字海洋驱动程序](img/B06455_03_08.jpg)  

这就涵盖了在 Digital Ocean 中启动主机；那么，如何启动比 Hello World 容器更令人兴奋的东西呢？  

没问题，让我们使用 Docker Compose 启动一个变化版本的 WordPress 堆栈，类似于我们在第二章中使用的那个，*使用 Docker 启动应用程序*。首先，进入`/bootcamp/chapter03/wordpress`文件夹，然后运行以下命令：  

```
docker-machine ls

```

为了检查你的 Docker 客户端是否已配置为使用你的 Digital Ocean Docker 主机，一旦确认客户端使用了远程主机，只需运行以下命令：  

```
docker-compose up -d

```

这将下载我们需要的镜像，并启动两个容器。此次你将能够通过`port 80`访问 Digital Ocean 主机上的 WordPress 安装。要找到你的主机的 IP 地址，你可以运行以下命令：

```
docker-machine ip douktest

```

或者在 Mac 或 Linux 机器上打开浏览器并访问你的安装页面，运行以下命令：

```
open http://$(docker-machine ip douktest)/

```

下面的终端会话展示了你可以从之前的命令中预期看到的输出：

![The Digital Ocean driver](img/B06455_03_10.jpg)

然后你将能够完成你的 WordPress 安装：

![The Digital Ocean driver](img/B06455_03_11.jpg)

### 提示

我不建议长时间将你的 WordPress 安装停留在安装页面上，因为有人可能会在你不知情的情况下完成安装并进行不当操作。

一旦你完成了 Digital Ocean 主机，运行以下命令来终止它：

```
docker-machine rm douktest

```

现在我们已经学会了如何在 Digital Ocean 上启动 Docker 主机，接下来让我们来了解一下 Amazon Web Services。

# Amazon Web Services 驱动程序

如果你还没有**Amazon Web Services**（**AWS**）账户，你应该在[`aws.amazon.com/`](http://aws.amazon.com/)注册一个；如果你是 AWS 新手，你将有资格获得他们的免费套餐，[`aws.amazon.com/free/`](http://aws.amazon.com/free/)。

如果你对 AWS 不熟悉，我建议在开始本章内容之前先阅读 Amazon 的入门指南；你可以在[`docs.aws.amazon.com/gettingstarted/latest/awsgsg-intro/gsg-aws-intro.html`](http://docs.aws.amazon.com/gettingstarted/latest/awsgsg-intro/gsg-aws-intro.html)找到该指南。

AWS 驱动程序与 Digital Ocean 驱动程序类似，具有一些合理的默认设置。我们不会详细讨论如何自定义 Docker Machine 启动的 EC2 实例，而是使用默认设置。对于 AWS 驱动程序，默认设置如下：

+   amazonec2-region = us-east-1（北弗吉尼亚）

+   amazonec2-ami = ami-fd6e3bea（Ubuntu 16.04）

+   amazonec2-instance-type = t2.micro

+   amazonec2-root-size = 16GB

+   amazonec2-security-group = docker-machine

请注意，如果**amazonec2-security-group**不存在，Docker Machine 会为你创建它；如果它已存在，则 Docker Machine 将使用现有规则。

在我们启动实例之前，我们还需要知道我们的**AWS Access**和**AWS Secret**密钥以及我们将要启动实例的**VPC ID**；要获取这些信息，请登录到 AWS 控制台，网址为[`console.aws.amazon.com/`](https://console.aws.amazon.com/)。

大多数人会使用 AWS 根账户进行登录。由于 AWS 根账户不应该关联任何 Access 和 Secret 密钥，因此我们应该为 Docker Machine 添加一个单独的用户，方法是进入**Services** | **IAM** | **Users**，然后选择你的用户并进入**Security Credentials**标签。

在那里，您应该能看到一个按钮，上面写着**添加用户**，点击此按钮，您将进入一个页面，在该页面上可以设置您的用户详情。输入**用户名** `docker-machine`，然后在**访问类型**中勾选**程序化访问**复选框：

![Amazon Web Services 驱动程序](img/B06455_03_13.jpg)

输入详情后，点击**下一步：权限**进入下一步。在权限页面上，点击**直接附加现有策略**，然后在**策略类型**搜索框中输入`SystemAdministrator`，按回车键以筛选策略：

勾选`SystemAdministrator`旁边的复选框，然后点击**下一步：审查**：

![Amazon Web Services 驱动程序](img/B06455_03_14.jpg)

在审查页面上，点击**创建用户**，几秒钟后，您应该会收到确认，您的用户已成功创建。

确保点击**下载 .csv**，因为您将不再看到密钥访问密钥。现在您已经拥有了您的访问密钥 ID 和密钥访问密钥。

在找到您的 VPC ID 之前，您应该通过确保 AWS 控制台右上角显示的是**北弗吉尼亚**来确认您处于正确的区域；如果不是，从下拉列表中选择它。

### 注意

亚马逊将 Amazon VPC（Amazon 虚拟私有云）描述为允许您提供 AWS 云的逻辑隔离部分，您可以在其中启动您定义的虚拟网络中的资源。您可以完全控制虚拟网络环境，包括选择自己的 IP 地址范围、创建子网，以及配置路由表和网络网关。

一旦确认您处于正确的区域，转到**服务**，然后选择**VPC**，点击**您的 VPC**；您无需担心创建和配置 VPC，因为 Amazon 在每个区域都提供了默认的 VPC。选择该 VPC，您应该看到类似以下的内容：

![Amazon Web Services 驱动程序](img/B06455_03_17.jpg)

### 注意

记下 VPC ID；您现在应该有足够的信息来使用 Docker Machine 启动您的实例。为此，请运行以下命令：

```
docker-machine create \
 --driver amazonec2 \
 --amazonec2-access-key AKIAIP26OOEA3D4SLW5A \
 --amazonec2-secret-key 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 \
 --amazonec2-vpc-id vpc-35c91750 \
 awstest

```

如果一切顺利，您应该看到类似以下的输出：

![Amazon Web Services 驱动程序](img/B06455_03_18.jpg)

您还应该能够通过点击**服务** | **EC2** | **实例**，在 AWS 控制台中看到一个已启动的 EC2 实例：

![Amazon Web Services 驱动程序](img/B06455_03_19.jpg)

您可能注意到，Docker Machine 创建了安全组并为实例分配了 SSH 密钥，而我们无需介入，这符合您无需成为专家即可配置启动 Docker 实例的环境的原则。

在我们终止实例之前，让我们将本地 Docker 客户端切换为使用 AWS 实例，并通过运行以下命令启动**Hello World**容器：

```
eval $(docker-machine env awstest)
docker-machine ls
docker container run hello-world
docker c
ontainer ls -a

```

如您所见，一旦使用 Docker Machine 启动实例并将本地 Docker 客户端切换到它，无论是在本地运行 Docker 还是在云提供商上运行，使用方法没有任何区别。

在我们开始累积成本之前，我们应该通过运行以下命令终止我们的测试 AWS 实例：

```
docker-machine rm awstest

```

然后在 AWS 控制台确认实例已正确终止：

![亚马逊 Web 服务驱动程序](img/B06455_03_21.jpg)

如果不这样做，EC2 实例将愉快地保持在那里，并且每小时将花费您 0.013 美元，直到终止。

### 注意

请注意，这不是 Docker for AWS，我们将在 第四章 中介绍该服务，*Docker Swarm*。

# Microsoft Azure 驱动程序

正如您可能从终端和浏览器截图中注意到的，到目前为止，我们一直在使用 Docker for Mac；现在让我们看看使用 Docker for Windows 的 Microsoft Azure 驱动程序。

首先，您需要一个 Microsoft Azure 账户；如果还没有，请访问 [`azure.microsoft.com/`](https://azure.microsoft.com/) 注册。一旦拥有账户，您需要开始的唯一信息是您的订阅 ID；您可以在门户的计费部分找到此信息。

一旦您有了订阅 ID，您可以使用 Azure 认证 Docker Machine，输入以下命令，确保用您自己的订阅 ID 替换：

```
docker-machine.exe create --driver azure --azure-subscription-id xxxxxxx-85a6-4ab4-b5c4-c18b54e014
98 azuretest

```

![Microsoft Azure 驱动程序](img/B06455_03_22.jpg)

您将收到一个激活码来授权 Docker Machine；请访问 [`aka.ms/devicelogin/`](https://aka.ms/devicelogin/) 并输入您收到的代码：

![Microsoft Azure 驱动程序](img/B06455_03_23.jpg)

点击 **Continue** 将带您到一个页面，显示 Docker Machine 将授予您的权限：

![Microsoft Azure 驱动程序](img/B06455_03_24.jpg)

一旦您点击 **Accept**，您将看到 Docker Machine 开始引导环境。此过程将花费几分钟；完成后，您应该看到类似以下输出：

![Microsoft Azure 驱动程序](img/B06455_03_25.jpg)

如您所见，Docker Machine 已执行以下操作：

+   创建了一个资源组

+   配置了网络安全组

+   配置了一个网络子网

+   创建了一个虚拟网络

+   分配了一个公共 IP 地址

+   创建了一个网络接口

+   创建了一个存储帐户

+   启动了一个虚拟机

如果您在 Azure 门户中查看资源组，您应该看到已启动和准备就绪的虚拟机：

![Microsoft Azure 驱动程序](img/B06455_03_26.jpg)

与 Mac 和 Linux 版本的 Docker Machine 一样，我们需要配置本地 Docker 客户端以与远程主机通信，为此我们需要运行以下命令：

```
docker-machine.exe env --shell powershell azuretest | Invoke-Expression

```

这相当于在 Mac 或 Linux 上运行以下命令：

```
eval $(docker-machine env azuretest)

```

您可以通过运行以下命令检查 Azure 是否处于活动状态：

```
docker-machine.exe ls

```

现在我们的客户端已连接到我们的 Azure 远程主机，我们可以通过运行以下命令启动**Hello World**容器：

```
docker container run hello-world
docker container ls -a

```

与 Docker Machine 的 Mac 和 Linux 版本一样，你可以通过运行以下命令 SSH 连接到你的 Azure 主机：

```
docker-machine.exe ssh azuretest

```

从下面的输出中你可以看到，我们可以看到 **Hello World** 容器：

![The Microsoft Azure driver](img/B06455_03_28.jpg)

从这里你可以像与任何其他 Docker 主机交互一样与 Azure 主机交互。一旦你准备好终止 Azure 主机，只需运行以下命令：

```
docker-machine.exe rm azuretest

```

移除主机及其相关资源需要一段时间，完成后你应该会看到类似以下内容：

![The Microsoft Azure driver](img/B06455_03_29.jpg)

在 Azure 门户中检查 **Activity log**，可以看到正在被移除的资源：

![The Microsoft Azure driver](img/B06455_03_30.jpg)

正如 PowerShell 输出中所强调的，最好检查所有内容是否已正确终止，最简单的方法是删除资源组本身，操作步骤是点击右侧 **...** 的三个点，来删除 **docker-machi** **ne** 资源。

在你按照屏幕上的提示操作后，包括输入你选择删除的资源名称，你应该会收到资源组已被移除的确认信息。

尽管我们使用 Windows 查看 Azure，其他操作与 Mac 和 Linux 上的过程相同，唯一不同的是将本地客户端切换为使用远程主机。

### 注意

请注意，这不是 Azure 的 Docker，我们将在 第四章 中讨论该服务，*Docker Swarm*。

# 参考资料

本章中使用的示例是启动 Ubuntu 实例。Docker Machine 还支持：

+   Debian (8.0+) - [`www.debian.org/`](https://www.debian.org/)

+   红帽企业版 Linux (7.0+) - [`www.redhat.com/`](https://www.redhat.com/)

+   CentOS (7+) - [`www.centos.org/`](https://www.centos.org/)

+   Fedora (21+) - [`getfedora.org/`](https://getfedora.org/)

+   RancherOS (0.3) - [`rancher.com/rancher-os/`](http://rancher.com/rancher-os/)

需要提到的另一件事是，Docker Machine 默认启用了崩溃报告功能，考虑到 Docker Machine 可以与不同的配置/环境组合一起使用，通知 Docker 任何问题是非常重要的，这有助于他们改进产品。

如果由于任何原因你想选择退出，那么运行以下命令将禁用崩溃报告：

```
mkdir -p ~/.docker/machine && touch ~/.docker/machine/no-error-report

```

获取更多关于 Docker Machine 的信息，请参考官方文档：

+   Docker Machine - [`docs.docker.com/machine/`](https://docs.docker.com/machine/)

+   Docker Machine 驱动程序 - [`docs.docker.com/machine/drivers/`](https://docs.docker.com/machine/drivers/)

+   Docker Machine 命令参考 - [`docs.docker.com/machine/reference/`](https://docs.docker.com/machine/reference/)

# 总结

从我们已经讨论过的例子可以看出，Docker Machine 是一个强大的工具，它让所有技能水平的用户都能轻松在云提供商中启动实例，而无需卷起袖子亲自配置服务器实例。

在我们接下来的章节中，我们将讨论如何在同一个云提供商中启动多个 Docker 主机，并配置 Docker Swarm 集群。
