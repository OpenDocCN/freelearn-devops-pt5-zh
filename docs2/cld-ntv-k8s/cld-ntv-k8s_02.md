# 第一章：*第一章*：与 Kubernetes 通信

本章包含容器编排的解释，包括其优点、用例和流行的实现方式。我们还将简要回顾 Kubernetes，包括架构组件的布局，以及有关授权、身份验证和与 Kubernetes 一般通信的入门知识。到本章结束时，您将了解如何对 Kubernetes API 进行身份验证和通信。

本章将涵盖以下主题：

+   容器编排入门

+   Kubernetes 的架构

+   Kubernetes 上的身份验证和授权

+   使用 kubectl 和 YAML 文件

# 技术要求

为了运行本章详细介绍的命令，您需要一台运行 Linux、macOS 或 Windows 的计算机。本章将教您如何安装 `kubectl` 命令行工具，您将在后续章节中使用该工具。

本章中使用的代码可以在本书的 GitHub 仓库中找到，链接如下：

[`github.com/PacktPublishing/Cloud-Native-with-Kubernetes/tree/master/Chapter1`](https://github.com/PacktPublishing/Cloud-Native-with-Kubernetes/tree/master/Chapter1)

# 介绍容器编排

我们无法在不介绍 Kubernetes 目的的情况下谈论它。Kubernetes 是一个容器编排框架，因此让我们回顾一下在本书中这个概念的含义。

## 什么是容器编排？

容器编排是运行现代应用程序的一种流行模式，适用于云端和数据中心。通过使用容器——预配置的应用单元，包含所有依赖项——作为基础，开发人员可以并行运行多个应用实例。

## 容器编排的好处

容器编排提供了许多好处，但我们将突出主要的几个。首先，它允许开发人员轻松构建**高可用性**的应用程序。通过运行多个应用实例，容器编排系统可以以一种配置方式工作，确保它能自动用新的实例替换任何故障的应用实例。

这可以通过将应用程序的多个实例分布在物理数据中心中扩展到云端，因此如果一个数据中心出现故障，其他实例的应用程序将继续运行，避免停机。

其次，容器编排可以支持高度**可扩展**的应用程序。由于可以轻松创建和销毁应用程序的新实例，编排工具可以根据需求自动进行上下扩展。无论是在云环境还是数据中心环境中，都可以向编排工具添加新的**虚拟机**（**VMs**）或物理机器，以提供更大的计算池供其管理。在云环境中，这一过程可以完全自动化，允许实现完全免人工干预的扩展，既可以在微观层面，也可以在宏观层面进行。

## 流行的编排工具

在生态系统中，有几种非常流行的容器编排工具：

+   **Docker Swarm**：Docker Swarm 是由 Docker 容器引擎背后的团队创建的。与 Kubernetes 相比，它的设置和运行更加简单，但灵活性稍差。

+   **Apache Mesos**：Apache Mesos 是一个低层次的编排工具，管理计算、内存和存储，在数据中心和云环境中均可使用。默认情况下，Mesos 不管理容器，但 Marathon —— 一个运行在 Mesos 上的框架 —— 是一个完整的容器编排工具。甚至可以在 Mesos 上运行 Kubernetes。

+   **Kubernetes**：截至 2020 年，容器编排的许多工作已经集中在 Kubernetes（koo-bur-net-ees）上，通常缩写为 k8s。Kubernetes 是一个开源容器编排工具，最初由 Google 创建，借鉴了其内部的编排工具 Borg 和 Omega，这些工具在 Google 内部已使用多年。自从 Kubernetes 开源以来，它的受欢迎程度迅速上升，成为企业环境中运行和编排容器的事实标准。原因有几个，其中之一是 Kubernetes 是一个成熟的产品，拥有极其庞大的开源社区。它的操作也比 Mesos 更简单，比 Docker Swarm 更加灵活。

从这个比较中最重要的结论是，尽管有多个相关的容器编排选项，并且其中一些在某些方面确实更好，但 Kubernetes 已成为事实上的标准。考虑到这一点，让我们来看看 Kubernetes 是如何工作的。

# Kubernetes 架构

Kubernetes 是一个可以运行在云虚拟机、数据中心的虚拟机或裸金属服务器上的编排工具。一般来说，Kubernetes 运行在一组节点上，每个节点可以是虚拟机或物理机器。

## Kubernetes 节点类型

Kubernetes 节点可以是许多不同的东西 —— 从虚拟机到裸金属主机，再到树莓派。Kubernetes 节点分为两类：首先是主节点，运行 Kubernetes 控制平面应用程序；其次是工作节点，运行你部署到 Kubernetes 上的应用程序。

一般来说，为了保证高可用性，Kubernetes 的生产环境部署应至少有三个主节点和三个工作节点，尽管大多数大型部署中，工作节点的数量远多于主节点。

## Kubernetes 控制平面

Kubernetes 控制平面是一组运行在主节点上的应用程序和服务。这里有多个高度专业化的服务，它们构成了 Kubernetes 功能的核心。它们如下所示：

+   **kube-apiserver**：这是 Kubernetes API 服务器。这个应用程序处理发送到 Kubernetes 的指令。

+   **kube-scheduler**：这是 Kubernetes 调度器。这个组件负责决定将工作负载放置在哪些节点上，这个过程可能变得非常复杂。

+   **kube-controller-manager**：这是 Kubernetes 控制器管理器。这个组件提供了一个高级控制循环，确保集群及其上运行的应用程序的期望配置得以实现。

+   **etcd**：这是一个分布式键值存储，包含集群配置。

通常，这些组件都以系统服务的形式运行在每个主节点上。如果你想手动启动集群，这些服务可以手动启动，但通过使用集群创建库或云提供商托管的服务，如 **弹性 Kubernetes 服务 (EKS)**，通常在生产环境中会自动完成。

## Kubernetes API 服务器

Kubernetes API 服务器是一个组件，接受 HTTPS 请求，通常使用端口 `443`。它会呈现一个证书，可以是自签名的，并且提供认证和授权机制，这些内容我们将在本章后面介绍。

当向 Kubernetes API 服务器发送配置请求时，它会检查当前集群配置中的 `etcd` 并在必要时进行更改。

Kubernetes API 通常是一个 RESTful API，针对每种 Kubernetes 资源类型提供端点，并带有一个 API 版本，在查询路径中传递；例如，`/api/v1`。

为了扩展 Kubernetes（参见 *第十三章*，*通过 CRD 扩展 Kubernetes*），API 还具有一组基于 API 组的动态端点，这些端点可以将相同的 RESTful API 功能暴露给自定义资源。

## Kubernetes 调度器

Kubernetes 调度器决定工作负载的实例应该运行在哪里。默认情况下，这个决定受到工作负载资源需求和节点状态的影响。你还可以通过在 Kubernetes 中可配置的放置控制来影响调度器（参见 *第八章*，*Pod 放置控制*）。这些控制可以作用于节点标签、节点上已经运行的其他 Pod，以及许多其他可能性。

## Kubernetes 控制器管理器

Kubernetes 控制器管理器是一个运行多个控制器的组件。控制器运行控制循环，确保集群的实际状态与存储在配置中的状态相匹配。默认情况下，这些包括以下内容：

+   节点控制器，确保节点正常运行

+   副本控制器，确保每个工作负载得到适当的扩展

+   端点控制器，负责处理每个工作负载的通信和路由配置（参见 *第五章**，服务与入口 – 与外界通信*）

+   服务账户和令牌控制器，负责创建 API 访问令牌和默认账户

## etcd

etcd 是一个分布式键值存储，用于以高度可用的方式存储集群的配置。每个主节点上运行一个 `etcd` 副本，并使用 Raft 一致性算法，确保在允许对键值进行任何更改之前，必须保持法定人数。

## Kubernetes 工作节点

每个 Kubernetes 工作节点都包含一些组件，使其能够与控制平面通信并处理网络。

首先是 **kubelet**，它确保集群配置要求的容器在节点上运行。其次，**kube-proxy** 提供了一个网络代理层，支持每个节点上运行的工作负载。最后，**容器运行时** 用于在每个节点上运行工作负载。

## kubelet

kubelet 是一个运行在每个节点上的代理（包括主节点，尽管在此上下文中具有不同的配置）。它的主要功能是接收一份 PodSpecs 列表（稍后将详细介绍），并确保这些 PodSpecs 所指定的容器在节点上运行。kubelet 可以通过几种不同的机制来获取这些 PodSpecs，但主要方式是通过查询 Kubernetes API 服务器。或者，kubelet 也可以通过文件路径启动，它将监视该文件路径中的 PodSpecs 列表、监视 HTTP 端点，或使用它自己的 HTTP 端点接收请求。

## kube-proxy

kube-proxy 是一个运行在每个节点上的网络代理。它的主要目的是将 TCP、UDP 和 SCTP 转发（通过流或轮询方式）到其节点上运行的工作负载。kube-proxy 支持 Kubernetes 的 `Service` 构造，我们将在*第五章*中讨论，*服务与入口——与外部世界通信*。

## 容器运行时

容器运行时在每个节点上运行，并且是实际运行你工作负载的组件。Kubernetes 支持 CRI-O、Docker、containerd、rktlet 和任何有效的 **容器运行时接口**（**CRI**）运行时。从 Kubernetes v1.14 版本开始，RuntimeClass 功能已从 alpha 版本移至 beta 版本，并允许针对特定工作负载选择运行时。

## 插件

除了核心集群组件外，典型的 Kubernetes 安装还包括插件，它们是提供集群功能的附加组件。

例如，`Calico`、`Flannel` 或 `Weave` 提供符合 Kubernetes 网络要求的覆盖网络功能。

CoreDNS 是一个流行的集群内 DNS 和服务发现插件。还有像 Kubernetes Dashboard 这样的工具，它提供了一个 GUI 用于查看和交互操作你的集群。

此时，你应该对 Kubernetes 的主要组件有一个高层次的了解。接下来，我们将回顾用户如何与 Kubernetes 交互以控制这些组件。

# Kubernetes 中的身份验证和授权

命名空间是 Kubernetes 中一个极其重要的概念，由于它们可能会影响 API 访问和授权，因此我们现在来讨论它们。

## 命名空间

Kubernetes 中的命名空间是一种允许你将集群中的 Kubernetes 资源进行分组的结构。它们是一种分隔方法，具有许多可能的用途。例如，你可以为每个环境（如开发、测试和生产）在集群中创建一个命名空间。

默认情况下，Kubernetes 会创建默认命名空间、`kube-system` 命名空间和 `kube-public` 命名空间。没有指定命名空间的资源将被创建在默认命名空间中。`kube-system` 包含集群服务，如 `etcd`、调度器和 Kubernetes 自身创建的任何资源，而不是用户创建的。`kube-public` 默认对所有用户可读，可以用于公共资源。

## 用户

Kubernetes 中有两种用户类型——普通用户和服务账户。

普通用户通常由集群外的服务管理，无论是私钥、用户名和密码，还是某种形式的用户存储。而服务账户则由 Kubernetes 管理，并且被限制在特定的命名空间内。要创建一个服务账户，Kubernetes API 可能会自动创建，或者可以通过调用 Kubernetes API 手动创建。

有三种可能的请求类型：与普通用户相关的请求、与服务账户相关的请求以及匿名请求。

## 认证方法

为了认证请求，Kubernetes 提供了几种不同的选项：HTTP 基本认证、客户端证书、承载令牌和基于代理的认证。

要使用 HTTP 认证，请求者发送包含 `Authorization` 头的请求，该头部的值为 bearer `"token value"`。

为了指定哪些令牌有效，启动 API 服务器应用程序时可以提供一个 CSV 文件，使用 `--token-auth-file=filename` 参数。一个新的 beta 特性（截至本书撰写时）叫做 *引导令牌*，允许在 API 服务器运行时动态交换和更改令牌，而无需重启。

也可以通过 `Authorization` 令牌进行基本的用户名/密码认证，使用请求头值 `Basic base64encoded(username:password)`。

## Kubernetes 的 TLS 和安全证书基础设施

为了使用客户端证书（X.509 证书），必须使用 `--client-ca-file=filename` 参数启动 API 服务器。此文件需要包含一个或多个 **证书颁发机构**（**CAs**），它们将在验证 API 请求中传递的证书时使用。

除了 `groups` 可以包含的内容，我们将在 *Authorization* 选项部分讨论。

例如，你可以使用以下内容：

```
openssl req -new -key myuser.pem -out myusercsr.pem -subj "/CN=myuser/0=dev/0=staging"
```

这将为用户 `myuser` 创建一个 CSR，该用户属于名为 `dev` 和 `staging` 的组。

一旦 CA 和 CSR 被创建，就可以使用 `openssl`、`easyrsa`、`cfssl` 或任何证书生成工具创建实际的客户端和服务器证书。此时还可以为 Kubernetes API 创建 TLS 证书。

因为我们的目标是尽快让你开始在 Kubernetes 上运行工作负载，所以我们将本书中省略所有可能的证书配置——但 Kubernetes 文档和文章 *Kubernetes The Hard Way* 提供了一些关于从零开始设置集群的很好的教程。在大多数生产环境中，你不会手动执行这些步骤。

## 授权选项

Kubernetes 提供了几种授权方法：节点、webhook、RBAC 和 ABAC。在本书中，我们将重点讨论 RBAC 和 ABAC，因为它们是最常用于用户授权的方式。如果你通过其他服务和/或自定义功能扩展集群，其他授权模式可能会变得更加重要。

## RBAC

`Role`、`ClusterRole`、`RoleBinding` 和 `ClusterRoleBinding`。要启用 RBAC 模式，API 服务器可以通过 `--authorization-mode=RBAC` 参数启动。

`Role` 和 `ClusterRole` 资源指定了一组权限，但并没有将这些权限分配给任何特定的用户。权限是通过 `resources` 和 `verbs` 来指定的。以下是一个指定 `Role` 的示例 YAML 文件。不要过于担心 YAML 文件的前几行——我们稍后会解释。重点关注 `resources` 和 `verbs` 行，看看如何将操作应用于资源：

Read-only-role.yaml

```
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: read-only-role
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list"]
```

`Role` 和 `ClusterRole` 之间唯一的区别在于，`Role` 限制在特定的命名空间中（在这种情况下是默认命名空间），而 `ClusterRole` 可以影响集群中该类型的所有资源访问，以及集群范围的资源，如节点。

`RoleBinding` 和 `ClusterRoleBinding` 是将 `Role` 或 `ClusterRole` 与用户或用户列表关联的资源。以下文件表示一个将 `read-only-role` 与用户 `readonlyuser` 连接的 `RoleBinding` 资源：

Read-only-rb.yaml

```
apiVersion: rbac.authorization.k8s.io/v1namespace.
kind: RoleBinding
metadata:
  name: read-only
  namespace: default
subjects:
- kind: User
  name: readonlyuser
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: read-only-role
  apiGroup: rbac.authorization.k8s.io
```

`subjects` 键包含所有要关联角色的实体列表；在此例中是用户 `alex`。`roleRef` 包含要关联的角色的名称，以及类型（`Role` 或 `ClusterRole`）。

## ABAC

`--authorization-mode=ABAC` 和 `--authorization-policy-file=filename` 参数。

在策略文件中，每个策略对象包含有关单个策略的信息：首先，它对应的主体是哪个，可以是用户或组；其次，哪些资源可以通过该策略进行访问。此外，可以包含一个布尔值 `readonly` 来限制策略仅对 `list`、`get` 和 `watch` 操作有效。

一种次要类型的策略与资源无关，而是与非资源请求类型相关，例如对 `/version` 端点的调用。

当在 ABAC 模式下向 API 发出请求时，API 服务器会检查用户及其所属的任何组是否在策略文件中的列表中，并检查是否有策略匹配用户尝试访问的资源或端点。如果匹配，API 服务器将授权该请求。

现在你应该对 Kubernetes API 如何处理身份验证和授权有了很好的理解。好消息是，尽管你可以直接访问 API，但 Kubernetes 提供了一个出色的命令行工具，可以轻松进行身份验证并发出 Kubernetes API 请求。

# 使用 kubectl 和 YAML

kubectl 是官方支持的用于访问 Kubernetes API 的命令行工具。它可以安装在 Linux、macOS 或 Windows 上。

## 设置 kubectl 和 kubeconfig

要安装 kubectl 的最新版本，你可以参考[`kubernetes.io/docs/tasks/tools/install-kubectl/`](https://kubernetes.io/docs/tasks/tools/install-kubectl/)的安装说明。

一旦安装了 kubectl，它需要设置为与一个或多个集群进行身份验证。这是通过使用`kubeconfig`文件来完成的，格式如下：

示例-kubeconfig

```
apiVersion: v1
kind: Config
preferences: {}
clusters:
- cluster:
    certificate-authority: fake-ca-file
    server: https://1.2.3.4
  name: development
users:
- name: alex
  user:
    password: mypass
    username: alex
contexts:
- context:
    cluster: development
    namespace: frontend
    user: developer
  name: development
```

该文件采用 YAML 格式，类似于我们稍后将介绍的其他 Kubernetes 资源规范——不同之处在于，这个文件仅存在于你的本地机器上。

`Kubeconfig` YAML 文件有三个部分：`clusters`、`users`和`contexts`：

+   `clusters`部分是你能够通过 kubectl 访问的集群列表，包括 CA 文件名和服务器 API 端点。

+   `users`部分列出了你可以授权的用户，包括任何用于认证的用户证书或用户名/密码组合。

+   最后，`contexts`部分列出了集群、命名空间和用户的组合，这些组合形成一个上下文。使用`kubectl config use-context`命令，你可以轻松在不同的上下文之间切换，从而实现集群、用户和命名空间组合的快速切换。

## 命令式与声明式命令

与 Kubernetes API 交互有两种范式：命令式和声明式。命令式命令允许你告诉 Kubernetes“做什么”——例如，“启动两个 Ubuntu 实例”，“将此应用程序扩展为五个副本”等等。

另一方面，声明式命令允许你编写一个文件，指定集群上应该运行的内容，并让 Kubernetes API 确保配置与集群配置匹配，并在必要时进行更新。

尽管命令式命令允许你快速开始使用 Kubernetes，但在运行生产工作负载或任何复杂工作负载时，最好编写一些 YAML 文件并使用声明式配置。原因在于，这使得跟踪变更更容易，例如通过 GitHub 仓库，或者在集群中引入 Git 驱动的**持续集成/持续交付**（**CI/CD**）。

一些基本的 kubectl 命令

kubectl 提供了许多方便的命令，用于检查集群的当前状态、查询资源和创建新资源。kubectl 的结构设计使得大多数命令能够以相同的方式访问资源。

首先，让我们学习如何查看集群中的 Kubernetes 资源。你可以使用 `kubectl get resource_type` 来做到这一点，其中 `resource_type` 是 Kubernetes 资源的全名，或者使用简短的别名。完整的别名列表（以及 `kubectl` 命令）可以在 Kubernetes 文档中找到：[`kubernetes.io/docs/reference/kubectl/overview`](https://kubernetes.io/docs/reference/kubectl/overview)。

我们已经了解了节点，所以让我们从这个开始。要查找集群中存在的节点，我们可以使用 `kubectl get nodes` 或别名 `kubectl get no`。

kubectl 的 `get` 命令返回当前集群中 Kubernetes 资源的列表。我们可以对任何 Kubernetes 资源类型运行此命令。为了在列表中添加更多信息，你可以加上 `wide` 输出标志：`kubectl get nodes -o wide`。

仅列出资源当然不够——我们需要能够查看特定资源的详细信息。为此，我们使用 `describe` 命令，它与 `get` 命令类似，唯一的区别是我们可以选择性地传递特定资源的名称。如果省略此最后一个参数，Kubernetes 将返回该类型所有资源的详细信息，这可能会导致终端输出过多信息。

例如，`kubectl describe nodes` 将返回集群中所有节点的详细信息，而 `kubectl describe nodes node1` 将返回名为 `node1` 的节点的描述。

正如你可能已经注意到的，这些命令都是以命令式风格书写的，这很合理，因为我们只是获取现有资源的信息，而不是创建新资源。要创建一个 Kubernetes 资源，我们可以使用以下命令：

+   `kubectl create -f /path/to/file.yaml`，这是命令式命令

+   `kubectl apply -f /path/to/file.yaml`，这是声明式的

这两个命令都需要一个文件路径，可以是 YAML 或 JSON 文件——或者你也可以直接使用 `stdin`。你还可以传递一个文件夹的路径，而不是文件，这样就会创建或应用该文件夹中的所有 YAML 或 JSON 文件。`create` 是命令式的，因此它会创建一个新的资源，但如果你再次使用相同的文件运行它，命令将失败，因为资源已经存在。`apply` 是声明式的，因此如果你第一次运行它，它会创建资源，随后的运行会更新 Kubernetes 中运行的资源，反映出任何变化。你可以使用 `--dry-run` 标志来查看 `create` 或 `apply` 命令的输出（即，哪些资源将被创建，或者如果存在错误，会显示错误信息）。

要命令式地更新现有资源，请使用 `edit` 命令，如下所示：`kubectl edit resource_type resource_name` —— 就像我们使用 `describe` 命令一样。这将打开默认的终端编辑器，显示现有资源的 YAML，不论你是以命令式还是声明式方式创建的。你可以编辑它并像往常一样保存，这将触发 Kubernetes 中资源的自动更新。

要声明式地更新现有资源，你可以编辑当初用来创建资源的本地 YAML 文件，然后运行 `kubectl apply -f /path/to/file.yaml`。删除资源最好通过命令式命令 `kubectl delete resource_type resource_name` 来完成。

本节我们要讨论的最后一个命令是 `kubectl cluster-info`，它将显示主要 Kubernetes 集群服务运行的 IP 地址。

## 编写 Kubernetes 资源 YAML 文件

与 Kubernetes API 进行声明性通信时，允许使用 YAML 和 JSON 格式。为了本书的目的，我们将坚持使用 YAML，因为它稍微更简洁，且占用页面空间更少。一个典型的 Kubernetes 资源 YAML 文件如下所示：

resource.yaml

```
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: ubuntu
    image: ubuntu:trusty
    command: ["echo"]
    args: ["Hello Readers"]
```

一个有效的 Kubernetes YAML 文件至少有四个顶级键。它们是 `apiVersion`、`kind`、`metadata` 和 `spec`。

`apiVersion` 决定将使用哪个版本的 Kubernetes API 来创建资源。`kind` 指定 YAML 文件所引用的资源类型。`metadata` 提供了命名资源的位置，并可以添加注解和命名空间信息（稍后会详细介绍）。最后，`spec` 键将包含 Kubernetes 创建资源所需的所有资源特定信息。

暂时不用担心 `kind` 和 `spec` —— 我们将在 *第三章*，*在 Kubernetes 上运行应用容器* 中详细介绍什么是 `Pod`。

# 总结

在这一章，我们学习了容器编排的背景，Kubernetes 集群的架构概述，集群如何验证和授权 API 调用，以及如何通过命令式和声明式模式使用 kubectl 与 API 进行通信，kubectl 是 Kubernetes 官方支持的命令行工具。

在下一章，我们将学习几种启动测试集群的方法，并掌握如何利用你迄今为止学到的 kubectl 命令。

# 问题

1.  什么是容器编排？

1.  Kubernetes 控制平面的组成部分有哪些？它们分别有什么作用？

1.  如何以 ABAC 授权模式启动 Kubernetes API 服务器？

1.  为什么在生产 Kubernetes 集群中有多个主节点很重要？

1.  `kubectl apply` 和 `kubectl create` 有什么区别？

1.  如何使用 `kubectl` 在不同上下文之间切换？

1.  在声明式地创建 Kubernetes 资源后，再进行命令式编辑有什么缺点？

# 深入阅读

+   Kubernetes 官方文档: [`kubernetes.io/docs/home/`](https://kubernetes.io/docs/home/)

+   *Kubernetes The Hard Way*: [`github.com/kelseyhightower/kubernetes-the-hard-way`](https://github.com/kelseyhightower/kubernetes-the-hard-way)
