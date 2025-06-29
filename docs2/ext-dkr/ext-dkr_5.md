# 第五章：构建你自己的插件

除了提供核心工具外，Docker 还记录了一个 API，允许核心 Docker 引擎与第三方开发者编写的插件服务进行通信。目前，这个 API 允许你将自己的存储和网络引擎接入 Docker。

这可能看起来像是将你限制在一个非常小众的插件集合中，确实如此。然而，Docker 作出这个决定是有充分理由的。

让我们来看一下我们在前几章已经安装的一些插件；然而，我们不会介绍它们的功能，而是看看它们背后的操作过程。

# 第三方插件

Docker 文档网站上关于插件的第一页列出了许多第三方插件。如前所述，让我们了解一下我们已经在第三章，*卷插件*和第四章，*网络插件*中安装并使用的插件背后的操作。

## Convoy

Convoy 是我们在第三章中查看的第一个第三方插件，*卷插件*。为了安装它，我们在 DigitalOcean 启动了一个 Docker 主机，因为我们需要一个比 Docker Machine 偏好的 Boot2Docker 操作系统更完整的底层操作系统。

为了安装 Convoy，我们从 GitHub 下载了一个发布文件。这个 tar 压缩包包含了运行 Convoy 所需的静态二进制文件，一旦静态二进制文件到位，我们创建了一个 Docker 插件文件夹，并添加了一个符号链接到 Convoy 在首次执行时创建的套接字文件。

然后我们继续配置了一个我们在卷上创建的回环设备。接着，我们指示 Convoy 使用新创建的卷，通过启动我们下载的 Convoy 静态二进制文件将 Convoy 作为守护进程运行。

### 注意

在多任务计算机操作系统中，守护进程是一个作为后台进程运行的计算机程序，而不是直接由交互式用户控制：

