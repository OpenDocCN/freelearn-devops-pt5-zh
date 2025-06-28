# 保护 Kubernetes 集群

安全实现是一场由一个拥有完全封锁策略的团队与一个通过为每个人提供完全自由来取得胜利的团队之间的博弈。你可以把它看作是无政府主义者和极权主义者之间的斗争。游戏能够获胜的唯一方式是两者融合成一种新的方式。唯一可行的策略是，在不牺牲安全性的情况下实现自由（尽可能少地牺牲安全性）。

目前，我们的集群已经尽可能地安全了。这里只有一个用户（你）。没有其他人能够操作它。其他人甚至不能列出集群中的 Pods。你是裁判、陪审团和执行者。你是无可争议的王者，拥有像神一样的权力，这些权力不与任何人共享。

“只有我能做事”策略在模拟笔记本电脑上的集群时非常有效。当唯一的目标是独自学习时，它就能达到目的。当我们创建一个“真实”的集群，整个公司将以某种形式进行协作时，我们就需要定义（并应用）认证和授权策略。如果你的公司很小，只有少数几个人会操作集群，那么为每个人提供相同的集群范围内的管理员权限是一种简单而合法的解决方案。但大多数情况下，情况并非如此。

你的公司可能有不同信任级别的人。即使不是这样，不同的人也会需要不同级别的访问权限。有些人可以做任何他们想做的事情，而其他人则没有任何类型的访问权限。大多数人能够在两者之间做一些事情。我们可能选择为每个人提供一个单独的 Namespace，并禁止他们访问其他 Namespace。有些人可能能够操作生产 Namespace，而其他人可能只对分配给开发和测试的 Namespace 感兴趣。我们可以应用的排列组合是无限的。然而，有一点是肯定的。我们将需要创建认证和授权机制。很可能，我们需要创建一些权限，这些权限有时会在整个集群范围内应用，而在其他情况下则仅限于某些 Namespaces。

这些以及其他许多策略可以通过使用 Kubernetes 授权和认证来创建。

# 访问 Kubernetes API

与 Kubernetes 的每一次交互都必须经过其 API，并且需要进行授权。这种通信可以通过用户或服务账户发起。目前在我们集群中运行的所有 Kubernetes 对象都是通过服务账户与 API 进行交互的。我们不会深入讨论这些内容，而是专注于人类用户的授权。

通常，Kubernetes API 是在一个安全的端口上提供服务的。我们的 Minikube 集群也不例外。我们可以通过 `kubectl` 配置查看端口。

