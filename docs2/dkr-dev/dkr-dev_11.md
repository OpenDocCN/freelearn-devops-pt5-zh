# *第九章*：使用 Spinnaker 进行云原生持续部署

将 Docker 容器作为云原生应用程序部署到 Kubernetes 面临着挑战，这些挑战可以通过专门的容器中心持续部署系统来解决。与我们之前在部署到单一主机时通过 Jenkins 运行自定义部署逻辑不同，我们可以使用 Spinnaker 来部署到 Kubernetes。因为 Spinnaker 可以与 Jenkins 配合使用，我们可以继续使用已经设置好的 Jenkins 服务器来构建 Docker 容器并准备部署的 Helm Charts。通过 Spinnaker，我们将使用其内置的 Helm Charts 和 Kubernetes 部署支持来部署应用程序。我们还将探讨 Spinnaker 的一些专门部署策略，看看它们如何应用于以 Kubernetes 为中心的环境。

在本章中，我们将学习在什么情况下以及为什么你会选择除了 Jenkins 之外还使用 Spinnaker。我们将学习如何通过配置 Spinnaker 并将其与 GitHub、Docker Hub 和 Jenkins 集成，来改进你的环境以支持 Kubernetes 应用程序的部署和维护。我们将学习如何通过 Spinnaker 管道和 AWS **Elastic Container Registry** (**ECR**)将应用程序部署到 Kubernetes，同时也将了解当你将 Spinnaker 与 Kubernetes 结合使用时，Spinnaker 支持不同部署和测试策略的方式是否适用。

本章将覆盖以下主题：

+   改进你的 Kubernetes 应用程序维护环境

+   Spinnaker —— 你何时以及为什么可能需要更复杂的部署

+   在 AWS EKS 集群中使用 Helm 设置 Spinnaker

+   在 Spinnaker 中使用简单的部署策略部署 ShipIt Clicker

+   了解 Spinnaker 在 Kubernetes 应用程序中支持不同部署和测试策略的情况

让我们先回顾一下本章的技术要求，然后继续学习 Spinnaker 平台。

# 技术要求

你需要在云中拥有一个可用的 Kubernetes 集群，如前一章所述。你可以复用该集群，或者使用相同的方法或`eksctl`为本章设置一个新的集群。请注意，本章所描述的 Spinnaker 版本与 Kubernetes 1.16 及以后的版本不兼容；务必在 Kubernetes 1.15 集群上安装此版本。你还需要在本地工作站上安装当前版本的 AWS `kubectl`和`helm` 3.x，正如前一章所述。本章中的 Helm 命令使用的是`helm` 3.x 语法。AWS **弹性 Kubernetes 服务**（**EKS**）集群必须配置有工作中的**应用负载均衡器**（**ALB**）Ingress Controller。我们还将使用前一章中设置的 AWS ECR Docker 仓库。你还需要具备在*第七章*中设置的 Jenkins 服务器，因为 Spinnaker 依赖 Jenkins 构建软件制品。

Spinnaker 比本地工作站可用的资源要求更多，我们还需要将其连接到外部服务，如 Jenkins 和 GitHub，这种方式可能无法与本地 Kubernetes 学习环境兼容。

查看以下视频，看看代码的实际应用：

