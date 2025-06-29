# 第二章 引入第一方工具

Docker 提供了多种工具来扩展核心 Docker 引擎之外的功能。在本章中，您将逐步了解如何安装、配置和运行以下工具：

+   Docker Toolbox

+   Docker Machine

+   Docker Swarm

+   Docker Compose

这些工具虽然没有我们将在接下来的章节中使用的高级工具功能强大，但它们将作为一个良好的入门，帮助您理解如何为核心 Docker 引擎添加额外功能，以及容器部署和编排的概念，我们将在本书后面更多地探讨这些内容。

# Docker Toolbox

在我们开始了解如何使用其他三种工具之前，应该先看一下如何在本地机器上安装它们。在上一章中，我们下载了 Docker 提供的脚本，并通过 bash 快速配置了官方的 Docker YUM 或 APT 仓库（具体取决于您使用的操作系统），我们执行的命令如下：

```
curl -sSL https://get.docker.com/ | sh

```

如果您已经在某个云服务或本地虚拟机上运行了基于 Linux 的服务器，这将非常有用；但是，如果您想在非 Linux 操作系统（如 Mac OSX 或 Windows）上安装 Docker，怎么办呢？

### 提示

始终检查源。最好的做法是检查您将要下载并安装的 bash 脚本的来源；在我们的案例中，您可以通过浏览器访问 [`get.docker.com/`](https://get.docker.com/) 来检查。

在我们查看 Docker 提供的工具之前，我们应该先回答一个问题：为什么？

## 为什么要在本地安装 Docker？

那么，为什么我们需要在非 Linux 系统上安装 Docker Toolbox、Compose、Machine 和 Swarm 呢？首先，您需要记住，Docker 本质上是一个针对基于 Linux 内核的技术的 API，例如 run ([`github.com/opencontainers/runc`](https://github.com/opencontainers/runc)) 和 LXC ([`linuxcontainers.org`](https://linuxcontainers.org))，因此虽然您无法在 Mac OS X 或 Windows 系统上启动容器，但您可以与 Linux 机器上的 Docker 安装进行交互。

能够从本地机器与 Docker 进行交互，意味着您可以跨多个主机启动并与容器进行交互，这些主机可以托管在公共云/托管服务上，也可以本地托管在虚拟机中。

幸运的是，Docker 提供了便捷的方式，帮助您在本地机器上安装 Docker 和本章要介绍的其他三种服务。

## 安装 Docker Toolbox

Docker 提供了一个名为 Docker Toolbox 的全球安装程序，它使得安装以下软件变得尽可能简单：

+   Docker 客户端

+   Docker Machine

+   Docker Compose

+   Docker Kitematic

+   虚拟机 VirtualBox

要开始，您需要运行一个安装有 Mac OS X 10.8+ 或 Windows 7+ 的机器。在我的例子中，我使用的是 Mac OS X 10.11（El Capitan）；Mac OS X 和 Windows 的安装程序差别很小：

1.  首先，要开始，您需要从 Docker 网站下载安装程序。您可以在[`www.docker.com/docker-toolbox/`](https://www.docker.com/docker-toolbox/)找到适用于您操作系统的可执行文件下载链接。![安装 Docker Toolbox](img/B05468_02_01.jpg)

1.  下载完安装程序后，您可以通过双击它来启动。接下来会显示一系列屏幕和安装选项。

    第一个屏幕是欢迎页面，确认您正在运行的工具箱版本。如果您是从前面截图中的页面下载的，那么您将始终拥有最新版本：

    ![安装 Docker Toolbox](img/B05468_02_02.jpg)

1.  要继续安装的下一步，请点击**继续**。

1.  下一屏幕将详细介绍将要安装的包以及它们将安装的位置。还有一个框，如果勾选此框，它将收集您安装 Docker Toolbox 的机器数据，对其进行匿名处理，然后提交给 Docker。

    这些信息有助于 Docker 了解您的软件安装在什么类型的机器上，还会反馈您在运行安装程序时遇到的任何错误：

    ![安装 Docker Toolbox](img/B05468_02_03.jpg)

    我总是建议保持勾选此框，因为这有助于 Docker 改进产品并提升未来版本安装程序的用户体验。

1.  要继续安装的下一步，点击**继续**。

1.  下一屏幕将允许您选择安装工具的磁盘位置。在大多数情况下，您应该使用默认设置，除非您在多个驱动器上运行应用程序：![安装 Docker Toolbox](img/B05468_02_04.jpg)

1.  要继续安装的下一步，再次点击**继续**按钮。

1.  对于大多数人来说，标准安装就足够了；但是，如果不打算安装某些工具，您可以点击**自定义**按钮。您必须安装的两个工具是 Docker 客户端和 Docker 机器。

    因为我想安装所有工具，所以我选择了标准安装：

    ![安装 Docker Toolbox](img/B05468_02_05.jpg)

1.  一旦您选择了标准或自定义安装，您可以点击**安装**按钮进行安装。

1.  安装本身只需要几分钟，在此期间，您将看到安装程序正在执行的任务反馈：![安装 Docker Toolbox](img/B05468_02_06.jpg)

1.  安装完成后，点击**继续**按钮。

    由于运行安装程序也会作为已安装组件的升级程序，它将检查服务管理的任何文件（例如各种工具使用的虚拟机映像）是否需要更新。

    根据更新的大小以及数据量的不同，这个过程可能需要几分钟。

    该过程仅适用于更新，因此如果你像我一样进行了全新安装，则此部分将被跳过。

1.  现在工具已经安装完成，你将获得启动**Docker 快速启动终端**或 Kitematic 的选项。为了本书的目的，我们将通过点击**继续**按钮跳过此屏幕：![安装 Docker 工具箱](img/B05468_02_07.jpg)

1.  如果一切顺利，你将看到一条消息，确认安装已经完成，你可以点击**关闭**按钮退出安装程序：![安装 Docker 工具箱](img/B05468_02_08.jpg)

现在，你已经在本地机器上安装了所有工具，可以继续本章及整本书的内容了。

在我们开始查看各个工具之前，我们需要配置 Docker 代理。为此，请运行**Docker 快速启动终端**应用程序。如果你安装了多个终端模拟器，它会弹出提示，询问你想使用哪个终端；我更喜欢使用 Mac OS X 自带的终端，因此我选择了终端。

一旦你做出选择，一个新的终端窗口将打开，应用程序将为你配置本地的 Docker 安装：

![安装 Docker 工具箱](img/B05468_02_09.jpg)

就我而言，在启动**Docker 快速启动终端**应用程序时，我得到了前面的终端输出。

# Docker 机器

因此，当你运行**Docker 快速启动终端**应用程序时，它会创建一堆证书、SSH 密钥，并配置你的用户环境以运行 Docker。它还启动了一个运行 Docker 的虚拟机。

## 本地开发

**Docker 快速启动终端**应用程序是通过 Docker 机器完成此操作的，你可以通过运行以下命令来检查该应用程序启动的机器状态：

```
docker-machine active

```

这将列出任何活动机器的名称，第一次安装 Docker 时启动的默认机器名为`default`，如果你运行：

```
docker-machine status default

```

它应该会告诉你虚拟机当前正在运行。最后，你应该能够通过运行以下命令 SSH 连接到虚拟机：

```
docker-machine ssh default

```

你会注意到，当你通过 SSH 连接到虚拟机时，它正在运行 Boot2Docker 发行版。

### 注意

Boot2Docker 是一个极其轻量的 Linux 发行版，基于 Tiny Core Linux，它的唯一目的是运行 Docker。由于这个原因，整个发行版的大小不到 30 MB，启动大约只需五秒钟，这使得它非常适合用于本地开发机器。关于 Boot2Docker 的更多信息，请参考 [`boot2docker.io/`](http://boot2docker.io/)，关于 Tiny Core Linux，请参考 [`tinycorelinux.net/`](http://tinycorelinux.net/)。

当你运行这些命令时，你应该看到类似下面的终端会话：

![本地开发](img/B05468_02_10.jpg)

虽然没有太多需要 SSH 进入虚拟机的必要，但由于通过工具箱安装的 Docker 客户端已被配置为连接到虚拟机上的 Docker 引擎，这意味着当你在本地运行 Docker 命令时，它会将所有请求传递给虚拟机上的 Docker 引擎。试着运行 `hello-world` 容器：

```
docker run hello-world

```

你应该看到以下输出：

![本地开发](img/B05468_02_10.jpg)

在这一阶段，你可能会想，这一切都很好，但这并不是一个值得兴奋的工具。嗯，你错了。Docker Machine 可不止能在本地启动 Boot2Docker 虚拟机，它还有更多的“隐藏技能”。

## 进入云端

Docker Machine 能够连接到以下服务，创建实例，并配置你的本地 Docker 客户端，使其能够与基于云的实例进行通信。

当前支持的公共云服务提供商如下：

+   **Amazon Web Services (AWS)**: [`aws.amazon.com/`](https://aws.amazon.com/)

+   **DigitalOcean**: [`www.digitalocean.com/`](https://www.digitalocean.com/)

+   **Microsoft Azure**: [`azure.microsoft.com/`](https://azure.microsoft.com/)

+   **Google Compute Engine**: [`cloud.google.com/compute/`](https://cloud.google.com/compute/)

+   **Rackspace**: [`www.rackspace.co.uk/cloud/`](http://www.rackspace.co.uk/cloud/)

+   **IBM SoftLayer**: [`www.softlayer.com`](http://www.softlayer.com)

+   **Exoscale**: [`www.exoscale.ch/`](https://www.exoscale.ch/)

+   **VMware vCloud Air**: [`vcloud.vmware.com/`](http://vcloud.vmware.com/)

以下自托管平台也可以使用：

+   **OpenStack**: [`www.openstack.org/`](https://www.openstack.org/)

+   **Microsoft Hyper-V**: [`www.microsoft.com/virtualization/`](http://www.microsoft.com/virtualization/)

+   **VMware vSphere**: [`www.vmware.com/uk/products/vsphere/`](http://www.vmware.com/uk/products/vsphere/)

### DigitalOcean 驱动程序

让我们开始在云中创建一些实例。首先，启动一个在 DigitalOcean 上的虚拟机。

在 DigitalOcean 中启动实例的两个先决条件是：首先需要一个 DigitalOcean 帐户，其次需要一个 API 令牌。

要注册一个 DigitalOcean 账户，请访问 [`www.digitalocean.com/`](https://www.digitalocean.com/) 并点击**注册**按钮。登录到您的账户后，您可以通过点击顶部菜单中的**API**链接生成 API 令牌。

要获取您的令牌，请点击**生成新令牌**按钮并按照屏幕上的说明操作：

### 提示

您只有一次机会记录下您的令牌，请确保将其存储在安全的地方，因为任何拥有它的人都能在您的账户中启动实例。

![DigitalOcean 驱动程序](img/B05468_02_12.jpg)

一旦您有了令牌，您可以使用 Docker Machine 启动您的实例。为此，运行以下命令；确保将示例 API 令牌替换为您自己的：

### 提示

使用反斜杠：由于我们需要传递很多选项给 `docker-machine` 命令，我们使用反斜杠 `\` 将命令分成多行，这样更容易跟踪发生了什么。

```
 docker-machine create \
    --driver digitalocean \
    --digitalocean-access-token sdnjkjdfgkjb345kjdgljknqwetkjwhgoih314rjkwergoiyu34rjkherglkhrg0 \
    dotest
```

这将把一个 `dotest` 实例启动到您的 DigitalOcean 账户中，您将看到类似以下终端输出的内容：

![DigitalOcean 驱动程序](img/B05468_02_13.jpg)

如果您检查 DigitalOcean 控制面板，您现在会看到由 Docker Machine 创建的实例已列在这里：

![DigitalOcean 驱动程序](img/B05468_02_14.jpg)

现在，我们通过 Docker Machine 启动了两个实例，一个是本地运行的，名为`default`，另一个是托管在 DigitalOcean 上的，名为`dotest`。我们可以通过运行以下命令来确认这一点：

```
docker-machine ls

```

这将返回所有正在运行的机器，并确认它们的状态、IP 地址、Docker 版本和名称。还有一列显示您本地环境配置为与哪台机器通信：

![DigitalOcean 驱动程序](img/B05468_02_15.jpg)

在前面的示例中，我们的本地 Docker 客户端配置为与 `default` 实例进行通信，它是在本地运行的。现在让我们将其更改为与 DigitalOcean 实例进行交互。

为了实现这一点，您需要更改一些本地环境变量，幸运的是，Docker Machine 提供了一种简单的方法来找出这些变量并进行更改。

要了解它们，只需运行以下命令：

```
docker-machine env dotest

```

这将告诉您需要运行什么命令来从 `default` 机器切换到 `dotest`。最棒的是，该命令本身会以可执行的方式格式化结果，因此我们再次运行该命令，但这次的输出将被执行：

```
eval $(docker-machine env dotest)

```

现在，如果您从 Docker Machine 获取列表，您会注意到 `dotest` 环境现在是活跃的：

![DigitalOcean 驱动程序](img/B05468_02_16.jpg)

现在，我们的 DigitalOcean 实例已激活，您可以在本地机器上运行 `docker` 命令，这些命令将会在 DigitalOcean 实例上执行。我们通过运行 hello-world 容器来测试这一点。

如果运行以下命令，你应该看到镜像下载，然后是运行 hello-world 容器的输出：

```
docker run hello-world

```

如果接着运行以下命令，你会看到几秒钟前退出的 hello-world 镜像：

```
docker ps –a

```

以下是终端输出的示例：

![DigitalOcean 驱动程序](img/B05468_02_17.jpg)

正如你所见，我使用 `ssh` 进入 DigitalOcean 实例，并运行了 `docker ps -a` 和 `docker images` 命令来演示本地运行的命令是在 DigitalOcean 实例上执行的；但这种设置的美妙之处在于，你不应该经常需要 SSH 进入实例。

你可能注意到的一件事是，我们告诉 Docker Machine 的只是我们要使用 DigitalOcean 和我们的 API 令牌；但从未告诉它在哪个区域启动实例，需要什么规格，或者要使用哪个 SSH 密钥。

Docker Machine 拥有一些合理的默认设置：

+   `digitalocean-image = ubuntu-15-10-x64`

+   `digitalocean-region = nyc3`

+   `digitalocean-size = 512mb`

由于我位于英国，让我们看看如何更改区域和机器规格。首先，我们应该通过运行以下命令删除 `dotest` 实例：

```
docker-machine rm dotest

```

这将终止在 NYC3 运行的 `512mb` 实例。

### 提示

终止不使用的实例非常重要，因为它们每小时处于活动状态都会产生费用。记住使用 Docker Machine 的关键优势之一是你可以快速启动实例，并尽可能少地进行交互。

现在我们已经删除了旧实例，让我们为 `docker-machine` 命令添加一些额外的标志，以在所需的区域和规格中启动新实例，我们将称新实例为 `douktest`。更新后的 `docker-machine create` 命令现在看起来类似于以下内容（记得用你自己的 API 令牌替换示例 API 令牌）：

```
docker-machine create \
    --driver digitalocean \
    --digitalocean-access-token sdnjkjdfgkjb345kjdgljknqwetkjwhgoih314rjkwergoiyu34rjkherglkhrg0 \
    --digitalocean-region lon1 \
    --digitalocean-size 1gb \
    douktest
```

一旦实例部署完成，你应该看到与之前命令相似的输出，然后可以通过运行以下命令使其激活：

```
eval $(docker-machine env douktest)

```

![DigitalOcean 驱动程序](img/B05468_02_18.jpg)

当你进入控制面板时，你会注意到实例已在指定的区域以及所需的规格下启动：

![DigitalOcean 驱动程序](img/B05468_02_19.jpg)

要获取每个区域的详细信息和每个区域可用的机器类型，请运行以下命令查询 DigitalOcean API（记得替换 API 令牌）：

```
curl -X GET -H "Content-Type: application/json" -H "Authorization: Bearer sdnjkjdfgkjb345kjdgljknqwetkjwhgoih314rjkwergoiyu34rjkherglkhrg0" "https://api.digitalocean.com/v2/regions" | python -mjson.tool

```

这将输出每个区域的信息。

还有最后一件事，我们还没弄清楚 SSH 密钥的情况。每次运行 Docker Machine，都会为你启动的实例创建一个新的 SSH 密钥并上传到提供者，每个密钥都存储在你用户主目录下的 `.docker` 文件夹中。例如，可以通过以下命令找到 `douktest` 的密钥：

```
cd ~/.docker/machine/machines/douktest/

```

在这里，您还会找到用于通过实例上的 Docker 安装认证 Docker 代理的证书以及相关配置：

![DigitalOcean 驱动程序](img/B05468_02_20.jpg)

这涵盖了 DigitalOcean，那么其他服务呢？让我们快速看一下 Amazon Web Services，以便我们可以对不同云服务提供商的驱动程序有个大致了解。

### Amazon Web Services 驱动程序

如果您还没有 Amazon Web Services 账户，应该在 [`aws.amazon.com/`](http://aws.amazon.com/) 注册一个。如果您是 AWS 新用户，您将有资格获得他们的免费层服务，详情请见 [`aws.amazon.com/free/`](http://aws.amazon.com/free/)。

如果您不熟悉 AWS，建议在开始本章内容之前先阅读 Amazon 的入门指南，您可以在 [`docs.aws.amazon.com/gettingstarted/latest/awsgsg-intro/gsg-aws-intro.html`](http://docs.aws.amazon.com/gettingstarted/latest/awsgsg-intro/gsg-aws-intro.html) 找到该指南。

AWS 驱动程序与 DigitalOcean 驱动程序类似，并具有一些合理的默认设置，考虑到我们不打算深入讨论如何自定义 Docker Machine 启动的 EC2 实例，我将坚持使用默认设置。对于 AWS 驱动程序，默认设置如下：

+   `amazonec2-region = us-east-1 (北弗吉尼亚)`

+   `amazonec2-ami = ami-26d5af4c (Ubuntu 15.10)`

+   `amazonec2-instance-type = t2.micro`

+   `amazonec2-root-size = 16GB`

+   `amazonec2-security-group = docker-machine`

在启动实例之前，我们还需要知道我们的 AWS 访问密钥和密钥对，此外，还需要知道 VPC ID，因为我们将在其中启动实例。要获取这些信息，请登录到 AWS 控制台，地址是 [`console.aws.amazon.com/`](https://console.aws.amazon.com/)。

您应该已经拥有您的访问密钥和密钥 ID 的副本，因为这些是在您的 AWS 用户首次创建时生成的。如果您丢失了它们，可以通过导航到 **服务** | **IAM** | **用户**，然后选择您的用户，最后转到 **安全凭证** 标签页来生成一对新密钥。在那里您应该能看到一个名为 **创建访问密钥** 的按钮。

### 注意

Amazon 将 **虚拟专用云**（**VPC**）描述为让您能够在 AWS 云中配置一个逻辑上隔离的区域，在该区域内您可以启动您定义的虚拟网络中的资源。您可以完全控制您的虚拟网络环境，包括选择您自己的 IP 地址范围、创建子网、配置路由表和网络网关。

在查找 VPC ID 之前，您应该确保您处于正确的区域，可以通过确保 AWS 控制台右上角显示 **N. Virginia** 来验证，如果没有显示，您可以从下拉列表中选择正确的区域。

确保你处于正确的区域后，前往**服务** | **VPC**，点击**您的 VPC**。你无需担心创建和配置 VPC，因为 Amazon 在每个区域都会为你提供默认 VPC。选择 VPC 后，你应该会看到类似于以下截图的内容：

![Amazon Web Services 驱动程序](img/B05468_02_21.jpg)

记下 VPC ID，你现在应该有足够的信息通过 Docker Machine 启动实例。为此，请运行以下命令：

```
docker-machine create \
 --driver amazonec2 \
 --amazonec2-access-key JHFDIGJKBDS8639FJHDS \
 --amazonec2-secret-key sfvjbkdsvBKHDJBDFjbfsdvlkb+JLN873JKFLSJH \
 --amazonec2-vpc-id vpc-35c91750 \
 awstest

```

如果一切顺利，你应该会看到类似以下的输出：

![Amazon Web Services 驱动程序](img/B05468_02_22.jpg)

你还应该能够通过导航到**服务** | **EC2** | **实例**来在 AWS 控制台中看到启动的 EC2 实例：

![Amazon Web Services 驱动程序](img/B05468_02_23.jpg)

你可能已经注意到，Docker Machine 创建了安全组，并且无需我们介入就为实例分配了 SSH 密钥，这符合我们不需要成为环境配置专家的原则，Docker 实例的启动和配置都已自动完成。

在我们终止实例之前，先将本地 Docker 客户端切换为使用 AWS 实例，并启动`Hello World`容器：

![Amazon Web Services 驱动程序](img/B05468_02_24.jpg)

如你所见，一旦使用 Docker Machine 启动了实例并切换本地 Docker 客户端，你会发现本地运行 Docker 和在云服务商上运行 Docker 在使用上没有区别。

在我们开始累积费用之前，我们应该通过运行以下命令来终止我们的测试 AWS 实例：

```
docker-machine rm awstest

```

然后确认实例在 AWS 控制台中已经正确终止：

![Amazon Web Services 驱动程序](img/B05468_02_25.jpg)

如果不这么做，EC2 实例将毫不犹豫地一直运行，每小时收取**$0.013**的费用，直到它被终止。

### 注意

请注意，这不是 Amazon 的**弹性容器服务**（**ECS**）。我们将在第七章，*查看调度器*中介绍 Amazon ECS。

## 其他考虑事项

如我们所做的示例所示，Docker Machine 是 Docker Toolbox 中的一个强大工具，它使所有技术水平的用户都能轻松启动本地或云提供商中的实例，而不需要动手配置服务器实例或本地 Docker 客户端。

本章中我们使用的示例均为启动 Boot2Docker 或 Ubuntu。Docker Machine 还支持以下操作系统：

+   **Debian（8.0+）**：[`www.debian.org/`](https://www.debian.org/)

+   **Red Hat Enterprise Linux（7.0+）**：[`www.redhat.com/`](https://www.redhat.com/)

+   **CentOS（7+）**：[`www.centos.org/`](https://www.centos.org/)

+   **Fedora（21+）**：[`getfedora.org/`](https://getfedora.org/)

+   **RancherOS（0.3）**：[`rancher.com/rancher-os/`](http://rancher.com/rancher-os/)

需要提到的关于 Docker Machine 的另一点是，默认情况下，它会启用崩溃报告功能。考虑到 Docker Machine 可以与许多不同的配置/环境组合一起使用，重要的是 Docker 能够收到任何问题的通知，以帮助他们改进产品。如果由于某种原因你想退出此功能，可以运行以下命令来禁用崩溃报告：

```
mkdir -p ~/.docker/machine && touch ~/.docker/machine/no-error-report

```

有关 Docker Machine 的更多信息，你可以参考官方文档：

+   **Docker Machine**: [`docs.docker.com/machine/`](https://docs.docker.com/machine/)

+   **Docker Machine Drivers**: [`docs.docker.com/machine/drivers/`](https://docs.docker.com/machine/drivers/)

+   **Docker Machine Command Reference**: [`docs.docker.com/machine/reference/`](https://docs.docker.com/machine/reference/)

# Docker Swarm

现在我们已经讨论了如何使用 Docker Machine 启动单个 Docker 实例，让我们更进一步，创建一个实例集群。为此，Docker 提供了一个名为 Swarm 的工具。当部署时，它充当你的 Docker 客户端与宿主 Docker 实例之间的调度器，根据调度规则决定在哪里启动容器。

## 创建本地集群

首先，我们将使用 Docker Machine 在本地通过 VirtualBox ([`www.virtualbox.org`](https://www.virtualbox.org)) 创建一个集群，VirtualBox 是 Docker Toolbox 的一部分。首先，我们将启动一个虚拟机来生成发现令牌。为此，运行以下命令：

```
docker-machine create -d virtualbox discover

```

然后配置你的 Docker 客户端以使用新创建的本地实例：

```
eval "$(docker-machine env discover)"

```

你可以通过运行`docker-machine ls`命令并确保 `discover` 在活动列中有星号，来检查你的 Docker 客户端是否配置为使用 `discover` 实例。

最后，你可以通过运行以下命令来安装发现服务：

```
docker run swarm create

```

这将下载并运行发现服务并生成令牌。在过程结束时，你将获得一个令牌；请务必记下这个令牌，以便在后续步骤中使用。如果一切顺利，你应该会看到类似于以下输出的内容：

![创建本地集群](img/B05468_02_26.jpg)

在上面的示例中，令牌是`40c3bf4866eed5ad14ade6633fc4cefc`。现在我们已经有了令牌，需要启动一个实例作为调度器，这个实例被称为 Swarm 管理器。

为此，请输入以下命令，并确保替换为你生成的令牌：

```
docker-machine create \
 -d virtualbox \
 --swarm \
 --swarm-master \
 --swarm-discovery token://40c3bf4866eed5ad14ade6633fc4cefc \
 swarm-master

```

现在我们已经启动了 Swarm 管理器虚拟机，可以开始启动充当集群节点的虚拟机。同样，使用发现令牌，运行以下命令以启动两个节点：

```
docker-machine create \
 -d virtualbox \
 --swarm \
 --swarm-discovery token://40c3bf4866eed5ad14ade6633fc4cefc \
 swarm-node-01

```

然后使用以下命令启动第二个节点：

```
docker-machine create \
 -d virtualbox \
 --swarm \
 --swarm-discovery token://40c3bf4866eed5ad14ade6633fc4cefc \
 swarm-node-02

```

我们可以通过运行 `docker-machine ls` 命令检查我们的虚拟机，然后运行以下命令将我们的 Docker 客户端切换到使用集群：

```
eval $(docker-machine env --swarm swarm-master)

```

现在您的 Docker 客户端已经与集群通信，您可以运行`docker info`来查找有关所有节点和集群本身的信息，您将看到类似以下屏幕截图的内容：

![创建本地集群](img/B05468_02_27.jpg)

现在，我们有一个三 CPU、3GB 的集群，运行在三个节点上。为了测试它，让我们运行`Hello World`容器，然后运行`docker ps -a`，以便查看容器启动在哪个节点上：

![创建本地集群](img/B05468_02_28.jpg)

如您从终端输出中看到的，容器是在`swarm-node-01`上启动的，再次运行容器应该会将其启动在我们的第二个节点上：

![创建本地集群](img/B05468_02_29.jpg)

所以，你看，这是一个非常基础的 Docker Swarm 集群，您可以使用本地 Docker 客户端将容器启动到其中，所有操作都是通过 Docker Machine 启动和管理的。

在我们进入下一部分之前，我们应该删除本地集群。为此，只需运行以下命令：

```
docker-machine rm discover swarm-master swarm-node-01 swarm-node-02

```

当提示时点击`yes`。然后，您可以通过运行`docker-machine ls`命令检查虚拟机是否已终止。

## 创建远程集群

在我们查看下一个工具之前，让我们在云中启动一个集群。我将使用 DigitalOcean 进行此操作。

首先，让我们创建一个新的发现令牌。由于我们所需要做的只是生成一个发现令牌，因此没有必要仅为此任务在 DigitalOcean 上启动实例，因此我们将简单地在本地启动一台机器，记下发现令牌后再将其移除：

```
docker-machine create -d virtualbox token
eval "$(docker-machine env token)"
docker run swarm create
docker-machine rm token

```

现在我们已经获得了发现令牌，让我们在 DigitalOcean 上启动我们的 Swarm 集群，首先我们将研究 Swarm 管理器：

```
docker-machine create \
 --driver digitalocean \
 --digitalocean-access-token sdnjkjdfgkjb345kjdgljknqwetkjwhgoih314rjkwergoiyu34rjkherglkhrg0 \
 --digitalocean-region lon1 \
 --swarm \
 --swarm-master \
 --swarm-discovery token://453sdfjbnfvlknmn3435mwedvmndvnwe \
 swarm-master

```

然后我们将使用这两个节点：

```
docker-machine create \
 --driver digitalocean \
 --digitalocean-access-token sdnjkjdfgkjb345kjdgljknqwetkjwhgoih314rjkwergoiyu34rjkherglkhrg0 \
 --digitalocean-region lon1 \
 --digitalocean-size 1gb \
 --swarm \
 --swarm-discovery token://453sdfjbnfvlknmn3435mwedvmndvnwe \
 swarm-node-01

docker-machine create \
 --driver digitalocean \
 --digitalocean-access-token sdnjkjdfgkjb345kjdgljknqwetkjwhgoih314rjkwergoiyu34rjkherglkhrg0 \
 --digitalocean-region lon1 \
 --digitalocean-size 1gb \
 --swarm \
 --swarm-discovery token://453sdfjbnfvlknmn3435mwedvmndvnwe \
 swarm-node-02

```

如以下屏幕截图所示，我在 DigitalOcean 的伦敦数据中心启动了集群，并为两个节点分配了更多资源：

![创建远程集群](img/B05468_02_30.jpg)

我们将配置本地 Docker 客户端，使用以下命令连接远程集群：

```
eval $(docker-machine env --swarm swarm-master)

```

这将为我们提供以下信息：

![创建远程集群](img/B05468_02_31.jpg)

我们将使用这个集群进行本章的下一部分，因此目前尽量保持它运行。如果您无法保持运行，则可以通过运行以下命令删除该集群：

```
docker-machine rm swarm-master swarm-node-01 swarm-node-02

```

您还应该检查 DigitalOcean 控制面板，以确保您的实例已正确终止。

### 注意

请记住，使用公共云服务时，您是按使用量付费的，因此如果您的实例保持开机状态，即使它处于`errored`状态，使用 Docker Machine 时，计费仍然在继续，您会产生费用。

## 发现后端

此时，值得指出的是，Docker 允许您更换发现后端，目前我们正在使用默认的托管发现与 Docker Hub，它不建议在生产环境中使用。

Swarm 支持以下发现服务：

+   **etcd**: [`coreos.com/etcd/`](https://coreos.com/etcd/)

+   **Consul**: [`www.consul.io/`](https://www.consul.io/)

+   **ZooKeeper**: [`zookeeper.apache.org/`](https://zookeeper.apache.org/)

目前，我们只会查看 Docker 提供的工具，而不会考虑任何第三方选项，因此我们将坚持使用默认的 Discovery 后端。

不幸的是，默认的 Discovery 后端没有提供高可用性，这意味着我们的 Swarm 管理器是单点故障。对于我们的需求来说，这不是一个问题；然而，我不建议在生产环境中运行这种配置。

有关不同 Discovery 后端和 Swarm 高可用性的更多信息，请参考以下 URL：

+   **Discovery 后端**: [`docs.docker.com/swarm/discovery/`](https://docs.docker.com/swarm/discovery/)

+   **Swarm 高可用性**: [`docs.docker.com/swarm/multi-manager-setup/`](https://docs.docker.com/swarm/multi-manager-setup/)

我们将在后面的章节中深入讨论调度器，所以现在让我们进入 Docker Toolbox 安装的最后一个服务。

# Docker Compose

到目前为止，在我们对 Docker Toolbox 工具的探索中，我们一直在使用管理 Docker 主机的服务，本章我们要看的最后一个服务是处理容器的服务。我相信您会同意，到目前为止，Docker 提供的工具非常直观，Docker Compose 也不例外。它最初作为一个名为 Fig 的第三方服务开始，由 Orchard Labs 编写（该项目的原始网站仍然可以访问：[`fig.sh/`](http://fig.sh/)）。

原始项目的目标如下：

> *"使用 Docker 提供快速、独立的开发环境"*

自从 Fig 成为 Docker 的一部分后，它们并没有偏离最初的目标：

> *"Compose 是一个定义和运行多容器 Docker 应用程序的工具。使用 Compose，您通过 Compose 文件配置应用程序的服务。然后，使用单个命令，您可以从配置中创建并启动所有服务。"*

在我们开始查看 Compose 文件并启动容器之前，让我们思考一下像 Compose 这样的工具为什么有用。

## 为什么选择 Compose？

启动单个容器就像运行以下命令一样简单：

```
docker run -i -t ubuntu /bin/bash

```

这将启动并附加到一个 Ubuntu 容器。如我们之前提到的，启动简单容器只是其中的一部分，Docker 并不是用来替代虚拟机的，而是用来运行单个应用程序的。

这意味着你不应该在单个容器中运行整个 LAMP 堆栈，应该将 Apache 和 PHP 运行在一个容器中，然后与一个运行 MySQL 的第二个容器连接。你可以更进一步，运行 NGINX 容器、PHP-FPM 容器以及 MySQL 容器。这就变得复杂了。突然间，你的简单启动命令变成了多行命令，所有命令必须按正确的顺序并带有正确的标志执行。

这正是 Docker Compose 试图解决的问题。你可以使用 YAML 文件定义容器，而不是执行多个长命令。这意味着你可以通过一个命令启动应用程序，并将容器启动顺序的逻辑交给 Compose 来处理。

### 注意

**YAML 不是标记语言**（**YAML**）是一种人类友好的数据序列化标准，适用于所有编程语言。

这也意味着你可以将应用程序的 Compose 文件与代码库一起发布，或者直接发送给另一个开发者/管理员，他们将能够按照你预期的方式启动应用程序。

## Compose 文件

几乎每个人在某个时候都会安装、使用或阅读过 WordPress，因此在接下来的几个例子中，我们将使用来自 Docker Hub 的官方 WordPress 容器，你可以在[`hub.docker.com/_/wordpress/`](https://hub.docker.com/_/wordpress/)上找到该容器的详细信息。

### 注意

WordPress 是一个可以用来创建美丽网站、博客或应用的 Web 软件。我们喜欢说 WordPress 既是免费的，又是无价的。欲了解更多信息，请查看[`wordpress.org/`](https://wordpress.org/)。

让我们从安装一个基本的 WordPress 开始，首先创建一个名为`wordpress`的文件夹，然后将以下内容添加到一个名为`docker-compose.yml`的文件中：

```
wordpress:
  container_name: my-wordpress-app
  image: wordpress
  ports:
    - "80:80"  
  links:
    - "mysql:mysql"
mysql:
  container_name: my-wordpress-database
  image: mysql
  environment:
    MYSQL_ROOT_PASSWORD: "password"
```

你将能够通过确保本地 Docker 客户端配置为使用 Swarm 集群来启动应用程序，运行`docker-machine ls`并确保其处于活动状态，然后运行以下命令：

```
eval $(docker-machine env --swarm swarm-master)

```

一旦你的客户端配置为与 Swarm 集群通信，在包含`docker-compose.yml`文件的文件夹中运行以下命令：

```
docker-compose up -d

```

使用命令末尾的`-d`标志会以分离模式启动容器，这意味着容器将在后台运行。如果我们不使用-d 标志，那么我们的容器将会在前台启动，我们将无法继续使用同一终端会话，而不停止正在运行的容器。

你将看到类似以下的输出：

![Compose 文件](img/B05468_02_32.jpg)

如你所见，你可以通过运行`docker ps`来查找启动 WordPress 应用程序的节点的 IP 地址。如果你访问图中显示的 IP 地址，其中列出了`端口 80`，你将看到一个 WordPress 安装界面：

![Compose 文件](img/B05468_02_33.jpg)

一个有趣的地方是，尽管 `my-wordpress-app` 容器在 `docker-compose.yml` 文件中最先定义，Compose 仍然识别它与 `my-wordpress-database` 容器的关联，并首先启动了后者。此外，你可能注意到 `wordpress:latest` 和 `mysql:latest` 镜像已经在 Swarm 集群中的所有节点上被拉取下来。

那么，`docker-compose.yml` 文件本身呢？让我们再看一遍，这次带上一些注释。

就 Compose 而言，我们的 WordPress 应用程序被拆分成两个应用程序，一个叫做 **wordpress**，另一个叫做 **mysql**。让我们再次看看 `docker-compose.yml` 文件：

```
wordpress:
  container_name: my-wordpress-app
  image: wordpress
  ports:
    - "80:80"  
  links:
    - "mysql:mysql"
mysql:
  container_name: my-wordpress-database
  image: mysql
  environment:
    MYSQL_ROOT_PASSWORD: "password"
```

在最顶层，我们有应用程序名称。从这里开始，我们定义应用程序的配置，通过给出键和值，确保你密切关注缩进。我倾向于使用两个空格来明确表示有缩进，但又不会使其变得难以阅读。

我们定义的第一个关键字是 `container_name`，我们不必这么做，因为 Compose 会根据我们所在的文件夹名称和应用程序名称自动命名我们的容器。如果我们没有定义这个关键字，那么我们的容器将被命名为 `wordpress_wordpress_01` 和 `wordpress_mysql_01`。

下一个关键字告诉应用程序使用哪个 `image`，这将直接从 Docker Hub 拉取镜像。

然后我们定义了 `ports`，注意我们只为 **wordpress** 应用程序定义了端口，而没有为 **mysql** 应用程序定义。因为我们希望 **wordpress** 应用程序监听宿主机的端口，所以我们给了 80:80。在这种情况下，第一个 80 是 **宿主机** 端口，第二个是我们希望暴露的 **容器** 端口。

同样，接下来的关键字只在 **wordpress** 应用程序中使用，它定义了 `links`。链接用于将容器连接在一起，这里将 **mysql** 容器暴露给 **wordpress** 容器。这意味着当 **wordpress** 容器启动时，它将知道 **mysql** 容器的 IP 地址，并且只有 **mysql** 容器的端口会暴露给 **wordpress** 容器。

我们定义的最后一个关键字是 `environment`，在这里我们传递更多的键和值，这些将在容器启动时设置为环境变量。

Compose 文件中所有可用键的详细说明可以在官方文档中找到：[`docs.docker.com/compose/compose-file/`](https://docs.docker.com/compose/compose-file/)。

## 启动更多

使用 Compose 的一个优势是它启动的每个环境都是隔离的，让我们使用以下 `docker-compose.yml` 文件启动另一个 WordPress 安装：

```
wordpress:
  container_name: my-other-wordpress-app
  image: wordpress
  ports:
    - "80:80"  
  links:
    - "mysql:mysql"
mysql:
  container_name: my-other-wordpress-database
  image: mysql
  environment:
    MYSQL_ROOT_PASSWORD: "password"
```

如你所见，除了容器名称外，它与我们之前启动的环境完全相同：

![启动更多](img/B05468_02_34.jpg)

你还会注意到，`my-other-wordpress`容器已在集群的第二个节点上启动。目前，每个 Compose 环境会在单一节点上启动。随着我们启动更多的环境，我们将需要开始调整端口分配，因为它们将在主机上发生冲突（也就是说，你不能将两个`port 80`分配给同一主机）。

### 注意

别忘了通过使用`docker-machine rm`命令删除你启动的任何基于云的实例，同时检查你的云服务提供商的控制面板，确保这些实例已正确终止。

# 总结

在本章中，我们介绍了 Docker 提供的附加客户端工具，这些工具可以扩展你核心 Docker 安装的功能。我们所查看的所有工具都旨在融入你的工作流程，并尽可能简单易用。在接下来的章节中，我们将探讨如何使用第三方服务扩展 Docker 的一些核心功能。到时，我们会重新审视本章中提到的一些工具，并看看它们如何为这些工具增加额外的功能。
