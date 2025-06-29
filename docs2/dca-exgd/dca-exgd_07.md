Docker 内容信任简介

在本章中，我们将学习 Docker 内容信任概念及其相关工具。为了在 Docker 环境中提供受信任的内容，我们将使用 Docker 内容信任来加密应用于 Docker 对象的元数据。因此，任何未经授权的更改或对象操作都会被报告。如果没有发现这些问题，我们就能够确保环境中的所有对象都是受信任的。

首先，我们将介绍更新框架，然后学习如何签名镜像。之后，我们将学习如何验证签名，以确保其优先性和所有权。最后，我们将应用这些概念，在生产环境中运行一个受信任的环境。

本章将涵盖以下主题：

+   更新框架

+   签名镜像

+   审查签名

+   在受信任的环境中创建和运行应用程序

让我们开始吧！

# 技术要求

在本章中，我们将学习关于各种 Docker 内容信任的概念。在本章末尾，我们将提供一些实验，帮助你理解并学习所展示的概念。这些实验可以在你的笔记本电脑或 PC 上运行，使用提供的 Vagrant 独立环境，或者使用你自己部署的任何 Docker 主机。你可以在本书的 GitHub 代码仓库中找到更多信息：[`github.com/PacktPublishing/Docker-Certified-Associate-DCA-Exam-Guide.git`](https://github.com/PacktPublishing/Docker-Certified-Associate-DCA-Exam-Guide.git)。

查看以下视频，了解代码如何运作：

"[`bit.ly/3b0qviR`](https://bit.ly/3b0qviR)"

# 更新框架

在学习 **更新框架**（也称为 **TUF**）之前，我们将介绍一些概念。以下概念将帮助我们理解为什么我们需要工具来管理应用程序更新：

+   **软件更新系统**：软件更新系统是一种持续寻找新更新的应用程序。当找到更新时，它会触发过程以获取这些更新并安装这些更改。一个很好的例子是 Google Chrome 浏览器的更新系统。它会持续寻找其组件的更新，一旦发现更新，就会向我们显示一条消息：有一个新版本，是否现在更新？

+   **库包管理器**：库包管理器将管理和更新编程语言库及其依赖项。Python 的 **Python 包安装器**（**PIP**）和 Node.js 的 **Node 包管理器**（**NPM**）就是很好的例子。这些应用程序会查找库的更新并安装它们及其所需的依赖项。

+   **操作系统组件更新**：在这种情况下，不同的软件包管理器将管理所有软件更新及其依赖项，并在某些情况下触发前面提到的某些解决方案（软件更新系统或库包管理器）。

应用程序更新通常包含三个逻辑步骤：

1.  它会查找任何更新或更改。

1.  它下载更新。

1.  它将更改应用于我们的系统。

如果这些更新是恶意的，因为代码被攻击者截获并篡改了，会发生什么情况？

TUF 被创建来防止这些情况。它将处理应用程序更新的步骤，以确保下载的更改是受信任的。不会允许篡改的更改。TUF 元数据包括与受信任的密钥、加密哈希、文件、组件版本、创建和过期日期以及签名相关的信息。一个需要多个更新的应用程序无需管理此验证过程。它将请求 TUF 来管理这些过程。总而言之，我们可以这样说，TUF 提供了一种安全的方法来获取受信任的文件。

TUF 目前由 Linux 基金会托管，作为 **云原生计算基金会**（**CNCF**）的一部分。它是开源的，可以在生产环境中使用。建议与一些厂商工具结合使用，这样更容易管理和使用。

TUF 元数据为软件更新系统提供了有关更新真实性的信息。然后，该组件将做出正确的决定（安装或拒绝更新）。这些元数据信息将以 JSON 格式呈现。我们将讨论四个级别的签名，并将其称为角色：

+   **根元数据（`root.json`）和角色**：此角色与变更的所有者相关。它是顶级角色，其他角色将与该角色相关联。

+   **目标元数据（`targets.json`）和角色**：此角色与包中包含的文件相关。

+   **快照元数据（`snapshot.json`）和角色**：除了 `timestamp.json` 之外的所有文件都将在该角色中列出，以确保更新的一致性。

+   **时间戳元数据（`timestamp.json`）和角色**：这个标志将确保更新的确切日期，并且在检查更新时它是唯一需要的标志，例如。

更新应用程序使用 TUF 与仓库和文件来源进行交互，同时管理它们的更新。角色、受信任的密钥和目标文件不应包含在这些仓库中，因为它们将用于管理这些内容。

这个框架应该有一个客户端，以便我们可以在其正常使用中包含所描述的角色。因此，客户端必须管理以下内容：

+   所有可能的所有者的受信任根密钥，必须被信任

+   目标委托，当一个目标有多个所有者时

+   使用时间戳角色日期检查更新

+   所有签名过程

现在我们已经了解了使用 TUF 来管理仓库更新的好处，接下来让我们回顾一下它在 Docker 中的实现方式。

Docker 内容信任是 Docker 实现的 TUF。它通过 Notary 集成，Notary 是一个用于发布和管理受信任内容的开源工具。Docker 客户端提供了一个界面，允许我们签名并验证内容发布者。

Notary 是一个独立的软件；可以下载并用于检查包含在 Docker 仓库中的密钥。Docker 使用其库集成了 Notary。因此，每当我们在启用 Docker 内容信任时拉取镜像（默认情况下为禁用）时，Docker 守护进程将验证其签名。镜像拉取是通过其摘要进行的。镜像名称和标签将不再使用。这确保了只有正确的镜像会被下载。

Notary 的使用超出了本书的范围。在撰写时，它并不是通过 DCA 认证考试的必需内容。不过，建议阅读以下链接提供的 Notary 特性：[`docs.docker.com/notary/getting_started`](https://docs.docker.com/notary/getting_started)。

当我们使用 Docker 内容信任并推送镜像时，Docker 客户端会要求我们在所有描述的级别上进行签名（根级别、目标级别、快照级别和时间戳级别）。

总结来说，Docker 内容信任（Docker 的 TUF 实现）将执行以下操作：

+   确保镜像来源的可信度

+   在分发之前签署内容

+   确保主机上运行的所有内容都是可信的。

在下一节中，我们将学习如何签署并使用经过 Docker 引擎验证的签名镜像。

# 签署镜像

到目前为止，我们已经了解了不同的角色和将用于验证和信任镜像内容的元数据。让我们在进入 Docker 签名操作之前，快速总结一下：

+   根密钥将验证其他密钥。它签署 `root.json` 文件，该文件包含根、目标、快照和时间戳公钥的 ID 列表。为了验证内容签名，Docker 客户端将使用这些公钥。根密钥是离线的，必须保管好。镜像集合的所有者应该保管此密钥。不要丢失此密钥。你可以重新创建它，但所有签名的镜像将无效。

+   目标密钥签署 `targets.json` 文件，其中包含内容文件名的列表，以及它们的大小和哈希值。该文件用于将信任委派给团队中的其他用户，以便其他人可以签署相同的仓库。此密钥由管理员和集合（仓库）所有者持有。

+   委派密钥用于签署委派元数据文件。此密钥由管理员和所有能为指定集合做出贡献的人持有。

+   快照密钥签署 `snapshot.json` 元数据文件。该文件还包含文件名、以及根、目标和委派文件的大小和哈希值。此密钥由管理员和集合所有者持有。如果我们使用 Notary 服务，此密钥也可以由该服务持有，以允许集合协作者签署。此密钥代表当前的包签名。

+   时间戳密钥确保集合的新鲜度。它用于验证 `snapshot.json` 文件的完整性。因为这个密钥只在一定时间内有效，所以最好将其保存在 Notary 中。在这种情况下，所有者不必在每次密钥过期时重新创建密钥。Notary 将根据需要重新生成这个密钥。

现在，让我们使用 Docker 客户端签名一个镜像。

首先，我们将启用 Docker 内容信任。默认情况下，它是禁用的。我们可以为所有 Docker 命令启用它，或者每次想启用时添加一个参数。要为所有后续的 Docker 命令启用 Docker 内容信任，我们需要定义 `DOCKER_CONTENT_TRUST` 变量：

```
$ export DOCKER_CONTENT_TRUST=1
```

另外，我们可以为指定的命令启用 Docker 内容信任：

```
$ docker pull --disable-content-trust=false busybox:latest
```

我们在这里使用了 `--disable-content-trust=false`，因为默认情况下，Docker 内容信任是禁用的。

现在我们通过设置 `DOCKER_CONTENT_TRUST=1` 为当前会话中的所有命令启用了 Docker 内容信任，我们可以使用 `docker image pull` 拉取镜像：

```
$ export DOCKER_CONTENT_TRUST=1

$ docker image pull busybox
Using default tag: latest
Pull (1 of 1): busybox:latest@sha256:1303dbf110c57f3edf68d9f5a16c082ec06c4cf7604831669faf2c712260b5a0
sha256:1303dbf110c57f3edf68d9f5a16c082ec06c4cf7604831669faf2c712260b5a0: Pulling from library/busybox
0f8c40e1270f: Pull complete 
Digest: sha256:1303dbf110c57f3edf68d9f5a16c082ec06c4cf7604831669faf2c712260b5a0
Status: Downloaded newer image for busybox@sha256:1303dbf110c57f3edf68d9f5a16c082ec06c4cf7604831669faf2c712260b5a0
Tagging busybox@sha256:1303dbf110c57f3edf68d9f5a16c082ec06c4cf7604831669faf2c712260b5a0 as busybox:latest
docker.io/library/busybox:latest
```

请注意，`docker image pull` 命令的输出发生了变化。事实上，下载的镜像是通过其哈希值进行管理的；在这种情况下，哈希值是 `busybox@sha256:1303dbf110c57f3edf68d9f5a16c082ec06c4cf7604831669faf2c712260b5a`。

Docker 的官方镜像和认证镜像始终是已签名的。官方镜像由 Docker 管理和构建，位于 `docker.io/<REPOSITORY>:<TAG>` 下。

让我们使用 `docker container run` 来运行这个镜像，看看会发生什么：

```
$ docker container run -ti busybox sh
/ # ls 
bin dev etc home proc root sys tmp usr var
/ # touch NEW_FILE
/ # exit
```

按预期，它成功了。我们添加了一个文件，因为我们想要在提交容器的内容之前修改容器，以创建一个新的、不受信任的镜像。对于这个过程，我们将运行 `docker container commit`，并为命令设置 `DOCKER_CONTENT_TRUST=0`。我们之所以这么做，是因为在当前会话中已启用内容信任：

```
$ DOCKER_CONTENT_TRUST=0 docker container commit 3da3b341e904 busybox:untrusted
sha256:67a6ce66451aa10011d379e4628205889f459c06a3d7793beca10ecd6c21b68a
```

现在，我们有一个不受信任的 `busybox` 镜像。如果我们尝试执行这个镜像，会发生什么呢？

```
$ docker container run -ti busybox:untrusted sh
docker: No valid trust data for untrusted.
See 'docker run --help'.
```

我们不能运行这个镜像，因为它不受信任；它没有任何内容信任元数据。因此，它无法通过验证，并且不会被允许运行。如果启用了 Docker 内容信任，则未签名的镜像将不被允许。

让我们签名这个镜像。在这种情况下，我们将更改镜像名称并创建一个新的 `trusted` 标签。签名过程需要两个密码短语，正如这里所述：

1.  首先，我们将被要求设置一个 `root` 密码短语。你将被再次询问两次，以验证输入的密码（因为密码不会显示）。

1.  然后，你会被要求设置一个 `repository` 密码短语。你将被再次询问两次，以验证输入的密码（因为密码不会显示）。

我们被要求输入密码短语两次，因为这是第一次设置它们的值。下次我们使用这些密钥推送或拉取到这个仓库时，只会被要求一次（如果密码输入错误，可能会要求更多次）。让我们执行 `docker image push`：

```
$ docker image push frjaraur/mybusybox:trusted
The push refers to repository [docker.io/frjaraur/mybusybox]
0736ae522762: Pushed 
1da8e4c8d307: Mounted from library/busybox 
trusted: digest: sha256:e58e349eee38baa38f8398510c44e63a1f331dc1d80d4ed6010fe34960b9945f size: 734
Signing and pushing trust metadata
You are about to create a new root signing key passphrase. This passphrase
will be used to protect the most sensitive key in your signing system. Please
choose a long, complex passphrase and be careful to keep the password and the
key file itself secure and backed up. It is highly recommended that you use a
password manager to generate the passphrase and keep it safe. There will be no
way to recover this key. You can find the key in your config directory.
Enter passphrase for new root key with ID 6e03824: 
Repeat passphrase for new root key with ID 6e03824: 
Enter passphrase for new repository key with ID b302395: 
Repeat passphrase for new repository key with ID b302395: 
Finished initializing "docker.io/frjaraur/mybusybox"
Successfully signed docker.io/frjaraur/mybusybox:trusted
```

根密码短语非常重要。请妥善保管，因为如果丢失，你需要重新开始。如果发生这种情况，你之前签名的镜像将变得不受信任，你需要更新它们。如果丢失了密钥，你需要联系 Docker 支持（`support@docker.com`）来重置仓库状态。

你为根密钥和你的仓库选择的密码短语应该是强密码。建议使用随机生成的密码。

现在，我们拥有一个已签名的镜像。它归我们所有（在这个例子中，我是`frjaraur/mybusybox:trusted`的所有者）。

现在，我们可以使用`docker container run`来执行这个新签名的（因此是受信的）镜像：

```
$ docker container run -ti frjaraur/mybusybox:trusted
/ # touch OTHERFILE
/ # exit
```

要管理 Docker 内容信任，我们可以使用`docker trust`及其可用的操作。我们将能够管理密钥（加载和撤销）并签名镜像（该过程类似于前面描述的过程）。我们可以使用`docker trust inspect`来查看这些签名：

```
$ docker trust inspect --pretty docker.io/frjaraur/mybusybox:trusted

Signatures for docker.io/frjaraur/mybusybox:trusted

SIGNED TAG DIGEST SIGNERS
trusted e58e349eee38baa38f8398510c44e63a1f331dc1d80d4ed6010fe34960b9945f (Repo Admin)

Administrative keys for docker.io/frjaraur/mybusybox:trusted

 Repository Key: b3023954026f59cdc9be0b7ba093039353ce6e2d1a06c1338e4387689663abc0
 Root Key: e9120faa839a565838dbad7d45edd3c329893ae1f2085f225dc039272dec98ed
```

请注意，我们使用了`docker.io/frjaraur/mybusybox:trusted`而不是`frjaraur/mybusybox:trusted`。这是因为如果我们没有使用注册表的**完全限定域名**（**FQDN**）且镜像在本地存在，它将用于检索所有签名信息，你将收到`WARN[0006] Error while downloading remote metadata, using cached timestamp - this might not be the latest version available remotely`的消息，因为你将使用缓存的时间戳，而不是实际的时间戳。

现在我们已经学习了如何签署内容——在这种情况下是镜像——接下来让我们学习如何验证签名。

# 审查签名

Docker 客户端将与内容信任相关的文件存储在用户主目录下的`.docker/trust`目录中。

如果我们导航到受信目录下，我们将在`.docker/trust/tuf`中找到不同的注册表文件。我们在本章的示例中使用了 Docker Hub。因此，我们会找到`docker.io`注册表和不同的仓库。在你的环境中可能会有所不同，你可能会有更多的注册表或仓库，这取决于你何时开始在 Docker 主机上使用 Docker 内容信任。根据前面部分的示例，我们将在`.docker`目录下找到类似树形结构的目录：

```
trust/tuf/docker.io/frjaraur/mybusybox/metadata:
total 16
-rw------- 1 zero zero  494 nov 30 17:29 timestamp.json
-rw------- 1 zero zero  531 nov 30 17:28 targets.json
-rw------- 1 zero zero  682 nov 30 17:28 snapshot.json
-rw------- 1 zero zero 2417 nov 30 17:03 root.json
...
...
trust/tuf/docker.io/library/busybox/metadata:
total 28
-rw------- 1 zero zero 498 nov 30 17:17 timestamp.json
-rw------- 1 zero zero 13335 nov 30 16:41 targets.json
-rw------- 1 zero zero 688 nov 30 16:41 snapshot.json
-rw------- 1 zero zero 2405 nov 30 16:41 root.json
```

记住前面一节中描述的 JSON 文件。所有这些文件都位于每个注册表和仓库的结构下。

Docker 客户端会将你的密钥存储在`.docker/trust/private`目录下。妥善保管这些密钥非常重要。要备份这些密钥，可以使用`$ umask 077; tar -zcvf private_keys_backup.tar.gz ~/.docker/trust/private; umask 022`命令。

Notary 将帮助我们管理签名。它是一个开源的服务器和客户端应用程序，可以从其 GitHub 项目页面下载([`github.com/theupdateframework/notary`](https://github.com/theupdateframework/notary))。

Notary 可以安装在 Linux 或 Windows 主机上。

我们将使用`curl`命令下载最新版本，并修改其权限和路径：

```
$ curl -o /tmp/notary -sL https://github.com/theupdateframework/notary/releases/download/v0.6.1/notary-Linux-amd64

$ sudo mv /tmp/notary /usr/local/bin/notary

$ sudo chmod 755 /usr/local/bin/notary
```

在本节中，我们将使用 Docker 自己发布的 Notary 服务器，该服务器已在互联网上发布（[`notary.docker.io`](https://notary.docker.io)），并且与 Docker Hub 相关联。

Docker 企业版将在你的环境中运行自己的 Docker Notary 服务器实现。

让我们验证，例如，与 Docker Hub 仓库关联的所有签名。在这个例子中，我们正在查看`busybox`仓库。我们使用`notary list`，并提供适当的服务器和目录参数：

```
$ notary -s https://notary.docker.io -d ~/.docker/trust list docker.io/library/busybox
NAME DIGEST SIZE (BYTES) ROLE
---- ------ ------------ ----
1 1303dbf110c57f3edf68d9f5a16c082ec06c4cf7604831669faf2c712260b5a0 1864 targets
...
...
1.31.1-uclibc 817e459ca73c567e9132406bad78845aaf72d2e0c0965ff68861b318591e949a 1210 targets
buildroot-2013.08.1 c0a08c5e4c15c53f03323bae8e82fdfd9f4fccb7fd01b97579b19e3e3205915c 5074 targets
buildroot-2014.02 ced99ae82473e7dea723e6c467f409ed8f051bda04760e07fd5f476638c33507 5071 targets
glibc 0ec061426ef36bb28e3dbcd005f9655b6bfa0345f0d219c8eb330e2954f192ac 1638 targets
latest 1303dbf110c57f3edf68d9f5a16c082ec06c4cf7604831669faf2c712260b5a0 1864 targets
...
...
uclibc 817e459ca73c567e9132406bad78845aaf72d2e0c0965ff68861b318591e949a 1210 targets
```

我们列出了远程受信集合中的所有目标——在这个例子中，是 Docker Hub 上的`busybox`集合（`docker.io/library/busybox`）。

现在，让我们学习如何自动化这些过程，并确保安全性，以便在我们的组织中构建一个受信环境。

# 在受信环境中创建和运行应用程序

在本节中，我们将考虑一个受信环境，在其中`CONTENT_TRUST_ENABLED`会用于所有操作。这将确保在该环境中构建的镜像始终会被签名。所有已推送和拉取的镜像都会被签名，我们只会运行基于受信镜像的容器。

将 CI/CD 编排工具加入这些流程是非常有趣的。在没有某些系统或更高安全政策的情况下，禁止非受信内容并不容易。如果我们将`DOCKER_CONTENT_TRUST`的值设置为仅允许 Docker 内容信任，但用户可以直接与 Docker 主机交互，那么他们可以在命令行禁用此功能。

自动化在生产环境中至关重要，尽管 Docker 企业版提供了其他方法，我们将在第十二章中讨论，*通用控制平面*。Kubernetes 也提供了强制执行受信内容安全的功能，但这一话题超出了本书的范围。

使用外部 CI/CD，我们可以自动化构建、共享或部署 Docker 内容。让我们看一个简单的例子，展示如何构建和推送一个镜像：

```
$ export DOCKER_CONTENT_TRUST_ROOT_PASSPHRASE="MyVerySecureRootPassphraseForAutomation"
$ export DOCKER_CONTENT_TRUST_REPOSITORY_PASSPHRASE="MyVerySecureRepositoryPassphraseForAutomation"

$ docker build -t docker/trusttest:testing .
Using default tag: latest
latest: Pulling from docker/trusttest
b3dbab3810fc: Pull complete
a9539b34a6ab: Pull complete
Digest: sha256:d149ab53f871

$ docker push docker/trusttest:latest
The push refers to a repository [docker.io/docker/trusttest] (len: 1)
a9539b34a6ab: Image already exists
b3dbab3810fc: Image already exists
latest: digest: sha256:d149ab53f871 size: 3355
Signing and pushing trust metadata
```

我们可以编写一个脚本，用于 CI/CD 编排任务，使用`root`和`repository`密码短语来确保在构建和推送到我们的注册表时应用内容信任。我们可以使用相同的方法在生产环境中进行部署，禁止任何用户与这个安全环境进行交互。在脚本中处理密码短语的环境变量时要小心，因为它们将是可见的。CI/CD 编排工具将提供安全的管理方法。这样，你就能了解如何使用你自己的管理配置工具来实现一个安全的链条。

现在，让我们回顾一个实验，更好地理解我们在本章中所学的内容。

# 本章实验

我们现在将完成一个实验，帮助我们提升对所学概念的理解。

如果你还没有部署过 `environments/standalone-environment`，请从本书的 GitHub 仓库（[`github.com/PacktPublishing/Docker-Certified-Associate-DCA-Exam-Guide.git`](https://github.com/PacktPublishing/Docker-Certified-Associate-DCA-Exam-Guide.git)）部署它。你也可以使用你自己的 CentOS 7 服务器。从 `environments/standalone-environment` 文件夹中使用 `vagrant up` 启动你的虚拟环境。

如果你正在使用独立环境，请等它运行起来。我们可以使用 `vagrant status` 检查节点的状态。使用 `vagrant ssh standalone` 连接到你的实验节点。`standalone` 是你的节点名称。你将使用具有管理员权限的 `vagrant` 用户，通过 `sudo` 执行操作。你应该会看到以下输出：

```
Docker-Certified-Associate-DCA-Exam-Guide/environments/standalone$ vagrant up
Bringing machine 'standalone' up with 'virtualbox' provider...
...
Docker-Certified-Associate-DCA-Exam-Guide/environments/standalone$ vagrant status
Current machine states:
standalone running (virtualbox)
...
Docker-Certified-Associate-DCA-Exam-Guide/environments/standalone$
```

现在，我们可以使用 `vagrant ssh standalone` 连接到独立节点。如果你之前已经部署过独立虚拟节点并且刚刚使用 `vagrant up` 启动它，那么此过程可能会有所不同：

```
Docker-Certified-Associate-DCA-Exam-Guide/environments/standalone$ vagrant ssh standalone
[vagrant@standalone ~]$ 
```

如果你正在重复使用你的独立环境，这意味着 Docker Engine 已经安装。如果你启动了一个新的实例，请执行 `/vagrant/install_requirements.sh` 脚本，以确保你拥有所有必需的工具（Docker Engine 和 `docker-compose`）：

```
[vagrant@standalone ~]$ /vagrant/install_requirements.sh 
```

现在，你已经准备好开始实验了。

## 为 Docker Hub 签名镜像

首先，如果你还没有 Docker Hub 账户，请登录 [`hub.docker.com/signup`](https://hub.docker.com/signup) 创建一个账户。你可以使用你自己的注册中心，但你需要有一个 Notary 服务器在运行。让我们开始吧：

本实验将使用 Docker Hub 上的 `frjaraur/pingo` 仓库。你必须将 `frjaraur` 替换为你的用户名。

1.  在本实验中，我们将从头开始。如果你之前已经签名过镜像，别删除你自己的 `.docker/trust` 目录。在这种情况下，请将信任目录备份到一个安全的位置，以便稍后恢复，或者干脆在 Docker 主机系统中创建一个虚拟用户。为了创建这个备份，我们将执行 `cp -pR ~/.docker/trust ~/.docker/trust.BKP`。在这些实验完成后，你可以恢复它：

```
[vagrant@standalone ~]$ rm -rf ~/.docker/trust/
```

1.  现在，启用 Docker 内容信任并为本实验创建一个目录：

```
[vagrant@standalone ~]$ export DOCKER_CONTENT_TRUST=1

[vagrant@standalone ~]$ cd $HOME
[vagrant@standalone ~]$ mkdir chapter6
[vagrant@standalone ~]$ cd chapter6 
```

1.  我们准备了一个相当简单的 Dockerfile，它将 `ping` 命令发送到 `8.8.8.8`，并执行 `300` 次。如果你从本书的 GitHub 仓库下载了书本示例文件，可以在 `chapter6` 目录中找到这些实验文件。使用你的文本编辑器创建一个包含以下内容的 `Dockerfile` 文件：

```
FROM alpine:3.8 
RUN apk add --update curl 
CMD ping 8.8.8.8 -c 300 
```

1.  现在，我们可以构建镜像。记得已经启用了 Docker 内容信任（Docker Content Trust）。我们将在你编写 Dockerfile 的目录中使用 `docker image build`：

```
[vagrant@standalone chapter6]$ docker image build -t frjaraur/pingo:trusted .

Sending build context to Docker daemon 2.048kB
Step 1/3 : FROM alpine@sha256:04696b491e0cc3c58a75bace8941c14c924b9f313b03ce5029ebbc040ed9dcd9
sha256:04696b491e0cc3c58a75bace8941c14c924b9f313b03ce5029ebbc040ed9dcd9: Pulling from library/alpine
c87736221ed0: Pull complete 
Digest: sha256:04696b491e0cc3c58a75bace8941c14c924b9f313b03ce5029ebbc040ed9dcd9
Status: Downloaded newer image for alpine@sha256:04696b491e0cc3c58a75bace8941c14c924b9f313b03ce5029ebbc040ed9dcd9
 ---> dac705114996
Step 2/3 : RUN apk add --update curl
...
...
Successfully built b3aba563b2ff
Successfully tagged frjaraur/pingo:trusted
Tagging alpine@sha256:04696b491e0cc3c58a75bace8941c14c924b9f313b03ce5029ebbc040ed9dcd9 as alpine:3.8
```

你可能注意到来自 Docker 守护进程的新消息。守护进程使用了`alpine:3.8`镜像哈希值`sha256:04696b491e0cc3c58a75bace8941c14c924b9f313b03ce5029ebbc040ed9dcd9`，而不是镜像名称和标签。如果我们本地有相同`image:tag`值的镜像，它将被验证。如果哈希不匹配，它将被避免，真实镜像将从 Docker Hub 下载。这将确保下载的是受信任的`alpine:3.8`镜像。

1.  现在，我们将使用`docker trust sign`对这个镜像进行签名。此过程将要求我们创建一个`root`密码短语、一个`repository`密码短语和一个`user`密码短语（这是本章的新内容，因为我们在之前的章节中没有使用 Docker Content Trust）。这将创建一个新的`trust`目录在`.docker`*.* 当镜像被推送时，你将再次被询问关于注册用户密码短语的问题。这不是你的 Docker Hub 密码，而是你为执行签名操作创建的密码短语。我们将使用`docker trust sign`：

```
[vagrant@standalone chapter6]$ docker trust sign frjaraur/pingo:trusted
You are about to create a new root signing key passphrase. This passphrase
will be used to protect the most sensitive key in your signing system. Please
choose a long, complex passphrase and be careful to keep the password and the
key file itself secure and backed up. It is highly recommended that you use a
password manager to generate the passphrase and keep it safe. There will be no
way to recover this key. You can find the key in your config directory.
Enter passphrase for new root key with ID 9e788ed: 
Repeat passphrase for new root key with ID 9e788ed: 
Enter passphrase for new repository key with ID fb7b8fd: 
Repeat passphrase for new repository key with ID fb7b8fd: 
Enter passphrase for new frjaraur key with ID f1916d7: 
Repeat passphrase for new frjaraur key with ID f1916d7: 
Created signer: frjaraur
Finished initializing signed repository for frjaraur/pingo:trusted
Signing and pushing trust data for local image frjaraur/pingo:trusted, may overwrite remote trust data
The push refers to repository [docker.io/frjaraur/pingo]
6f02cc23eebe: Pushed 
d9ff549177a9: Mounted from library/alpine 
trusted: digest: sha256:478cd976c78306bbffd51a4b5055e28873697d01504e70ef85bddd9cc348450b size: 739 
Signing and pushing trust metadata 
Enter passphrase for frjaraur key with ID f1916d7: 
Successfully signed docker.io/frjaraur/pingo:trusted
```

1.  这样，镜像就已经签名并推送到 Docker Hub。我们可以使用`curl`验证镜像是否已上传：

```
[vagrant@standalone chapter6]$ curl -s https://hub.docker.com/v2/repositories/frjaraur/pingo/tags|jq
{
 "count": 1,
 "next": null,
 "previous": null,
 "results": [
 {
 "name": "trusted",
 "full_size": 4306493,
 "images": [
 {
 "size": 4306493,
 "digest": "sha256:478cd976c78306bbffd51a4b5055e28873697d01504e70ef85bddd9cc348450b",
 "architecture": "amd64",
 "os": "linux",
 "os_version": null,
 "os_features": "",
 "variant": null,
 "features": ""
 }
 ],
 "id": 78277337,
 "repository": 8106864,
 "creator": 380101,
 "last_updater": 380101,
 "last_updater_username": "frjaraur",
 "image_id": null,
 "v2": true,
 "last_updated": "2019-11-30T22:03:28.820429Z"
 }
 ]
}
```

1.  最后，我们将使用`docker trust inspect`查看镜像签名：

```
[vagrant@standalone chapter6]$ docker trust inspect --pretty frjaraur/pingo:trusted
Signatures for frjaraur/pingo:trusted
SIGNED TAG DIGEST SIGNERS
trusted 478cd976c78306bbffd51a4b5055e28873697d01504e70ef85bddd9cc348450b frjaraur
List of signers and their keys for frjaraur/pingo:trusted
SIGNER KEYS
frjaraur f1916d7ad60b
Administrative keys for frjaraur/pingo:trusted
Repository Key: fb7b8fdaa22738c44b927110c377aaa7c56a6a15e2fa0ebc554fe92a57b5eb0b
 Root Key: 4a739a076032b94a79c6d376721649c79917f4b5f8c8035ca11e36a0ed0696b4
```

现在，在我们看一些问题之前，让我们简要总结一下本章所涵盖的主题。

# 总结

Docker Content Trust 帮助我们确保容器环境中的内容安全，确保镜像的来源和受信任的内容。在生产环境中，能够确保任何正在运行的容器是由受信任内容生成的至关重要。如果无法验证镜像的安全性，则不应允许基于该镜像运行容器。

我们已经了解到，Content Trust 通过四个基本密钥提高了 Docker 仓库的安全性。根密钥确保所有权，目标密钥允许验证特定集合或仓库中的内容。这些密钥将受到密码短语的保护，签名时会要求我们输入这些密码。快照和时间戳密钥不需要用户交互，将自动生成，以保证内容密钥文件以及签名镜像的日期和过期时间。

在下一章，我们将介绍编排的概念。我们将回顾管理基于容器的应用程序在分布式环境中的所有功能。

# 问题

1.  以下哪句话是不正确的？

a) Docker Content Trust 基于 TUF。

b) TUF（信任更新框架）被开发用来确保软件更新过程。

c) 无法验证新的软件版本。

d) 以上所有陈述都是错误的。

1.  以下哪项名称表示用于验证镜像内容的 Docker Content Trust 密钥？

a) 目标

b) 用户

c) 组