本章中的所有命令都可以在 [`12-auth.sh`](https://gist.github.com/f2c4a72a1e010f1237eea7283a9a0c11) ([`gist.github.com/vfarcic/f2c4a72a1e010f1237eea7283a9a0c11`](https://gist.github.com/vfarcic/f2c4a72a1e010f1237eea7283a9a0c11)) Gist 中找到。

```
kubectl config view \
    -o jsonpath='{.clusters[?(@.name=="minikube")].cluster.server}'  
```

我们使用 `jsonpath` 输出了位于名为 `minikube` 的集群中的 `cluster.server` 条目。

输出如下：

```
https://192.168.99.105:8443
```

我们可以看到 `kubectl` 正在通过 `8443` 端口访问 Minikube Kubernetes API。由于访问已被加密，它需要证书，这些证书存储在 `certificate-authority` 条目中。让我们看一下。

```
kubectl config view \
 -o jsonpath='{.clusters[?(@.name=="minikube")].cluster.certif
icate-authority}'  
```

输出如下：

```
/Users/vfarcic/.minikube/ca.crt  
```

`ca.crt` 证书是与 Minikube 集群一起创建的，目前，它是我们访问 API 的唯一方式。

如果这是一个“真实”的集群，我们还需要为其他用户启用访问权限。我们可以将已经拥有的证书发送给他们，但那样会非常不安全，并且可能导致许多潜在问题。稍后我们将探讨如何安全地为其他用户启用集群访问权限。目前，我们将专注于探索 Kubernetes 用于授权 API 请求的过程。

每个对 API 的请求都经过三个阶段。首先需要进行身份验证，其次需要授权，最后需要通过准入控制。

身份验证过程是从 HTTP 请求中获取用户名。如果请求无法通过身份验证，则操作将以 401 状态码中止。

一旦用户通过身份验证，授权过程将验证是否允许执行指定的操作。授权可以通过 ABAC、RBAC 或 Webhook 模式进行。

最后，一旦请求被授权，它将经过准入控制器。准入控制器在对象被持久化之前拦截请求，并可以对其进行修改。它们是高级主题，本文不会涵盖这些内容。

身份验证过程相当标准，没有太多要说的。另一方面，准入控制器是非常复杂的内容，不适合在此详细讨论。因此，我们将重点研究授权这一主题。

# 授权请求

就像 Kubernetes 中的几乎所有其他功能一样，授权是模块化的。我们可以选择使用 *Node*、*ABAC*、*Webhook* 或 *RBAC* 授权。Node 授权用于特定目的。它根据 kubelet 被调度运行的 Pods 授予权限。**基于属性的访问控制**（**ABAC**）基于属性和策略的组合，并且由于 RBAC 的引入，它被认为已经过时。Webhook 用于通过 HTTP POST 请求进行事件通知。最后，**基于角色的访问控制**（**RBAC**）根据单个用户或组的角色授予（或拒绝）对资源的访问权限。

在四种授权方法中，RBAC 是基于用户授权的正确选择。由于本章将重点探讨授权人类的方式，RBAC 将是我们的主要关注点。

我们可以通过 RBAC 做什么？首先，我们可以通过仅允许授权用户访问来确保集群的安全。我们可以定义不同的角色，为用户和组授予不同级别的访问权限。有些角色可能拥有类似上帝的权限，几乎可以做任何事，而其他角色则可能仅限于基本的非破坏性操作。介于两者之间可能还有许多其他角色。我们可以将 RBAC 与命名空间结合使用，只允许用户在集群的特定部分进行操作。根据具体的使用案例，我们还可以应用许多其他组合。

由于我对过多的理论感到不舒服，我们将把其余的内容留到以后，通过一些示例来探索细节。正如你可能已经猜到的，我们将从一个新的 Minikube 集群开始。

# 创建集群

创建 Minikube 集群的命令如下：

```
cd k8s-specs

git pull

minikube start --vm-driver virtualbox 
kubectl config current-context  
```

从 minikube v0.26 开始，RBAC 默认已安装。如果你的版本较旧，则需要添加 `--extra-config apiserver.Authorization.Mode=RBAC` 参数。或者，更好的是，升级你的 minikube 二进制文件。

在集群中拥有一些对象可能会派上用场，因此我们将部署 `go-demo-2` 应用程序。我们将使用它来测试我们很快将使用的不同授权策略的各种组合。

`go-demo-2` 应用程序的定义与我们在前几章中创建的相同，因此我们将跳过解释，直接执行 `kubectl create`：

```
kubectl create \
 -f auth/go-demo-2.yml \
 --record --save-config  
```

# 创建用户

Kubernetes 的强大功能在你的公司中传播开来了。人们变得好奇，并希望尝试一下。作为 Kubernetes 专家，你接到 John Doe 的电话也就不足为奇了。他想“玩” Kubernetes，但他没有时间自己搭建集群。由于他知道你已经搭建好了集群，他希望你能让他使用你的集群。

由于你不打算将证书给 John，因此决定让他使用自己的用户进行身份验证。

你需要为他创建证书，因此第一步是验证你的笔记本是否已安装 OpenSSL。

```
openssl version  
```

安装的 OpenSSL 版本不应该有问题。我们输出`version`只是为了验证软件是否正常工作。如果输出类似于`command not found: openssl`，则需要安装二进制文件（[`wiki.openssl.org/index.php/Binaries`](https://wiki.openssl.org/index.php/Binaries)）。

我们要做的第一件事是为 John 创建一个私钥。我们假设 John Doe 的用户名是 `jdoe`。

```
mkdir keys

openssl genrsa \
 -out keys/jdoe.key 2048
```

我们创建了 `keys` 目录，并生成了一个私钥 `jdoe.key`。

接下来，我们将使用私钥生成证书：

```
openssl req -new \
    -key keys/jdoe.key \
    -out keys/jdoe.csr \
    -subj "/CN=jdoe/O=devs"
```

给 Windows 用户的提示

如果你收到类似于`Subject does not start with '/'。制作证书请求时出现问题`的错误，请将前一个命令中的`-subj "/CN=jdoe/O=devs"`替换为`-subj "//CN=jdoe\O=devs"`，然后重新执行该命令。

我们创建了证书 `jdoe.csr`，其中包含一个特定的主题，这有助于我们识别约翰。`CN` 是用户名，`O` 表示他所属的组织。约翰是开发者，因此 `devs` 应该是合适的。

对于最终的证书，我们需要集群的**证书颁发机构**（**CA**）。它将负责批准请求并生成约翰用于访问集群的必要证书。由于我们使用了 Minikube，证书颁发机构在集群创建时已经为我们生成。它应该位于操作系统用户的主文件夹中的 `.minikube` 目录下。让我们确认一下它是否在那里。

```
ls -1 ~/.minikube/ca.*  
```

Minikube 的目录可能在其他地方。如果是这种情况，请将 `~/.minikube` 替换为正确的路径。

输出结果如下：

```
/Users/vfarcic/.minikube/ca.crt
/Users/vfarcic/.minikube/ca.key
/Users/vfarcic/.minikube/ca.pem  
```

现在我们可以通过批准证书签名请求 `jdoe.csr` 来生成最终的证书。

```
openssl x509 -req \
    -in keys/jdoe.csr \
    -CA ~/.minikube/ca.crt \
    -CAkey ~/.minikube/ca.key \
    -CAcreateserial \
    -out keys/jdoe.crt \
    -days 365
```

由于我们心胸宽广，我们让证书 `jdoe.crt` 的有效期为一年（365 天）。

为了简化过程，我们将集群的证书颁发机构复制到 `keys` 目录中。

```
cp ~/.minikube/ca.crt keys/ca.crt  
```

让我们检查一下我们生成的内容：

```
ls -1 keys  
```

输出结果如下：

```
ca.crt
jdoe.crt
jdoe.csr
jdoe.key  
```

约翰不需要 `jdoe.csr` 文件。我们只用了它来生成最终证书 `jdoe.crt`。不过，他确实需要其他所有的文件。

除了密钥之外，约翰还需要知道集群的地址。在本章的开始，我们已经创建了 `jsonpath`，它可以检索服务器地址，所以这部分应该很简单。

```
SERVER=$(kubectl config view \
 -o jsonpath='{.clusters[?(@.name=="minikube")].cluster.server
}')

echo $SERVER  
```

输出结果如下：

```
https://192.168.99.106:8443  
```

配备了新的证书、密钥、集群证书颁发机构和服务器地址，约翰可以配置他的 `kubectl` 安装。

由于约翰不在场，我们将进行角色扮演，代替他执行操作。

约翰首先需要使用我们发送给他的地址和证书颁发机构来设置集群。

```
kubectl config set-cluster jdoe \
    --certificate-authority \
    keys/ca.crt \
    --server $SERVER
```

我们创建了一个名为 `jdoe` 的新集群。

接下来，他将需要使用我们为他创建的证书和密钥来设置凭证。

```
kubectl config set-credentials jdoe \
 --client-certificate keys/jdoe.crt \
 --client-key keys/jdoe.key  
```

我们创建了一组名为 `jdoe` 的新凭证。

最后，约翰需要创建一个新的上下文：

```
kubectl config set-context jdoe \
 --cluster jdoe \
 --user jdoe

kubectl config use-context jdoe  
```

我们创建了使用新创建的集群和用户的 `jdoe` 上下文，并确保我们正在使用这个新创建的上下文。

让我们看一下配置：

```
kubectl config view  
```

限制在约翰设置中的输出结果如下：

```
...
clusters:
- cluster:
 certificate-authority: /Users/vfarcic/IdeaProjects/k8s-specs/
keys/ca.crt
 server: https://192.168.99.106:8443
 name: jdoe
...
contexts:
- context:
 cluster: jdoe
 user: jdoe
 name: jdoe
...
current-context: jdoe
...
users:
- name: jdoe
 user:
 client-certificate: /Users/vfarcic/IdeaProjects/k8s-specs/key
s/jdoe.crt
 client-key: /Users/vfarcic/IdeaProjects/k8s-specs/keys/jdoe.k
ey
...  
```

约翰应该会很高兴地认为他可以访问我们的集群。因为他是一个好奇的人，他肯定会想看看我们正在运行的 Pods。

```
kubectl get pods  
```

输出结果如下：

```
Error from server (Forbidden): pods is forbidden: User "jdoe" can
not list pods in the namespace "default"
```

这真让人沮丧。约翰可以访问我们的集群，但他无法获取 Pods 的列表。由于希望永不熄灭，约翰可能会检查是否被禁止查看其他类型的对象。

```
kubectl get all  
```

输出是一个长长的列表，列出了他被禁止查看的所有对象。

约翰拿起手机，不仅请求你给予他访问集群的权限，还请求他有权限“玩”集群。

在修改约翰的权限之前，我们应该先了解一下参与 RBAC 授权过程的组件。

# 探索 RBAC 授权

管理 Kubernetes RBAC 需要了解一些元素。具体来说，我们需要了解规则（Rules）、角色（Roles）、主体（Subjects）和 RoleBindings。

*Rule* 是一组操作（动词）、资源和 API 组。动词描述了可以在属于不同 API 组的资源上执行的活动。

通过规则定义的权限是累加的。我们无法拒绝对某些资源的访问。

当前支持的动词如下：

| **动词** | **描述** |
| --- | --- |
| get | 检索特定对象的信息 |
| list | 检索一组对象的信息 |
| create | 创建特定对象 |
| update | 更新特定对象 |
| patch | 补丁特定对象 |
| watch | 监听对象的变化 |
| proxy | 代理请求 |
| redirect | 重定向请求 |
| delete | 删除特定对象 |
| deletecollection | 删除一组对象 |

例如，如果我们只希望允许用户创建对象并获取其信息，我们将使用动词 `get`、`list` 和 `create`。动词可以是星号（`*`），从而允许所有动词（操作）。

动词与 Kubernetes 资源结合使用。例如，如果我们只希望允许用户创建 Pods 并获取其信息，我们将 `get`、`list` 和 `create` 动词与 `pods` 资源结合使用。

规则的最后一个元素是 API 组。RBAC 使用 `rbac.authorization.k8s.io` 组。如果我们切换到其他授权方法，我们还需要更改该组。

*Role* 是一组规则（Rules）。它定义一个或多个可以绑定到用户或用户组的规则。角色的关键在于它们是应用于某个 Namespace 的。如果我们希望创建一个指向整个集群的角色，我们将使用 *ClusterRole*。两者的定义方式相同，唯一的区别在于作用范围（Namespace 或整个集群）。

授权机制的下一个元素是 *Subjects*。它们定义了执行操作的实体。Subject 可以是 *用户*（User）、*组*（Group）或 *服务帐户*（Service Account）。用户是一个或一个位于集群外的进程。服务帐户用于在 Pods 内部运行的进程，且这些进程想要使用 API。由于本章关注人类身份验证，因此我们现在不讨论它们。最后，组是用户或服务帐户的集合。某些组是默认创建的（例如，`cluster-admin`）。

最后，我们需要 *RoleBindings*。顾名思义，它们将 Subjects 绑定到 Roles。由于 Subjects 定义了用户，RoleBindings 实际上是将用户（或组或服务帐户）绑定到 Roles，从而授予他们在特定 Namespace 内执行某些操作的权限。与角色一样，RoleBindings 有一个集群范围的替代方案，叫做 *ClusterRoleBindings*。唯一的区别是它们的作用范围不限于某个 Namespace，而是应用于整个集群。

所有这些可能看起来令人困惑和压倒性。你甚至可能会说你什么都没理解。别担心。我们将通过实际示例更详细地探讨每个 RBAC 组件。我们先进行了说明，因为人们说应该先解释理论，后演示。我认为这不是正确的方法，但我不希望你说我没有提供理论。不管怎样，接下来的示例会澄清一切。

让我们回到 John 的问题，并尝试解决它。

# 查看预定义的集群角色

John 感到沮丧。他可以访问集群，但不被允许执行任何操作。他甚至不能列出 Pods。自然，他请求我们更加宽容，允许他在我们的集群中“玩”。

由于我们不做任何假设，我们决定第一步应该是验证 John 的说法。是不是他连获取集群中的 Pods 都做不到？

在继续之前，我们将停止模拟 John 的身份，回到使用`minikube`用户授予的类似神的管理员权限来使用集群。

```
kubectl config use-context minikube

kubectl get all  
```

现在我们切换到`minikube`上下文（以及`minikube`用户），我们重新获得了完全的权限，`kubectl get all`返回了`default`命名空间中的所有对象。

让我们验证一下 John 确实不能在`default`命名空间中列出 Pods。

我们可以配置与他使用相同的证书，但那会使过程更加复杂。相反，我们将使用一个`kubectl`命令，允许我们检查如果我们是特定用户，是否能执行某个操作。

```
kubectl auth can-i get pods --as jdoe  
```

响应是`no`，表明`jdoe`不能`get pods`。`--as`参数是一个全局选项，可以应用于任何命令。`kubectl auth can-i`是一个“特殊”命令。它不会执行任何操作，而只是验证是否可以执行某个操作。如果没有`--as`参数，它将验证当前用户（在此情况下为`minikube`）是否可以执行某个操作。

我们已经简要讨论了角色（Roles）和集群角色（ClusterRoles）。让我们看看集群中或`default`命名空间中是否已配置这些角色。

```
kubectl get roles  
```

输出结果显示`no resources`被`found`。在`default`命名空间中没有任何角色。这个结果是预期的，因为 Kubernetes 集群没有预定义的角色。我们需要自己创建需要的角色。

那么集群角色怎么样呢？让我们检查一下。

```
kubectl get clusterroles  
```

这次我们获得了相当多的资源。我们的集群已经默认定义了一些集群角色。那些以`system:`开头的角色是为 Kubernetes 系统保留的集群角色。修改这些角色可能导致集群无法正常工作，因此我们不应更新它们。相反，我们将跳过系统角色，专注于应该分配给用户的角色。

输出结果仅限于那些应分配给用户的集群角色，具体如下：

```
NAME          AGE
admin         1h
cluster-admin 1h
edit          1h
view          1h  
```

权限最少的集群角色是`view`。我们来仔细看看它：

```
kubectl describe clusterrole view  
```

限制输出为前几行，如下所示：

```
Name:        view
Labels:      kubernetes.io/bootstrapping=rbac-defaults
Annotations: rbac.authorization.kubernetes.io/autoupdate=true
PolicyRule:
 Resources              Non-Resource URLs Resource Names Verbs
 ---------              ----------------- -------------- -----
 bindings               []                []             [get li
st watch]
 configmaps             []                []             [get li
st watch]
 cronjobs.batch         []                []             [get li
st watch]
 daemonsets.extensions  []                []             [get li
st watch]
 deployments.apps       []                []             [get li
st watch]
 ...  
```

它包含一个长长的资源列表，所有资源都具有`get`、`list`和`watch`操作权限。看起来，它会允许绑定该角色的用户检索所有资源。我们尚未验证这些资源列表是否真的完整。现在来看，它似乎是一个非常适合分配给那些应拥有非常有限权限的用户的角色。与绑定到特定命名空间的角色不同，集群角色可在整个集群中使用。这是一个显著的区别，我们稍后会利用这一点。

让我们探索另一个预定义的集群角色。

```
kubectl describe clusterrole edit  
```

限制输出为 Pods，如下所示：

```
...
pods             [] [] [create delete deletecollection get list p
atch update watch]
pods/attach      [] [] [create delete deletecollection get list p
atch update watch]
pods/exec        [] [] [create delete deletecollection get list p
atch update watch]
pods/log         [] [] [get list watch]
pods/portforward [] [] [create delete deletecollection get list p
atch update watch]
pods/proxy       [] [] [create delete deletecollection get list p
atch update watch]
pods/status      [] [] [get list watch]
...  
```

如我们所见，`edit`集群角色允许我们对 Pods 执行任何操作。如果我们浏览整个列表，会看到`edit`角色允许我们对任何 Kubernetes 对象执行几乎所有操作。它看起来赋予了我们无限的权限。然而，仍有一些资源未列出。我们可以通过集群角色`admin`来观察这些差异。

```
kubectl describe clusterrole admin  
```

如果你仔细观察，你会注意到集群角色`admin`有一些额外的条目。

限制输出为不包含在集群角色`edit`中的记录，如下所示：

```
...
localsubjectaccessreviews.authorization.k8s.io [] [] [create]
rolebindings.rbac.authorization.k8s.io         [] [] [create dele
te deletecollection get list patch update watch]
roles.rbac.authorization.k8s.io                [] [] [create dele
te deletecollection get list patch update watch]
...  
```

`edit`和`admin`的主要区别在于，后者允许我们操作角色和角色绑定。而`edit`允许我们执行与 Kubernetes 对象如 Pods 和 Deployments 相关的几乎所有操作，`admin`则更进一步，提供了一个额外的功能，允许我们通过修改现有角色或创建新的角色和角色绑定来定义其他用户的权限。`admin`角色的主要限制是，它无法更改命名空间本身，也无法更新资源配额（我们尚未探索这些配额）。

只剩下一个预定义的非系统集群角色。

```
kubectl describe clusterrole \
 cluster-admin  
```

输出如下所示：

```
Name:        cluster-admin
Labels:      kubernetes.io/bootstrapping=rbac-defaults
Annotations: rbac.authorization.kubernetes.io/autoupdate=true
PolicyRule:
 Resources Non-Resource URLs Resource Names Verbs
 --------- ----------------- -------------- -----
 [*]               []             [*]
 *.*       []                []             [*] 
```

集群角色`cluster-admin`毫无保留地提供权限。星号（`*`）意味着所有内容。它赋予了类似上帝般的权限。绑定该角色的用户可以执行任何操作，没有任何限制。`cluster-admin`角色是绑定到`minikube`用户的。我们可以通过执行以下命令轻松确认这一点：

```
kubectl auth can-i "*" "*"  
```

输出为`yes`。尽管我们并未真正确认`cluster-admin`角色已绑定到`minikube`，但我们确实验证了它可以执行任何操作。

# 创建角色绑定和集群角色绑定

角色绑定将用户（或组，或服务账户）绑定到角色（或集群角色）。由于 John 希望能更多地了解我们的集群，我们将创建一个角色绑定，允许他查看`default`命名空间中（几乎）所有的对象。这应该是我们赋予 John 适当权限之旅的良好开始：

```
kubectl create rolebinding jdoe \
 --clusterrole view \
 --user jdoe \
 --namespace default \
 --save-config

kubectl get rolebindings  
```

我们创建了一个名为`jdoe`的角色绑定。由于集群角色`view`已经提供了我们大致需要的权限，我们直接使用它，而不是创建一个全新的角色。

后者命令的输出证明新创建的角色绑定`jdoe`确实已创建。

这是一个很好的时机来澄清一下，Role Binding 并不一定只能与 Role 配合使用，它也可以与 Cluster Role 结合使用（如我们的示例所示）。作为一个经验法则，当我们认为某个角色可能会在集群范围内使用（配合 Cluster Role Bindings）或在多个命名空间中使用（配合 Role Bindings）时，我们会定义 Cluster Roles。权限的作用范围是通过绑定类型来定义的，而不是通过角色类型来定义的。由于我们使用的是 Role Binding，作用范围仅限于单个命名空间，在我们的例子中是 `default`。

让我们查看新创建的 Role Binding 的详细信息：

```
kubectl describe rolebinding jdoe
Name:        jdoe
Labels:      <none>
Annotations: <none>
Role:
 Kind: ClusterRole
 Name: view
Subjects:
 Kind Name Namespace
 ---- ---- ---------
 User jdoe  
```

我们可以看到，Role Binding `jdoe` 只有一个主体，即用户 `jdoe`。命名空间为空可能有点令人困惑，您可能会认为这个 Role Binding 适用于所有命名空间。但这种假设是错误的。请记住，Role Binding 始终与特定命名空间相关联，我们刚刚描述的是在 `default` 命名空间中创建的 Role Binding。相同的 Role Binding 不应该在其他地方可用。让我们确认这一点：

```
kubectl --namespace kube-system \
 describe rolebinding jdoe  
```

我们描述了命名空间 `kube-system` 中的 Role Binding `jdoe`。

输出如下：

```
Error from server (NotFound): rolebindings.rbac.authorization.k8s
.io "jdoe" not found  
```

命名空间 `kube-system` 没有该 Role Binding。我们从未创建过它。

通过 `kubectl auth can-i` 命令验证我们的权限是否设置正确可能会更容易：

```
kubectl auth can-i get pods \
 --as jdoe

kubectl auth can-i get pods \
 --as jdoe --all-namespaces  
```

第一个命令验证了用户 `jdoe` 是否可以从 `default` 命名空间中 `获取 Pods`。答案是 `yes`。第二个命令检查了 Pods 是否可以从所有命名空间中检索，答案是 `no`。目前，John 只能看到 `default` 命名空间中的 Pods，且无法查看其他命名空间中的 Pods。

从现在开始，John 应该能够查看 `default` 命名空间中的 Pods。然而，既然他和我们在同一家公司工作，我们应该对他有更多的信任。为什么不赋予他查看任何命名空间 Pods 的权限呢？为什么不将相同的权限应用于整个集群？在我们这么做之前，我们将删除之前创建的 Role Binding 并重新开始：

```
kubectl delete rolebinding jdoe  
```

我们将修改 John 的 `view` 权限，使其能够跨整个集群应用。我们不再执行其他临时的 kubectl 命令，而是以 YAML 格式定义 `ClusterRoleBinding` 资源，以便更好地记录这个变更。

让我们查看 `auth/crb-view.yml` 文件中的定义。

```
cat auth/crb-view.yml  
```

输出如下：

```
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
 name: view
subjects:
- kind: User
 name: jdoe
 apiGroup: rbac.authorization.k8s.io
roleRef:
 kind: ClusterRole
 name: view
 apiGroup: rbac.authorization.k8s.io  
```

功能上，区别在于这次我们创建的是 `ClusterRoleBinding`，而不是 `RoleBinding`。此外，我们显式地指定了 `apiGroup`，这样可以明确指出 `ClusterRole` 是 RBAC 类型。

```
kubectl create -f auth/crb-view.yml \
 --record --save-config
```

我们创建了 YAML 文件中定义的角色，输出确认了 `clusterrolebinding "view"` 已被 `created`。

我们可以通过描述新创建的角色来进一步验证一切是否正常。

```
kubectl describe clusterrolebinding \
    view
```

输出如下：

```
Name:         view
Labels:       <none>
Annotations:  <none>
Role:
 Kind:  ClusterRole
 Name:  view
Subjects:
 Kind  Name  Namespace
 ----  ----  ---------
 User  jdoe  
```

最后，我们将模拟 John 并验证他是否能够从任何命名空间中获取 Pods：

```
kubectl auth can-i get pods \
 --as jdoe --all-namespaces
```

输出是`yes`，因此确认`jdoe`可以查看 Pods。

我们非常激动，迫不及待地想告诉 John，他已经被授予了权限。然而，通话开始一分钟后，他提出了一个问题。虽然能够查看跨集群的 Pods 是一个好的开始，但他需要一个地方，让他和其他开发人员能拥有更多的自由。他们需要能够部署、更新、删除并访问他们的应用程序。他们可能需要做更多的事情，但他们无法提供更多的信息。他们对 Kubernetes 还不太熟悉，因此不知道该期待什么。他请求你找到一个解决方案，让他们能够执行帮助他们开发和测试软件的操作，而不会影响集群中的其他用户。

这个新请求为结合命名空间与角色绑定提供了一个很好的机会。我们可以创建一个`dev`命名空间，并允许一组选定的用户在其中几乎做任何事情。这应该能够在`dev`命名空间内为开发人员提供足够的自由，同时避免影响其他命名空间中资源的风险。

让我们来看看`auth/rb-dev.yml`的定义：

```
cat auth/rb-dev.yml  
```

输出如下：

```
apiVersion: v1
kind: Namespace
metadata:
 name: dev

---

apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
 name: dev
 namespace: dev
subjects:
- kind: User
 name: jdoe
 apiGroup: rbac.authorization.k8s.io
roleRef:
 kind: ClusterRole
 name: admin
 apiGroup: rbac.authorization.k8s.io  
```

第一部分定义了`dev`命名空间，而第二部分指定了同名的绑定。由于我们使用的是`RoleBinding`（而非`ClusterRoleBinding`），因此影响将仅限于`dev`命名空间。目前，只有一个主体（用户`jdoe`）。我们可以预计，随着时间推移，列表会增长。

最后，`roleRef`使用的是`ClusterRole`（而不是`Role`）类型。尽管集群角色在整个集群中可用，但由于我们将其与`RoleBinding`结合使用，这将使其仅限于指定的命名空间。

`admin`集群角色有一整套广泛的资源和操作，用户（目前只有`jdoe`）将能够在`dev`命名空间内几乎执行任何操作。

让我们创建新资源：

```
ubectl create -f auth/rb-dev.yml \
    --record --save-config  
```

输出如下：

```
namespace "dev" created
rolebinding "dev" created  
```

我们可以看到命名空间和角色绑定已经创建。

让我们验证一下，例如，`jdoe`是否可以创建和删除 Deployments：

```
kubectl --namespace dev auth can-i \
 create deployments --as jdoe

kubectl --namespace dev auth can-i \
 delete deployments --as jdoe
```

在这两种情况下，输出都是`yes`，确认`jdoe`至少可以执行`create`和`delete`操作。由于我们已经查看了`admin`集群角色中定义的资源列表，我们可以假设，如果我们检查其他操作，也会得到相同的响应。

但是，John 仍然没有获得一些权限。只有`cluster-admin`角色涵盖了所有权限。`admin`集群角色非常广泛，但它并不包含所有资源和操作。我们可以通过以下命令确认这一点：

```
kubectl --namespace dev auth can-i \
 "*" "*" --as jdoe
```

输出是`no`，表示在`dev`命名空间内，John 仍然有一些操作是被禁止的。这些操作主要与集群管理相关，仍然在我们的控制之中。

John 很高兴。他和他的开发同事们有一个集群区域，他们可以在其中几乎做任何事情，而不会影响其他命名空间。

John 是一个团队合作者，但他也希望有自己的空间。现在他知道为开发者创建命名空间有多简单，他在想是否可以为他自己生成一个命名空间。你开始觉得他是一个不知足的人，总是要求更多，但你不能否认他的新请求是有道理的。创建他的个人命名空间应该很简单，那为什么不满足他的愿望呢？

让我们来看一下另一个 YAML 定义：

```
cat auth/rb-jdoe.yml
```

输出如下：

```
apiVersion: v1
kind: Namespace
metadata:
    name: jdoe

---

apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
 name: jdoe
 namespace: jdoe
subjects:
- kind: User
 name: jdoe
 apiGroup: rbac.authorization.k8s.io
roleRef:
 kind: ClusterRole
 name: cluster-admin
 apiGroup: rbac.authorization.k8s.io  
```

这个定义与之前的差别不大。重要的变化是命名空间是`jdoe`，而且 John 可能是唯一的用户，至少在他决定添加其他人之前。通过引用 `cluster-admin` 角色，他被赋予了在该命名空间内做任何事情的完全权限。他可能会部署一些酷东西，并授权其他人查看。每个人偶尔都会想炫耀一下。无论如何，那将是他的决定。毕竟，这是他的命名空间，他应该可以在其中做任何他喜欢的事。

让我们创建新的资源：

```
kubectl create -f auth/rb-jdoe.yml \
 --record --save-config
```

在继续之前，我们将确认 John 是否真的可以在 `jdoe` 命名空间中做任何他喜欢的事情。

```
kubectl --namespace jdoe auth can-i \
    "*" "*" --as jdoe
```

正如预期的那样，回应是`yes`，这意味着 John 在他自己小小的星系中是一个神一般的人物。

John 喜欢拥有自己命名空间的想法。他将其作为自己的游乐场。然而，他还有一件事没有满足。他恰好是一个发布经理。与其他开发者不同，他负责将新版本部署到生产环境。他计划通过 Jenkins 来自动化这个过程。然而，这需要一些时间，直到那时，他应该被允许手动执行部署。我们已经决定生产版本应部署到 `default` 命名空间，因此他将需要额外的权限。

经过短暂的讨论，我们决定发布经理所需的最小权限是对 Pods、Deployments 和 ReplicaSets 执行操作。拥有该角色的人应该能够做几乎所有与 Pods 相关的事情，而对 Deployments 和 ReplicaSets 的允许操作应仅限于 `create`、`get`、`list`、`update` 和 `watch`。我们认为他们不应该能删除它们。

我们并不完全确信这些就是发布经理所需要的全部权限，但这是一个很好的开始。如果有需要，我们以后可以随时更新角色。

目前，John 将是唯一的发布经理。一旦我们确信该角色按预期工作，我们会添加更多用户。

既然我们已经有了计划，我们可以继续创建一个角色和绑定，来定义发布经理的权限。我们首先需要做的是弄清楚将要使用的资源、操作动词和 API 组。我们可能需要参考一下`admin`集群角色，以获取灵感：

```
kubectl describe clusterrole admin  
```

仅限 Pods 的输出如下：

```
...
pods             [] [] [create delete deletecollection get list p
atch update watch]
pods/attach      [] [] [create delete deletecollection get list p
atch update watch]
pods/exec        [] [] [create delete deletecollection get list p
atch update watch]
pods/log         [] [] [get list watch]
pods/portforward [] [] [create delete deletecollection get list p
atch update watch]
pods/proxy       [] [] [create delete deletecollection get list p
atch update watch]
pods/status      [] [] [get list watch]
...  
```

如果我们仅将`pods`作为规则资源进行指定，可能无法创建我们需要的所有与 Pods 相关的权限。尽管大多数对 Pods 的操作可以通过`pods`资源来实现，我们可能还需要添加一些子资源。例如，如果我们希望能够检索日志，就需要`pods/log`资源。在这种情况下，`pods`将是一个命名空间资源，而`log`将是`pods`的子资源。

Deployment 和 ReplicaSet 对象带来了不同的挑战。如果我们回顾一下`kubectl describe clusterrole admin`命令的输出，我们会注意到`deployments`有 API 组。与通过斜杠（`/`）分隔的子资源不同，API 组是通过点（`.`）来分隔的。因此，当我们看到像`deployments.apps`这样的资源时，意味着它是通过 API 组`apps`的 Deployment。核心 API 组则被省略。

通过探索`auth/crb-release-manager.yml`中的定义，理解子资源和 API 组可能会更容易：

```
cat auth/crb-release-manager.yml  
```

该定义的大部分内容遵循我们已经使用过几次的相同模式。我们将重点关注`ClusterRole`的`rules`部分。其内容如下：

```
...
rules:
- resources: ["pods", "pods/attach", "pods/exec", "pods/log", "po
ds/status"]
 verbs: ["*"]
 apiGroups: [""]
- resources: ["deployments", "replicasets"]
 verbs: ["create", "get", "list", "watch"]
 apiGroups: ["", "apps", "extensions"]
...  
```

发布经理需要的访问级别在 Pods 和 Deployments、ReplicaSets 之间有所不同。因此，我们将它们分为两组。

第一组规则指定了`pods`资源，并附带了几个子资源（`attach`、`exec`、`log`和`status`）。这应该涵盖了我们迄今为止探索的所有用例。由于我们没有创建 Pod 代理或端口转发，它们没有被包括在内。

我们已经说过发布经理应该能够对 Pods 执行任何操作，所以`verbs`只包含一个带星号（`*`）的条目。另一方面，所有 Pod 资源都属于同一个 Core 组，因此我们无需在`apiGroups`字段中指定任何内容。

第二组规则针对的是`deployments`和`replicasets`资源。考虑到我们决定对它们采取更严格的控制，我们指定了更具体的`verbs`，只允许发布经理执行`create`、`get`、`list`和`watch`操作。由于我们没有指定`delete`、`deletecollection`、`patch`和`update`动词，发布经理将无法执行相关操作。

正如你所看到的，RBAC 规则可以从非常简单到非常精细化，取决于具体需求。我们可以自行决定希望达到的粒度级别。

让我们创建与发布经理相关的角色和绑定。

```
kubectl create \
 -f auth/crb-release-manager.yml \
    --record --save-config
```

为了安全起见，我们将描述新创建的 Cluster Role，并确认它具备我们需要的权限。

```
kubectl describe \
 clusterrole release-manager
```

输出如下：

```
Name:         release-manager
Labels:       <none>
Annotations:  kubectl.kubernetes.io/last-applied-configuration={"
apiVersion":"rbac.authorization.k8s.io/v1","kind":"ClusterRole","
metadata":{"annotations":{},"name":"release-manager","namespace":
""},"rules":[{"apiG...
 kubernetes.io/change-cause=kubectl create --filenam
e=auth/crb-release-manager.yml --record=true --save-config=true
PolicyRule:
 Resources              Non-Resource URLs Resource Names Verbs
 ---------              ----------------- -------------- -----
 deployments            []                []             [create 
get list update watch]
 deployments.apps       []                []             [create 
get list update watch]
 deployments.extensions []                []             [create 
get list update watch]
 pods                   []                []             [*]
 pods/attach            []                []             [*]
 pods/exec              []                []             [*]
 pods/log               []                []             [*]
 pods/status            []                []             [*]
 replicasets            []                []             [create 
get list update watch]
 replicasets.apps       []                []             [create 
get list update watch]
 replicasets.extensions []                []             [create 
get list update watch]  
```

正如你所看到的，被分配到该角色的用户几乎可以对 Pods 执行任何操作，而他们在 Deployments 和 ReplicaSets 上的权限仅限于创建和查看。他们不能更新或删除这些资源。访问任何其他资源是被禁止的。

目前，John 是唯一被绑定到`release-manager`角色的用户。我们将模拟他，并验证他是否能够做一些与 Pods 相关的操作：

```
kubectl --namespace default auth \
 can-i "*" pods --as jdoe
```

我们将进行类似的验证，但仅限于创建 Deployments。

```
kubectl --namespace default auth \
    can-i create deployments --as jdoe
```

在这两种情况下，我们都得到了`yes`的答案，从而确认了 John 可以执行这些操作。

在告诉 John 他的新权限之前，我们最后一次验证将是确认他不能删除 Deployments。

```
kubectl --namespace default auth can-i \
 delete deployments --as jdoe
```

输出是`no`，明确表示此操作是被禁止的。

我们打电话给 John，告诉他他作为发布经理现在在集群中被允许做的所有事情。

让我们看看 John 在获得新权限后会做些什么。我们将通过切换到`jdoe`上下文来模拟他。

```
kubectl config use-context jdoe  
```

通过 Mongo DB 可以快速验证 John 是否能够创建 Deployments。

```
kubectl --namespace default \
 run db --image mongo:3.3
```

John 成功地在`default`命名空间中创建了 Deployment。

```
kubectl --namespace default \
    delete deployment db
```

输出如下：

```
Error from server (Forbidden): replicasets.extensions "db-649df9d
899" is forbidden: User "jdoe" cannot delete replicasets.extensio
ns in the namespace "default"  
```

我们可以看到，John 无法删除由 Deployment 创建的 ReplicaSet。

让我们检查 John 是否可以在自己的命名空间中执行任何操作：

```
kubectl config set-context jdoe \
    --cluster jdoe \
    --user jdoe \
    --namespace jdoe

kubectl config use-context jdoe

kubectl run db --image mongo:3.3  
```

我们更新了`jdoe`的上下文，使其使用与默认命名空间相同名称的命名空间。接下来，我们确保使用该上下文，并基于`mongo`镜像创建了新的 Deployment。

由于 John 应该能够在他的命名空间内执行任何操作，他应该也能够删除该 Deployment。

```
kubectl delete deployment db  
```

最后，让我们尝试一些需要较高权限的操作：

```
kubectl create rolebinding mgandhi \
    --clusterrole=view \
    --user=mgandhi \
    --namespace=jdoe
```

输出如下：

```
rolebinding "mgandhi" created  
```

John 甚至能够向他的命名空间添加新用户，并将他们绑定到任何角色（只要不超出他的权限）。

# 用组替代用户

定义一个可以访问`jdoe`命名空间的单一用户可能是最好的方法。我们预计只有 John 会想要访问它。他是该命名空间的所有者，这是他的私人游乐场。即使他选择添加更多用户，他也可能会独立于我们的 YAML 定义进行操作。毕竟，如果不给他类似神的权限，那么赋予他这种权限又有什么意义呢？从我们的角度看，该命名空间只有一个用户，且将继续只有一个用户。

我们无法对`default`和`dev`命名空间中的权限应用相同的逻辑。我们可能会选择为我们组织中的每个人在`default`命名空间中授予`view`角色。类似地，我们公司中的开发人员应该能够从`dev`命名空间中`部署`、`更新`和`删除`资源。总的来说，我们可以预期，随着时间的推移，`view`和`dev`绑定中的用户数量会增加。不断地添加新用户是一个重复、无聊且容易出错的过程，你可能不希望去做。与其成为一个讨厌自己繁琐工作的人的人，不如创建一个基于角色将用户分组的系统。在我们创建约翰的证书时，我们已经朝这个方向迈出了第一步。

让我们再看看我们之前创建的证书的主题。

```
openssl req -in keys/jdoe.csr \
    -noout -subject
```

输出如下：

```
subject=/CN=jdoe/O=devs  
```

我们可以看到用户名是`jdoe`，他属于`devs`组织。我们将忽略他可能应该至少属于另一个组织（`release-manager`）这一事实。

如果你仔细观察，你可能记得我提到过几次，RBAC 可以与用户、组和服务帐户一起使用。组与用户相同，只不过它们验证附加在请求 API 的证书是否属于指定的组（`O`），而不是名称（`CN`）。

让我们快速看一下另一个 YAML 定义。

```
cat auth/groups.yml  
```

输出如下：

```
apiVersion: v1
kind: Namespace
metadata:
 name: dev

---

apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
 name: dev
 namespace: dev
subjects:
- kind: Group
 name: devs
 apiGroup: rbac.authorization.k8s.io
roleRef:
 kind: ClusterRole
 name: admin
 apiGroup: rbac.authorization.k8s.io

---

apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
 name: view
subjects:
- kind: Group
 name: devs
 apiGroup: rbac.authorization.k8s.io
roleRef:
 kind: ClusterRole
 name: view
 apiGroup: rbac.authorization.k8s.io  
```

你会注意到，角色绑定`dev`和集群角色绑定`view`几乎与我们之前使用的相同。唯一的区别在于`subjects.kind`字段。这次，我们使用`Group`作为值。因此，我们将授予属于`devs`组织的所有用户权限。

在我们应用更改之前，我们需要将上下文切换回`minikube`。

```
kubectl config use-context minikube

kubectl apply -f auth/groups.yml \
    --record
```

输出如下：

```
namespace "dev" configured
rolebinding "dev" configured
clusterrolebinding "view" configured  
```

我们可以看到，新定义重新配置了一些资源。

现在新定义已应用，我们可以验证约翰是否仍然能够在`dev`命名空间中创建对象。

```
kubectl --namespace dev auth \
 can-i create deployments --as jdoe
```

输出是`no`，表示`jdoe`不能`创建部署`。在你开始想知道哪里出错之前，我应该告诉你，这是预期且正确的响应。`--as`参数模拟了约翰的身份，但证书仍然来自`minikube`。Kubernetes 无法知道`jdoe`属于`devs`组。至少，在约翰发出自己的证书请求之前，是无法知道的。

我们将不再使用`--as`参数，而是切换回`jdoe`上下文，并尝试创建一个部署。

```
kubectl config use-context jdoe

kubectl --namespace dev \
    run new-db --image mongo:3.3
```

这次输出是`deployment "new-db" created`，这明确表明，作为`devs`组成员的约翰可以`创建部署`。

从现在开始，任何证书的主题中包含`/O=devs`的用户，都将拥有与约翰在`dev`命名空间内相同的权限，并且在其他地方拥有`view`权限。我们刚刚避免了不断修改 YAML 文件和应用更改的麻烦。

# 现在怎么办？

授权和认证是至关重要的安全组件。如果没有适当的权限设置，我们可能会面临暴露风险，带来潜在的严重后果。此外，通过合适的规则（Rules）、角色（Roles）和角色绑定（RoleBindings），我们不仅可以让集群更安全，还可以促进组织内部不同成员之间的协作。唯一的挑战是找到安全性和自由之间的平衡。这需要时间，直到达成平衡。

结合命名空间和 RBAC 提供了很好的隔离。如果没有命名空间，我们就需要创建多个集群。如果没有 RBAC，这些集群要么会暴露，要么仅限于少数几个用户。两者结合提供了一种在不牺牲安全性的前提下，增加协作的优秀方式。

我们没有深入探讨服务账户（Service Accounts）。它们是除用户（Users）和组（Groups）之外的第三种主题（Subjects）。我们将留到以后再讨论，因为它们主要用于需要访问 Kubernetes API 的 Pod。本章重点讨论的是人类以及我们如何以安全、受控的方式让他们访问集群。

我们仍然缺少一个重要的限制条件。通过结合命名空间（Namespaces）和基于角色的访问控制（RBAC），我们可以限制用户的操作。然而，这并不能防止他们部署可能会导致整个集群崩溃的应用程序。我们需要将资源配额（Resource Quotas）加入其中。这将是下一章的内容。

现在，我们将销毁集群，休息一下。本章我们涵盖了很多内容，值得休息一下。

```
minikube delete  
```

如果你想了解更多关于角色的信息，请查阅 Role v1 rbac 的 API 文档（[`v1-9.docs.kubernetes.io/docs/reference/generated/kubernetes-api/v1.9/#role-v1-rbac`](https://v1-9.docs.kubernetes.io/docs/reference/generated/kubernetes-api/v1.9/#role-v1-rbac)）和 ClusterRole v1 rbac 的 API 文档（[`v1-9.docs.kubernetes.io/docs/reference/generated/kubernetes-api/v1.9/#clusterrole-v1-rbac`](https://v1-9.docs.kubernetes.io/docs/reference/generated/kubernetes-api/v1.9/#clusterrole-v1-rbac)）。同样，你可能也想访问 RoleBinding v1 rbac 的 API 文档（[`v1-9.docs.kubernetes.io/docs/reference/generated/kubernetes-api/v1.9/#rolebinding-v1-rbac`](https://v1-9.docs.kubernetes.io/docs/reference/generated/kubernetes-api/v1.9/#rolebinding-v1-rbac)）和 ClusterRoleBinding v1 rbac 的 API 文档（[`v1-9.docs.kubernetes.io/docs/reference/generated/kubernetes-api/v1.9/#clusterrolebinding-v1-rbac`](https://v1-9.docs.kubernetes.io/docs/reference/generated/kubernetes-api/v1.9/#clusterrolebinding-v1-rbac)）。

# Kubernetes RBAC 与 Docker Swarm RBAC 对比

Docker 也有 RBAC。与 Kubernetes 一样，它围绕主题（subjects）、角色（roles）和资源集合（resource collections）进行组织。在许多方面，二者提供了非常相似的功能。我们是否应该迅速宣布平局？

Kubernetes RBAC 和 Docker 提供的 RBAC 之间有一个关键区别。后者不是免费的。你需要购买 Docker **企业版** (**EE**) 来保障集群的安全，而不仅仅是“只有持证人才能访问”。如果你已经购买了 Docker EE，那么你已经做出了决定，关于是否使用 Docker 或 Kubernetes 的讨论也就结束了。Docker EE 很棒，而且很快它不仅会与 Swarm 一起工作，还会与 Kubernetes 一起工作。你已经购买了它，没有太多理由去切换到其他东西。然而，这个对比专注于开源核心版本能提供的功能，忽略了第三方和企业版本的扩展。

如果我们坚持做一个“仅限开箱即用”的对比，Kubernetes 显然是赢家。它有 RBAC，而 Docker Swarm 没有。问题不在于 Swarm 没有 RBAC，而在于它没有任何内建的基于用户的认证机制。因此，这是一个非常简短的对比。如果你不想购买企业产品，并且确实需要一个授权和认证机制，那么 Kubernetes 是唯一的选择。就像命名空间一样，Kubernetes 通过其众多 Swarm 所没有的功能展现了它的优势。