[`bit.ly/2DUGumq`](https://bit.ly/2DUGumq)

使用更新的 ShipIt Clicker v5

我们将使用以下 GitHub 仓库中的`chapter9`目录中的 ShipIt Clicker 版本：

[`github.com/PacktPublishing/Docker-for-Developers/`](https://github.com/PacktPublishing/Docker-for-Developers/)

这个版本与前一个版本有所不同。它只包含一个 Helm Charts 副本，位于`chapter9/shipitclicker`，并包含几个用于集群部署的覆盖 YAML 文件：`values-eks.yaml`和`values-spin.yaml`。

在前一章中，我们保持了多个冗余的模板和配置文件目录，但在 Helm Charts 中的唯一差异是`values`文件中的覆盖内容。本章中的示例采用了更简洁的策略。事实证明，你可以使用多个 YAML 配置文件，这些文件只覆盖每个部署或环境中需要更改的设置。在本章中，我们将把示例应用程序的容器仓库从 Docker Hub 迁移到 ECR，先手动部署一次，然后切换到使用 Spinnaker 部署 ShipIt Clicker。

# 改进 Kubernetes 应用程序维护的设置

为了部署和维护 Spinnaker，我们需要能够从本地工作站与 Kubernetes 集群进行通信。我们还希望能够使用**安全套接字层**（**SSL**）保护的通信与 Kubernetes 托管的资源进行交互。让我们一步步进行，以便为你的本地工作站和 AWS 账户准备更多的高级部署。

## 从本地工作站管理 EKS 集群

为了更轻松地管理 EKS 集群并与其一起工作，您需要设置本地工作站以与集群通信。在上一章中，我们使用 AWS CLI 配置了 AWS IAM 管理员帐户，并使用它设置了 EKS 集群。在本章中，我们将进一步构建，以确保能够从本地工作站高效管理集群及其内部的应用程序。

要在本地工作站上执行以下操作，以使`kubectl`和其他 Kubernetes 实用程序能够与您的 EKS 集群通信，请按照这里的说明进行操作：

[`aws.amazon.com/premiumsupport/knowledge-center/eks-cluster-connection`](https://aws.amazon.com/premiumsupport/knowledge-center/eks-cluster-connection)

在前面的链接中，关于执行从本地工作站发出的`aws cli`命令的重要部分涉及。执行此命令以更新 `.kube/config`，并添加一个条目，使您能够连接到 EKS 集群，但请用您的 EKS 集群名称替换 `EKS-VIVLKQ5X`：

```
aws eks --region us-east-2 update-kubeconfig --name EKS- VIVLKQ5X
```

然后，测试您是否能够与集群通信：

```
kubectl get nodes
```

如果成功，您将看到组成您的 EKS 集群节点的 EC2 主机列表。

## 故障排除 kubectl 连接失败

如果前面的`aws eks`命令产生错误消息或访问被拒绝消息，或者未能完成操作，则需要先进行故障排除。请按照以下部分的步骤进行操作，并查看 AWS 指南以解决此通信失败问题：

[`aws.amazon.com/premiumsupport/knowledge-center/eks-cluster-connection/`](https://aws.amazon.com/premiumsupport/knowledge-center/eks-cluster-connection/)

### 确保您已激活正确的 AWS CLI 配置文件

如果您有多个 AWS CLI 配置文件，则默认用户可能与预期的用户不匹配。您可以通过`--profile`参数显式告知 AWS CLI 使用配置文件，或者在执行`aws eks`命令之前通过设置`AWS_DEFAULT_PROFILE`变量来强制使用特定配置文件：

```
export AWS_DEFAULT_PROFILE=my-eks-profile
```

现在我们已经使用配置文件设置了 AWS CLI，我们必须再次检查我们是否可以通过检查 CloudFormation 模板访问控制列表来到达我们的 EKS 集群。

### 确保您的 CloudFormation 模板已配置为允许访问

在上一章中，当我们设置 EKS 集群时，我们在 `192.2.0.15/32` 中输入了我们的 IPv4 地址。请通过 [`whatismyip.com/`](https://whatismyip.com/) 仔细检查您的地址。如果这些设置不正确，请使用这些值更新 CloudFormation 堆栈。

CLI 配置文件必须与您用于创建 EKS 集群的 AWS Quick Start 中使用的 IAM 用户匹配。

这将适当地配置 IAM 和 EKS。

## 在本地和集群上下文之间切换

当您配置了多个 Kubernetes 上下文时，您可以通过以下命令切换它们：`kubectl config get-contexts` 和 `kubectl config use-context`：

```
$ kubectl config get-contexts
CURRENT   NAME                                                      CLUSTER                                                   AUTHINFO                                                  NAMESPACE
*         arn:aws:eks:us-east-2:143970405955:cluster/EKS-8PWG76O8   arn:aws:eks:us-east-2:143970405955:cluster/EKS-8PWG76O8   arn:aws:eks:us-east-2:143970405955:cluster/EKS-8PWG76O8
          docker-desktop                                            docker-desktop                                            docker-desktop
$ kubectl config use-context docker-desktop
Switched to context "docker-desktop".
$ kubectl get nodes
NAME             STATUS   ROLES    AGE   VERSION
docker-desktop   Ready    master   21d   v1.15.5
$ kubectl config use-context arn:aws:eks:us-east-2:143970405955:cluster/EKS-VIVLKQ5X
Switched to context "arn:aws:eks:us-east-2:143970405955:cluster/EKS-VIVLKQ5X".
 $ kubectl get nodes
NAME                                        STATUS   ROLES    AGE    VERSION
ip-10-0-31-183.us-east-2.compute.internal   Ready    <none>   2d9h   v1.15.10-eks-bac369
ip-10-0-57-2.us-east-2.compute.internal     Ready    <none>   2d9h   v1.15.10-eks-bac369
ip-10-0-90-115.us-east-2.compute.internal   Ready    <none>   2d9h   v1.15.10-eks-bac369
```

在前面的列表中，我们可以看到所有已定义的上下文。我们还可以看到，当我们使用`docker-desktop`上下文时，只会看到一个节点，但当我们使用 EKS 上下文时，我们可以看到多个 EC2 服务器节点。在本章的剩余部分，我们将针对 EKS 上下文执行与 Kubernetes 相关的命令。

## 验证你是否拥有一个工作正常的 ALB Ingress Controller

在上一章中，我们设置了一个带有 ALB Ingress Controller 的 EKS 集群，以便让外界能够访问 ShipIt Clicker 应用。如果你重新使用那个 EKS 集群，并且 ALB Ingress Controller 正常工作，你可以跳到下一节。

如果你设置了一个新集群，你可以按照上一章的说明来使 ALB Ingress Controller 工作，或者你可以运行本章中提供的其中一个 Shell 脚本，作为快捷方式，如果新集群没有 ALB Ingress Controller。

要使用 ALB Ingress Controller 安装脚本，记下你的 EKS 集群名称，并确保已安装 Helm 和`eksctl`。

然后，从你的本地工作站运行`deploy-alb-ingress-controller.sh`脚本来设置 ALB Ingress Controller（将`EKS-8PWG76O8`替换为你的 EKS 集群名称）：

```
chapter9/bin/deploy-alb-ingress-controller.sh EKS-8PWG76O8
```

现在你已经安装了 ALB Ingress Controller，可以继续在 AWS 中获取一个域名并生成 SSL 证书。

## 准备 Route 53 域名和证书

为了确保你的 EKS 集群与外部世界之间的通信安全，我们将使用以下服务来管理**域名服务器**（**DNS**）条目和服务器证书：

+   **AWS Route 53**：[`aws.amazon.com/route53/`](https://aws.amazon.com/route53/)

+   **AWS 证书管理器**（**ACM**）：[`aws.amazon.com/certificate-manager/`](https://aws.amazon.com/certificate-manager/)

在*第七章*《使用 Jenkins 进行持续部署》中，我们配置了 Jenkins，使用域名映射 ShipIt Clicker 的暂存和生产环境条目。在本章中，我们将使用 Route 53 来管理 DNS 条目，并使用 ACM 管理证书，以帮助确保通信的安全。

你可以将你正在使用的顶级域名转移到 Route 53，或者你可以将你控制的现有域名的子域名（例如`eks.example.com`）委托给 Route 53。请参阅 AWS 指南，了解如何将子域名委托给 Route 53：

[`docs.aws.amazon.com/Route53/latest/DeveloperGuide/CreatingNewSubdomain.html`](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/CreatingNewSubdomain.html)

一旦你将域名委托给 Route 53，验证你是否能够查看该域名的 SOA 记录（将你的域名替换为 eks.example.com）：

```
$ host -t soa eks.example.com eks.example.com has SOA record ns-1372.awsdns-43.org. awsdns-hostmaster.amazon.com. 1 7200 900 1209600 86400
```

如果返回的 SOA 记录与前面的日志类似，那么你已经设置好了。如果出现未找到错误，你需要进一步排查问题。

一旦你的域名解析正常，前往 ACM 控制台 https://us-east-2.console.aws.amazon.com/acm/home?region=us-east-2#/ 并生成包含两个域名的新的公共证书 – `*.eks.example.com` 和 `eks.example.com`（将 `example.com` 替换为你的域名）。以 `*` 开头的域名称为通配符证书，因为它匹配所有具有相同域名后缀的域名。使用这个证书可以让我们拥有一个覆盖多个域名的证书。

使用 DNS 验证方法。由于你的域名已经在 Route 53 中进行管理，你可以展开该域名并点击快捷按钮**在 Route 53 中创建记录**，该按钮应该类似于以下内容：

![](img/B11641_09_001.jpg)

图 9.1 – 在 ACM 中请求证书

这将向你的 Route 53 区域添加验证记录，从而加速证书的颁发。证书可能需要 5 分钟到 1 小时才能颁发，除非 DNS 验证记录有问题，比如域名没有正确地从上一级的名称服务器委派过来。等待证书颁发，并记下证书的 ARN —— 你稍后会用到它。

## 构建并部署 ShipIt Clicker v5

为了验证我们是否支持 SSL 保护的网站，我们将把 ShipIt Clicker 部署到 EKS 并启用 ALB 负载均衡器对 HTTPS 的支持。为了演示我们可以使用 AWS ECR 容器注册表，我们还将把容器推送到 ECR，并使用该注册表部署应用程序。

将 `chapter9/values-eks.yaml` 复制到 `chapter9/values.yaml`，然后编辑 `values.yaml` 文件，如下所示。首先更改文件开头的镜像名称，并将其前缀设置为你的 ECR 容器注册表名称（将 `143970405955` 替换为你的 AWS 账户 ID，并确保区域——在此为 `us-east-2`——与你使用的区域匹配）：

```
---
image:
  repository: 143970405955.dkr.ecr.us-east-2.amazonaws.com/                   dockerfordevelopers/shipitclicker:0.5.0
```

请注意，`values.yaml` 文件中有注释，表明 ALB 应该监听 `80` 和 `443` 端口，并且它在 `host` 设置中有一个完全限定的域名。编辑以下主机条目中的值，以便 `shipit-v5.eks.example.com` 域名与 ACM 中的通配符 SSL 证书匹配的域名一致：

```
ingress:
  enabled: true
  annotations:
    kubernetes.io/ingress.class: alb
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/listen-ports: '[{"HTTPS":443},{"HTTP":80}]'
    alb.ingress.kubernetes.io/target-type: ip
  hosts:	
    - host: "shipit-v5.eks.example.com"
      paths: ['/*']
```

现在我们已经准备好 `values.yml` 文件，我们将构建容器并将其推送到 EKS。

切换到 `Docker-for-Developers/chapter9` 目录并执行以下命令来构建并部署 ShipIt Clicker 到集群中，以测试 ALB 集成（将 `143970405955.dkr.ecr.us-east-2.amazonaws.com` 替换为你的 ECR 注册表）：

```
docker build . -t dockerfordevelopers/shipitclicker:0.5.0
docker tag dockerfordevelopers/shipitclicker:0.5.0 143970405955.dkr.ecr.us-east-2.amazonaws.com/dockerfordevelopers/shipitclicker:0.5.0
aws ecr get-login-password --region us-east-2 | docker login --username AWS --password-stdin 143970405955.dkr.ecr.us-east-2.amazonaws.com
docker push 143970405955.dkr.ecr.us-east-2.amazonaws.com/dockerfordevelopers/shipitclicker:0.5.0
helm install shipit-v5 -f values.yaml ./shipitclicker
```

几分钟后，你应该能够验证 Ingress Controller 是否正常工作：

```
$ kubectl get ingress
NAME                      HOSTS                             ADDRESS                                                                 PORTS   AGE
shipit-v5-shipitclicker   shipit-v5.eks.shipitclicker.com   9bbd6f9c-default-shipitv5s-051a-795288134.us-east-2.elb.amazonaws.com   80      90m
```

如果没有显示，请检查 Ingress Controller 日志，以下是一些排错线索：

```
kubectl logs -n kube-system deployment.apps/alb-ingress-controller
```

接下来，我们需要创建一个 DNS 地址映射记录，也就是前面`kubectl get ingress`输出中的`HOSTS`列。进入 AWS Route 53 控制台，选择你的域名，创建一个类型为 A 的新记录，指向`shipit-v5.eks`。将此记录设置为别名记录，并输入`kubectl get ingress`输出中 ALB 的`HOSTS`列中的 DNS 名称。执行此操作的表单应如下图所示：

![](img/B11641_09_002.jpg)

图 9.2 – 在 AWS Route 53 中创建作为别名的 A 记录

按下`example.com`（使用你自己的域名）来验证你是否可以通过 HTTPS 访问它。

现在，你已经确保可以从本地环境管理 EKS 集群，将演示应用程序的容器推送到 ECR，使用 Helm 将演示应用程序部署到 Kubernetes，并配置 HTTPS 支持以安全地访问一个托管在 EKS 中的服务，你可以继续进行 Spinnaker 的安装。

# Spinnaker – 你可能需要更复杂部署的时机和原因

为了可靠地部署你的应用程序，你可以手动编写许多脚本并使用持续集成系统。然而，许多人已经考虑到在 Kubernetes 中部署应用程序所固有的问题。Kubernetes 确实具有强大的部署能力，尤其是当你使用部署控制器时。但这种方法并不适合所有人。一些人开发了专门的系统来减少处理这些任务的复杂性。像 Jenkins-X、Weaveworks、CodeFresh 和 Spinnaker 这样的系统适合这一需求。我们将更详细地研究 Spinnaker，这是一个持续部署工具集([`www.spinnaker.io/`](https://www.spinnaker.io/))。

我们将首先介绍 Spinnaker 的核心概念，并强调它与其他平台（如 Kubernetes）共享的术语，特别是它们的含义不同之处。

## Spinnaker 简介

Spinnaker 是一个**持续交付**（**CD**）平台，跨多个云提供商工作，并且是开源的。Netflix 最初编写 Spinnaker 是为了帮助管理他们的多云部署，采用不可变服务器模式（请参见[`martinfowler.com/bliki/ImmutableServer.html`](https://martinfowler.com/bliki/ImmutableServer.html)）。Spinnaker 具有一个镜像工厂功能，涉及将应用程序代码与操作系统镜像及支持库结合，然后保存（烘焙）成一个不可变的机器镜像，例如 AWS 的**Amazon 机器镜像**（**AMI**）或 VMware 的**虚拟机磁盘**（**VMDK**）镜像，以加速部署并最小化运行时配置。可以阅读更多关于镜像工厂及其在 Spinnaker 中的使用的相关文章：

+   [`netflixtechblog.com/how-we-build-code-at-netflix-c5d9bd727f15`](https://netflixtechblog.com/how-we-build-code-at-netflix-c5d9bd727f15)

+   [`docs.armory.io/spinnaker-install-admin-guides/packer/`](https://docs.armory.io/spinnaker-install-admin-guides/packer/)

这种模式在大规模应用中表现良好，但 Docker 和以容器为中心的运行时（如 Kubernetes）的出现，提供了一种不同的方法来实现相同的目标。

Spinnaker 已被调整为支持 Kubernetes 和 Docker，并且支持其原始的部署策略——使用镜像工厂和不可变服务器模式。你可以在官方 GitHub 仓库中找到该平台的源代码及其他项目：

[`github.com/spinnaker`](https://github.com/spinnaker)

在安装应用程序之前，我们应当熟悉一些该技术的核心概念。我们首先要了解的概念是应用程序管理。

### 应用程序管理

我们可以使用管理功能来管理和查看我们的云资源。通过 Spinnaker，我们围绕服务器组和集群等概念来建模我们的应用程序。有关这些概念的完整概述，请参阅 Spinnaker 文档：

[`spinnaker.io/concepts/`](https://spinnaker.io/concepts/)

应用程序是顶级容器，可以部署在 Spinnaker 维护的基础设施上，包括集群和服务器组。每个集群作为组织服务器组的机制。Spinnaker 将在 Kubernetes 中以 Pod 形式运行的 Docker 容器视为服务器组的成员。这些 Docker 镜像可能包含诸如 ShipIt Clicker 之类的服务以及任何相关工具，如在*第十五章*中介绍的 Datadog 监控代理，*扫描、监控和使用第三方工具*。

现在我们已经理解了容器化项目在 Spinnaker 中的表现形式，我们应考虑如何通过该框架将其部署到我们 AWS 中的 EKS 集群。

### 应用程序部署

应用程序部署这一环节在 Spinnaker 用户界面中通过管道图形化呈现。管道可以手动启动，也可以作为其他事件触发的一部分自动启动，比如源代码控制系统的推送。管道会告诉我们完成过程中需要执行的所有步骤（称为**阶段**）——例如，如何获取 Docker 容器，安装它，并在我们的云环境中对其进行后续更新。

以下截图展示了部署管道及其各个阶段的样子：

![](img/B11641_09_003.jpg)

图 9.3 – Spinnaker 管道

这个管道中的每个阶段可以视为一个独立的任务。每个任务按照顺序或并行执行，具体取决于管道是否分叉。正如我们接下来会看到的，Spinnaker 提供了多个预定义的阶段，我们可以将它们融入到自定义管道中。

将管道与构建服务器和源代码控制库绑定是有优势的，这样，当你将更改推送到应用程序及其 Helm Charts 时，Spinnaker 可以适当地打包、测试和部署它们。

现在，我们简要介绍了 Spinnaker 的两个主要概念，接下来让我们着手构建一些基础设施和管道，以便更好地理解各个阶段是如何工作的，以及可能的部署策略类型。

# 使用 Helm 在 AWS EKS 集群中设置 Spinnaker

设置一个生产级的 Spinnaker 集群需要一些精心的规划，但为了学习的目的，我们将使用其中一种简化的方法。完整的 Spinnaker 设置指南可以在 [`www.spinnaker.io/setup/`](https://www.spinnaker.io/setup/) 找到。

为了演示使用 Spinnaker 的概念验证，我们将使用以下链接中的 Helm Chart 来部署 Spinnaker：

[`github.com/helm/charts/tree/master/stable/spinnaker`](https://github.com/helm/charts/tree/master/stable/spinnaker)

Spinnaker Helm Chart 警告不适合用于生产环境

尽管这个 Helm Chart 声明它不适合用于生产环境，但我们可以使用它来演示构建、测试和部署应用程序的概念验证。Spinnaker 设置指南提供了设置生产级 Spinnaker 系统的指导。最重要的是，这包括将 Spinnaker 安装与托管最终用户消费应用程序的集群分开。为了节省时间和金钱，也为了便于演示，我们将在本章忽略这一建议。如果你打算大规模采用 Spinnaker，请务必认真采纳这一建议，并根据他们的最佳实践文档，在单独的集群中设置 Spinnaker。

确保你连接到正确的 Kubernetes 上下文，目标是你的 EKS 集群，并输入以下命令将 Spinnaker 部署到它自己的命名空间中：

```
$ kubectl create namespace spinnaker
$ helm install spinnaker stable/spinnaker --namespace spinnaker --version 1.23.3 --timeout 600s
```

Spinnaker 部署可能需要几分钟才能完成。当它完成时，你应该看到类似以下内容的输出：

![](img/B11641_09_004.jpg)

图 9.4 – Spinnaker Helm Chart 安装

接下来，我们将连接到刚刚安装的 Spinnaker 系统。

## 通过 kubectl proxy 连接到 Spinnaker

为了进行初步测试，请注意你从运行 `helm install` 命令（在上一节中创建端口转发隧道）中收到的输出中的建议。它应该类似于前一节中显示的输出。你应该在本地工作站上设置两个单独的控制台窗口或标签页，然后运行 `helm install spinnaker` 命令输出中 `NOTES` 部分列出的命令对，以设置端口转发隧道，每个控制台窗口或标签页执行一对命令。然后，你可以在浏览器中访问 [`127.0.0.1:9000`](http://127.0.0.1:9000) 来验证 Spinnaker 是否已启动并运行。

## 通过 ALB Ingress 控制器公开 Spinnaker

将 Spinnaker 与 EKS 集成的说明（[`www.spinnaker.io/setup/install/providers/kubernetes-v2/aws-eks/`](https://www.spinnaker.io/setup/install/providers/kubernetes-v2/aws-eks/)）描述了使用具有 LoadBalancer 注释的服务来公开服务的解决方案。然而，由于我们已经配置了 ALB Ingress 控制器、Route 53 和 ACM，因此最好通过 ALB Ingress 控制器来公开它们。编辑 `chapter9/spinnaker-alb-ingress.yaml` 文件，并对 `spin-deck` 和 `spin-gate` 的 ingress 配置做以下更改（文件中有两组配置）：

+   将 eks.example.com 替换为你已使用 ACM 通配符证书配置的域名。

+   将 `192.2.0.10/32` 替换为你的公共 IP 地址（CIDR 格式）（与你用于锁定 EKS API 的格式相同）。

+   将 `192.2.0.200/32` 替换为你的 Jenkins 服务器的公共 IP 地址。

    安全通知

    添加前述的 IP 地址限制非常重要，因为默认情况下，Spinnaker 的用户界面以集群管理员身份运行。如果你允许 `0.0.0.0/0`（整个互联网）访问，那么有人可能会以集群管理员身份运行进程，并修改或接管你的集群。如果你有动态 IP 地址，你可能需要多次更改，首先从 CloudFormation 模板开始。

然后，应用配置模板以创建 ALB Ingress 控制器：

```
kubectl apply -n spinnaker -f spinnaker-alb-ingress.yaml
```

几秒钟后，执行以下命令以验证是否成功（请查找你的域名，而不是 eks.example.com）：

```
$ kubectl get -n spinnaker ingress
NAME        HOSTS                                  ADDRESS                                                                  PORTS   AGE
spin-deck   spinnaker.eks.example.com              9bbd6f9c-spinnaker-spindec-5f03-917097792.us-east-2.elb.amazonaws.com    80      10m
spin-gate   spinnaker-gate.eks.example.com         9bbd6f9c-spinnaker-spingat-712f-2021704484.us-east-2.elb.amazonaws.com   80      10m
```

该列表中 `HOSTS` 列下列出的 DNS 名称是我们打算用来调用服务的名称。`ADDRESS` 列下的 DNS 地址是 ALB Ingress 控制器通过 AWS ALB 创建的实际 DNS 名称。为了将这两个名称连接起来，我们需要在我们的域中创建两个 DNS 记录，以便使用更友好的名称访问 Spinnaker 服务。请注意从该列表中 `ADDRESS` 列下的 Ingress 控制器的 DNS 名称。然后，进入 AWS Route 53 控制台，为你的域创建两个新的 A 类型 DNS 记录，并将它们设置为别名记录。

将第一个命名为 `spinnaker`，并将 `spin-deck` 条目中 `ADDRESS` 列显示的值作为其值。

将第二个条目的名称命名为 `spinnaker-gate`，并将 `spin-gate` 条目中 `ADDRESS` 列显示的值作为其值。

这样做的结果将是两个新的 DNS 记录，类似于以下内容（将你的域名替换为 example.com）：

+   spinnaker.eks.example.com

+   spinnaker-gate.eks.example.com

在等待大约 5 分钟，直到 DNS 记录可用并且 ALB 完全激活时，使用 Halyard 配置 Spinnaker，并使用这些 URL 的 HTTPS 版本。

## 使用 Halyard 配置 Spinnaker

现在我们已经为 Spinnaker 安装分配了友好的 DNS 名称，我们需要配置 Spinnaker 使其理解必须遵守这些名称。从你的本地工作站，连接到 Halyard 维护 pod：

```
kubectl exec --namespace spinnaker -it spinnaker-spinnaker-halyard-0 bash
```

一旦你连接到 pod，你将看到 `spinnaker@spinnaker-spinnaker-halyard-0:/workdir$` 提示符。然后，输入这些命令，将 `example.com` 替换为你的域名：

```
$ hal config security api edit --override-base-url https://spinnaker-gate.eks.example.com  --cors-access-pattern https://spinnaker.eks.example.com
$ hal config security ui edit --override-base-url https://spinnaker.eks.example.com
$ hal deploy apply
```

最后的 `hal` 命令将重新部署 Spinnaker 应用程序。

等待 5 分钟，以便 DNS 记录激活并且 ALB 完全创建。一旦完成，使用其完全合格的域名访问 Spinnaker 网站，将 example.com 替换为你的域名：

[`spinnaker.eks.example.com/`](http://spinnaker.eks.example.com/)

你应该被重定向到网站的 HTTPS 版本。

## 将 Spinnaker 连接到 Jenkins

为了让 Spinnaker 接收来自 Jenkins 的工件，我们必须通过 Jenkins 管理员 API 令牌将其连接。Spinnaker 提供了相关说明，可以在 [`www.spinnaker.io/setup/ci/jenkins/`](https://www.spinnaker.io/setup/ci/jenkins/) 上找到。

进入你在前一章节中使用的 Jenkins 服务器。登录后，转到类似于 [`jenkins.example.com/user/admin/configure`](https://jenkins.example.com/user/admin/configure) 的用户配置页面（将 jenkins.example.com 替换为你的 Jenkins URL）。然后，生成 Spinnaker 的 API 令牌：

![](img/B11641_09_005.jpg)

图 9.5 – Jenkins API 令牌生成

如 *使用 Halyard 配置 Spinnaker* 部分所示，从你的本地工作站连接到 `hal` 维护 pod：

```
kubectl exec --namespace spinnaker -it spinnaker-spinnaker-halyard-0 bash
```

然后，在该 pod 的终端中执行这些命令来配置 Jenkins，将 `BASEURL`、`APIKEY` 和 `USERNAME` 右侧的值替换为你安装的相应值：

```
$ hal config ci jenkins enable
$ BASEURL=https://jenkins.example.com
$ APIKEY=123456789012345678901234567890
$ USERNAME=admin
$ echo $APIKEY | hal config ci jenkins \
  master add my-jenkins-master  \  --address $BASEURL --username $USERNAME --password
$ hal deploy apply
```

现在 Spinnaker 已经配置为与 Jenkins 通信，我们将继续为 Jenkins 配置一组 Spinnaker 将使用的额外构建任务。

## 设置 Jenkins 与 Spinnaker 和 ECR 的集成

为了运行 Spinnaker 特定的任务并将 Jenkins 与 ECR 集成，我们需要为 Jenkins 配置额外的插件和凭据，以便它能够将容器推送到 AWS ECR，并且还需要设置一个新的多分支流水线项目，以便使用存储在 GitHub 仓库中的此章节的 `chapter9/Jenkinsfile`。

在接下来的章节中，我们将进行所有必要的更改，使 Jenkins 能够同时与 ECR 和 Spinnaker 配合使用。

### 安装 AWS ECR Jenkins 插件

以管理员身份登录到你的 Jenkins 服务器，然后在左侧菜单中进入 `ECR`，并在 **Filter** 框中查看类似这样的内容：

![](img/B11641_09_006.jpg)

图 9.6 – 通过 Jenkins 插件管理器安装 Amazon ECR 插件

点击 **安装** 复选框，选择 **Amazon ECR** 插件旁的 **立即下载并在重启后安装** 按钮。您将看到如下截图：

![](img/B11641_09_007.jpg)

图 9.7 – 正在安装 Amazon ECR Jenkins 插件

Jenkins 可能需要 5 到 15 分钟的时间才能重新启动并再次可用。一旦它可用，再次以 Jenkins 管理员用户身份登录。接下来，我们将创建一个具有有限权限的 AWS IAM 用户，并使用这些凭证配置 Jenkins。

### 为 Jenkins 创建一个受限的 AWS IAM 用户

在上一章中，我们使用 AWS 控制台为账户创建了一个管理员 IAM 用户。这一次，我们将使用 AWS CLI 来创建一个 Jenkins 用户，赋予其比管理员用户更有限的权限，以便它仅能管理 ECR 仓库并将 Docker 镜像推送到这些仓库中。这符合最小权限原则，系统应仅授予所需的*最小权限*。要创建该用户，请附加适当的策略，创建访问密钥，并执行以下列出的三个 `aws iam` 命令来设置 Jenkins 用户（您应预期看到的输出与这些命令一致）：

```
$ aws iam create-user --user-name Jenkins
{
    "User": {
        "Path": "/",
        "UserName": "Jenkins",
        "UserId": "AIDASDBKOBZBU6ZX6SQ7U",
        "Arn": "arn:aws:iam::143970405955:user/Jenkins",
        "CreateDate": "2020-05-03T02:45:34Z"
    }
}
$ aws iam attach-user-policy --user-name Jenkins --policy-arn  arn:aws:iam::aws:policy/AmazonEC2ContainerRegistryPowerUser
$ aws iam create-access-key --user-name Jenkins
{
    "AccessKey": {
        "UserName": "Jenkins",
        "AccessKeyId": "AKIASDBKOBZBYFDCBLMR",
        "Status": "Active",
        "SecretAccessKey": "q+1z7wt/FsbYOv5Yy7HRUSZI0OsLbANV7a8nIQDy",
        "CreateDate": "2020-05-03T02:46:00Z"
    }
}
```

请注意您命令输出中与 `AccessKeyId` 和 `SecretAccessKey` 相关的值。您需要这些值来在下一节配置 Jenkins 的 AWS 访问凭证。接下来，让我们使用 AWS 凭证来配置 Jenkins。

### 使用 AWS 和 ECR 凭证配置 Jenkins

我们需要告诉 Jenkins 我们的 AWS 凭证，以便它可以将构建的 Docker 容器推送到 ECR。此外，我们还需要配置 Jenkins 以知道使用哪个 ECR 注册表。在 *第六章*，*使用 Docker Compose 部署应用程序* 中，我们为 Jenkins 配置了 GitHub 和 Docker Hub 的凭证。现在，我们将为 AWS IAM 用户和 ECR 容器注册表配置额外的凭证。

在使用管理员用户登录 Jenkins 服务器时，进入首页，在左侧菜单中找到 `shipit.aws.key` ID，`ShipIt Clicker AWS API Keys` 描述，以及上一节中的访问密钥 ID 和密钥访问密钥。您应该会看到如下的凭证表单：

![](img/B11641_09_008.jpg)

图 9.8 – 配置 Jenkins 中的 AWS 凭证

完成后，在最后添加一个额外的凭证，引用 `dockerfordevelopers/shipitclicker:0.5.0`：

+   **作用域**：**全局**

+   `143970405955.dkr.ecr.us-east-2.amazonaws.com`

+   `shipit.ecr.container.id`

+   `ShipIt Clicker ECR 容器 ID`

按下 **确定** 按钮保存此凭证。

现在我们已经为 Jenkins 配置了连接 AWS 和 ECR 所需的凭证，接下来让我们为本章的代码配置一个新的多分支流水线。

### 配置 Jenkins 使用多分支流水线进行 Jenkinsfile 配置

接下来，我们将配置 Jenkins 使用一个额外的多分支管道项目，该项目从同一个 GitHub 仓库拉取代码，但配置为使用 `chapter9/Jenkinsfile`，而不是仓库根目录下的 Jenkinsfile。登录 Jenkins，并从主页导航到 `Spinnaker`，然后使用你的 GitHub 仓库凭证进行配置，类似于以下截图所示（将 `PacktPublishing/Docker-for-Developers` 替换为你在 *第七章* 中设置的 GitHub 组织和仓库名称，*使用 Jenkins 进行持续部署*）：

![](img/B11641_09_009.jpg)

图 9.9 – Jenkins 多分支管道设置

配置完成后，新的项目应能连接到 GitHub 仓库，并将容器构建并推送到 AWS ECR。检查该新项目中的主分支的控制台输出，确保构建成功，并且 Docker 镜像已推送到 AWS ECR 仓库。

现在你已经使用 ECR 插件配置了 Jenkins，创建了一个 Jenkins IAM 用户，配置了该用户的凭证，并且配置了新的凭证以反映 AWS 集成，同时还添加了新的 Jenkins 多分支设置，那么你可以继续将其他服务连接到 Spinnaker。接下来，我们将连接 GitHub。

## 将 Spinnaker 连接到 GitHub

我们将按照 [`www.spinnaker.io/setup/artifacts/github/`](https://www.spinnaker.io/setup/artifacts/github/) 上的指导，将 Spinnaker 连接到 Jenkins，以便它能够从 GitHub 读取工件。前往你的 GitHub 用户账户，在 **Developer Settings** 中生成一个带有 repo 范围的访问 token，供 Spinnaker 使用。

从你的本地工作站，连接到 Halyard 维护 pod，如 *使用 Halyard 配置 Spinnaker* 部分所示，将 GitHub token 放入主目录下的一个文件中，然后执行以下命令（将 `xxxx` 替换为你的 GitHub token，将 `my-github-user` 替换为你的 GitHub 用户名）：

```
TOKEN=xxxx
GH_ACCOUNT=my-github-user
TOKEN_FILE=~/.github-token.txt
echo "$TOKEN" > $TOKEN_FILE
hal config artifact github enable
hal config artifact github account add $GH_ACCOUNT --token-file $TOKEN_FILE
hal deploy apply
```

完成这些操作后，Spinnaker 应该能够与 GitHub 通信。接下来，我们将把 Spinnaker 连接到 Docker Hub。

## 将 Spinnaker 连接到 Docker Hub

你还需要将 Spinnaker 连接到 Docker Hub，以便它可以读取你的仓库和 `library/redis` 仓库。将 Spinnaker 集成到 Docker Hub 需要你将所有模板使用的仓库列入白名单。默认的 Docker Hub 集成有一个短小的白名单，包含最常用的库。

我们将按照 [`www.spinnaker.io/setup/install/providers/docker-registry/`](https://www.spinnaker.io/setup/install/providers/docker-registry/) 上的指导，将 Docker Hub 添加到 Spinnaker。

登录到你的 Docker Hub 账户，并从 [`hub.docker.com/settings/security`](https://hub.docker.com/settings/security) 为 Spinnaker 安装生成一个新的 API token。

从你的本地工作站，连接到 Halyard 维护 pod：

```
kubectl exec --namespace spinnaker -it spinnaker-spinnaker-halyard-0 bash
```

然后，执行以下命令（将 `xxxx` 替换为您的 Docker Hub 令牌，将 `my-dockerhub-user` 替换为您的 Docker Hub 用户名）：

```
$ ADDRESS=index.docker.io
$ REPOSITORIES="library/redis dockerhub-user/shipitclicker"
$ USERNAME=dockerhub-user
$ PASSWORD=xxxx
$ REPOSITORIES="library/redis dockerhub-user/shipitclicker"
$ echo $PASSWORD | hal config provider docker-registry \
    account add my-docker-registry \
    --address $ADDRESS \
    --repositories $REPOSITORIES \
    --username $USERNAME \
    --password
$ hal deploy apply
```

一旦连接了 Docker Hub，您就可以开始在 Spinnaker 中设置应用程序和流水线。但在此之前，让我们谈谈如何排除 Spinnaker 问题。

## 排除 Spinnaker 问题

如果您在使 Spinnaker 流水线执行工作时遇到任何困难，或者在设置和配置 Spinnaker 方面有其他问题，用户界面具有最少的错误报告能力。它可能看起来不透明且令人生畏。

例如，假设您在其中一个 artifact 定义中有拼写错误 – 例如，[gitgub.com](http://gitgub.com) 而不是 [github.com](http://github.com)。由于主机名查找失败，当流水线尝试检索该 artifact 时可能会导致流水线失败。

而不是尝试找出 Spinnaker pods 中可能记录了错误的哪一个，您可以同时查看所有 Spinnaker pods 的所有日志：

```
kubectl logs -n spinnaker -f -l app=spin --all-containers --max-log-requests 10
```

如果您在控制台输出中搜索 `exception` 这个词，您可能会找到一些线索，比如在排除 Spinnaker 问题时找到的这个：

```
com.netflix.spinnaker.clouddriver.artifacts.exceptions.FailedDownloadException: Unable to determine the download URL of artifact Artifact(type=github/file, customKind=false, name=chapter9/helm.tar.gz, version=staging, location=null, reference=https://api.gitgub.com/repos/PacktPublishing/Docker-for-Developers/contents/chapter9/helm.tar.gz, metadata={id=8ebb0ad7-2d14-4882-9b77-fde3a03e3c45}, artifactAccount=obscurerichard, provenance=null, uuid=null): api.gitgub.com: Try again
```

分析类似这样的日志文件确实可以帮助您摆脱困境。接下来，我们将使用 Spinnaker 部署 ShipIt Clicker。

# 使用简单的部署策略在 Spinnaker 中部署 ShipIt Clicker

让我们通过部署我们的 ShipIt Clicker 应用程序来深入了解 Spinnaker。为此，我们将使用 Helm Charts，并使用 `chapter9` 目录中的应用程序版本。

Spinnaker 需要 Helm 存档文件来运行

为了简化 Helm Charts 的部署，我们已经在 `chapter9/helm.tar.gz` 中创建了 `chapter9/shipitclicker` Helm Chart 目录的归档文件，因为 Spinnaker 期望将此格式的归档文件作为其输入之一。我们也可以将此归档文件输出到 AWS S3 对象，甚至作为 GitHub 发布 artifact，但这超出了本章的范围。如果您更改 `chapter9/shipitclicker` 目录中的 Helm Charts，请务必更新 `helm.tar.gz` 归档文件并在使用 Spinnaker 构建之前进行提交和推送。

## 添加一个 Spinnaker 应用程序

在网页浏览器中转到您的 Spinnaker 安装地址 [`spinnaker.eks.example.com`](https://spinnaker.eks.example.com)（将 example.com 替换为您的域名）。添加一个名为 `shipandspin` 的应用程序，然后，在 `Docker-for-Developers` 代码中：

![](img/B11641_09_010.jpg)

图 9.10 – Spinnaker 中的新应用对话框

当您提交此表单时，它将带您进入一个基础设施定义表单。在此停止，并不要填写或提交基础设施定义表单。此表单适用于其他类型的 Spinnaker 部署，而不是专注于 Kubernetes 的部署。当您部署应用程序时，它将定义 Spinnaker 在 Kubernetes 中理解的基础设施。

## 添加一个 Spinnaker 流水线

转到 **流水线** 屏幕：

![](img/B11641_09_011.jpg)

图 9.11 – Spinnaker 中的 PIPELINES 屏幕示例

创建一个名为`shipit-eks-staging`的管道，然后添加两个工件——一个是 Helm Chart，另一个是`values-spin.yaml`覆盖。

对于第一个，选择 GitHub 帐户，给它`chapter9/helm.tar.gz` Helm 工件，并点击**使用默认工件**。然后，提供该工件的完整 API URL，并将其更改为匹配你的帐户和仓库名称（提交前请仔细检查是否正确）：

[`api.github.com/repos/PacktPublishing/Docker-for-Developers/contents/chapter9/helm.tar.gz`](https://api.github.com/repos/PacktPublishing/Docker-for-Developers/contents/chapter9/helm.tar.gz)

告诉它使用`staging`分支。定义后，它应该看起来像这样：

![](img/B11641_09_012.jpg)

图 9.12 – Spinnaker 中覆盖工件：Helm Chart 归档

为`chapter9/values-spin.yaml`覆盖文件添加另一个工件。设置`chapter9/values-spin.yaml`文件路径和`values-spin.yaml`显示名称，选择`staging`分支（将`PacktPublishing/Docker-for-Developers`替换为你在*第七章*中设置的 GitHub 组织和分叉仓库的名称，*使用 Jenkins 进行持续部署*）：

![](img/B11641_09_013.jpg)

图 9.13 – Spinnaker 中覆盖工件：Helm Chart 归档

然后，配置`build.properties`文件用于**属性文件**，这是一个 Jenkins 归档文件，Spinnaker 将使用它来获取 Jenkins 构建的容器版本：

![](img/B11641_09_014_New.jpg)

图 9.14 – Spinnaker 中 Jenkins 自动触发器屏幕

前往表单底部并保存**配置**阶段。

现在，让我们添加下一个阶段，创建来自 Helm Charts 的 Kubernetes 清单。

### 添加 Bake（Manifest）阶段

保存配置阶段后，你仍然会停留在非常长的阶段定义网页表单底部。返回表单顶部，添加一个名为`shipit-staging`的额外阶段，并告诉它部署到默认命名空间。为它设置一个**模板工件**，值为**helm.tar.gz**。

对于`image.repository`名称和`${trigger["properties"]["imageName"]}`值，添加一个覆盖键值对，键为`ingress.hosts[0].host`，值为 shipit-stage.eks.example.com，将 example.com 替换为你的域名。

我们将在此部署的 Ingress Controller 上设置一个 Route 53 DNS 条目。表单应该类似于以下内容：

![](img/B11641_09_015.jpg)

图 9.15 – Spinnaker 中 Bake（Manifest）模板渲染器配置屏幕

然后，在表单底部，编辑`kube-templates.yaml`并保存表单。它应该看起来像这样：

![](img/B11641_09_016.jpg)

图 9.16 – Spinnaker 中的 Bake（Manifest）生成工件部分

配置此阶段将设置 Helm 模板渲染过程。然后保存表单。接下来，我们将设置 **Deploy (Manifest)** 阶段。

### 添加部署（Manifest）阶段

在保存了上述配置更改之后，再次返回配置表单顶部，添加另一个阶段，`kube-templates.yaml` 用于 **Manifest Artifact** 进行部署。不要选择 **Rollout Strategy Options** 设置，因为这仅适用于只有一个 ReplicaSet 并放弃使用 **Deployments** 作为 Kubernetes 控制器的情况。它会类似于这样：

![](img/B11641_09_017.jpg)

图 9.17 – Spinnaker 中部署（Manifest）配置

现在，我们准备触发部署。在屏幕顶部单击 **PIPELINES**，然后单击 **Start Manual Execution** 链接。它应该会从 GitHub 获取最新构建，然后使用 Helm Charts 烘焙清单并部署。

因为我们使用 Jenkins 发布了一个 `build.properties` 文件，并在模板中使用了 `image.repository` 字段，所以我们将使用 Jenkins 作业连接的触发器构建的特定容器。有关 SPEL 表达式和 Spinnaker 流水线的更多信息，请参考以下链接：

[`www.spinnaker.io/guides/user/pipeline/expressions/`](https://www.spinnaker.io/guides/user/pipeline/expressions/)

如果您在某些必需配置中有拼写错误，可能需要解决一些问题。如果一切顺利，它应该看起来类似于这样：

![](img/B11641_09_018.jpg)

图 9.18 – Spinnaker 中显示已完成作业的流水线

您可以查看 **Execution Details** 和 **INFRASTRUCTURE** 面板，因为 Spinnaker 将显示有关正在运行的应用程序的一些信息。它甚至可以显示您运行的 Pod 的日志。

## 为 Ingress 控制器设置 DNS 记录条目

要从外部查看正在运行的应用程序，您需要设置 DNS 记录。发出 `kubectl get ingress` 命令以确定 `shipit-eks-staging` 的 Ingress 控制器的 DNS 别名，然后在 Route 53 中设置 DNS 别名，以匹配您为 shipit-stage.eks.example.com 设置的覆盖（将 example.com 替换为您的域名）。

完成后，您应该能够访问 [`shipit-stage.eks.example.com/`](https://shipit-staging.eks.example.com/)（将 example.com 替换为您的域名），并查看运行中的 ShipIt Clicker 游戏。

接下来，我们将了解 Spinnaker 对不同类型部署的支持及其在 Kubernetes 部署中的应用（或不适用）。

# 浏览 Spinnaker 的部署和测试功能

在本章早些时候关于 Spinnaker 的介绍中，我们指出您将有机会了解可用的各种部署方法。现在让我们深入探讨这些概念，包括金丝雀和红/黑部署，并描述它们在使用 Spinnaker 管理 Kubernetes 部署时的相关性。

## 金丝雀部署

金丝雀部署是一种将应用程序暴露给用户的方法，你可以在新版本的部署中运行一部分流量，同时大部分流量仍然流向当前部署的版本。这可以帮助你测试新版本是否适合生产环境，而无需立即将所有流量都导向新版本。

Kubernetes v2 Spinnaker 提供程序不支持金丝雀部署。

尽管这是 Spinnaker 最受欢迎的特性之一，但 Kubernetes v2 云提供程序不支持金丝雀部署，因此我们不会在 ShipIt Clicker 中使用它。如果我们使用的是非 Kubernetes 云提供程序，例如 AWS、Google Compute Engine 或 Azure 提供程序，那么这将是一个更自然的模式。有关 Spinnaker 云提供程序的完整列表，请参阅 [`spinnaker.io/setup/install/providers/`](https://spinnaker.io/setup/install/providers/)。

## 红/黑部署

现在我们来看看红/黑部署方法是如何工作的。这是另一种为人熟知的蓝/绿部署策略的名称。在红/黑策略中，部署期间会保持两组服务器或容器可用，流量每次只会流向其中一个。假设在部署开始时，红组接收流量。在部署过程中，你将部署到黑组。一旦健康检查通过，你就会将流量切换到黑组，但仍保留红组，以便如果出现问题，你可以将流量切换回红组，而无需重新部署。

Spinnaker 于 2019 年宣布通过 Kubernetes v2 提供程序支持红/黑部署：

[`blog.spinnaker.io/introducing-rollout-strategies-in-the-kubernetes-v2-provider-8bbffea109a`](https://blog.spinnaker.io/introducing-rollout-strategies-in-the-kubernetes-v2-provider-8bbffea109a)

然而，这有一些显著的限制。它意味着你不能使用 Kubernetes 部署对象，而必须改用低级别的 ReplicaSet 注释。Helm Chart 生成器会生成一个带有部署的框架，它是建立在 ReplicaSets 之上的，因此如果你希望使用 Spinnaker 的红/黑支持与 Kubernetes 配合使用，你将需要对 Helm Charts 进行大幅度的修改。请参考关于 Kubernetes v2 提供程序的建议：

https://www.spinnaker.io/guides/user/kubernetes-v2/traffic-management/#you-must-use-replica-sets

Spinnaker *确实*支持的 Kubernetes 部署（仅使用 ReplicaSets）部署策略如下：

+   **暗部署**：部署到一个新的 ReplicaSet，该 ReplicaSet 不连接到实时负载均衡器。

+   **红/黑**：部署一个新的 ReplicaSet，并使用 Spinnaker 在新旧两组之间来回切换。

+   **Highlander**：部署一个新的 ReplicaSet，并在新的 ReplicaSet 开始接收流量时销毁旧的 ReplicaSet（只能有一个 ReplicaSet）。

如果你使用的是 Kubernetes 部署控制器，那么你将获得类似于 Spinnaker Highlander 策略的行为。因此，如果你使用 Kubernetes，可能不需要使用 Spinnaker 支持的高级部署策略。

## 回滚

那么，如果部署失败会发生什么呢？我们将需要以安全的方式回滚到之前的版本。对于 Spinnaker 管理机器镜像部署的方式，它会协调此回滚。然而，对于 Kubernetes 操作员，它依赖于 Kubernetes 部署机制，使用存活和就绪探针来检查部署是否有效。

Spinnaker 确实支持通过其界面直接撤销一组模板的发布。然而，如果模板中的所有资源没有独立的版本修订（例如，单独版本和标记的 Docker 容器），这可能不起作用。有关 Spinnaker 和 Kubernetes 回滚的更多信息，请参见此处：

[`www.spinnaker.io/guides/user/kubernetes-v2/automated-rollbacks/`](https://www.spinnaker.io/guides/user/kubernetes-v2/automated-rollbacks/)

## 使用 Spinnaker 进行测试

使用 Spinnaker 时，你可以使用手动判断阶段为人员提供时间对应用程序进行手动测试，或者使用脚本化流水线阶段在 Jenkins 中运行自动化测试套件以测试你的应用程序。如果你在多个环境中部署或使用红/黑策略，这能给你更好的机会在将应用程序部署到生产环境或公开给世界之前执行测试。

你可以在各自的 Spinnaker 文档中找到更多关于使用这两种策略进行测试的信息，文档地址为 [`www.spinnaker.io/guides/tutorials/codelabs/safe-deployments/`](https://www.spinnaker.io/guides/tutorials/codelabs/safe-deployments/) 和 [`www.spinnaker.io/setup/features/script-stage/`](https://www.spinnaker.io/setup/features/script-stage/)。

# 概述

在本章中，我们探讨了如何使用 Spinnaker 框架在 AWS 中进行持续部署的主题。我们首先配置了 Spinnaker，使其与 Jenkins、GitHub、AWS ECR 和 Docker Hub 配合使用。然后，我们使用它将 ShipIt Clicker 应用程序部署到 EKS 上的 Kubernetes，同时为 Spinnaker 和 ShipIt Clicker 应用程序配置了 SSL 安全。

在此之后，我们了解了 Spinnaker 提供的一些高级部署策略，以及在配置基于 Kubernetes 的 Docker 应用程序以利用这些策略时可能需要做出的一些权衡。我们还了解了如何通过 Spinnaker 触发测试的执行（手动或自动）。通过实践中学到的这些章节中的经验，你可以构建使用简单 Jenkins 构建任务和 Spinnaker 流水线将 Docker 应用程序部署到 Kubernetes 的持续部署系统。你所掌握的关于将 Spinnaker 与 Kubernetes 集成的技能也适用于将其他软件包与 Kubernetes 集成。

在下一章中，我们将探讨如何使用 Prometheus、Grafana 和 Jaeger 监控我们的 Docker 容器。

# 进一步阅读

使用以下资源扩展你对 Spinnaker 和 EKS 的知识：

+   Spinnaker 不是一个构建服务器，以及其他误解：[`www.armory.io/blog/spinnaker-is-not-a-build-server-and-other-misconceptions/`](https://www.armory.io/blog/spinnaker-is-not-a-build-server-and-other-misconceptions/)

+   AWS 的一篇博客文章，描述了如何使用 Jenkins 和 ECR 完全安装 Kubernetes 和 Spinnaker 的部署流水线：[`aws.amazon.com/blogs/opensource/deployment-pipeline-spinnaker-kubernetes/`](https://aws.amazon.com/blogs/opensource/deployment-pipeline-spinnaker-kubernetes/)

+   一篇关于如何将 Kubernetes 服务暴露给世界的好文章：[`medium.com/google-cloud/kubernetes-nodeport-vs-loadbalancer-vs-ingress-when-should-i-use-what-922f010849e0`](https://medium.com/google-cloud/kubernetes-nodeport-vs-loadbalancer-vs-ingress-when-should-i-use-what-922f010849e0)

+   AWS 官方文档关于 ALB Ingress 控制器：[`docs.aws.amazon.com/eks/latest/userguide/alb-ingress.html`](https://docs.aws.amazon.com/eks/latest/userguide/alb-ingress.html)

+   Spinnaker CLI：[`www.spinnaker.io/guides/spin/`](https://www.spinnaker.io/guides/spin/)

+   一个 Kubernetes 外部 DNS 提供者，你可以使用它来注释你的模板，以避免手动设置 DNS 别名：[`github.com/kubernetes-sigs/external-dns`](https://github.com/kubernetes-sigs/external-dns)

Spinnaker 不是你应该了解的唯一先进的 Kubernetes 感知 CD 系统；你还应考虑其他替代方案，并在这个领域进行新的研究，因为这个领域变化迅速：

+   Jenkins-X – 一个以 Kubernetes 为中心的 CI/CD 系统的权威性意见：[`jenkins-x.io/`](https://jenkins-x.io/)

+   Argo Project – 工作流、CD 等。作为 2020 年 7 月孵化阶段的 CNCF 项目：[`argoproj.github.io/`](https://argoproj.github.io/)

+   WeaveWorks – 一个使用 Kubernetes 进行 CD 的 GitOps 系统：[`www.weave.works/technologies/ci-cd-for-kubernetes/`](https://www.weave.works/technologies/ci-cd-for-kubernetes/)