d) 时间戳

1.  我们如何确保`busybox:latest`发布的镜像确实是最新的？

a) 我们无法确保镜像的最新性。

b) `busybox:latest`表示这是创建的最新镜像。

c) 内容信任将验证镜像的新鲜度；因此，我们可以确保主机确实执行的是`busybox:latest`镜像，尽管我们无法确保它是最新的。

d) 之前的所有陈述都是错误的。

1.  为什么我们在尝试签署`busybox:trusted`时会遇到`denied: requested access to the resource is denied`错误？

a) 此镜像不存在。

b) 我们不被允许修改该仓库。

c) Docker 内容信任可能没有启用。

d) 之前的所有内容。

1.  我们丢失了根密钥，因为我们更换了笔记本电脑。以下哪一项陈述是正确的？

a) 如果我们在`.docker/trust/private`下没有密钥，在签名时会生成一个新的密钥。

b) 如果我们进行备份，我们可以恢复私钥根密钥。

c) 如果我们生成了新的密钥，我们的旧镜像将变得不可信，并且我们需要重新签名它们。

d) 之前的所有陈述都是正确的。

# 进一步阅读

您可以参考以下链接，获取有关本章所涵盖主题的更多信息：

+   TUF: [`theupdateframework.io/`](https://theupdateframework.io/)

+   TUF 规范: [`github.com/theupdateframework/specification`](https://github.com/theupdateframework/specification)

+   Notary: [`github.com/theupdateframework/notary`](https://github.com/theupdateframework/notary)

+   Docker 内容信任: [`docs.docker.com/engine/security/trust/content_trust/`](https://docs.docker.com/engine/security/trust/content_trust/)