[`en.wikipedia.org/wiki/Daemon(computing)`](https://en.wikipedia.org/wiki/Daemon(computing))。

就 Docker 而言，当使用`--volume-driver=convoy`标志启动容器时，每个请求将简单地将与卷相关的任何任务交给守护进程化的 Convoy 进程处理。

如果你回顾一下第三章中的*Convoy*部分，*卷插件*，你会注意到我们与 Convoy 的所有交互都使用`convoy`命令而不是`docker`命令，实际上，Convoy 客户端使用的是与我们符号链接到`Docker 插件`文件夹的同一个套接字文件。

## REX-Ray

接下来，我们安装了 REX-Ray。为此，我们运行了一个命令，该命令从[`dl.bintray.com/emccode/rexray/install`](https://dl.bintray.com/emccode/rexray/install)下载并执行了一个 bash 脚本。

该脚本会判断您正在运行的操作系统，然后下载并安装 DEB 或 RPM 文件。正如您可能已经猜到的，这些软件包会安装适合您操作系统的静态二进制文件。

REX-Ray 更进一步，还通过安装 init、upstart 或 systemd 服务脚本来启动守护进程，这意味着您可以像管理 Docker 主机上的其他服务一样启动和停止它。

再次说明，一旦我们安装了 REX-Ray，我们与该工具的唯一交互方式就是使用 `rexray` 命令。

## Flocker

Flocker 更进一步，而不是安装安装脚本，我们使用 Cluster HQ 提供的 AWS CloudFormation 模板来为我们引导环境。

该程序完成了显而易见的任务：启动 Docker 主机、设置安全组，并安装和配置 Docker 和 Flocker。

Flocker 比 Convoy 和 REX-Ray 更进一步，通过安装一个与远程托管的 Web API（卷中心）交互的代理。

如本章所述，Flocker 在卷插件概念出现之前就已经存在。因此，许多与 Flocker 的交互是在 Docker 之外进行的；实际上，Cluster HQ 编写了他们自己的 Docker 封装程序，以便在 Docker 内部选项出现之前，您就可以轻松创建 Flocker 卷。

## Weave

这是我们查看的唯一第三方网络插件。像 Flocker 一样，Weave 在 Docker 推出插件功能之前就已经存在。

Weave 与我们查看的其他第三方工具略有不同。在 Weave 中，下载的实际上是一个 Bash 脚本，而不是静态二进制文件。

### 注意

该脚本用于配置主机，并从 Weaveworks Docker Hub 帐户下载容器，您可以在 [`hub.docker.com/u/weaveworks/`](https://hub.docker.com/u/weaveworks/) 找到该帐户。

脚本启动并配置容器，给予足够的权限以便与主机机器交互。该脚本还负责通过 `docker exec` 命令向运行中的容器发送命令，并在主机上配置 `iptables`。

## 插件之间的共性

如您所见，正如您所经历的，这些插件都有脚本和二进制文件，它们是 Docker 本身之外的外部文件。

它们几乎都是用与 Docker 相同的语言编写的：

| 插件 | 语言 |
| --- | --- |
| Convoy | Go |
| REX-Ray | Go |
| Flocker | Python |
| Weave | Go |

大多数服务都是用 Go 编写的，唯一的例外是 Flocker，它主要是用 Python 编写的：

> *Go 语言简洁、清晰、有效率，其并发机制使得编写高效利用多核和联网机器的程序变得容易，而其新颖的类型系统则使得灵活且模块化的程序构建成为可能。Go 语言快速编译为机器代码，同时具备垃圾回收的便利性和运行时反射的强大功能。它是一种快速的静态类型编译语言，使用起来却像是一种动态类型解释语言。[`golang.org/`](https://golang.org/)。*

# 了解插件

到目前为止，我们已经确定，所有我们安装的插件实际上与 Docker 没有直接关系，那么插件到底做什么呢？

Docker 将插件描述为：

> *“Docker 插件是外部进程扩展，能够为 Docker 引擎添加功能。”*

这正是我们在安装第三方工具时所看到的，它们都作为独立的守护进程与 Docker 并行运行。

假设我们将在本章的其余部分创建一个名为`mobyfs`的卷插件。mobyfs 插件是一个虚构的服务，它是用 Go 语言编写的，并作为一个守护进程运行。

## 发现

通常，插件会安装在与 Docker 二进制文件相同的主机上。我们可以通过在`/run/docker/plugins`目录下创建以下文件（如果是 Unix 套接字文件），或者在`/etc/docker/plugins`或`/usr/lib/docker/plugins`目录下创建（如果是其他两种文件）来将 mobyfs 插件注册到 Docker 中：

+   `mobyfs.sock`

+   `mobyfs.spec`

+   `mobyfs.json`

使用 Unix 套接字文件的插件必须与 Docker 安装在同一主机上。使用`.spec`或`.json`文件的插件，如果你的守护进程支持 TCP 连接，可以在外部主机上运行。

如果你使用的是.spec 文件，那么文件只需包含一个 URL，指向 TCP 主机和端口或本地套接字文件。以下三种示例都是有效的：

```
tcp://192.168.1.1:8080
tcp://localhost:8080
unix:///other.sock
```

如果你想使用`.json`文件，它的内容应类似于以下代码：

```
{
  "Name": "mobyfs",
  "Addr": "https:// 192.168.1.1:8080",
  "TLSConfig": {
    "InsecureSkipVerify": false,
    "CAFile": "/usr/shared/docker/certs/example-ca.pem",
    "CertFile": "/usr/shared/docker/certs/example-cert.pem",
    "KeyFile": "/usr/shared/docker/certs/example-key.pem",
  }
}
```

JSON 文件中的`TLSConfig`部分是可选的；然而，如果你将服务运行在 Docker 主机之外，我建议使用 HTTPS 进行 Docker 与插件之间的通信。

## 启动顺序

理想情况下，插件服务应在 Docker 之前启动。如果你正在运行的主机安装了`systemd`，则可以通过使用类似以下的`systemd`服务文件来实现这一点，文件应命名为`mobyfs.service`：

```
[Unit]
Description= mobyfs
Before=docker.service

[Service]
EnvironmentFile=/etc/mobyfs/mobyfs.env
ExecStart=/usr/bin/mobyfs start -p 8080
ExecReload=/bin/kill -HUP $MAINPID
KillMode=process

[Install]
WantedBy=docker.service
```

这将确保你的插件服务始终在主 Docker 服务之前启动。

如果你将插件服务托管在外部主机上，可能需要重启 Docker 才能让 Docker 开始与插件服务通信。

你可以将插件打包在容器内。为了避免 Docker 必须在插件服务之前启动，每个激活请求将在 30 秒内尝试多次。

这将为容器提供足够的时间来启动，并在绑定自己到容器的端口之前运行插件服务的任何引导过程。

## 激活

现在插件服务已经启动，我们需要告诉 Docker 在调用插件服务时应该将请求发送到哪里。根据我们的示例，服务是一个卷插件，我们应该运行类似于以下命令的内容：

```
docker run -ti -v volumename:/data --volume-driver=mobyfs russmckendrick/base bash

```

这将把我们已经在插件服务中配置的 `volumename` 卷挂载到容器中的 `/data`，该容器运行我的基础容器镜像，并将我们连接到一个 shell。

当调用 mobyfs 卷驱动程序时，Docker 将在我们在 *发现* 部分中讨论的三个插件目录中进行搜索。默认情况下，Docker 将始终先查找套接字文件，然后是 `.spec` 或 `.json` 文件。插件名称必须与文件扩展名前的文件名匹配。如果不匹配，Docker 将无法识别该插件。

## API 调用

一旦插件被调用，Docker 守护进程将通过 HTTP 使用 RPC 风格的 JSON 向插件服务发送 POST 请求，使用 `.spec` 或 `.json` 文件中定义的套接字文件或 URL。

这意味着你的插件服务必须实现一个 HTTP 服务器，并将其绑定到你在发现部分中定义的套接字或端口。

Docker 发出的第一个请求将是 `/Plugin.Activate`。你的插件服务必须对三种响应之一作出回应。由于 mobyfs 是一个卷插件，响应将如下所示：

```
{
    "Implements": ["VolumeDriver"]
}
```

如果它是一个网络驱动程序，那么我们的插件服务应给出的响应将如下所示：

```
{
    "Implements": ["NetworkDriver"]
}
```

插件服务的最终响应如下所示：

```
{
    "Implements": ["authz"]
}
```

任何其他响应都将被拒绝，激活将失败。现在 Docker 已经激活了插件，它将继续根据调用 `/Plugin.Activate` 时收到的响应，向插件服务发送 POST 请求。

# 编写你的插件服务

如前节所述，Docker 将通过发出 HTTP 调用与插件服务进行交互。这些调用的文档可以在以下页面中找到：

+   **卷** **驱动插件**：[`docs.docker.com/engine/extend/plugins_volume/`](https://docs.docker.com/engine/extend/plugins_volume/)

+   **网络** **驱动插件**：[`docs.docker.com/engine/extend/plugins_network/`](https://docs.docker.com/engine/extend/plugins_network/)

+   **授权** **插件**：[`docs.docker.com/engine/extend/plugins_authorization/`](https://docs.docker.com/engine/extend/plugins_authorization/)

Docker 还提供了一个 SDK，作为 Go 帮助程序的集合，这些可以在以下 URL 找到：

[`github.com/docker/go-plugins-helpers`](https://github.com/docker/go-plugins-helpers)

每个帮助程序都附带示例以及开源项目的链接，这些项目提供了如何实现该帮助程序的进一步示例。

这些 API 请求不应与 Docker 远程 API 混淆，Docker 远程 API 的文档可在以下网址查看：

[`docs.docker.com/engine/reference/api/docker_remote_api/`](https://docs.docker.com/engine/reference/api/docker_remote_api/)

这是 API，它允许您的应用与 Docker 进行交互，而不是 Docker 与您的应用进行交互。

# 总结

正如您所看到的，我们只讨论了 Docker 如何与您编写的插件服务进行交互，并没有涉及如何实际编写插件服务。

这样做的原因是由于我们需要涵盖的插件服务，我们还需要以下功能：

+   使用 Go 编写

+   能够作为守护进程运行

+   包含一个绑定到 Unix 套接字或 TCP 端口的 HTTP 服务器

+   能够接受并回答 Docker 守护进程向其发出的请求

+   将 Docker 发出的 API 请求转化为文件系统或网络服务

如您所想，这本身就有可能成为一本完整的书。

此外，构建自己的插件是一个相当大的工程，因为您已经需要编写服务的基础部分。虽然似乎有很多 Docker 插件，但在 GitHub 上搜索 Docker 插件只会返回一些几十个已经使用 Docker 插件 API 编写的插件。

其余返回的项目都是与 Docker 远程 API 通信的第三方服务工具或插件（例如 Jenkins、Maven 等）。

在下一章，我们将探讨第三方工具，以便在使用 Docker Machine 之外扩展您的基础设施。
