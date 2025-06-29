# *第十二章*：Kubernetes 安全性与合规性

在本章节中，你将了解一些 Kubernetes 安全性的关键内容。我们将讨论一些近期的 Kubernetes 安全问题，以及对 Kubernetes 进行的最新审计发现。然后，我们将讨论如何在集群的各个层级实现安全性，从 Kubernetes 资源及其配置的安全性开始，接着是容器安全，最后是通过入侵检测实现的运行时安全。首先，我们将讨论与 Kubernetes 相关的一些关键安全概念。

在本章节中，我们将讨论以下内容：

+   理解 Kubernetes 安全性

+   审查 Kubernetes 的 CVE 和安全审计

+   实现集群配置和容器安全的工具

+   在 Kubernetes 上处理入侵检测、运行时安全性和合规性

# 技术要求

为了运行本章节中详细介绍的命令，你需要一台支持 `kubectl` 命令行工具的计算机，以及一个正常运行的 Kubernetes 集群。请参阅 *第一章*，*与 Kubernetes 通信*，了解几种快速启动 Kubernetes 的方法，以及如何安装 `kubectl` 工具的说明。

此外，你还需要一台支持 Helm CLI 工具的计算机，通常它的前提条件与 `kubectl` 相同—详细信息请查看 Helm 文档，地址为 [`helm.sh/docs/intro/install/`](https://helm.sh/docs/intro/install/)。

本章节中使用的代码可以在本书的 GitHub 仓库找到，地址为 [`github.com/PacktPublishing/Cloud-Native-with-Kubernetes/tree/master/Chapter12`](https://github.com/PacktPublishing/Cloud-Native-with-Kubernetes/tree/master/Chapter12)。

# 理解 Kubernetes 安全性

在讨论 Kubernetes 安全性时，特别需要注意安全边界和共享责任。*共享责任模型*是一个常用术语，用来描述公共云服务中如何处理安全问题。该模型指出，客户负责应用程序的安全性以及公共云组件和服务配置的安全性。而公共云提供商则负责服务本身的安全性，以及它们运行的基础设施的安全性，直到数据中心和物理层。

同样地，Kubernetes 上的安全性也是共享的。虽然上游 Kubernetes 并不是一个商业产品，但成千上万的 Kubernetes 贡献者和大型科技公司背后的组织力量确保了 Kubernetes 组件的安全性得以维护。此外，广泛的个人贡献者和公司使用该技术，确保了 Kubernetes 在 CVE 被报告和处理时不断改进。不幸的是，正如我们在下一节将讨论的那样，Kubernetes 的复杂性意味着存在许多潜在的攻击途径。

然后，应用共享责任模型，作为开发者，你需要负责如何配置 Kubernetes 组件的安全性、在 Kubernetes 上运行的应用程序的安全性，以及集群配置中的访问级别安全性。虽然应用程序和容器本身的安全性不在本书的讨论范围内，但它们对 Kubernetes 安全性至关重要。我们将花费大部分时间讨论配置级别的安全性、访问安全性和运行时安全性。

无论是 Kubernetes 本身还是 Kubernetes 生态系统，都提供了工具、库和完整的产品来处理各个层次的安全性——我们将在本章中回顾其中的一些选项。

在我们讨论这些解决方案之前，最好先从一个基本的理解出发，弄清楚它们为什么在一开始就可能是需要的。接下来，让我们进入下一部分，详细说明 Kubernetes 在安全领域遇到的一些问题。

# 审查 Kubernetes 的 CVE 和安全审计

Kubernetes 遇到了多个 `kubernetes`。每个问题都直接或间接地与 Kubernetes 本身，或与在 Kubernetes 上运行的常见开源解决方案（例如 NGINX Ingress 控制器）相关。

其中一些问题严重到需要对 Kubernetes 源代码进行热修复，因此它们在 CVE 描述中列出了受影响的版本。所有与 Kubernetes 相关的 CVE 完整列表可以在 [`cve.mitre.org/cgi-bin/cvekey.cgi?keyword=kubernetes`](https://cve.mitre.org/cgi-bin/cvekey.cgi?keyword=kubernetes) 找到。为了让你了解一些已发现的问题，我们将按时间顺序回顾其中的一些 CVE。

## 理解 CVE-2016-1905 —— 不当的准入控制

这个 CVE 是 Kubernetes 生产环境中遇到的第一个重大安全问题之一。国家漏洞数据库（NVD，国家标准与技术研究院网站）给出了 7.7 的基础分数，将其归类为高影响级别。

这个问题的关键在于，Kubernetes 的准入控制器未能确保 `kubectl patch` 命令遵循准入规则，从而允许用户完全绕过准入控制器——在多租户场景中，这是一个噩梦。

## 理解 CVE-2018-1002105 —— 后端连接升级

这个 CVE 可能是迄今为止 Kubernetes 项目中最为关键的一个。事实上，NVD 给它的严重性评分为 9.8！在这个 CVE 中，发现某些版本的 Kubernetes 中，攻击者可以利用来自 Kubernetes API 服务器的错误响应，并升级连接。一旦连接被升级，就可以向集群中的任何后端服务器发送经过身份验证的请求。这使得恶意用户能够在没有适当凭据的情况下，基本模拟一个完全认证的 TLS 请求。

除了这些 CVE（并且可能部分由它们驱动），CNCF 在 2019 年资助了对 Kubernetes 的第三方安全审计。审计结果是开源的，并且可以公开访问，值得一读。

## 理解 2019 年的安全审计结果

如我们在上一节提到的，2019 年的 Kubernetes 安全审计是由第三方进行的，审计结果是完全开源的。完整的审计报告可以在 [`www.cncf.io/blog/2019/08/06/open-sourcing-the-kubernetes-security-audit/`](https://www.cncf.io/blog/2019/08/06/open-sourcing-the-kubernetes-security-audit/) 找到。

一般来说，这次审计关注了 Kubernetes 功能的以下几个方面：

+   `kube-apiserver`

+   `etcd`

+   `kube-scheduler`

+   `kube-controller-manager`

+   `cloud-controller-manager`

+   `kubelet`

+   `kube-proxy`

+   容器运行时

审计的目的是聚焦于 Kubernetes 中最重要和最相关的安全部分。审计结果不仅包括完整的安全报告，还包括威胁模型、渗透测试和白皮书。

深入审查审计结果不在本书的范围之内，但有一些主要的结论可以为我们提供许多 Kubernetes 安全问题的核心窗口。

简而言之，审计发现，由于 Kubernetes 是一个复杂的、网络高度集成的系统，拥有许多不同的设置，缺乏经验的工程师可能会执行某些配置，从而使集群暴露于外部攻击者。

Kubernetes 足够复杂，以至于一个不安全的配置可能很容易发生，这一点很重要，需要牢记。

整个审计报告值得阅读——对于那些具有丰富网络安全和容器知识的人来说，这是 Kubernetes 平台开发过程中做出的某些安全决策的绝佳视角。

现在我们已经讨论了 Kubernetes 安全问题的发现，我们可以开始探讨如何提高集群的安全性。让我们从 Kubernetes 的一些默认安全功能开始。

# 实施集群配置和容器安全的工具

Kubernetes 为集群配置和容器权限的安全性提供了许多内建选项。既然我们已经讨论过 RBAC、TLS Ingress 和加密的 Kubernetes Secrets，那么接下来我们将讨论一些尚未讨论的概念：入站控制器、Pod 安全策略和网络策略。

## 使用入站控制器

入站控制器是一个常常被忽视，但非常重要的 Kubernetes 特性。Kubernetes 的许多高级特性在背后使用了入站控制器。此外，你还可以创建新的入站控制器规则，为集群添加自定义功能。

入站控制器有两种主要类型：

+   修改入站控制器

+   验证准入控制器

变异准入控制器接收 Kubernetes 资源规范并返回更新后的资源规范。它们还执行副作用计算或进行外部调用（对于自定义准入控制器而言）。

另一方面，验证准入控制器仅仅接受或拒绝 Kubernetes 资源 API 请求。需要了解的是，这两种类型的控制器仅对创建、更新、删除或代理请求起作用。这些控制器不能变异或更改列出资源的请求。

当此类请求进入 Kubernetes API 服务器时，它会首先通过所有相关的变异准入控制器进行处理。然后，可能被变异的输出会通过验证准入控制器，最后在 API 服务器中执行（如果被准入控制器拒绝，则不会执行）。

从结构上来看，Kubernetes 提供的准入控制器是作为 Kubernetes API 服务器的一部分运行的函数或“插件”。它们依赖于两个 webhook 控制器（它们本身也是准入控制器，只不过是特殊的）：**MutatingAdmissionWebhook** 和 **ValidatingAdmissionWebhook**。所有其他准入控制器在内部使用这两个 webhook 中的一个，具体取决于它们的类型。此外，您编写的任何自定义准入控制器都可以附加到这两个 webhook 中的任意一个。

在我们探讨如何创建自定义准入控制器之前，让我们回顾一下 Kubernetes 提供的几个默认准入控制器。欲了解完整列表，请参考 Kubernetes 官方文档：[`kubernetes.io/docs/reference/access-authn-authz/admission-controllers/#what-does-each-admission-controller-do`](https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/#what-does-each-admission-controller-do)。

### 了解默认准入控制器

在典型的 Kubernetes 设置中，存在相当多的默认准入控制器，其中许多对于一些非常重要的基本功能是必需的。以下是一些默认准入控制器的示例。

#### NamespaceExists 准入控制器

**NamespaceExists** 准入控制器检查任何传入的 Kubernetes 资源（除了命名空间本身）。这是为了检查资源所附加的命名空间是否存在。如果不存在，它会在准入控制器级别拒绝该资源请求。

#### PodSecurityPolicy 准入控制器

**PodSecurityPolicy** 准入控制器支持 Kubernetes Pod 安全策略，我们稍后将了解该策略。此控制器防止不遵循 Pod 安全策略的资源被创建。

除了默认的准入控制器，我们还可以创建自定义准入控制器。

### 创建自定义准入控制器

创建自定义准入控制器可以通过动态使用两种 webhook 控制器之一来完成。其工作方式如下：

1.  你必须编写自己的服务器或脚本，它需要独立于 Kubernetes API 服务器运行。

1.  然后，你需要配置前面提到的两种 webhook 触发器之一，向你的自定义服务器控制器发送包含资源数据的请求。

1.  根据结果，webhook 控制器会告知 API 服务器是否继续执行。

让我们从第一步开始：编写一个简单的准入服务器。

### 编写自定义准入控制器的服务器

要创建我们的自定义准入控制器服务器（它将接收来自 Kubernetes 控制平面的 webhook），我们可以使用任何编程语言。与大多数 Kubernetes 扩展一样，Go 语言拥有最好的支持和库，使得编写自定义准入控制器变得更加容易。现在，我们将使用一些伪代码。

我们服务器的控制流大致如下：

Admission-controller-server.pseudo

```
// This function is called when a request hits the
// "/mutate" endpoint
function acceptAdmissionWebhookRequest(req)
{
  // First, we need to validate the incoming req
  // This function will check if the request is formatted properly
  // and will add a "valid" attribute If so
  // The webhook will be a POST request from Kubernetes in the
  // "AdmissionReviewRequest" schema
  req = validateRequest(req);
  // If the request isn't valid, return an Error
  if(!req.valid) return Error; 
  // Next, we need to decide whether to accept or deny the Admission
  // Request. This function will add the "accepted" attribute
  req = decideAcceptOrDeny(req);
  if(!req.accepted) return Error;
  // Now that we know we want to allow this resource, we need to
  // decide if any "patches" or changes are necessary
  patch = patchResourceFromWebhook(req);
  // Finally, we create an AdmissionReviewResponse and pass it back
  // to Kubernetes in the response
  // This AdmissionReviewResponse includes the patches and
  // whether the resource is accepted.
  admitReviewResp = createAdmitReviewResp(req, patch);
  return admitReviewResp;
}
```

现在我们已经有了一个简单的服务器来处理我们的自定义准入控制器，我们可以配置一个 Kubernetes 准入 webhook 来调用它。

### 配置 Kubernetes 调用自定义准入控制器服务器

为了让 Kubernetes 调用我们的自定义准入服务器，它需要一个调用的地址。我们可以将自定义准入控制器部署在任何地方——它不必部署在 Kubernetes 上。

话虽如此，在本章的目的下，在 Kubernetes 上运行它是很容易的。我们不会详细介绍完整的清单，但假设我们有一个 Service 和一个指向它的 Deployment，运行着一个容器作为我们的服务器。Service 的配置大致如下：

Service-webhook.yaml

```
apiVersion: v1
kind: Service
metadata:
  name: my-custom-webhook-server
spec:
  selector:
    app: my-custom-webhook-server
  ports:
    - port: 443
      targetPort: 8443
```

需要注意的是，我们的服务器必须使用 HTTPS，以便 Kubernetes 能接受 webhook 响应。配置方式有很多种，我们在本书中不会深入讨论。证书可以是自签名的，但证书的通用名称和 CA 必须与设置 Kubernetes 集群时使用的名称匹配。

既然我们的服务器已经在运行并接收 HTTPS 请求，让我们告诉 Kubernetes 在哪里可以找到它。为此，我们使用`MutatingWebhookConfiguration`。

以下代码块展示了 `MutatingWebhookConfiguration` 的示例：

Mutating-webhook-config-service.yaml

```
apiVersion: admissionregistration.k8s.io/v1beta1
kind: MutatingWebhookConfiguration
metadata:
  name: my-service-webhook
webhooks:
  - name: my-custom-webhook-server.default.svc
    rules:
      - operations: [ "CREATE" ]
        apiGroups: [""]
        apiVersions: ["v1"]
        resources: ["pods", "deployments", "configmaps"]
    clientConfig:
      service:
        name: my-custom-webhook-server
        namespace: default
        path: "/mutate"
      caBundle: ${CA_PEM_B64}
```

让我们详细分析一下 `MutatingWebhookConfiguration` 的 YAML 配置。正如你所见，我们可以在这个配置中配置多个 webhook——尽管在这个示例中我们只配置了一个。

对于每个 webhook，我们设置`name`、`rules` 和 `configuration`。`name`只是 webhook 的标识符。`rules` 让我们配置 Kubernetes 在哪些情况下应该向我们的准入控制器发起请求。在此案例中，我们已经将 webhook 配置为在 `pods`、`deployments` 和 `configmaps` 类型的资源发生 `CREATE` 事件时触发。

最后，我们有`clientConfig`，在这里我们指定 Kubernetes 应该如何以及在何处发起 webhook 请求。由于我们在 Kubernetes 上运行自定义服务器，因此除了指定服务器路径（`"/mutate"` 是此处的最佳实践）外，我们还需要指定服务名称，正如之前的 YAML 文件中所示，并提供集群的 CA 以便与 HTTPS 终止证书进行比较。如果您的自定义准入服务器运行在其他地方，还有其他可能的配置字段——如果需要，请查阅文档（[`kubernetes.io/docs/reference/access-authn-authz/admission-controllers/`](https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/)）。

一旦我们在 Kubernetes 中创建了 `MutatingWebhookConfiguration`，测试验证就变得很容易。我们需要做的就是像往常一样创建一个 Pod、Deployment 或 ConfigMap，并检查我们的请求是否根据服务器中的逻辑被拒绝或修补。

假设我们的服务器目前被设置为拒绝任何名称中包含字符串 `deny-me` 的 Pod。它还设置了一个错误响应到 `AdmissionReviewResponse`。

让我们使用如下的 Pod 规格：

To-deny-pod.yaml

```
apiVersion: v1
kind: Pod
metadata:
  name: my-pod-to-deny
spec:
  containers:
  - name: nginx
    image: nginx
```

现在，我们可以创建 Pod 来检查准入控制器。我们可以使用以下命令：

```
kubectl create -f to-deny-pod.yaml
```

这将导致以下输出：

```
Error from server (InternalError): error when creating "to-deny-pod.yaml": Internal error occurred: admission webhook "my-custom-webhook-server.default.svc" denied the request: Pod name contains "to-deny"!
```

就这样！我们的自定义准入控制器成功地拒绝了一个与我们服务器中指定条件不匹配的 Pod。对于被修补的资源（不是被拒绝，而是被修改），`kubectl` 不会显示任何特殊响应。您需要获取相关资源，查看补丁的效果。

现在我们已经探索了自定义准入控制器，让我们看看另一种实施集群安全实践的方法——Pod 安全策略。

## 启用 Pod 安全策略

Pod 安全策略的基本原理是，它们允许集群管理员创建规则，要求 Pods 必须遵循这些规则才能被调度到节点上。从技术上讲，Pod 安全策略仅是另一种类型的准入控制器。然而，这一功能已被 Kubernetes 官方支持，值得深入讨论，因为有许多可用的选项。

Pod 安全策略可以用来防止 Pods 以 root 身份运行，限制使用的端口和卷，限制特权提升等。我们现在将回顾 Pod 安全策略的一部分功能，但要查看完整的 Pod 安全策略配置类型列表，请查阅官方的 PSP 文档 [`kubernetes.io/docs/concepts/policy/pod-security-policy/`](https://kubernetes.io/docs/concepts/policy/pod-security-policy/)。

最后需要注意的是，Kubernetes 还支持用于控制容器权限的低级原语——即 *AppArmor*、*SELinux* 和 *Seccomp*。这些配置超出了本书的范围，但它们在高安全性环境中可能会非常有用。

### 创建 Pod 安全策略的步骤

实现 Pod 安全策略的步骤如下：

1.  首先，必须启用 Pod 安全策略准入控制器。

1.  这将阻止在集群中创建所有 Pod，因为它要求匹配的 Pod 安全策略和角色才能创建 Pod。由于这个原因，您可能希望在启用准入控制器之前先创建 Pod 安全策略和角色。

1.  启用准入控制器后，必须创建相应的策略。

1.  然后，必须创建一个`Role`或`ClusterRole`对象，并授予其访问 Pod 安全策略的权限。

1.  最后，可以将该角色与`accountService`账户绑定，从而允许使用该服务账户创建的 Pod 使用 Pod 安全策略中可用的权限。

在某些情况下，您的集群可能默认未启用 Pod 安全策略准入控制器。让我们来看一下如何启用它。

### 启用 Pod 安全策略准入控制器

为了启用 PSP 准入控制器，`kube-apiserver`必须使用一个标志启动，该标志指定要启动的准入控制器。在托管 Kubernetes（如 EKS、AKS 等）上，PSP 准入控制器可能会默认启用，同时为初始管理员用户创建了一个特权 Pod 安全策略。这可以防止 PSP 在新集群中创建 Pod 时引发任何问题。

如果您是自行管理 Kubernetes 并且尚未启用 PSP 准入控制器，您可以通过以下标志重新启动`kube-apiserver`组件来启用它：

```
kube-apiserver --enable-admission-plugins=PodSecurityPolicy,ServiceAccount…<all other desired admission controllers>
```

如果您的 Kubernetes API 服务器是通过`systemd`文件运行的（如果按照*Kubernetes: The Hard Way*的方式进行部署），则应在此文件中更新标志。通常，`systemd`文件位于`/etc/systemd/system/`文件夹中。

为了找出哪些准入插件已经启用，您可以运行以下命令：

```
kube-apiserver -h | grep enable-admission-plugins
```

此命令将列出启用的所有准入插件。例如，您将在输出中看到以下准入插件：

```
NamespaceLifecycle, LimitRanger, ServiceAccount…
```

既然我们已经确认 PSP 准入控制器已启用，我们实际上可以创建一个 PSP。

### 创建 PSP 资源

Pod 安全策略本身可以使用典型的 Kubernetes 资源 YAML 文件创建。下面是一个特权 Pod 安全策略的 YAML 文件：

privileged-psp.yaml

```
apiVersion: policy/v1beta1
kind: PodSecurityPolicy
metadata:
  name: privileged-psp
  annotations:
    seccomp.security.alpha.kubernetes.io/allowedProfileNames: '*'
spec:
  privileged: true
  allowedCapabilities:
  - '*'
  volumes:
  - '*'
  hostNetwork: true
  hostPorts:
  - min: 2000
    max: 65535
  hostIPC: true
  hostPID: true
  allowPrivilegeEscalation: true
  runAsUser:
    rule: 'RunAsAny'
  supplementalGroups:
    rule: 'RunAsAny'
  fsGroup:
    rule: 'RunAsAny'
```

此 Pod 安全策略允许用户或服务账户（通过`PodSecurityPolicy`）绑定主机网络上的`2000`至`65535`端口，可以以任何用户身份运行，并且可以绑定任何类型的卷。此外，我们还为`allowedProfileNames`添加了一个`seccomp`限制注释—以帮助您理解`Seccomp`和`AppArmor`注释如何与`PodSecurityPolicies`配合使用。

如前所述，仅创建 PSP 并不会做任何事情。对于任何将创建特权 Pod 的服务账户或用户，我们需要通过`ClusterRole`和`ClusterRoleBinding`为他们提供对 Pod 安全策略的访问权限。

为了创建一个可以访问这个 PSP 的 `ClusterRole`，我们可以使用以下 YAML：

Privileged-clusterrole.yaml

```
apiVersion: rbac.authorization.k8s.io
kind: ClusterRole
metadata:
  name: privileged-role
rules:
- apiGroups: ['policy']
  resources: ['podsecuritypolicies']
  verbs:     ['use']
  resourceNames:
  - privileged-psp
```

现在，我们可以将新创建的 `ClusterRole` 绑定到我们打算用来创建特权 Pod 的用户或服务账户。我们可以通过 `ClusterRoleBinding` 来实现：

Privileged-clusterrolebinding.yaml

```
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: privileged-crb
roleRef:
  kind: ClusterRole
  name: privileged-role
  apiGroup: rbac.authorization.k8s.io
subjects:
- kind: Group
  apiGroup: rbac.authorization.k8s.io
  name: system:authenticated
```

在我们的案例中，我们希望让集群中的每个经过身份验证的用户都能创建特权 Pod，因此我们将其绑定到 `system:authenticated` 组。

现在，可能我们不希望所有用户或 Pod 都具备特权。一个更现实的 Pod 安全策略是对 Pod 能做的事情进行限制。

让我们来看一下一个示例 YAML，它展示了具有以下限制的 PSP：

unprivileged-psp.yaml

```
apiVersion: policy/v1beta1
kind: PodSecurityPolicy
metadata:
  name: unprivileged-psp
spec:
  privileged: false
  allowPrivilegeEscalation: false
  volumes:
    - 'configMap'
    - 'emptyDir'
    - 'projected'
    - 'secret'
    - 'downwardAPI'
    - 'persistentVolumeClaim'
  hostNetwork: false
  hostIPC: false
  hostPID: false
  runAsUser:
    rule: 'MustRunAsNonRoot'
  supplementalGroups:
    rule: 'MustRunAs'
    ranges:
      - min: 1
        max: 65535
  fsGroup:
    rule: 'MustRunAs'
    ranges:
      - min: 1
        max: 65535
  readOnlyRootFilesystem: false
```

正如你所看到的，这个 Pod 安全策略在对创建的 Pod 施加的限制方面与其他策略有很大不同。根据该策略，没有 Pod 被允许以 root 身份运行或提升为 root 权限。它们在绑定卷类型上也有所限制（这一部分在前面的代码片段中已被突出显示）——并且它们不能使用主机网络或直接绑定到主机端口。

在这个 YAML 中，`runAsUser` 和 `supplementalGroups` 部分控制可以由容器运行或添加的 Linux 用户 ID 和组 ID，而 `fsGroup` 键控制容器可以使用的文件系统组。

除了使用像 `MustRunAsNonRoot` 这样的规则之外，还可以直接指定容器可以运行的用户 ID——任何未在规范中明确指定该 ID 的 Pod 将无法调度到节点上。

对于一个限制用户为特定 ID 的示例 PSP，请参考以下 YAML：

Specific-user-id-psp.yaml

```
apiVersion: policy/v1beta1
kind: PodSecurityPolicy
metadata:
  name: specific-user-psp
spec:
  privileged: false
  allowPrivilegeEscalation: false
  hostNetwork: false
  hostIPC: false
  hostPID: false
  runAsUser:
    rule: 'MustRunAs'
    ranges:
      - min: 1
        max: 3000
  readOnlyRootFilesystem: false
```

这个 Pod 安全策略应用后，将防止任何 Pod 以用户 ID `0` 或 `3001` 或更高的 ID 运行。为了创建符合这一条件的 Pod，我们在 Pod 规范中的 `securityContext` 中使用 `runAs` 选项。

这是一个满足此限制的 Pod 示例，即使在启用该 Pod 安全策略的情况下，它也会成功调度：

Specific-user-pod.yaml

```
apiVersion: v1
kind: Pod
metadata:
  name: specific-user-pod
spec:
  securityContext:
    runAsUser: 1000
  containers:
  - name: test
    image: busybox
    securityContext:
      allowPrivilegeEscalation: false
```

如你所见，在这个 YAML 中，我们为 Pod 指定了一个特定的用户 ID `1000`。我们还禁止 Pod 提升为 root 权限。即使在 `specific-user-psp` 策略生效的情况下，这个 Pod 规范也能成功调度。

现在我们已经讨论了如何通过对 Pod 运行的限制来保障 Kubernetes 的安全，接下来我们可以讨论网络策略，利用它我们可以限制 Pod 之间的网络通信。

## 使用网络策略

Kubernetes 中的网络策略与防火墙规则或路由表类似。它们允许用户通过选择器指定一组 Pod，并确定这些 Pod 如何以及在哪里进行通信。

为了使网络策略生效，您选择的 Kubernetes 网络插件（如*Weave*、*Flannel*或*Calico*）必须支持网络策略规格。网络策略可以像所有其他 Kubernetes 资源一样通过 YAML 文件创建。我们从一个非常简单的网络策略开始。

这是一个限制访问标签为`app=server`的 Pod 的网络策略规格：

Label-restriction-policy.yaml

```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: frontend-network-policy
spec:
  podSelector:
    matchLabels:
      app: server
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
    ports:
    - protocol: TCP
      port: 80
```

现在，让我们解析这个网络策略 YAML，因为它将帮助我们在接下来的内容中解释一些更复杂的网络策略。

首先，在我们的规格中，我们有一个`podSelector`，其功能类似于节点选择器。这里，我们使用`matchLabels`来指定此网络策略将只影响标签为`app=server`的 Pod。

接下来，我们为网络策略指定一个策略类型。有两种策略类型：`ingress`和`egress`。一个网络策略可以指定一个或两个类型。`ingress`指的是为连接到匹配的 Pod 的流量设置网络规则，而`egress`指的是为离开匹配 Pod 的连接设置网络规则。

在这个特定的网络策略中，我们仅定义了一个`ingress`规则：只有来自标签为`app:frontend`的 Pod 的流量才会被标签为`app=server`的 Pod 接受。此外，标签为`app=server`的 Pod 上唯一会接受流量的端口是`80`。

在`ingress`策略集中可以有多个`from`块，它们对应多个流量规则。类似地，在`egress`中，也可以有多个`to`块。

需要注意的是，网络策略是按命名空间工作的。默认情况下，如果一个命名空间中没有任何网络策略，则该命名空间内的 Pod 之间没有限制通信。然而，一旦某个特定 Pod 被单个网络策略选中，所有进出该 Pod 的流量必须明确匹配网络策略规则。如果不匹配规则，则会被阻止。

有了这个考虑，我们可以轻松创建强制执行对 Pod 网络的广泛限制的策略。让我们来看一下以下网络策略：

Full-restriction-policy.yaml

```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: full-restriction-policy
  namespace: development
spec:
  policyTypes:
  - Ingress
  - Egress
  podSelector: {}
```

在这个`NetworkPolicy`中，我们指定将同时包含`Ingress`和`Egress`策略，但我们没有为它们编写任何块。这会导致自动拒绝任何`Egress`和`Ingress`流量，因为没有规则可以匹配流量。

此外，我们的`{}` Pod 选择器值对应于选择命名空间中的每个 Pod。此规则的最终结果是，`development`命名空间中的每个 Pod 都无法接受`ingress`流量或发送`egress`流量。

重要说明

还需要注意的是，网络策略是通过将所有影响 Pod 的单独网络策略结合起来进行解释，然后将所有这些规则的组合应用于 Pod 流量。

这意味着，即使我们在前面的例子中已经限制了`development`命名空间中所有的进出流量，我们仍然可以通过添加另一个网络策略来为特定 Pod 启用流量。

假设现在我们的`development`命名空间对 Pod 之间的流量进行了完全限制，我们希望允许一部分 Pod 在`443`端口接收网络流量，并在`6379`端口向数据库 Pod 发送流量。为了实现这一点，我们只需要创建一个新的网络策略，通过策略的累加特性，允许这些流量。

这是网络策略的样子：

Override-restriction-network-policy.yaml

```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: override-restriction-policy
  namespace: development
spec:
  podSelector:
    matchLabels:
      app: server
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
    ports:
    - protocol: TCP
      port: 443
  egress:
  - to:
    - podSelector:
        matchLabels:
          app: database
    ports:
    - protocol: TCP
      port: 6379
```

在这个网络策略中，我们允许`development`命名空间中的服务器 Pod 接收来自前端 Pod 的`443`端口流量，并将流量发送到数据库 Pod 的`6379`端口。

如果我们想要开放所有 Pod 之间的通信，而没有任何限制，同时仍然实际实施一个网络策略，我们可以使用以下的 YAML 配置：

All-open-network-policy.yaml

```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-all-egress
spec:
  podSelector: {}
  egress:
  - {}
  ingress:
  - {}
  policyTypes:
  - Egress
  - Ingress
```

现在我们已经讨论了如何使用网络策略来设置 Pod 之间流量的规则。然而，网络策略也可以作为外部防火墙使用。为此，我们需要创建基于外部 IP 地址的网络策略规则，而不是基于 Pod 的来源或目的地。

让我们看一个例子，网络策略限制了一个 Pod 的进出通信，并且指定了一个特定的 IP 范围作为目标：

External-ip-network-policy.yaml

```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: specific-ip-policy
spec:
  podSelector:
    matchLabels:
      app: worker
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - ipBlock:
        cidr: 157.10.0.0/16
        except:
        - 157.10.1.0/24
  egress:
  - to:
    - ipBlock:
        cidr: 157.10.0.0/16
        except:
        - 157.10.1.0/24
```

在这个网络策略中，我们指定了一个`Ingress`规则和一个`Egress`规则。每个规则基于网络请求的源 IP 地址来决定是否接受或拒绝流量，而不是基于流量来自哪个 Pod。

在我们的例子中，我们为`Ingress`和`Egress`规则选择了一个`/16`子网掩码范围（并指定了一个`/24`CIDR 例外）。这会导致任何来自集群内的流量无法到达这些 Pod，因为在默认的集群网络设置中，我们的 Pod IP 地址都不会与这些规则匹配。

然而，来自指定子网掩码内（而不在例外范围内）集群外部的流量将能够向`worker` Pod 发送流量，并且能够接收来自`worker` Pod 的流量。

在我们讨论完网络策略之后，我们可以转到安全堆栈的完全不同层次——运行时安全性和入侵检测。

# 处理 Kubernetes 中的入侵检测、运行时安全性和合规性

一旦你设置了 Pod 安全策略和网络策略，并且基本确保你的配置尽可能地严密，Kubernetes 中仍然存在许多潜在的攻击途径。在这一部分，我们将重点讨论来自 Kubernetes 集群内部的攻击。即使你设置了高度特定的 Pod 安全策略（这无疑会有所帮助，明确一点），集群中运行的容器和应用程序仍然可能执行意外或恶意操作。

为了解决这个问题，许多专业人员依赖于运行时安全工具，这些工具可以持续监控并警报应用进程。对于 Kubernetes，一个常用的开源工具是 *Falco*。

## 安装 Falco

Falco 将自己定义为 Kubernetes 上进程的 *行为活动监控器*。它可以监控运行在 Kubernetes 上的容器化应用程序以及 Kubernetes 组件本身。

Falco 是如何工作的？在实时情况下，Falco 解析来自 Linux 内核的系统调用。然后，它通过规则对这些系统调用进行过滤——这些规则是可以应用于 Falco 引擎的一组配置。每当一个系统调用违反规则时，Falco 就会触发警报。就是这么简单！

Falco 默认带有一套详尽的规则，这些规则提供了内核级别的显著可观察性。Falco 当然支持自定义规则——我们将展示如何编写这些规则。

然而，在此之前，我们需要先在集群中安装 Falco！幸运的是，Falco 可以通过 Helm 安装。不过，需要特别注意的是，安装 Falco 有几种不同的方法，在发生安全漏洞时，不同的方法效果会有显著差异。

我们将使用 Helm 图表来安装 Falco，这种方式简单并且适用于管理型 Kubernetes 集群，或任何可能无法直接访问工作节点的场景。

然而，为了获得最佳的安全防护，应该将 Falco 直接安装到 Linux 层的 Kubernetes 节点上。使用 DaemonSet 的 Helm 图表非常便捷，但从安全角度来看，直接安装 Falco 比通过 Helm 图表安装要更为安全。要将 Falco 直接安装到你的节点，请查看安装说明：[`falco.org/docs/installation/`](https://falco.org/docs/installation/)。

在上述说明之后，我们可以使用 Helm 安装 Falco：

1.  首先，我们需要将 `falcosecurity` 仓库添加到本地 Helm 中：

    ```
    helm repo add falcosecurity https://falcosecurity.github.io/charts
    helm repo update
    ```

    接下来，我们可以继续使用 Helm 安装 Falco。

    重要说明

    Falco Helm 图表有许多可以在 values 文件中更改的变量 – 如果要查看完整的配置项，可以查看官方的 Helm 图表仓库：[`github.com/falcosecurity/charts/tree/master/falco`](https://github.com/falcosecurity/charts/tree/master/falco)。

1.  要安装 Falco，请运行以下命令：

    ```
    helm install falco falcosecurity/falco
    ```

该命令将使用默认值安装 Falco，默认值可以在 [`github.com/falcosecurity/charts/blob/master/falco/values.yaml`](https://github.com/falcosecurity/charts/blob/master/falco/values.yaml) 查看。

接下来，让我们深入了解 Falco 能为注重安全的 Kubernetes 管理员提供哪些功能。

## 了解 Falco 的功能

如前所述，Falco 自带一套默认规则，但我们可以通过新的 YAML 文件轻松添加更多规则。由于我们使用的是 Helm 版本的 Falco，向 Falco 传递自定义规则只需创建一个新的 values 文件或编辑默认文件并添加自定义规则即可。

添加自定义规则如下所示：

Custom-falco.yaml

```
customRules:
  my-rules.yaml: |-
    Rule1
    Rule2
    etc...
```

现在是讨论 Falco 规则结构的好时机。为说明这一点，让我们借用 Falco Helm Chart 中随附的 `Default` Falco 规则集中的几行规则。

在指定 Falco 配置时，我们可以使用三种不同类型的键来帮助编写规则。这些键分别是宏、列表和规则本身。

我们在这个示例中查看的特定规则叫做 `Launch Privileged Container`。这个规则将检测何时启动一个特权容器，并将容器的一些信息记录到 `STDOUT`。规则可以在警报方面执行各种操作，但记录到 `STDOUT` 是当发生高风险事件时提高可观察性的好方法。

首先，让我们看看规则条目本身。这个规则使用了一些辅助条目、几个宏和列表——稍后我们会详细介绍这些内容：

```
- rule: Launch Privileged Container
  desc: Detect the initial process started in a privileged container. Exceptions are made for known trusted images.
  condition: >
    container_started and container
    and container.privileged=true
    and not falco_privileged_containers
    and not user_privileged_containers
  output: Privileged container started (user=%user.name command=%proc.cmdline %container.info image=%container.image.repository:%container.image.tag)
  priority: INFO
  tags: [container, cis, mitre_privilege_escalation, mitre_lateral_movement]
```

如你所见，Falco 规则有几个部分。首先，我们有规则的名称和描述。接下来，我们指定规则的触发条件——这充当 Linux 系统调用的过滤器。如果某个系统调用符合 `condition` 块中的所有逻辑过滤条件，则触发该规则。

当规则被触发时，`output` 键允许我们设置输出文本的显示格式。`priority` 键让我们为规则指定优先级，优先级可以是 `emergency`、`alert`、`critical`、`error`、`warning`、`notice`、`informational` 或 `debug` 之一。

最后，`tags` 键为相关规则添加标签，从而使规则分类更容易。这在使用并非单纯文本的 `STDOUT` 警报时尤其重要。

`condition` 的语法在这里尤为重要，我们将重点讲解这一过滤系统的工作原理。

首先，由于过滤器本质上是逻辑语句，如果你曾经编写过程序或伪代码，你会看到一些熟悉的语法——例如 if、and、not 等。这些语法非常简单易学，关于 *Sysdig* 过滤器语法的详细讨论可以在 [`github.com/draios/sysdig/wiki/sysdig-user-guide#filtering`](https://github.com/draios/sysdig/wiki/sysdig-user-guide#filtering) 中找到。

需要注意的是，Falco 开源项目最初是由*Sysdig*创建的，这也是为什么它使用了常见的*Sysdig*过滤器语法。

接下来，你会看到对`container_started`和`container`的引用，以及`falco_privileged_containers`和`user_privileged_containers`。这些并非普通字符串，而是宏的使用 —— 引用 YAML 中的其他块来指定附加功能，通常使得编写规则更加简便，无需重复大量配置。

为了更好地理解这个规则的工作原理，我们来看一下前面规则中引用的所有宏的完整参考：

```
- macro: container
  condition: (container.id != host)
- macro: container_started
  condition: >
    ((evt.type = container or
     (evt.type=execve and evt.dir=< and proc.vpid=1)) and
     container.image.repository != incomplete)
- macro: user_sensitive_mount_containers
  condition: (container.image.repository = docker.io/sysdig/agent)
- macro: falco_privileged_containers
  condition: (openshift_image or
              user_trusted_containers or
              container.image.repository in (trusted_images) or
              container.image.repository in (falco_privileged_images) or
              container.image.repository startswith istio/proxy_ or
              container.image.repository startswith quay.io/sysdig)
- macro: user_privileged_containers
  condition: (container.image.repository endswith sysdig/agent)
```

你会看到前面的 YAML 中，每个宏实际上只是一个可重用的`Sysdig`过滤器语法块，通常使用其他宏来实现规则功能。列表（此处未显示）类似于宏，但它们不描述过滤逻辑。相反，它们包含一组字符串值，可以作为比较的一部分，使用过滤语法进行处理。

例如，`(``trusted_images)` 在 `falco_privileged_containers` 宏中引用了一个名为`trusted_images`的列表。以下是该列表的来源：

```
- list: trusted_images
  items: []
```

正如你所看到的，默认规则中这个特定的列表是空的，但自定义规则集可以在此列表中使用受信任的镜像列表，随后所有使用`trusted_image`列表作为过滤规则的一部分的宏和规则都会自动使用该列表。

如前所述，除了跟踪 Linux 系统调用外，Falco 从 v0.13.0 版本开始还可以跟踪 Kubernetes 控制平面事件。

### 理解 Falco 中的 Kubernetes 审计事件规则

结构上，这些 Kubernetes 审计事件规则与 Falco 的 Linux 系统调用规则工作方式相同。以下是 Falco 中默认 Kubernetes 规则的一个示例：

```
- rule: Create Disallowed Pod
  desc: >
    Detect an attempt to start a pod with a container image outside of a list of allowed images.
  condition: kevt and pod and kcreate and not allowed_k8s_containers
  output: Pod started with container not in allowed list (user=%ka.user.name pod=%ka.resp.name ns=%ka.target.namespace images=%ka.req.pod.containers.image)
  priority: WARNING
  source: k8s_audit
  tags: [k8s]
```

该规则作用于 Falco 中的 Kubernetes 审计事件（本质上是控制平面事件），当创建一个不在`allowed_k8s_containers`列表中的 Pod 时，触发警报。默认的`k8s`审计规则包含许多类似的规则，其中大多数在触发时会输出格式化的日志。

如前所述，本章早些时候我们提到过 Pod 安全策略 —— 你可能会发现 PSP 和 Falco Kubernetes 审计事件规则之间有一些相似之处。例如，来看一下默认的 Kubernetes Falco 规则中的这条记录：

```
- rule: Create HostNetwork Pod
  desc: Detect an attempt to start a pod using the host network.
  condition: kevt and pod and kcreate and ka.req.pod.host_network intersects (true) and not ka.req.pod.containers.image.repository in (falco_hostnetwork_images)
  output: Pod started using host network (user=%ka.user.name pod=%ka.resp.name ns=%ka.target.namespace images=%ka.req.pod.containers.image)
  priority: WARNING
  source: k8s_audit
  tags: [k8s]
```

当 Pod 尝试使用主机网络启动时，这条规则会被触发，直接映射到主机网络的 PSP 设置。

Falco 利用这种相似性，让我们可以通过使用 Falco 来`试用`新的 Pod 安全策略，而无需将其应用于整个集群，从而避免影响正在运行的 Pod。

为此，`falcoctl`（Falco 的命令行工具）提供了`convert psp`命令。该命令接受一个 Pod 安全策略定义，并将其转化为一组 Falco 规则。这些 Falco 规则在触发时仅会将日志输出到`STDOUT`（而不是像 PSP 不匹配时导致 Pod 调度失败），这使得在现有集群中测试新的 Pod 安全策略变得更加容易。

要学习如何使用 `falcoctl` 转换工具，请参阅官方的 Falco 文档：[`falco.org/docs/psp-support/`](https://falco.org/docs/psp-support/)。

现在我们对 Falco 工具有了基本了解，让我们讨论它如何用于实施合规性控制和运行时安全性。

## 将 Falco 应用于合规性和运行时安全的使用案例

由于其可扩展性以及能够审计低级 Linux 系统调用，Falco 是一个非常适合持续合规性和运行时安全性的工具。

在合规性方面，可以利用专门映射到合规性标准要求的 Falco 规则集——例如 PCI 或 HIPAA。这使得用户能够迅速检测并对任何不符合相关标准的进程采取行动。有多个标准的开源和闭源 Falco 规则集可供使用。

类似地，对于运行时安全性，Falco 提供了一个警报/事件系统，这意味着任何触发警报的运行时事件也可以触发自动干预和修复过程。这对安全性和合规性都适用。举个例子，如果一个 Pod 触发了 Falco 警报，表示不符合规定，则可以根据该警报执行一个流程，并立即删除违规的 Pod。

# 总结

在本章中，我们了解了 Kubernetes 环境中的安全性。首先，我们回顾了 Kubernetes 安全的基础知识——哪些安全层与我们的集群相关，以及如何管理这一复杂性的一些大致方法。接着，我们了解了 Kubernetes 遇到的一些主要安全问题，并讨论了 2019 年安全审计的结果。

然后，我们在 Kubernetes 的两层堆栈中实施了安全性——首先，在配置中使用 Pod 安全策略和网络策略，最后，通过 Falco 实现运行时安全性。

在下一章中，我们将学习如何通过构建自定义资源来使 Kubernetes 适应自己的需求。这将使你能够为集群添加重要的新功能。

# 问题

1.  自定义准入控制器可以使用的两个 webhook 控制器的名称是什么？

1.  空白的 `NetworkPolicy` 对入站流量有什么影响？

1.  为了防止攻击者更改 Pod 功能，跟踪哪些类型的 Kubernetes 控制平面事件是有价值的？

# 进一步阅读

+   Kubernetes CVE 数据库：[`cve.mitre.org/cgi-bin/cvekey.cgi?keyword=kubernetes`](https://cve.mitre.org/cgi-bin/cvekey.cgi?keyword=kubernetes)
