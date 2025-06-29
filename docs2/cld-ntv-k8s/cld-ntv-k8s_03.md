# *第二章*：设置您的 Kubernetes 集群

本章将回顾一些创建 Kubernetes 集群的可能性，这些内容对于学习本书中的其他概念至关重要。我们将从 minikube 开始，这是一款用于创建简单本地集群的工具，然后简要介绍一些附加的、更高级（并且适用于生产环境）的工具，并回顾主要的公共云提供商的托管 Kubernetes 服务，最后我们将介绍从零开始创建集群的策略。

本章将涵盖以下主题：

+   创建第一个集群的选项

+   minikube – 一种简便的入门方式

+   托管服务 – EKS、GKE、AKS 等

+   Kubeadm – 简单的一致性

+   Kops – 基础设施引导

+   Kubespray – 基于 Ansible 的集群创建

+   完全从零开始创建集群

# 技术要求

要运行本章中的命令，您需要安装 kubectl 工具。安装说明请参见*第一章*，*与 Kubernetes 通信*。

如果您打算使用本章中的任何方法创建集群，您需要查看相关项目文档中的每种方法的具体技术要求。特别是对于 minikube，大多数运行 Linux、macOS 或 Windows 的机器都能正常工作。对于大型集群，请查看您计划使用的工具的具体文档。

本章中使用的代码可以在本书的 GitHub 仓库中找到，链接如下：

