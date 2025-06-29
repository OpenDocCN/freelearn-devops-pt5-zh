# 第十五章：*第十四章*：高级 Docker 安全性——秘密、秘密命令、标记和标签

到目前为止，我们已经看到多个需要使用包含秘密数据的文件的例子。我们可以将“秘密”视为一个通用术语，指的是通常存储在配置文件和环境文件中的敏感数据类型，如数据库访问凭证或 API 令牌。Docker 提供了一种方便的方法来保护这类数据并进行共享。对于使用 Swarm 模式而非 Kubernetes 的遗留系统，了解如何在这些环境中应用安全措施非常重要，因为在你的职业生涯中，可能需要对这些环境进行后期修复。

除了管理秘密数据外，我们还可以使用标签和标记来帮助确保我们在进行安全性工作时考虑到这些因素。你在上一章中已经看到了标记，我们将在本章后续内容中进一步探讨这些。

此外，我们还将探讨如何使用元数据标签提供容器的额外信息，以及如何使用 `security.txt` 文件。

在本章中，我们将涵盖以下主要主题：

+   在 Docker 中安全存储秘密的介绍

+   什么是秘密以及我们为什么需要它们

+   遍历 Raft 日志文件

+   向 Swarm 中添加、编辑和移除秘密

+   更有效地使用标签，确保我们使用安全的镜像

+   实现元数据标签和 secrets.txt 文件

让我们从了解 Docker 秘密是什么以及它们为什么有益开始。

# 技术要求

本章，你需要访问一台运行 Docker 的 Linux 机器。我们建议你使用本书中到目前为止所使用的设置。

此外，你还需要一个 Docker Hub 账户，以便访问那里的镜像。如果你还没有设置，可以通过以下链接进行设置：

