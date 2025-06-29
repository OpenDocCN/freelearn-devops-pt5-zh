# 第六章： Kubernetes 应用程序配置

本章描述了 Kubernetes 提供的主要配置工具。我们将从讨论一些将配置注入容器化应用程序的最佳实践开始。接下来，我们将讨论 ConfigMaps，这是一种 Kubernetes 资源，旨在为应用程序提供配置数据。最后，我们将介绍 Secrets，这是在 Kubernetes 上存储和提供敏感数据给应用程序的一种安全方式。总的来说，本章将为您提供一套用于配置生产环境中 Kubernetes 应用程序的绝佳工具。

本章将涵盖以下主题：

+   使用最佳实践配置容器化应用程序

+   实现 ConfigMaps

+   使用 Secrets

# 技术要求

为了运行本章中详细说明的命令，您需要一台支持 `kubectl` 命令行工具的计算机，以及一个正常运行的 Kubernetes 集群。请查看 *第一章*，*与 Kubernetes 通信*，了解快速启动 Kubernetes 的几种方法，以及如何安装 `kubectl` 工具的说明。

本章使用的代码可以在本书的 GitHub 仓库中找到，链接地址为 [`github.com/PacktPublishing/Cloud-Native-with-Kubernetes/tree/master/Chapter6`](https://github.com/PacktPublishing/Cloud-Native-with-Kubernetes/tree/master/Chapter6)。

# 使用最佳实践配置容器化应用程序

到目前为止，我们已经了解了如何有效地部署（见 *第四章*，*扩展与部署您的应用程序*）和暴露（见 *第五章*，*服务与 Ingress* – *与外界通信*）容器化应用程序在 Kubernetes 上。这足以在 Kubernetes 上运行非平凡的无状态容器化应用程序。然而，Kubernetes 还提供了更多的工具来进行应用程序配置和 Secrets 管理。

由于 Kubernetes 运行容器，您可以始终配置您的应用程序使用嵌入在 Dockerfile 中的环境变量。但这绕过了像 Kubernetes 这样的编排工具的某些实际价值。我们希望能够在不重建 Docker 镜像的情况下更改我们的应用程序容器。为此，Kubernetes 为我们提供了两个关注配置的资源：ConfigMaps 和 Secrets。我们首先来看一下 ConfigMaps。

## 理解 ConfigMaps

在生产环境中运行应用程序时，开发人员希望能够快速、轻松地注入应用程序配置。实现这一目标的方式有很多——从使用一个单独的配置服务器进行查询，到使用环境变量或环境文件。这些策略在安全性和可用性方面各有不同。

对于容器化应用程序，环境变量通常是最简便的方式——但以安全的方式注入这些变量可能需要额外的工具或脚本。在 Kubernetes 中，ConfigMap 资源让我们可以以灵活、简便的方式做到这一点。ConfigMap 允许 Kubernetes 管理员指定并注入配置数据，可以是文件，也可以是环境变量。

对于像密钥这样的高度敏感信息，Kubernetes 提供了另一种类似的资源——Secrets。

## 理解 Secrets

Secrets 是指需要以稍微更安全的方式存储的额外应用配置项——例如，限制 API 的主密钥、数据库密码等。Kubernetes 提供了一种叫做 Secret 的资源，它以编码方式存储应用配置数据。这本身并不使 Secret 更加安全，但 Kubernetes 通过不自动在 `kubectl get` 或 `kubectl describe` 命令中打印出秘密信息来尊重保密的概念。这样可以防止 Secret 被意外记录到日志中。

为了确保 Secrets 确实保密，必须在集群上启用静态加密来保护 secret 数据——我们将在本章稍后讨论如何做到这一点。自 Kubernetes 1.13 起，这个功能允许 Kubernetes 管理员防止 Secrets 在 `etcd` 中以未加密形式存储，并限制 `etcd` 管理员的访问权限。

在深入讨论 Secrets 之前，我们先来讨论一下 ConfigMaps，它更适合存储非敏感信息。

# 实现 ConfigMap

ConfigMap 提供了一种简单的方式来存储和注入运行在 Kubernetes 上的容器的应用配置数据。

创建 ConfigMap 很简单——它们提供了两种方式来注入应用配置数据：

+   作为环境变量注入

+   作为文件注入

虽然第一个选项只是通过容器环境变量在内存中操作，但后一个选项涉及到某些卷的概念——卷是 Kubernetes 的一种存储介质，将在下一章中进行详细介绍。我们现在简要回顾一下，并将其作为对卷的介绍，卷的内容将在下一章 *第七章* *Kubernetes 存储* 中展开。

在使用 ConfigMap 时，通过命令式的 `Kubectl` 命令来创建它可能会更简单。创建 ConfigMap 有几种可能的方法，这也会导致数据存储和访问方式的不同。第一种方式是直接从文本值创建，如我们接下来将看到的。

## 来自文本值

通过命令从文本值创建 ConfigMap 的方法如下：

```
kubectl create configmap myapp-config --from-literal=mycategory.mykey=myvalue 
```

上述命令创建了一个名为 `myapp-config` 的 `configmap`，它有一个键，叫做 `mycategory.mykey`，值为 `myvalue`。你也可以创建一个包含多个键值对的 ConfigMap，如下所示：

```
kubectl create configmap myapp-config2 --from-literal=mycategory.mykey=myvalue
--from-literal=mycategory.mykey2=myvalue2 
```

前面的命令将导致 ConfigMap 在 `data` 部分中包含两个值。

要查看你的 ConfigMap，运行以下命令：

```
kubectl get configmap myapp-config2
```

你将看到以下输出：

configmap-output.yaml

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: myapp-config2
  namespace: default
data:
  mycategory.mykey: myvalue
  mycategory.mykey2: myvalue2
```

当你的 ConfigMap 数据较长时，直接从文本值创建它并不太合适。对于较长的配置，我们可以从文件中创建 ConfigMap。

## 从文件中

为了更方便地创建包含多种不同值的 ConfigMap，或重用已有的环境文件，你可以通过以下步骤从文件中创建 ConfigMap：

1.  让我们从创建文件开始，我们将其命名为 `env.properties`：

    ```
    myconfigid=1125
    publicapikey=i38ahsjh2
    ```

1.  然后，我们可以通过运行以下命令来创建我们的 ConfigMap：

    ```
    kubectl create configmap my-config-map --from-file=env.properties
    ```

1.  为了检查我们的 `kubectl create` 命令是否正确创建了 ConfigMap，我们可以使用 `kubectl describe` 来描述它：

    ```
    kubectl describe configmaps my-config-map
    ```

这应该会输出以下内容：

```
Name:           my-config-map
Namespace:      default
Labels:         <none>
Annotations:    <none>
Data
====
env.properties:        39 bytes
```

如你所见，这个 ConfigMap 包含了我们的文本文件（以及字节数）。我们的文件在此情况下可以是任何文本文件——但是如果你知道你的文件已经正确格式化为环境文件，你可以告诉 Kubernetes，这样可以让你的 ConfigMap 更容易阅读。让我们来学习如何做到这一点。

## 从环境文件中

如果我们知道我们的文件格式是普通的环境文件，包含键值对，那么我们可以使用稍微不同的方法来创建 ConfigMap——环境文件方法。此方法会使数据在 ConfigMap 对象中更加直观，而不是隐藏在文件内部。

让我们使用之前完全相同的文件，进行环境特定的创建：

```
kubectl create configmap my-env-config-map --from-env-file=env.properties
```

现在，让我们使用以下命令来描述我们的 ConfigMap：

```
> kubectl describe configmaps my-env-config-map
```

我们得到以下输出：

```
Name:         my-env-config-map
Namespace:    default
Labels:       <none>
Annotations:  <none>
Data
====
myconfigid:
----
1125
publicapikey:
----
i38ahsjh2
Events:  <none>
```

如你所见，使用 `-from-env-file` 方法，`env` 文件中的数据在运行 `kubectl describe` 时很容易查看。这也意味着我们可以直接将 ConfigMap 挂载为环境变量——稍后会详细说明。

## 将 ConfigMap 挂载为卷

要在 Pod 中使用 ConfigMap 中的数据，你需要将其挂载到 Pod 的配置中。这与在 Kubernetes 中挂载卷的方式非常相似（我们稍后会明白这样做的原因），卷是一种提供存储的资源。不过现在，不必担心卷的问题。

让我们来看一下我们的 Pod 配置，它将 `my-config-map` ConfigMap 挂载为 Pod 上的卷：

pod-mounting-cm.yaml

```
apiVersion: v1
kind: Pod
metadata:
  name: my-pod-mount-cm
spec:
  containers:
    - name: busybox
      image: busybox
      command:
      - sleep
      - "3600"
      volumeMounts:
      - name: my-config-volume
        mountPath: /app/config
  volumes:
    - name: my-config-volume
      configMap:
        name: my-config-map
  restartPolicy: Never
```

如你所见，我们的 `my-config-map` ConfigMap 被挂载为卷（`my-config-volume`），并通过 `/app/config` 路径供容器访问。我们将在下一个关于存储的章节中进一步了解它是如何工作的。

在某些情况下，你可能希望将 ConfigMap 作为环境变量挂载到容器中——我们将在接下来学习如何做到这一点。

## 将 ConfigMap 挂载为环境变量

你也可以将 ConfigMap 作为环境变量进行挂载。这个过程与将 ConfigMap 挂载为卷非常相似。

让我们来看一下我们的 Pod 配置：

pod-mounting-cm-as-env.yaml

```
apiVersion: v1
kind: Pod
metadata:
  name: my-pod-mount-env
spec:
  containers:
    - name: busybox
      image: busybox
      command:
      - sleep
      - "3600"
      env:
        - name: MY_ENV_VAR
          valueFrom:
            configMapKeyRef:
              name: my-env-config-map
              key: myconfigid
  restartPolicy: Never
```

如你所见，我们没有将 ConfigMap 挂载为卷，而是通过容器环境变量 `MY_ENV_VAR` 来引用它。为了做到这一点，我们需要在 `valueFrom` 键中使用 `configMapRef`，并引用我们的 ConfigMap 的名称以及要查看的键。

如我们在本章开始时的 *使用最佳实践配置容器化应用程序* 部分所提到的，ConfigMaps 默认不安全，它们的数据以明文形式存储。为了增加安全性，我们可以使用 Secrets 替代 ConfigMaps。

# 使用 Secrets

Secrets 的工作方式与 ConfigMaps 非常相似，不同之处在于它们以编码文本（具体来说是 Base64）而非明文形式存储。

因此，创建 Secret 与创建 ConfigMap 非常相似，但有一些关键的不同之处。首先，命令式创建 Secret 会自动对 Secret 中的数据进行 Base64 编码。首先，让我们看看如何通过一对文件命令式地创建 Secret。

## 从文件

首先，让我们尝试从文件创建一个 Secret（这对于多个文件也有效）。我们可以使用 `kubectl create` 命令来实现：

```
> echo -n 'mysecretpassword' > ./pass.txt
> kubectl create secret generic my-secret --from-file=./pass.txt
```

这应该会产生以下输出：

```
secret "my-secret" created
```

现在，让我们通过 `kubectl describe` 查看我们的 Secret 长什么样：

```
> kubectl describe secrets/db-user-pass
```

这个命令应该会产生以下输出：

```
Name:            my-secret
Namespace:       default
Labels:          <none>
Annotations:     <none>
Type:            Opaque
Data
====
pass.txt:    16 bytes
```

如你所见，`describe` 命令显示了 Secret 中包含的字节数，以及它的类型 `Opaque`。

创建 Secret 的另一种方式是使用声明式方法手动创建它。接下来让我们看看如何操作。

## 手动声明式方法

当从 YAML 文件声明式地创建 Secret 时，您需要使用编码工具（如 Linux 上的 `base64` 管道）预先对要存储的数据进行编码。

让我们在这里使用 Linux 的 `base64` 命令对我们的密码进行编码：

```
> echo -n 'myverybadpassword' | base64
bXl2ZXJ5YmFkcGFzc3dvcmQ=
```

现在，我们可以使用 Kubernetes YAML 规格声明性地创建我们的 Secret，并将其命名为 `secret.yaml`：

```
apiVersion: v1
kind: Secret
metadata:
  name: my-secret
type: Opaque
data:
  dbpass: bXl2ZXJ5YmFkcGFzc3dvcmQ=
```

我们的 `secret.yaml` 规格包含我们创建的 Base64 编码字符串。

要创建 Secret，请运行以下命令：

```
kubectl create -f secret.yaml
```

现在你已经知道如何创建 Secrets。接下来，让我们学习如何将 Secret 挂载供 Pod 使用。

## 将 Secret 挂载为卷

挂载 Secrets 与挂载 ConfigMaps 非常相似。首先，让我们看看如何将 Secret 作为卷（文件）挂载到 Pod。

让我们看一下我们的 Pod 规格。在这个例子中，我们运行一个示例应用程序来测试我们的 Secret。这里是 YAML：

pod-mounting-secret.yaml

```
apiVersion: v1
kind: Pod
metadata:
  name: my-pod-mount-cm
spec:
  containers:
    - name: busybox
      image: busybox
      command:
      - sleep
      - "3600"
      volumeMounts:
      - name: my-config-volume
        mountPath: /app/config
        readOnly: true
  volumes:
    - name: foo
      secret:
      secretName: my-secret
  restartPolicy: Never
```

与 ConfigMap 的区别在于，我们在卷上指定了 `readOnly`，以防止 Pod 运行时对 Secret 进行任何更改。其他方面与挂载 Secret 的方式相同。

同样，在下一章节*第七章*，*在 Kubernetes 上存储*中，我们将深入研究卷。

## 将 Secret 作为环境变量挂载

类似于文件挂载，我们可以像 ConfigMap 挂载一样将我们的 Secret 作为环境变量挂载。

让我们再看一个 Pod YAML。在这种情况下，我们将把我们的 Secret 作为环境变量挂载：

pod-mounting-secret-env.yaml

```
apiVersion: v1
kind: Pod
metadata:
  name: my-pod-mount-env
spec:
  containers:
    - name: busybox
      image: busybox
      command:
      - sleep
      - "3600"
      env:
        - name: MY_PASSWORD_VARIABLE
          valueFrom:
            secretKeyRef:
              name: my-secret
              key: dbpass
  restartPolicy: Never
```

在使用 `kubectl apply` 创建了前述 Pod 后，让我们运行一个命令来查看我们的 Pod 是否正确初始化了该变量。这与 `docker exec` 的操作方式完全相同：

```
> kubectl exec -it my-pod-mount-env -- /bin/bash
> printenv MY_PASSWORD_VARIABLE
myverybadpassword
```

它起作用了！您现在应该对如何创建、挂载和使用 ConfigMaps 和 Secrets 有了很好的理解。

作为关于 Secrets 的最后一个话题，我们将学习如何使用 Kubernetes 的 `EncryptionConfig` 创建安全的加密 Secrets。

## 实现加密 Secrets

几个托管的 Kubernetes 服务（包括亚马逊的 `etcd` 数据在静止时 - 因此您不需要执行任何操作来实现加密 Secrets。集群提供者如 Kops 有一个简单的标志（例如 `encryptionConfig: true`）。但是，如果您采用 *硬核方式* 创建集群，则需要使用标志 `--encryption-provider-config` 和一个 `EncryptionConfig` 文件启动 Kubernetes API 服务器。

重要提示

完全从头开始创建一个集群超出了本书的范围（请查看 *Kubernetes The Hard Way*，其中有一个很好的指南，网址为 [`github.com/kelseyhightower/kubernetes-the-hard-way`](https://github.com/kelseyhightower/kubernetes-the-hard-way))。

想要快速了解加密如何处理，请查看以下 `EncryptionConfiguration` YAML，它在启动时传递给 `kube-apiserver`：

encryption-config.yaml

```
apiVersion: apiserver.config.k8s.io/v1
kind: EncryptionConfiguration
resources:
  - resources:
    - secrets
    providers:
    - aesgcm:
        keys:
        - name: key1
          secret: c2VjcmV0IGlzIHNlY3VyZQ==
        - name: key2
          secret: dGhpcyBpcyBwYXNzd29yZA==
```

前述的 `EncryptionConfiguration` YAML 获取了应在 `etcd` 中加密的资源列表，以及可用于加密数据的一个或多个提供者。截至 Kubernetes `1.17`，允许使用以下提供者：

+   **Identity**：无加密。

+   **Aescbc**：推荐的加密提供者。

+   **Secretbox**：比 Aescbc 更快，也更新。

+   **Aesgcm**：请注意，您需要自行实现 Aesgcm 的密钥轮换。

+   **Kms**：与 Vault 或 AWS KMS 等第三方 Secrets 存储一起使用。

若要查看完整列表，请参阅 https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/#providers。当列表中添加多个提供者时，Kubernetes 将使用第一个配置的提供者来加密对象。在解密时，Kubernetes 将逐个尝试使用每个提供者解密 - 如果都不起作用，将返回错误。

一旦我们创建了一个 secret（可以参考我们之前的任何示例），并且我们的 `EncryptionConfig` 是激活的，我们就可以检查我们的 Secrets 是否真正加密了。

## 检查你的 Secrets 是否已加密

检查你的 secret 是否真正加密在 `etcd` 中的最简单方法是直接从 `etcd` 获取值并检查加密前缀：

1.  首先，让我们使用 `base64` 创建一个 secret 密钥：

    ```
    > echo -n 'secrettotest' | base64
    c2VjcmV0dG90ZXN0
    ```

1.  创建一个名为 `secret_to_test.yaml` 的文件，内容如下：

    ```
    apiVersion: v1
    kind: Secret
    metadata:
     name: secret-to-test
    type: Opaque
    data:
      myencsecret: c2VjcmV0dG90ZXN0
    ```

1.  创建 Secret：

    ```
    kubectl apply -f secret_to_test.yaml
    ```

1.  创建好 Secret 后，让我们通过直接查询它来检查它是否已在 `etcd` 中加密。你不应该经常直接查询 `etcd`，但如果你有访问用于引导集群的证书，这是一个简单的过程：

    ```
    k8s:enc:kms:v1:azurekmsprovider.
    ```

1.  现在，检查一下 Secret 是否已经正确解密（它仍然是编码过的），通过 `kubectl` 来查看：

    ```
    > kubectl get secrets secret-to-test -o yaml
    ```

输出应该是 `myencsecret: c2VjcmV0dG90ZXN0`，这是我们未加密的、编码后的 Secret 值：

```
> echo 'c2VjcmV0dG90ZXN0' | base64 --decode
> secrettotest
```

成功！

我们现在已经在集群中运行加密了。让我们找出如何移除它。

## 禁用集群加密

我们也可以相对容易地从我们的 Kubernetes 资源中移除加密。

首先，我们需要使用一个空白的加密配置 YAML 文件重新启动 Kubernetes API 服务器。如果你是自行搭建的集群，这应该比较容易，但在 EKS 或 AKS 上，手动操作是不可能的。你需要按照云服务提供商的特定文档来禁用加密。

如果你是自行搭建集群或使用了 Kops 或 Kubeadm 等工具，那么你可以在所有主节点上重新启动 `kube-apiserver` 进程，并使用以下 `EncryptionConfiguration`：

encryption-reset.yaml

```
apiVersion: apiserver.config.k8s.io/v1
kind: EncryptionConfiguration
resources:
  - resources:
    - secrets
    providers:
    - identity: {}
```

重要提示

注意，身份提供者不需要是唯一列出的提供者，但它需要是第一个，因为正如我们之前提到的，Kubernetes 使用第一个提供者来加密 `etcd` 中的新对象或更新对象。

现在，我们将手动重新创建所有的 Secrets，到时它们将自动使用身份提供者（未加密）：

```
kubectl get secrets --all-namespaces -o json | kubectl replace -f -
```

此时，我们所有的 Secrets 都是未加密的！

# 总结

在本章中，我们回顾了 Kubernetes 提供的注入应用配置的方法。首先，我们回顾了一些配置容器化应用的最佳实践。接着，我们复习了 Kubernetes 提供的第一个方法——ConfigMaps，并介绍了如何创建和挂载它们到 Pods。最后，我们介绍了 Secrets，在加密后，它们是一种更安全的方式来处理敏感配置。现在，你应该已经掌握了为应用提供安全和不安全配置值所需的所有工具。

在下一章中，我们将深入探讨一个我们已经触及过的主题——挂载 Secrets 和 ConfigMaps——Kubernetes 卷资源，以及更广泛的 Kubernetes 存储。

# 问题

1.  Secrets 和 ConfigMaps 之间有哪些区别？

1.  Secrets 是如何编码的？

1.  从普通文件创建 ConfigMap 和从环境文件创建 ConfigMap 之间的主要区别是什么？

1.  如何在 Kubernetes 上保证 Secrets 的安全？为什么它们默认情况下不安全？

# 进一步阅读

+   有关 Kubernetes 数据加密配置的信息可以在官方文档中找到，网址是 [`kubernetes.io/docs/tasks/administer-cluster/encrypt-data/`](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/)。