[`github.com/PacktPublishing/Cloud-Native-with-Kubernetes/tree/master/Chapter2`](https://github.com/PacktPublishing/Cloud-Native-with-Kubernetes/tree/master/Chapter2)

# 创建集群的选项

有许多方法可以创建 Kubernetes 集群，从简单的本地工具到完全从零开始创建集群。

如果您刚开始学习 Kubernetes，可能希望使用如 minikube 之类的工具来启动一个简单的本地集群。

如果您想为应用程序构建生产集群，您有几个选项：

+   您可以使用如 Kops、Kubespray 或 Kubeadm 等工具程序化地创建集群。

+   您可以使用托管的 Kubernetes 服务。

+   您可以在虚拟机或物理硬件上完全从零开始创建集群。

除非您的集群配置有非常具体的需求（即使是那样），通常不建议完全从头开始创建集群而不使用引导工具。

对于大多数使用案例，决策通常是在使用云提供商的托管 Kubernetes 服务和使用引导工具之间选择。

在隔离的系统中，使用引导工具是唯一可行的方式——但对于特定的使用案例，某些工具比其他工具更合适。特别地，Kops 旨在简化在 AWS 等云提供商上创建和管理集群的过程。

重要提示

本节未讨论第三方托管服务或集群创建和管理工具，如 Rancher 或 OpenShift。在选择用于生产环境的集群时，考虑多种因素至关重要，包括当前基础设施、业务需求等。为了简化起见，本书将重点关注生产集群，假设没有其他基础设施或超具体的业务需求——可以说是一个“白纸”。

# minikube – 一种简单的启动方式

minikube 是开始使用简单本地集群的最简便方法。此集群不会设置为高可用性，并且不适合生产用途，但它是一个在几分钟内开始在 Kubernetes 上运行工作负载的绝佳方式。

## 安装 minikube

minikube 可以安装在 Windows、macOS 和 Linux 上。以下是所有三个平台的安装说明，你也可以通过访问 [`minikube.sigs.k8s.io/docs/start`](https://minikube.sigs.k8s.io/docs/start) 获取这些信息。

### 在 Windows 上安装

在 Windows 上最简单的安装方法是从[`storage.googleapis.com/minikube/releases/latest/minikube-installer.exe`](https://storage.googleapis.com/minikube/releases/latest/minikube-installer.exe) 下载并运行 minikube 安装程序。

### 在 macOS 上安装

使用以下命令下载并安装二进制文件。你也可以在代码库中找到它：

Minikube-install-mac.sh

```
     curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-darwin-amd64 \
&& sudo install minikube-darwin-amd64 /usr/local/bin/minikube
```

### 在 Linux 上安装

使用以下命令下载并安装二进制文件：

Minikube-install-linux.sh

```
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 \
&& sudo install minikube-linux-amd64 /usr/local/bin/minikube
```

## 在 minikube 上创建集群

要开始在 minikube 上使用集群，只需运行`minikube start`，它将使用默认的 VirtualBox VM 驱动程序创建一个简单的本地集群。minikube 还有几个额外的配置选项，可以在文档网站上查看。

运行 `minikube` `start` 命令将自动配置你的 `kubeconfig` 文件，这样你就可以在新创建的集群上运行 `kubectl` 命令，而无需进行进一步配置。

# 托管 Kubernetes 服务

提供托管 Kubernetes 服务的云提供商数量在不断增长。然而，出于本书的目的，我们将重点关注主要的公共云及其特定的 Kubernetes 服务。这包括以下内容：

+   **亚马逊网络服务**（**AWS**） – **弹性 Kubernetes 服务**（**EKS**）

+   Google Cloud – **Google Kubernetes 引擎**（**GKE**）

+   Microsoft Azure – **Azure Kubernetes 服务**（**AKS**）

    重要提示

    管理型 Kubernetes 服务的数量和实施方式始终在变化。选择 AWS、Google Cloud 和 Azure 作为本书这一章节的内容，因为它们很可能会继续以相同的方式运行。无论使用哪个管理型服务，都要确保检查服务附带的官方文档，以确保集群创建过程仍与本书中所述相同。

## 管理型 Kubernetes 服务的好处

一般来说，主要的管理型 Kubernetes 服务提供一些好处。首先，我们审查的三个管理型服务都提供完全托管的 Kubernetes 控制平面。

这意味着，当您使用这些管理型 Kubernetes 服务时，您无需担心主节点。它们已被抽象化，几乎可以认为它们不存在。这三个管理型集群都允许您在创建集群时选择工作节点的数量。

管理型集群的另一个好处是可以无缝地将 Kubernetes 从一个版本升级到另一个版本。一般来说，一旦为管理型服务验证了一个新的 Kubernetes 版本（并不总是最新版本），您应该能够通过一键操作或相对简单的程序来进行升级。

## 管理型 Kubernetes 服务的缺点

尽管管理型 Kubernetes 集群在许多方面可以简化操作，但也有一些缺点。

对于许多可用的管理型 Kubernetes 服务，管理集群的最低成本远超手动创建或使用像 Kops 这样的工具创建的最小集群成本。对于生产使用场景，这通常不是问题，因为生产集群应包含最低数量的节点，但对于开发环境或测试集群，额外的成本可能无法根据预算与操作简便性相匹配。

此外，虽然抽象化主节点使得操作更简单，但它也限制了主节点功能的微调或高级功能，这些功能在具有定义主节点的集群中可能是可用的。

# AWS – 弹性 Kubernetes 服务

AWS 的管理型 Kubernetes 服务称为 EKS（Elastic Kubernetes Service）。有几种不同的方式可以开始使用 EKS，但我们将介绍最简单的方式。

## 开始使用

为了创建一个 EKS 集群，您必须配置适当的**虚拟私有云（VPC）**和**身份与访问管理（IAM）**角色设置——此时您就可以通过控制台创建集群。这些设置可以通过控制台手动创建，或者使用像 CloudFormation 和 Terraform 这样的基础设施配置工具来创建。有关通过控制台创建集群的完整说明，请参见 [`docs.aws.amazon.com/en_pv/eks/latest/userguide/getting-started-console.html`](https://docs.aws.amazon.com/en_pv/eks/latest/userguide/getting-started-console.html)。

假设您是从头开始创建集群和 VPC，则可以使用名为`eksctl`的工具来提供您的集群。

若要安装`eksctl`，您可以在 macOS、Linux 和 Windows 上找到安装说明，请访问[`docs.aws.amazon.com/eks/latest/userguide/getting-started-eksctl.html`](https://docs.aws.amazon.com/eks/latest/userguide/getting-started-eksctl.html)。

一旦安装了`eksctl`，创建集群就像使用`eksctl create cluster`命令一样简单：

Eks-create-cluster.sh

```
eksctl create cluster \
--name prod \
--version 1.17 \
--nodegroup-name standard-workers \
--node-type t2.small \
--nodes 3 \
--nodes-min 1 \
--nodes-max 4 \
--node-ami auto
```

这将创建一个包含三个`t2.small`实例作为工作节点的集群，并设置为具有一个节点的自动伸缩组和最多四个节点。使用的 Kubernetes 版本将为`1.17`。重要的是，`eksctl`从默认区域开始，并根据选择的节点数量将它们分布在该区域的多个可用区中。

`eksctl`还将自动更新您的`kubeconfig`文件，因此在集群创建过程完成后，您应该能够立即运行`kubectl`命令。

使用以下代码测试配置：

```
kubectl get nodes
```

您应该看到节点及其关联 IP 的列表。您的集群已准备就绪！接下来，让我们看一看 Google 的 GKE 设置过程。

# Google Cloud – Google Kubernetes Engine

GKE 是 Google Cloud 的托管 Kubernetes 服务。使用 gcloud 命令行工具，快速启动 GKE 集群非常容易。

## 入门

若要使用 gcloud 在 GKE 上创建集群，您可以使用 Google Cloud 的 Cloud Shell 服务，或在本地运行命令。如果要在本地运行命令，则必须通过 Google Cloud SDK 安装 gcloud CLI。请参阅[`cloud.google.com/sdk/docs/quickstarts`](https://cloud.google.com/sdk/docs/quickstarts)获取安装说明。

安装完 gcloud 后，您需要确保已在您的 Google Cloud 帐户中激活了 GKE API。

要轻松完成此操作，请转到[`console.cloud.google.com/apis/library`](https://console.cloud.google.com/apis/library)，然后在搜索栏中搜索`kubernetes`。点击**Kubernetes Engine API**，然后点击**启用**。

现在 API 已激活，请使用以下命令在 Google Cloud 中设置您的项目和计算区域：

```
gcloud config set project proj_id
gcloud config set compute/zone compute_zone
```

在命令中，`proj_id`对应于您希望在其中创建集群的 Google Cloud 项目 ID，而`compute_zone`对应于您在 Google Cloud 中所需的计算区域。

实际上，GKE 上有三种类型的集群，每种具有不同（增加）的可靠性和容错能力：

+   单区域集群

+   多区域集群

+   区域集群

在 GKE 中，**单区域**集群意味着具有单个控制平面副本和一个或多个工作节点的集群，这些节点运行在同一 Google Cloud 区域中。如果区域发生故障，控制平面和工作节点（因此工作负载）将同时停机。

在 GKE 中，**多区域**集群意味着一个具有单个控制平面副本和两个或更多运行在不同 Google Cloud 区域的工作节点的集群。这意味着如果一个单独的区域（即使是包含控制平面的区域）宕机，运行在集群中的工作负载仍将持久存在，但 Kubernetes API 将不可用，直到控制平面区域恢复为止。

最后，在 GKE 中，**区域**集群意味着一个具有多区域控制平面和多区域工作节点的集群。如果任何区域宕机，控制平面和工作在工作节点上的负载都将持久存在。这是最昂贵和可靠的选择。

现在，要实际创建您的集群，您可以运行以下命令来创建一个名为`dev`且具有默认设置的集群：

```
gcloud container clusters create dev \
    --zone [compute_zone]
```

这个命令将在您选择的计算区域内创建一个单区域集群。

为了创建一个多区域集群，您可以运行以下命令：

```
gcloud container clusters create dev \
    --zone [compute_zone_1]
    --node-locations [compute_zone_1],[compute_zone_2],[etc]
```

在这里，`compute_zone_1`和`compute_zone_2`是不同的 Google Cloud 区域。此外，可以通过`node-locations`标志添加更多区域。

最后，要创建一个区域集群，您可以运行以下命令：

```
gcloud container clusters create dev \
    --region [region] \
    --node-locations [compute_zone_1],[compute_zone_2],[etc]
```

在这种情况下，`node-locations`标志实际上是可选的。如果省略，集群将在区域内所有区域的工作节点中创建。如果您希望更改此默认行为，可以使用`node-locations`标志进行覆盖。

现在，您的集群正在运行，您需要配置您的`kubeconfig`文件以与集群通信。为此，只需将集群名称传递到以下命令中：

```
gcloud container clusters get-credentials [cluster_name]
```

最后，使用以下命令测试配置：

```
kubectl get nodes
```

与 EKS 一样，您应该看到所有已提供节点的列表。成功！最后，让我们看一下 Azure 的托管服务。

# Microsoft Azure – Azure Kubernetes Service

Microsoft Azure 的托管 Kubernetes 服务称为 AKS。可以通过 Azure CLI 在 AKS 上创建集群。

## 入门

要在 AKS 上创建集群，您可以使用 Azure CLI 工具，并运行以下命令来创建服务主体（集群用于访问 Azure 资源的角色）：

```
az ad sp create-for-rbac --skip-assignment --name myClusterPrincipal
```

此命令的结果将是一个包含服务主体信息的 JSON 对象，我们将在下一步中使用它。此 JSON 对象看起来像以下内容：

```
{
  "appId": "559513bd-0d99-4c1a-87cd-851a26afgf88",
  "displayName": "myClusterPrincipal",
  "name": "http://myClusterPrincipal",
  "password": "e763725a-5eee-892o-a466-dc88d980f415",
  "tenant": "72f988bf-90jj-41af-91ab-2d7cd011db48"
}
```

现在，您可以使用上一个 JSON 命令中的值实际创建您的 AKS 集群：

Aks-create-cluster.sh

```
az aks create \
    --resource-group devResourceGroup \
    --name myCluster \
    --node-count 2 \
    --service-principal <appId> \
    --client-secret <password> \
    --generate-ssh-keys
```

这个命令假设已经存在一个名为`devResourceGroup`的资源组和一个名为`devCluster`的集群。对于`appId`和`password`，请使用服务主体创建步骤中的值。

最后，为了在您的机器上生成正确的`kubectl`配置，您可以运行以下命令：

```
az aks get-credentials --resource-group devResourceGroup --name myCluster
```

到了这一步，您应该能够正确运行`kubectl`命令。使用`kubectl get nodes`命令测试配置。

# 编程化集群创建工具

有几种工具可用于在各种非托管环境中引导 Kubernetes 集群。我们将重点介绍三种最流行的工具：Kubeadm、Kops 和 Kubespray。每种工具都有不同的使用场景，并且通常采用不同的方法。

## Kubeadm

Kubeadm 是由 Kubernetes 社区创建的工具，用于简化在已配置好的基础设施上创建集群。与 Kops 不同，Kubeadm 没有在云服务上配置基础设施的能力。它仅创建一个符合最佳实践的集群，能够通过 Kubernetes 一致性测试。Kubeadm 与基础设施无关——它应该可以在任何能够运行 Linux 虚拟机的地方工作。

## Kops

Kops 是一个流行的集群配置工具。它为集群配置底层基础设施，安装所有集群组件，并验证集群功能。它还可以用于执行各种集群操作，如升级、节点轮换等。Kops 目前支持 AWS，且（在撰写本书时）对 Google Compute Engine 和 OpenStack 提供 beta 支持，对 VMware vSphere 和 DigitalOcean 提供 alpha 支持。

## Kubespray

Kubespray 与 Kops 和 Kubeadm 有所不同。与 Kops 不同，Kubespray 本身并不会自动配置集群资源。相反，Kubespray 允许您选择使用 Ansible 或 Vagrant 来进行配置、编排和节点设置。

与 Kubeadm 相比，Kubespray 集成的集群创建和生命周期管理过程要少得多。Kubespray 的较新版本允许您在节点设置后，专门使用 Kubeadm 来创建集群。

重要提示

由于使用 Kubespray 创建集群需要一些特定于 Ansible 的领域知识，我们将在本书中不讨论这一部分内容——但关于 Kubespray 的所有内容可以参考[`github.com/kubernetes-sigs/kubespray/blob/master/docs/getting-started.md`](https://github.com/kubernetes-sigs/kubespray/blob/master/docs/getting-started.md)。

# 使用 Kubeadm 创建集群

要使用 Kubeadm 创建一个集群，您需要提前为节点进行配置。与任何其他 Kubernetes 集群一样，我们需要运行 Linux 的虚拟机或裸金属服务器。

本书将展示如何仅使用单个主节点来引导一个 Kubeadm 集群。对于高可用设置，您需要在其他主节点上运行额外的加入命令，相关内容可以在[`kubernetes.io/docs/setup/production-environment/tools/kubeadm/high-availability/`](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/high-availability/)找到。

## 安装 Kubeadm

首先，您需要在所有节点上安装 Kubeadm。每个支持的操作系统的安装说明可以在[`kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm`](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm)找到。

对于每个节点，还需要确保检查所有必需的端口是否已开放，并且已安装你所选择的容器运行时。

## 启动主节点

要快速启动主节点，可以使用 Kubeadm 运行一个命令：

```
kubeadm init
```

此初始化命令可以接受多个可选参数——根据你选择的集群设置、网络配置等，可能需要使用这些参数。

在`init`命令的输出中，你将看到一个`kubeadm join`命令。请确保保存这个命令。

## 启动工作节点

为了启动工作节点，你需要运行之前保存的`join`命令。该命令的格式如下：

```
kubeadm join --token [TOKEN] [IP ON MASTER]:[PORT ON MASTER] --discovery-token-ca-cert-hash sha256:[HASH VALUE]
```

此命令中的令牌是一个引导令牌。它用于对节点进行身份验证并将新节点加入集群。拥有此令牌的访问权限意味着你有能力将新节点加入集群，因此请谨慎对待它。

## 配置 kubectl

使用 Kubeadm 时，`kubectl`已经在主节点上正确配置。不过，如果你希望在其他机器或集群外部使用`kubectl`，可以将配置从主节点复制到本地机器：

```
scp root@[IP OF MASTER]:/etc/kubernetes/admin.conf .
kubectl --kubeconfig ./admin.conf get nodes 
```

此`kubeconfig`将是集群管理员配置——若要为其他用户指定权限，你需要添加新的服务帐户，并为其生成`kubeconfig`文件。

# 使用 Kops 创建集群

由于 Kops 将为你提供基础设施，因此无需预先创建任何节点。你只需安装 Kops，确保你的云平台凭证正常工作，然后一键创建集群。Kops 可以在 Linux、macOS 和 Windows 上安装。

本教程将演示如何在 AWS 上创建集群，但你可以在 Kops 文档中找到其他支持的平台的安装说明：[`github.com/kubernetes/kops/tree/master/docs`](https://github.com/kubernetes/kops/tree/master/docs)。

## 在 macOS 上安装

在 OS X 上，安装 Kops 的最简单方法是使用 Homebrew：

```
brew update && brew install kops
```

或者，你也可以从 Kops 的 GitHub 页面下载最新的稳定版 Kops 二进制文件：[`github.com/kubernetes/kops/releases/tag/1.12.3`](https://github.com/kubernetes/kops/releases/tag/1.12.3)。

## 在 Linux 上安装

在 Linux 上，你可以通过以下命令安装 Kops：

Kops-linux-install.sh

```
curl -LO https://github.com/kubernetes/kops/releases/download/$(curl -s https://api.github.com/repos/kubernetes/kops/releases/latest | grep tag_name | cut -d '"' -f 4)/kops-linux-amd64
chmod +x kops-linux-amd64
sudo mv kops-linux-amd64 /usr/local/bin/kops
```

## 在 Windows 上安装

要在 Windows 上安装 Kops，你需要从[`github.com/kubernetes/kops/releases/latest`](https://github.com/kubernetes/kops/releases/latest)下载最新的 Windows 版本，重命名为`kops.exe`，并将其添加到你的`path`变量中。

## 配置 Kops 凭证

为了使 Kops 正常工作，你需要在机器上配置 AWS 凭证，并授予一些必要的 IAM 权限。为了安全起见，你应为 Kops 创建一个专用的 IAM 用户。

首先，为`kops`用户创建一个 IAM 组：

```
aws iam create-group --group-name kops_users
```

然后，为 `kops_users` 组附加所需的角色。为了正常运行，Kops 需要 `AmazonEC2FullAccess`、`AmazonRoute53FullAccess`、`AmazonS3FullAccess`、`IAMFullAccess` 和 `AmazonVPCFullAccess`。我们可以通过运行以下命令来实现：

提供 AWS 策略给 kops.sh

```
aws iam attach-group-policy --policy-arn arn:aws:iam::aws:policy/AmazonEC2FullAccess --group-name kops
aws iam attach-group-policy --policy-arn arn:aws:iam::aws:policy/AmazonRoute53FullAccess --group-name kops
aws iam attach-group-policy --policy-arn arn:aws:iam::aws:policy/AmazonS3FullAccess --group-name kops
aws iam attach-group-policy --policy-arn arn:aws:iam::aws:policy/IAMFullAccess --group-name kops
aws iam attach-group-policy --policy-arn arn:aws:iam::aws:policy/AmazonVPCFullAccess --group-name kops
```

最后，创建 `kops` 用户，将其添加到 `kops_users` 组中，并创建编程访问密钥，请保存这些密钥：

```
aws iam create-user --user-name kops
aws iam add-user-to-group --user-name kops --group-name kops_users
aws iam create-access-key --user-name kops
```

为了允许 Kops 访问您的新 IAM 凭证，您可以使用以下命令将访问密钥和密钥对配置到 AWS CLI 中，这些密钥来自前一个命令（`create-access-key`）：

```
aws configure
export AWS_ACCESS_KEY_ID=$(aws configure get aws_access_key_id)
export AWS_SECRET_ACCESS_KEY=$(aws configure get aws_secret_access_key)
```

## 设置状态存储

配置好适当的凭证后，我们可以开始创建集群。在本例中，我们将构建一个简单的基于 gossip 的集群，因此不需要处理 DNS。要查看可能的 DNS 设置，可以查看 Kops 文档 ([`github.com/kubernetes/kops/tree/master/docs`](https://github.com/kubernetes/kops/tree/master/docs))。

首先，我们需要一个位置来存储我们的集群规格。由于我们使用的是 AWS，S3 是一个完美的选择。

和往常一样，S3 存储桶的名称需要唯一。您可以使用 AWS SDK 轻松创建一个存储桶（确保将 `my-domain-dev-state-store` 替换为您希望使用的 S3 存储桶名称）：

```
aws s3api create-bucket \
    --bucket my-domain-dev-state-store \
    --region us-east-1
```

最佳实践是启用存储桶加密和版本控制：

```
aws s3api put-bucket-versioning --bucket prefix-example-com-state-store  --versioning-configuration Status=Enabled
aws s3api put-bucket-encryption --bucket prefix-example-com-state-store --server-side-encryption-configuration '{"Rules":[{"ApplyServerSideEncryptionByDefault":{"SSEAlgorithm":"AES256"}}]}'
```

最后，要为 Kops 设置变量，请使用以下命令：

```
export NAME=devcluster.k8s.local
export KOPS_STATE_STORE=s3://my-domain-dev-cluster-state-store
```

重要提示

Kops 支持多个状态存储位置，例如 AWS S3、Google Cloud Storage、Kubernetes、DigitalOcean、OpenStack Swift、阿里云和 memfs。然而，您也可以将 Kops 状态保存到本地文件中并使用它。使用云端状态存储的好处是多个基础设施开发人员可以访问并更新它，同时进行版本控制。

## 创建集群

使用 Kops，我们可以部署任意大小的集群。为了本指南的目的，我们将通过让工作节点和主节点跨越三个可用区来部署一个生产就绪的集群。我们将使用 US-East-1 区域，主节点和工作节点将是 `t2.medium` 实例。

要为该集群创建配置，您可以运行以下 `kops create` 命令：

Kops-create-cluster.sh

```
kops create cluster \
    --node-count 3 \
    --zones us-east-1a,us-east-1b,us-east-1c \
    --master-zones us-east-1a,us-east-1b,us-east-1c \
    --node-size t2.medium \
    --master-size t2.medium \
    ${NAME}
```

要查看已创建的配置，请使用以下命令：

```
kops edit cluster ${NAME}
```

最后，要创建我们的集群，请运行以下命令：

```
kops update cluster ${NAME} --yes
```

集群创建过程可能需要一些时间，但一旦完成，您的 `kubeconfig` 应该已经正确配置，可以与新的集群一起使用 kubectl。

# 从头创建集群

从头创建 Kubernetes 集群是一个多步骤的过程，可能会跨越本书的多个章节。然而，由于我们的目的是让你尽快启动并运行 Kubernetes，我们将避免描述整个过程。

如果您有兴趣从零开始创建集群，无论是出于教育目的还是需要精细定制您的集群，一个很好的指南是*Kubernetes The Hard Way*，这是由*Kelsey Hightower*编写的完整集群创建教程。您可以在[`github.com/kelseyhightower/kubernetes-the-hard-way`](https://github.com/kelseyhightower/kubernetes-the-hard-way)找到它。

既然我们已经解决了这个问题，我们可以继续概述手动集群创建过程。

## 配置您的节点

首先，您需要一些基础设施来运行 Kubernetes。通常，虚拟机是一个不错的选择，尽管 Kubernetes 也可以在裸机上运行。如果您工作在一个无法轻松添加节点的环境中（这会减少云的许多扩展优势，但在企业环境中是完全可能的），您需要足够的节点来满足应用程序的需求。这在隔离环境中更可能成为问题。

您的一些节点将用于主控平面，而其他节点将仅作为工作节点。无需在内存或 CPU 方面使主节点和工作节点完全相同——您甚至可以有一些较弱和一些更强大的工作节点。这种模式导致一个非同质的集群，其中某些节点更适合特定的工作负载。

## 为 TLS 创建 Kubernetes 证书颁发机构

为了正常运行，所有主要的控制平面组件都需要 TLS 证书。为此，必须创建一个**证书颁发机构**（**CA**），该 CA 将进一步创建 TLS 证书。

为了创建 CA，必须启动一个**公共密钥基础设施**（**PKI**）。对于此任务，您可以使用任何 PKI 工具，但 Kubernetes 文档中使用的是 cfssl。

一旦为所有组件创建了 PKI、CA 和 TLS 证书，下一步是为控制平面和工作节点组件创建配置文件。

## 创建配置文件

需要为`kubelet`、`kube-proxy`、`kube-controller-manager`和`kube-scheduler`组件创建配置文件。它们将使用这些配置文件中的证书与`kube-apiserver`进行身份验证。

## 创建 etcd 集群并配置加密

创建数据加密配置是通过包含数据加密密钥的 YAML 文件来处理的。此时，必须启动`etcd`集群。

为此，在每个节点上创建带有`etcd`进程配置的`systemd`文件。然后，在每个节点上使用`systemctl`启动`etcd`服务器。

这是一个`etcd`的`systemd`文件示例。其他控制平面组件的`systemd`文件将类似于此：

示例-systemd-control-plane

```
[Unit]
Description=etcd
Documentation=https://github.com/coreos
[Service]
Type=notify
ExecStart=/usr/local/bin/etcd \\
  --name ${ETCD_NAME} \\
  --cert-file=/etc/etcd/kubernetes.pem \\
  --key-file=/etc/etcd/kubernetes-key.pem \\
  --peer-cert-file=/etc/etcd/kubernetes.pem \\
  --peer-key-file=/etc/etcd/kubernetes-key.pem \\
  --trusted-ca-file=/etc/etcd/ca.pem \\
  --peer-trusted-ca-file=/etc/etcd/ca.pem \\
  --peer-client-cert-auth \\
  --initial-cluster-state new \\
  --data-dir=/var/lib/etcd
Restart=on-failure
RestartSec=5
[Install]
WantedBy=multi-user.target
```

该服务文件为我们的`etcd`组件提供了运行时定义，`etcd`将被启动在每个主节点上。要在节点上实际启动`etcd`，我们运行以下命令：

```
{
  sudo systemctl daemon-reload
  sudo systemctl enable etcd
  sudo systemctl start etcd
}
```

这使得`etcd`服务能够启用，并在节点重启时自动重启。

## 引导控制平面组件

在主节点上引导控制平面组件的过程类似于创建`etcd`集群的过程。为每个组件——API 服务器、控制器管理器和调度器——创建`systemd`文件，然后使用`systemctl`命令启动每个组件。

之前创建的配置文件和证书也需要包含在每个主节点上。

让我们来看一下我们为`kube-apiserver`组件定义的服务文件，按以下部分分解。`Unit`部分只是我们`systemd`文件的简短描述：

```
[Unit]
Description=Kubernetes API Server
Documentation=https://github.com/kubernetes/kubernetes
```

Api-server-systemd-example

这第二部分是服务的实际启动命令，以及要传递给服务的任何变量：

```
[Service]
ExecStart=/usr/local/bin/kube-apiserver \\
  --advertise-address=${INTERNAL_IP} \\
  --allow-privileged=true \\
  --apiserver-count=3 \\
  --audit-log-maxage=30 \\
  --audit-log-maxbackup=3 \\
  --audit-log-maxsize=100 \\
  --audit-log-path=/var/log/audit.log \\
  --authorization-mode=Node,RBAC \\
  --bind-address=0.0.0.0 \\
  --client-ca-file=/var/lib/kubernetes/ca.pem \\
  --enable-admission-plugins=NamespaceLifecycle,NodeRestriction,LimitRanger,ServiceAccount,DefaultStorageClass,ResourceQuota \\
  --etcd-cafile=/var/lib/kubernetes/ca.pem \\
  --etcd-certfile=/var/lib/kubernetes/kubernetes.pem \\
  --etcd-keyfile=/var/lib/kubernetes/kubernetes-key.pem \\
  --etcd-
  --service-account-key-file=/var/lib/kubernetes/service-account.pem \\
  --service-cluster-ip-range=10.10.0.0/24 \\
  --service-node-port-range=30000-32767 \\
  --tls-cert-file=/var/lib/kubernetes/kubernetes.pem \\
  --tls-private-key-file=/var/lib/kubernetes/kubernetes-key.pem \\
  --v=2
```

最后，`Install`部分允许我们指定一个`WantedBy`目标：

```
Restart=on-failure
RestartSec=5
 [Install]
WantedBy=multi-user.target
```

`kube-scheduler`和`kube-controller-manager`的服务文件将与`kube-apiserver`的定义非常相似，一旦我们准备好在节点上启动这些组件，过程就很简单：

```
{
  sudo systemctl daemon-reload
  sudo systemctl enable kube-apiserver kube-controller-manager kube-scheduler
  sudo systemctl start kube-apiserver kube-controller-manager kube-scheduler
}
```

类似于`etcd`，我们希望在节点关闭时确保服务能够重启。

## 引导工作节点

在工作节点上情况类似。需要为`kubelet`、容器运行时、`cni`和`kube-proxy`创建并使用`systemctl`运行服务规格。`kubelet`配置将指定前面提到的 TLS 证书，以便它能通过 API 服务器与控制平面进行通信。

让我们来看一下`kubelet`服务定义的样子：

Kubelet-systemd-example

```
[Unit]
Description=Kubernetes Kubelet
Documentation=https://github.com/kubernetes/kubernetes
After=containerd.service
Requires=containerd.service
[Service]
ExecStart=/usr/local/bin/kubelet \\
  --config=/var/lib/kubelet/kubelet-config.yaml \\
  --container-runtime=remote \\
  --container-runtime-endpoint=unix:///var/run/containerd/containerd.sock \\
  --image-pull-progress-deadline=2m \\
  --kubeconfig=/var/lib/kubelet/kubeconfig \\
  --network-plugin=cni \\
  --register-node=true \\
  --v=2
Restart=on-failure
RestartSec=5
[Install]
WantedBy=multi-user.target
```

如你所见，这个服务定义引用了`cni`、容器运行时和`kubelet-config`文件。`kubelet-config`文件包含了我们为工作节点所需的 TLS 信息。

在引导工作节点和主节点之后，通过使用在 TLS 设置过程中创建的管理员`kubeconfig`文件，集群应该可以正常工作。

# 总结

在本章中，我们回顾了几种创建 Kubernetes 集群的方法。我们讨论了使用 minikube 创建最小化本地集群、在 Azure、AWS 和 Google Cloud 上设置托管的 Kubernetes 服务、使用 Kops 配置工具创建集群，以及最后从头开始手动创建集群。

现在，我们掌握了在多个不同环境中创建 Kubernetes 集群的技能，可以继续使用 Kubernetes 来运行应用程序。

在下一章中，我们将学习如何在 Kubernetes 上运行应用程序。你已经掌握了 Kubernetes 在架构层面是如何工作的知识，这将使你更容易理解接下来的几章中的概念。

# 问题

1.  minikube 的作用是什么？

1.  使用托管的 Kubernetes 服务有什么缺点？

1.  Kops 和 Kubeadm 有什么区别？主要区别是什么？

1.  Kops 支持哪些平台？

1.  手动创建集群时，如何指定主要的集群组件？它们如何在每个节点上运行？

# 进一步阅读

+   官方 Kubernetes 文档: [`kubernetes.io/docs/home/`](https://kubernetes.io/docs/home/)

+   *Kubernetes The Hard Way*: [`github.com/kelseyhightower/kubernetes-the-hard-way`](https://github.com/kelseyhightower/kubernetes-the-hard-way)