[`hub.docker.com`](https://hub.docker.com)

最后，为了探索使用 Docker 秘密，你需要设置至少两个容器并使用 Docker 的 Swarm 功能。你可以在这里了解更多关于 Swarm 模式的信息：[`docs.docker.com/engine/swarm/`](https://docs.docker.com/engine/swarm/)

查看以下视频，看看代码是如何运行的：

[`bit.ly/3iDsjkA`](https://bit.ly/3iDsjkA)

# 在 Docker 中安全存储秘密

处理复杂的网络化软件项目时，不可避免的一部分就是要处理秘密数据。这可以包括像 SSH 访问的私钥、SSL 证书、密码和 API 密钥等各种数据。

为了在多个容器之间安全共享秘密，当然，你需要避免尝试将秘密存储在容器本身中，避免以允许潜在攻击者访问的方式存储秘密。这个抽象层不仅对于根据环境管理不同的凭证集有用，而且还提供了一层额外的安全保障，以防容器以某种方式被破坏。

幸运的是，Docker 提供了一个有用的功能来实现这一目标，名为 Docker secrets。要使用这个功能或 Kubernetes 对应的功能，你需要实现 Swarm 服务或 Kubernetes 本身。正如我们在本书的其他地方所建议的，如果可能的话，你可能希望避免使用 Swarm 服务，转而使用 Kubernetes。然而，在处理使用 Swarm 的遗留系统时，你可能需要与之合作，因此理解在这种情况下如何使用 secrets 是很重要的。考虑到这一点，容器应作为服务运行。

Mirantis 在 2020 年 2 月收购 Docker 后，承诺对 Docker Swarm 提供长期支持（[`www.mirantis.com/blog/mirantis-will-continue-to-support-and-develop-docker-swarm/`](https://www.mirantis.com/blog/mirantis-will-continue-to-support-and-develop-docker-swarm/)）。你可能已经在 *第五章*《*在生产环境中部署和运行容器的替代方案*》中熟悉过这个概念；然而，如果你需要复习，可以按照 Docker 网站上提供的步骤开始使用 Swarm 模式，作为 Kubernetes 的替代方案：

[`docs.docker.com/engine/swarm/swarm-tutorial/`](https://docs.docker.com/engine/swarm/swarm-tutorial/)

Swarm 和 Kubernetes 中的机密功能允许你集中管理数据（如密码和 API 密钥），然后将其安全地共享给你选择的容器。这避免了在容器内不安全地硬编码值，或者让所有容器都能访问敏感数据。

此外，当通过 Docker secrets 与 Swarm 中的其他容器共享机密时，这些机密会通过 SSL/TLS 加密的安全连接进行传输。现在，让我们更深入地了解 Docker secrets 如何在基础层面上工作，包括一个名为 Raft 日志的重要功能。

## Raft 日志

为了在 Swarm 节点之间共享内容，我们需要确保既有共识又有容错能力。简而言之，这意味着网络中的所有节点都需就某一组值达成一致，以保持一致的状态。

Docker Swarm 使用的算法叫做 Raft。你可以在论文《*寻找可理解的共识算法*》中阅读更多技术细节，论文可在 Raft 的 GitHub 账户上找到：

[`raft.github.io/raft.pdf`](https://raft.github.io/raft.pdf)

Docker Swarm 使用一个称为 Raft 日志文件的文件，作为其实现算法的一部分。这个文件的好处在于它可以用于存储机密数据，这些数据随后需要在 1 到 *n* 个节点之间共享。当通过 `docker secret` 命令添加机密时，一个值会被添加到 Raft 日志文件中，然后通过临时文件系统提供，如下所示的示例：

```
/run/secrets/apikey
```

本质上，这就是如何在 Docker Swarm 中的多个容器之间共享一个秘密。应用程序读取秘密的方式取决于你使用的语言。例如，如果你在修改 ShipIt Clicker 应用程序，你会使用 JavaScript。如果我们有一个 API 密钥文件之类的秘密，我们可以直接在 JavaScript 源代码中使用 `fs` 模块访问它，如以下示例所示：

```
fs.readFile('/run/secrets/apikey', 'utf8')
```

如你所见，这是一种相当简单的方法。

尽管此文件是加密的，但我们也可以通过锁定提供额外的安全层。

可以使用 `--autolock` 标志锁定 Swarm，以防止攻击者解密 Raft 日志文件。

请参阅 Docker 文档以获取更多详细信息：

[`docs.docker.com/engine/swarm/swarm_manager_locking/`](https://docs.docker.com/engine/swarm/swarm_manager_locking/)

现在你已经基本了解了 Docker secrets 功能的工作原理，我们来看看如何使用它。

# 添加、查看和删除秘密

现在我们将开始探索与 secrets 相关的各种命令。

如果你愿意尝试 Kubernetes 等效的命令，也可以替换本节中的命令。你可以在 [`kubernetes.io/docs/concepts/configuration/secret/`](https://kubernetes.io/docs/concepts/configuration/secret/) 上找到 `kubectl` 命令的列表。

或者你可以参考 *第八章*，*将 Docker 应用部署到 Kubernetes*，我们在那里通过 `kubectl` 创建、描述、检索和编辑了秘密。

关于 Docker，我们将首先从创建秘密开始。

## 创建

`create` 命令是我们将新秘密添加到 Raft 日志文件中的方法。其基本格式如下：

```
docker secret create [OPTIONS] SECRET [file|-]
```

你可能会注意到这与 `kubectl` 中的命令类似，即 `kubectl create secret`。

创建秘密时，我们可以使用 `-l` 标志为秘密添加标签，如下所示：

```
docker secret create -l key=val api_key -
```

这使我们能够为值添加标签，从而知道它们是针对哪个环境的。例如，我们可以为环境添加一个键值对，如**质量保证**（**QA**）、**开发**（**DEV**）和**生产**（**PROD**）。

一个秘密也可以是一个文件。例如，如果我们想添加一个私钥，可能会如下操作：

```
docker secret create my_key ./id_rsa
```

如果你希望将秘密添加或更新到正在运行的服务中，你需要在 `update` 命令上使用 `--secret-add` 标志。例如，可以按如下方式操作：

```
docker service update --secret-add <secret> <service>
```

在添加了一个秘密后，让我们来探讨一下如何查看它。

## 查看

有很多方法可以用来查看 Docker secrets。要列出添加到 Raft 日志文件中的任何秘密，我们可以使用 `ls` 命令：

```
$ docker secret ls.
```

执行此命令后，当前的秘密将被显示，如下所示：

```
ID             NAME      CREATED         UPDATED
123345         my_key    2 weeks ago     2 weeks ago 
```

我们可以使用 `inspect` 命令收集有关该秘密的更多信息。

其格式如下：

```
docker secret inspect [OPTIONS] SECRET [SECRET...]
```

所以，使用前面的示例，我们可以如下执行命令：

```
$ docker secret inspect my_key
```

然后，这将返回一个包含 ID、版本、创建和更新日期的 JSON 对象，并且包含标签和名称的`spec`对象。现在提供了一个该输出的示例：

```
[
    {
        "ID": "ae4kfwe6s56sgop7vn1kxap59",
        "Version": {
            "Index": 10
        },
        "CreatedAt": "2020-01-26T07:15:29.674382561Z",
        "UpdatedAt": "2020-01-26T07:15:29.674382561Z",
        "Spec": {
            "Name": "my_key",
            "Labels": {
                "env": "dev",
                "rev": "20200126"
            }
        }
    }
]
```

我们已经添加并检查了机密，现在我们将探索如何在不再需要它们时将其删除。

## 删除

删除一个机密与添加一个机密一样简单，语法与 Linux 删除文件的命令`rm`相同。

命令的格式如下：

```
docker secret rm SECRET [SECRET...]
```

在 Kubernetes 中，相应的命令是`kubectl delete secret`。

要删除我们之前的示例机密，我们可以运行如下命令：

```
docker secret rm my_key
```

如果你希望删除当前服务正在使用的机密，你需要在`update`命令中使用`--secret-rm`标志，如以下示例所示：

```
docker service update --secret-rm <secret> <service>
```

如你所见，添加、删除和检查机密非常简单。现在，让我们尝试使用*第十三章*中的 SSH 文件，*Docker 安全基础与最佳实践*，运行前面的命令。

# 机密操作 – 示例

现在是时候尝试我们刚才回顾过的命令（`create`/`inspect`/`ls`/`rm`）了。确保你的设置已经配置为使用 Swarm。你也可以重用上一章中的镜像进行此部分的操作。可以使用以下命令获取该镜像：

```
$ docker pull docker pull dockerfordevelopers/shipitclicker@ sha256:39eda93d15866957feaee28f8fc5adb545276a64147445c64992ef 69804dbf01
```

重要提示

记住，你可以使用`docker swarm init`命令初始化 swarm。也可以使用`--advertise-addr`标志，并指定你初始容器的 IP 地址。

之前，我们使用以下命令为 SCP 添加了一个 SSH 私钥，供单个容器使用：

```
$ docker build --build-arg ssh_prv_key="$(cat ~/.ssh/id_rsa_test)" .
```

要将此密钥添加到我们的 swarm 中，我们将使用以下命令：

```
$ docker secret create -l env=dev ssh_prv_key ~/.ssh/id_rsa_test
```

在这里，我们创建了一个与之前使用的构建参数相同名称的新机密，并将我们的私钥内容输出到其中。我们还包含了一个标签，标签的`key=val`对表示我们所在的环境。在此情况下，它是开发环境。

现在让我们检查一下我们是否正确添加了它。我们可以通过运行`ls`命令来验证：

```
$ docker secret ls
ID         NAME          CREATED        UPDATED
To5jj...   ssh_prv_key   1 minutes ago  1minutes ago
```

在这里，我们可以看到机密的 ID 和名称。看起来不错！现在我们使用`NAME`值在密钥上执行`inspect`命令：

```
$ docker secret inspect ssh_prv_key
```

现在，你应该能看到一个类似以下内容的 JSON 对象：

```
[
    {
        "ID": "to5jjgshjqaddhf56ty89rss42",
        "Version": {
            "Index": 17
        },
        "CreatedAt": "2019-11-25T07:11:03.335174723Z",
        "UpdatedAt": "2019-11-25T07:11:03.335174723Z",
        "Spec": {
            "Name": "ssh_prv_key",
            "Labels": {
                "env": "dev",
                "rev": "20181125"
            }
        }
    }
]
```

如果你在 Swarm 中有多个容器，那么你可以授予它们访问这个机密的权限。以下示例演示了如何将我们刚刚创建的机密发送到一个使用我们示例镜像的新容器：

```
$ docker service create --name second_container --secret source=ssh_prv_key,target=second_ssh_prv_key,mode=0400 dockerfordevelopers/shipitclicker@ sha256:39eda93d15866957feaee28f8fc5adb545276a64147445c64992ef 69804dbf01
```

在这里，`--secret source`值设置为我们创建的 Docker 机密的名称。然后，我们将它存储在目标值中定义的变量里。为了清晰起见，我们将其命名为`second_ssh_prv_key`。模式已设置为`0400`，以便使机密可访问，并选择我们标记过的镜像作为`create`命令的源镜像。

要确认秘密是否可用，我们可以检查我们早些时候讨论过的临时文件系统。为此，您需要获取新容器的容器 ID。您可以使用`docker ps`命令来执行此操作。

接下来，使用容器 ID 如下：

```
$ docker exec -it <id> cat /run/secrets/second_ssh_prv_key
```

您应该看到秘密的内容与您传递给第一个容器的内容相同，即我们迄今为止一直在测试的私有 SSH 密钥。

其他选项

除了使用本机 Docker 和 Kubernetes 工具外，还存在各种其他选项可用于在基于云的系统中存储秘密。AWS、GCP 和 Azure 提供本机支持，而 HashiCorp 通过 HashiCorp Vault 提供了一种全面的与云无关的秘密管理机制，网址为[`www.vaultproject.io/`](https://www.vaultproject.io/)。

现在，我们将进一步了解 Docker 秘密，以及如何使用标签。

# Docker 安全标签

我们刚刚看到如何确保在容器集群中安全共享秘密。在*第十二章*，*容器安全介绍*，我们学习了如何使用标签结合其他安全功能，确保使用正确的镜像。

现在，我们将看到如何通过使用标签与秘密和标签结合，我们可以注释给定秘密和标签在哪个环境中使用。

作为良好的安全实践，我们应始终在不同环境中使用不同的秘密。例如，在您的开发、演示和生产实例中，数据库访问密码不应相同。通常，在您的开发过程中，与生产环境相比，您可能会在研究、开发和 QA 环境中使用更新版本的容器。

我们可以使用 Docker 标签来帮助确保一旦为开发环境设置了凭据/秘密，我们也会拉取正确的镜像；也就是说，我们打算用于开发目的的镜像与我们创建的开发凭据。使用固定的标签通过不可变性提供了一层安全性，并防止意外地在开发环境之外使用可能包含安全漏洞的实验性镜像。

通常，应采用语义化版本控制（[`semver.org/`](https://semver.org/)）。这将导致标签使用一种格式，以传达您在使用版本时应期待的变更级别。主要版本号指示了一组不向后兼容的更改。次要版本通常是对现有版本的新功能。最后，我们有了修补程序版本，这可能是一个小的安全修复或类似的内容。一个典型的格式可能如下：

```
1.1.2
```

在选择标签时，与您的版本控制系统一致，选择最接近您希望部署的环境的标签。例如，选择`:1.1.2-dev`而不是`:1`。

在这个例子中，你知道你将要拉取补丁发布。然后，你可以通过`docker secret`部署凭证，专门为这个构建和你部署的环境。例如，一种有用的方法是将密钥标签与使用的标签版本配对，如以下代码所示：

```
$docker secret create  --label ver=1.1.2-dev \
                       --label env=dev \
                       ssh_prv_key ~/.ssh/id_rsa_test
```

在这个示例中，已经创建了一个密钥（一个 SSH 密钥），并且我们知道它应该与标签版本`1.1.2`一起使用，并且这是一个开发环境。这里，标签提供了注释，给我们提供了密钥的上下文。像这样的简单技巧有助于向工程团队提供更多信息，避免生产凭证被错误地与实验性开发容器一起使用或用于错误的环境。

我们已经看到如何将标签、密钥和标签结合使用。现在让我们看看其他标记选项。

# 使用标签进行元数据应用

元数据标签是一种在容器中注释额外信息的方式，旨在为开发团队提供有用的信息。当团队中的其他开发者需要了解镜像的关键特性，比如版本号和描述时，这会非常有用。

我们通过`docker secrets`命令看到了如何通过命令行添加标签。使用元数据标签，我们还可以将标签添加到 Dockerfile 中，这样在构建新容器时，这些信息就会被内嵌。

标签采用以下格式：

```
LABEL key=value
```

在我们之前的示例基础上，我们可以通过 Dockerfile 在容器内设置版本，代码如下：

```
LABEL "version"="1.1.2-test"
LABEL "description"=" Development environment container for testing the newest security patch. Not for production release yet"
```

一旦你构建了一个容器，你可以使用`docker inspect`命令查看任何你添加的元数据：

```
"Labels" :{
       "version"="1.1.2-test",
       "description"=" Development environment container for testing the newest security patch. Not for production release yet"
}
```

在发布供公众使用的软件时，你还应考虑链接到一个`security.txt`文件。就像行为规范或贡献者指南一样，它提供了一种机制，提醒安全研究人员如何负责任地披露他们可能发现的任何安全问题。

你可以从以下网站自动生成一个`security.txt`文件：

[`securitytxt.org/`](https://securitytxt.org/)

将此文件保存到你的代码库中，然后通过 Dockerfile 中的`LABEL`链接它，如以下示例所示：

```
LABEL "security.txt"="https://respository.example.com/my_project/security.txt"
```

这就是我们关于密钥、标签和标签的指南总结。让我们回顾一下到目前为止学到的内容。

# 总结

在这一章中，我们了解了关于 Docker 密钥的所有内容，它是 Kubernetes 密钥的对等物。我们看到如果需要使用这种技术而不是 Kubernetes 时，如何安全地在容器之间共享敏感数据。我们还了解了这对于基于工作环境划分凭证集有多么有用。最后，我们演示了如何创建、检查和删除这些密钥。

接下来，我们再次回顾了标签，并讨论了如何使用这些标签确保从正确的环境中拉取正确的镜像。我们展示了环境相关的密钥和标签的结合，帮助你进一步保护开发过程。

最后，我们讨论了如何为容器添加元数据标签。这也包括了使用`security.txt`文件。

在下一章，我们将探讨如何利用第三方工具帮助我们保护容器，并加强我们迄今为止学到的一些实践方法。
