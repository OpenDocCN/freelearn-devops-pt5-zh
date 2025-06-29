# 第五章：Fn 项目

本章介绍了一个 FaaS 平台，*Fn 项目*。它是由 Oracle Inc. 团队开发的另一个出色的 FaaS 框架。Fn 是其中一个最简单的项目，允许我们在纯 Docker 基础设施上部署 FaaS 平台。

本章从讨论 Fn 项目是什么开始。然后我们将继续探讨它的组件如何组织，以及它的整体架构。接下来，我们将学习如何使用 Fn CLI 准备和部署函数。本章最后将讨论如何使用 Fn 子项目来管理其 UI、扩展和监控 Fn 集群本身。

本章将涵盖以下内容：

+   Fn 项目

+   Fn 的架构

+   使用 Fn CLI

+   部署本地函数

+   在 Docker Swarm 上部署 Fn

+   使用内置 UI 监控 Fn

+   使用熟悉的工具进行日志分析

# Fn 项目

Fn 项目最初由*Iron.io*团队（[`www.iron.io/`](https://www.iron.io/)）在 Iron function 名称下构思。此后，两位创始人加入了 Oracle，并将 Iron function 分支为新的项目 Fn。

Fn 是一个框架和系统，用于开发和部署无服务器/FaaS 应用程序。与 OpenFaaS 不同，Fn 不使用任何编排器级别的功能来管理函数容器。

Fn 不仅支持通过其自身基础设施进行部署；它还允许你将相同的函数部署到 AWS Lambda。然而，我们这里只讨论如何将函数部署到其自身的基础设施，当然，这个基础设施是基于 Docker 的。

Fn 背后有几个设计原因。

Fn 项目致力于开源。它原生支持 Docker，这意味着我们可以将 Docker 容器作为其部署单元——**一个函数**。Fn 支持任何编程语言的开发。Fn 基础设施是用 Go 编程语言编写的，旨在能够在各处部署，包括公共云、私有云，甚至混合基础设施。Fn 还支持从 AWS 导入 Lambda 函数并将它们部署到自身的基础设施中。

如前所述，基于 Docker 的无服务器/FaaS 基础设施基本上旨在平衡控制整个系统与基础设施的维护和管理的便利性。Fn 也有与这一理念相一致的设计目标。

# Fn 的架构

Fn 服务器的最简单设置只是启动一个独立的 Fn 容器；然而，更完整的架构将如图所示。本章最后将演示集群实现。下图展示了 Fn 架构的概览：

![](img/31b1eb88-ecb6-4b85-9faf-6251d64d41ca.png)

图 5.1：Fn FaaS 集群的架构

与常见的 FaaS 架构一样，Fn 也有*API 网关*，在前面的图示中为**Fn LB**。Fn LB 基本上是一个负载均衡器。它将来自客户端的请求转发到每个**Fn Server**。在 Fn Server 的实现中，没有像 Fn 架构核心中的事件总线那样的*发起者*和*执行者*的分离概念。因此，Fn Server 也充当执行者，在其关联的 Docker 引擎上执行函数。

Fn 服务器连接到一个**Log Store**，它可以是一个独立的数据库系统或一个数据库管理系统的集群。所有从 Fn 函数发送到标准错误的数据显示都会记录到**Log Store**。

**Fn UI** 和 **Fn LB** 是额外的组件，有助于在生产环境中改进 Fn 项目。Fn UI 是用户界面服务器，如仪表盘，用于 Fn，而 Fn LB 是负载均衡器，用于在集群中的 Fn 节点之间进行轮询。

在 Fn Server 中有一个*执行者代理*的概念。该代理负责控制运行时环境。在 Fn 中，运行时是 Docker。因此，在本章节中，执行者代理也称为*Docker 代理*。在默认配置下，Fn Server 中的 Docker 代理连接到本地 Docker 引擎，并通过本地 Unix 套接字启动 Fn 函数：

![](img/1e846299-3e5e-4981-9e49-1368f4711128.png)

图 5.2：显示 Fn 集群在 Swarm 范围网络上的示意图

上面的图示展示了一个在 Swarm 范围覆盖网络上的运行 Fn 集群。为了组成一个集群，我们将使用一个可附加的 Swarm 范围网络。每个 Fn Server 实例都需要连接到该网络。当请求被发送到网关或直接发送到服务器时，它将被传递到**EntryPoint**。EntryPoint 是一个特定语言的程序，它包装了真正的函数程序。例如，在使用 Java 构建的 Fn 函数中，EntryPoint 是类 `com.fnproject.fn.runtime.EntryPoint`。这个 Java 类内部的代码通过 Java 的反射技术调用真正的函数：

![](img/b2280d72-5efe-49de-8417-07de9a774c2e.png)

图 5.3：一个 Fn 函数与 STDIN、STDOUT 交互并将日志写入 STDERR，且将日志委托给存储

Fn 服务器将请求主体以**STDIN**的形式发送到**Function**容器。当**EntryPoint**接收到**STDIN**流后，它会将数据内容转换为匹配函数签名类型的格式。在前面的图示中，签名是**String**。因此，函数体被转换为字符串。发送到**STDOUT**的输出将被转发到**Fn Server**并作为结果发送出去，而发送到**STDERR**的输出将被捕获并存储在**Log Store**中。

# 使用 Fn CLI

本节将讨论如何使用 Fn CLI 的基本功能，这是一个控制 Fn 的命令行工具。我们先从 Fn CLI 的安装开始。确保你的系统中存在`curl`命令：

```
$ curl -LSs https://raw.githubusercontent.com/fnproject/cli/master/install | sh
```

安装完前面的命令后，通过输入`fn`检查其版本和帮助信息。撰写时，命令行的当前版本是`0.4.43`。由于变化迅速，你可以预期使用不同版本的情况：

```
$ fn
fn 0.4.43

Fn command line tool

ENVIRONMENT VARIABLES:
 FN_API_URL - Fn server address
 FN_REGISTRY - Docker registry to push images to, use username only to push to Docker Hub - [[registry.hub.docker.com/]USERNAME]

COMMANDS:
...
```

`fn`提供了几个子命令，例如：

+   `fn start`是`docker run`命令的一个简单包装器。此子命令启动新的 Fn Server 实例。默认地址将是`http://localhost:8080`。然而，Fn CLI 将尝试连接到在环境变量中定义的`FN_API_URL`地址（如果已设置）。

+   `fn update`是用于将最新版本的 Fn Server 拉取到本地 Docker 镜像的命令。

+   `fn init`是用于初始化一个骨架以开发新函数的命令。它接受`--runtime`参数，用于生成特定语言的模板，例如 Go 语言。

+   `fn apps`包含用于创建、更新和删除应用程序的子命令，是一种命名空间或包，用于将多个函数组合在一起。要求函数必须在一个应用程序下定义。

+   `fn routes`是一组用于定义指向函数容器的路由的命令。例如，我们有一个名为`demo`的应用程序，然后可以定义路由`hello`并将其指向 Docker 容器镜像`test/hello:v1`。一个应用程序可以有多个路由：

![](img/2f66488b-55b8-42eb-941a-17a11968e653.png)

图 5.4：Fn 应用程序及其路由之间的关系

这是 Fn 如何在应用程序下组织路由的示例。例如，Fn 的 API URL 是`http://localhost:8080`。我们可能有一个名为`demo`的应用程序，其中包含一个名为`hello`的路由，该路由为容器镜像`test/hello:v1`创建。所有这些组合在一起形成了一个访问该函数的完整 URL。

# 让我们部署一个本地函数

首先，执行`fn start`以启动一个独立的 Fn Server 实例。服务器通过设置日志级别为`info`（默认设置）启动。Fn Server 然后连接到数据存储，即日志存储。当前的实现是 SQLite3。之后，代理将启动。Docker 代理使用其默认配置连接到本地 Docker 引擎。最后，Fn 开始监听端口`8080`：

```
$ fn start

time="2018-03-17T08:48:39Z" level=info msg="Setting log level to" level=info
time="2018-03-17T08:48:39Z" level=info msg="datastore dialed" datastore=sqlite3 max_idle_connections=256
time="2018-03-17T08:48:40Z" level=info msg="agent starting cfg=&{MinDockerVersion:17.06.0-ce FreezeIdle:50ms EjectIdle:1s HotPoll:200ms HotLauncherTimeout:1h0m0s AsyncChewPoll:1m0s MaxResponseSize:0 MaxLogSize:1048576 MaxTotalCPU:0 MaxTotalMemory:0 MaxFsSize:0}"
time="2018-03-17T08:48:40Z" level=info msg="no docker auths from config files found (this is fine)" error="open /root/.dockercfg: no such file or directory"

 ______
 / ____/___
 / /_ / __ \
 / __/ / / / /
 /_/ /_/ /_/
 v0.3.381

time="2018-03-17T08:48:41Z" level=info msg="available memory" availMemory=12357627495 cgroupLimit=9223372036854771712 headRoom=1373069721 totalMemory=13730697216
time="2018-03-17T08:48:41Z" level=info msg="sync and async ram reservations" ramAsync=9886101996 ramAsyncHWMark=7908881596 ramSync=2471525499
time="2018-03-17T08:48:41Z" level=info msg="available cpu" availCPU=4000 totalCPU=4000
time="2018-03-17T08:48:41Z" level=info msg="sync and async cpu reservations" cpuAsync=3200 cpuAsyncHWMark=2560 cpuSync=800
time="2018-03-17T08:48:41Z" level=info msg="Fn serving on `:8080`" type=full
```

为了检查 Docker 是否正确启动了 Fn Server，我们可以使用`docker ps`查看正在运行的容器。可以在另一个终端执行此操作：

```
$ docker ps --format="table {{.ID}}\t{{.Names}}\t{{.Ports}}"
CONTAINER ID    NAMES       PORTS
ab5cd794b787    fnserver    2375/tcp, 0.0.0.0:8080->8080/tcp
```

好的，现在我们已经看到 Fn Server 正在端口`8080`上运行，并且通过`docker ps`看到映射`0.0.0.0:8080->8080/tcp`。

在启动`fn start`命令的当前目录中，容器将其`data`目录映射到主机的`$PWD/data`。该目录包含 SQLite3 数据库文件，用于存储日志和信息。在生产环境中，我们将用 MySQL DBMS 替换它，例如：

```
$ tree
.
└── data
 ├── fn.db
 └── fn.mq

1 directory, 2 files
```

要查看应用程序列表，只需使用`fn apps list`命令：

```
$ fn apps list
no apps found
```

好吧，由于我们刚刚启动了服务器实例，因此没有新创建的应用程序。我们将创建一个，命名为`demo`，然后再次使用`fn apps list`命令来双重确认创建的应用：

```
$ fn apps create demo
Successfully created app: demo

$ fn apps list
demo
```

现在我们将开始开发一个函数。在这个示例中，我们使用 Java 运行时，稍后我们将尝试使用 Go 的另一种运行时。

让我们初始化新函数。我们使用`fn init`来创建一个新的函数项目。此命令需要`--runtime`来指定我们希望使用的语言运行时。

`func.yaml`是我们的函数描述文件。它包含版本号、运行时和函数的入口点：

```
$ fn init --runtime java hello
Creating function at: /hello
Runtime: java
Function boilerplate generated.
func.yaml created.
```

我们将尝试学习如何构建和部署一个函数。所以让我们先不修改任何内容，直接构建它。要构建函数，只需使用`fn build`。而要部署函数，我们有`fn deploy`来为我们处理整个过程。

这是 Fn 的构建行为。在调用`fn build`命令后，构建过程开始，使用生成的 Dockerfile。生成的镜像将被打标签并由 Docker 引擎本地存储。例如，示例中的镜像将被本地标记为`hello:0.0.1`。然后，使用`fn deploy`命令时，需要`--registry`来将镜像远程存储在 Docker Hub 上。在此示例中，使用的是我的 Docker ID，请记得将其更改为您的 ID。

`fn deploy`命令的工作方式如下。

首先，它增加了函数的版本号。其次，它使用`--registry`将函数的镜像推送到 Docker Hub，作为仓库名称。因此，`hello:0.0.2`在 Docker Hub 上变成了`chanwit/hello:0.0.2`。

然后，`fn deploy`将使用新构建镜像的名称，在`--app`指定的应用程序下注册一个新路由：

```
$ fn build
Building image hello:0.0.1 
Function hello:0.0.1 built successfully.

$ fn deploy --app demo --registry chanwit
Deploying hello to app: demo at path: /hello
Bumped to version 0.0.2
Building image chanwit/hello:0.0.2 
Pushing chanwit/hello:0.0.2 to docker registry...The push refers to repository [docker.io/chanwit/hello]
07a85412c682: Pushed 
895a2a3582de: Mounted from fnproject/fn-java-fdk 
5fb388f17d37: Mounted from fnproject/fn-java-fdk 
c5e4fcfb11b0: Mounted from fnproject/fn-java-fdk 
ae882186dfca: Mounted from fnproject/fn-java-fdk 
aaf375487746: Mounted from fnproject/fn-java-fdk 
51980d95baf3: Mounted from fnproject/fn-java-fdk 
0416abcc3238: Mounted from fnproject/fn-java-fdk 
0.0.2: digest: sha256:c7539b1af68659477efac2e180abe84dd79a3de5ccdb9b4d8c59b4c3ea429402 size: 1997
Updating route /hello using image chanwit/hello:0.0.2...
```

让我们检查一下新注册的路由。我们使用`fn routes list <app>`命令来列出应用程序`<app>`下的所有路由。在下面的示例中，列出了`demo`的所有路由：

```
$ fn routes list demo
path     image                 endpoint
/hello   chanwit/hello:0.0.2   localhost:8080/r/demo/hello
```

上一个命令还列出了每个路由的端点。通过端点，我们基本上可以像普通 HTTP 端点一样使用`curl`与其交互。不要忘记为`curl`设置`-v`详细选项。通过此选项，我们可以检查 HTTP 头部中隐藏的内容。

让我们看一下 HTTP 响应头中标记为粗体的行。这里有一些额外的条目，`Fn_call_id`和`Xxx-Fxlb-Wait`。

头部信息，`Fn_call_id`，是每次调用的标识符。此 ID 在我们启用 Fn 的分布式追踪时也将使用。头部信息，`Xxx-Fxlb-Wait`，是 Fn LB 收集的信息，它可以知道此函数的等待时间：

```
$ curl -v localhost:8080/r/demo/hello

* Trying 127.0.0.1...
* Connected to localhost (127.0.0.1) port 8080 (#0)
> GET /r/demo/hello HTTP/1.1
> Host: localhost:8080
> User-Agent: curl/7.47.0
> Accept: */*
> 
< HTTP/1.1 200 OK
< Content-Length: 13
< Content-Type: text/plain
< Fn_call_id: 01C8SPGSEK47WGG00000000000
< Xxx-Fxlb-Wait: 78.21µs
< Date: Sat, 17 Mar 2018 10:01:43 GMT
< 
* Connection #0 to host localhost left intact
Hello, world!
```

# 再试一次使用 Golang

让我们尝试使用另一种运行时 Go 创建下一个函数。与 Java 不同，Go 语言代码在 Fn 函数内没有明确的入口点概念。幸运的是，Fn 的执行模型足够简单，因此这个问题非常微不足道：

```
$ fn init --runtime go hello_go

Creating function at: /hello_go
Runtime: go
Function boilerplate generated.
func.yaml created.
```

这是 Go Fn 函数的文件列表：

```
$ cd hello_go

$ tree .
.
├── func.go
├── func.yaml
└── test.json
```

文件`func.go`当然就是*函数程序*本身，而`func.yaml`是 Fn 的函数*描述符*。其中一个有趣的文件是`test.json`——它是包含功能测试的测试数据文件。目前，我们可以使用`fn test`命令来测试正向路径，但无法测试负向结果。

我们首先来看一下`func.yaml`，了解它的内容。每次部署时，`version`会自动增加。这里的`runtime`是`go`，因为我们在`fn init`时指定了`--runtime`参数。这里的`entrypoint`不应修改。就保持原样，信任我：

```
$ cat func.yaml

version: 0.0.1
runtime: go
entrypoint: ./func
```

Go 代码可以直接消费 STDIN。最好的方法是将输入作为 JSON 传递，并使用 Go 的`encoding/json`包来处理数据。以下是从原始 Fn 示例修改的程序。这个程序被修改为简化输出过程，并添加了错误检查和日志记录：

```
package main

import (
  "encoding/json"
  "fmt"
  "os"
)

type Message struct {
  Name string
}

func main() {
  m := &Message{Name: "world"}

  err := json.NewDecoder(os.Stdin).Decode(m)
  if err != nil {
    fmt.Fprintf(os.Stderr, "err JSON Decode: %s\n", err.Error())
    os.Exit(250)
  }

  fmt.Printf(`{"success": "Hello %s"}`, m.Name);
  os.Exit(0)
}
```

在每个程序中，我们都需要检查错误并处理它们。如前面的示例所示，我们检查编码时发生的错误，然后将错误消息打印到 Go 中的标准错误文件`os.Stderr`。然后我们只需使用代码`> 0`退出进程。在这里，我们使用`250`。

让我们总结一下 Fn 中的错误处理和日志记录。首先，写入 STDERR 的消息将被存储在日志中。其次，使用错误代码退出进程，即`> 0`。Fn 随后会将容器执行标记为*错误*。

让我们来看一下实际操作。确保我们在`func.go`中有之前的代码示例，并使用`fn deploy`命令将其部署：

```
$ fn deploy --app demo --registry chanwit

Deploying hello_go to app: demo at path: /hello_go
Bumped to version 0.0.2
Building image chanwit/hello_go:0.0.2 .......
Pushing chanwit/hello_go:0.0.2 to docker registry...The push refers to repository [docker.io/chanwit
/hello_go]
00a6a1467505: Pushed 
96252b84ae14: Pushed 
97dedccb7128: Mounted from fnproject/go 
c9e8b5c053a2: Mounted from fnproject/go 
0.0.2: digest: sha256:8a57737bff7a8e4444921959532716654230af0534b93dc6be247ac88e4e7ef2 size: 1155
Updating route /hello_go using image chanwit/hello_go:0.0.2...
```

如果`fn deploy`的最后一行显示路由已更新，那就表示已经准备好。

接下来，我们将使用`fn call`命令来调用该函数，该函数现在已经注册为`demo`应用程序下的一个路由。尝试在没有参数的情况下调用它，这会导致错误：

```
$ fn call demo /hello_go 

{"error":{"message":"container exit code 250"}}
ERROR: error calling function: status 502
```

这是我们预期的结果。它是一次没有输入的调用。因此，`encoding/json`抛出了错误，程序将日志消息写入了 STDERR（在之前的代码中没有显示）。最后，函数返回了`250`。通过这个消息，我们看到`fn call`打印出函数容器以`250`代码退出。所以错误得到了正确处理。

这里没有日志消息，但我们稍后会回到它们。

接下来，我们将进行一次成功的调用。为了使其显示为绿色，只需使用`echo`命令传递 JSON 主体。JSON 主体将通过管道传递给`fn call`，并转换为 HTTP 请求，然后它将被 Fn 服务器接收并再次序列化为函数程序的 STDIN。

成功的 JSON 块是我们对一个正常工作的程序的预期输出。

使用`fn call`调用远程函数的语法是，我们需要传递*应用程序名称*和*路由名称*，这样它才能被调用：

```
$ echo '{"Name": "chanwit"}' | fn call demo /hello_go 

{"success": "Hello chanwit"}
```

# 检查呼叫日志和错误

要查看所有调用日志，请使用`fn calls`命令。请注意，命令是带有 *s* 的`calls`。`fn calls list`命令接受应用程序的名称。需要关注的属性是`ID`和`Status`。以下示例显示了两个调用日志，第一个是`error`，第二个是`success`，按时间倒序排列：

```
$ fn calls list demo 

ID: 01C8VRGN9R47WGJ00000000000
App: demo
Route: /hello_go
Created At: 2018-03-18T05:15:04.376Z
Started At: 2018-03-18T05:15:04.738Z
Completed At: 2018-03-18T05:15:07.519Z
Status: success

ID: 01C8VRFE3647WGE00000000000
App: demo
Route: /hello_go
Created At: 2018-03-18T05:14:24.230Z
Started At: 2018-03-18T05:14:24.566Z
Completed At: 2018-03-18T05:14:27.375Z
Status: error
```

现在，我们选择第二个调用 ID 来获取日志消息。用于检索日志的命令是`fn logs get`。它需要应用程序名称和调用 ID：

```
$ fn logs get demo 01C8VRFE3647WGE00000000000
err JSON Decode: EOF
```

之前的日志消息是 Go 程序输出到`os.Stderr`的内容。

# 在 Docker Swarm 上部署 Fn

在这个示例中，我们在 Swarm 范围内的网络上启动一个 Fn 集群。

从部署网络开始，我们使用`weaveworks/net-plugin`作为骨干网络，以确保稳定性。请注意，网络必须是可附加的，并且子网必须位于`10.32.0.0/16`的范围内。所以，`10.32.3.0/24`在这里完全合适：

```
$ docker network create \
 --driver weaveworks/net-plugin:2.1.3
 --attachable \
 --subnet 10.32.3.0/24 \
 fn_net
```

然后，我们为数据存储准备一个卷。由于本节还希望展示一个产品级的设置，我们使用 MySQL 作为存储，而不是默认的 SQLite3。使用 MySQL 使我们能够横向扩展 Fn 服务器的数量。

卷将使用`docker volume create`命令创建。如果我们想要设置一个 MySQL 集群，设置会比这个稍微复杂一些，但这本书不会涉及：

```
$ docker volume create mysql_vol
```

这是启动 MySQL 实例的`docker run`命令。我们将实例连接到先前创建的网络`fn_net`。我们在此指定网络别名，以确保服务必须通过名称`mysql`来访问。所有环境变量的设计是为了设置用户名、密码和默认数据库`fn_db`。不要忘记将卷`mysql_vol`绑定到容器内的`/var/lib/mysql`。这是为了确保数据在容器被移除时仍然存活：

```
$ docker run \
 --detach \
 --name mysql \
 --network fn_net \
 --network-alias mysql \
 -e MYSQL_DATABASE=fn_db \
 -e MYSQL_USER=func \
 -e MYSQL_PASSWORD=funcpass \
 -e MYSQL_RANDOM_ROOT_PASSWORD=yes \
 -v mysql_vol:/var/lib/mysql \
 mysql
```

下一步是启动 Fn 服务器。本节展示了如何启动两个指向相同日志存储（MySQL）的 Fn 服务器。每个 Fn 服务器都连接到`fn_net`。第一个实例命名为`fn_0`。Fn 服务器需要`FN_DB_URL`来指向外部日志存储，可能是 PostgreSQL 或 MySQL。只需像以下命令中所示，输入完整的 URL。我们还将容器命名为`fn_0`，以便于管理。

当有这样的设置时，Fn 服务器变得完全无状态，所有状态将被存储在数据库外部。所以，当出现问题时，完全可以安全地移除 Fn 服务器容器：

```
$ docker run --privileged \
 --detach \
 --network fn_net \
 --network-alias fn_0 \
 --name fn_0 \
 -e "FN_DB_URL=mysql://func:funcpass@tcp(mysql:3306)/fn_db" \
 fnproject/fnserver
```

让我们启动另一个，`fn_1`。基本上，这应该在一个单独的节点（物理或虚拟）上完成：

```
$ docker run --privileged \
 --detach \
 --network fn_net \
 --network-alias fn_1 \
 --name fn_1 \
 -e "FN_DB_URL=mysql://func:funcpass@tcp(mysql:3306)/fn_db" \
 fnproject/fnserver
```

好的，在设置好所有 Fn Server 实例后，现在是时候将它们聚合在一起了。我们使用 Fn LB 作为所有 Fn Servers 前面的负载均衡器。与其他容器类似，我们只需创建并将其附加到 `fn_net`。作为 FaaS 网关，我们还将其端口暴露到 `8080`（从其内部端口 `8081`），使 Fn CLI 可以连接到 Fn 集群而无需任何特殊设置。网络别名仅在我们需要其他服务连接到该网关时使用。

接下来，发送一个 Fn Server 节点列表作为命令行参数。

当前，节点列表配置仅允许直接传递给容器。只需以 `<name>:<port>` 格式输入它们，并用 *逗号* 分隔：

```
$ docker run --detach \
 --network fn_net \
 --network-alias fnlb \
 --name fnlb \
 -p 8080:8081 \
 fnproject/fnlb:latest --nodes fn_0:8080,fn_1:8080
```

好的，现在是时候验证一切是否正常运行了。我们使用 `docker ps` 命令仔细检查所有容器：

```
$ docker ps --format "table {{.ID}}\t{{.Names}}\t{{.Command}}\t{{.Ports}}"
CONTAINER ID   NAMES     COMMAND                  PORTS
ce4f8e9bc300   fnlb      "./fnlb --nodes fn_0…"   0.0.0.0:8080->8081/tcp
dae4fb892b4d   fn_1      "preentry.sh ./fnser…"   2375/tcp
8aefeb9e19ef   fn_0      "preentry.sh ./fnser…"   2375/tcp
67bd136c331a   mysql     "docker-entrypoint.s…"   3306/tcp
```

在接下来的两部分中，我们将介绍如何通过 Fn UI 监控 Fn 的运行情况，以及如何查看并可能进一步分析存储在数据库中的日志。

# 使用 Fn UI 进行监控

Fn UI 是为 Fn 创建的用户界面项目。它提供了一个简单的仪表盘，并配备易于使用的时间序列图表，以便实时监控函数的运行情况。要启动 Fn UI，我们创建并将容器附加到 `fn_net`，同时将端口发布到 `4000`。Fn UI 需要一个 Fn Server 的 URL。但是它们都位于 Fn LB 后面，因此我们只需将 `FN_API_URL` 设置为 Fn LB 的位置。

请注意，它们都在 `fn_net` 网络内部相互连接，因此 URL 显示为 `http://fnlb:8081`，使用的是 `fnlb` 在网络中的实际名称和端口：

```
$ docker run --detach \
 --network fn_net \
 --network-alias fnui \
 -p 4000:4000 \
 -e "FN_API_URL=http://fnlb:8081" fnproject/ui
```

设置好 Fn UI 实例后，浏览到 `localhost:8080` 打开仪表盘。我们将看到列出的所有应用程序，如以下截图所示。也可以在此管理应用程序，例如创建或删除。如果不希望屏幕一直自动刷新，可以取消选中“自动刷新”：

![](img/b661769f-08bb-4570-b7ea-b308152d87e0.png)

图 5.5：显示 Fn 应用程序列表的 Fn 仪表盘

选择一个应用程序后，您可以通过单击仪表盘中的“运行函数”按钮来执行函数，如以下截图所示。如果在执行函数时发生错误并且执行失败，例如，将弹出通知，如以下示例所示。

要执行该函数，请将有效载荷以 JSON 形式放入并按下运行按钮：

![](img/ba0c4b6d-cf17-4869-8495-81fc624414f3.png)

图 5.6：用于调用函数的对话框

当函数调用完成时，它的名称和计数将出现在已完成的图表中。以下是调用函数的 `curl` 命令。多次运行它以查看图表变化：

```
$ curl -X POST -d '{"Name":"chanwit"}' http://localhost:8080/r/demo/hello_go
```

还有一个运行图表，显示了仍在并行运行的函数数量。以下截图展示了这些图表的运行情况：

![](img/8fd504f0-5a58-4c7c-a6c0-cb593dc8fd26.png)

图 5.7：显示 Fn 函数不同状态的图表

让我们看看当我们运行一些无效输入的请求时会发生什么。以下是命令：

```
$ curl -X POST -d '' http://localhost:8080/r/demo/hello_go
```

这样，`hello_go` 函数将以代码 `250` 退出，并出现在失败图表中。我们反复运行它，以使失败次数增加，如下图所示：

![](img/9df97d8e-b9fd-4735-bf73-159286be42c6.png)

图 5.8：右下方显示失败函数增量的图表

我们现在已经知道如何使用 Fn UI 来监控函数调用。接下来，我们将使用一个简单的 DBMS 界面来帮助浏览 Fn 服务器收集的日志。

# 使用 MyAdmin 查看呼叫日志

以 MySQL 作为中央日志存储，我们可以通过任何工具轻松访问 MySQL 以查询或分析日志。在这个例子中，我们使用一个简单的 MyAdmin 界面连接到 MySQL 后端。以下是启动 MyAdmin 的 `docker run` 命令。

我们只需将 MyAdmin 实例附加到相同的网络，并告诉 MyAdmin 连接到 `mysql`，即后端数据库的服务名称：

```
$ docker run --detach \
 --name myadmin \
 --network fn_net \
 --network-alias myadmin \
 -p 9000:80 \
 -e PMA_HOST=mysql \
 phpmyadmin/phpmyadmin
```

浏览到暴露的端口，在这个例子中是端口号 `9000`，并使用在 MySQL 设置期间设置的用户名和密码（`func`/`funcpass`）登录。以下截图显示了 phpMyAdmin 的登录页面：

![](img/2484860b-e84b-4181-b0b9-006df403a81a.png)

图 5.9：将连接到 Fn 日志数据库的 phpMyAdmin 登录页面

在 phpMyAdmin 面板内，查看 `fn_db` 参数，我们将看到用于存储 Fn 信息的所有表，如下图所示。表 `apps` 的数据是通过命令 `fn apps create` 创建的。例如，我们想要查看的是 `calls` 表和 `logs` 表。`calls` 表的内容可以通过 `fn calls list` 检索，`logs` 表的内容也可以通过类似的方式使用 `fn logs get` 检索。但是，当我们能够直接访问 `logs` 时，我们甚至可以直接使用可用数据进行一些分析：

![](img/244670aa-29fb-4a4c-abc4-125ad310557d.png)

图 5.10：phpMyAdmin 中所有 Fn 表的列表

以下截图显示了 `calls` 表的内容。表中有一个状态列，允许我们有效地筛选出呼叫的状态：成功或错误。还有一个 `stats` 列，包含一些时间信息，将由 Fn UI 检索并显示：

![](img/0dd3299e-c9ad-4236-aee5-b6e38705ae9f.png)

图 5.11：`calls` 表中的 Fn 呼叫日志数据

以下截图显示了 `logs` 表。在 `logs` 表中，它只是为每个条目打上呼叫 ID 的标记。`log` 列显示了我们打印到 STDERR 的日志消息。我们可以通过尝试向 `hello_go` 函数发送一些无效输入来查看不同的错误行为。由于这个表如此易于访问，我们可以有效地排除 Fn 函数的问题，而不需要安装其他额外的工具：

![](img/54b6697b-726d-40d1-b74b-9a9c5ed215cf.png)

图 5.12：从函数的 STDERR 捕获的 Fn 日志数据

好的，如果我们能够让 MyAdmin 显示日志数据，看来一切正常。最后，为了确认所有容器是否都在运行以及它们的状态，只需再次使用`docker ps`命令检查所有正在运行的容器：

```
$ docker ps --format "table {{.ID}}\t{{.Names}}\t{{.Command}}\t{{.Ports}}"
CONTAINER ID   NAMES     COMMAND                  PORTS
70810f341284   fnui      "npm start"              0.0.0.0:4000->4000/tcp
ce4f8e9bc300   fnlb      "./fnlb --nodes fn_0…"   0.0.0.0:8080->8081/tcp
dae4fb892b4d   fn_1      "preentry.sh ./fnser…"   2375/tcp
8aefeb9e19ef   fn_0      "preentry.sh ./fnser…"   2375/tcp
8645116af77d   myadmin   "/run.sh phpmyadmin"     9000/tcp, 0.0.0.0:9000->80/tcp
67bd136c331a   mysql     "docker-entrypoint.s…"   3306/tcp
```

# 练习

现在是时候回顾本章的所有内容了：

1.  Fn 架构是什么样的？

1.  该架构与其他 FaaS 平台有何不同？

1.  Fn 服务器的角色是什么？

1.  我们如何配置 Fn 服务器以使用外部数据存储？

1.  Fn 的 Java 运行时和 Go 运行时所使用的技术有何不同？

1.  应用程序和路由是如何组织的？

1.  Fn LB 的角色是什么？

1.  Fn UI 的角色是什么？

1.  我们如何查看之前调用的结果？

1.  我们如何检查失败调用的日志信息？

1.  描述一个 Fn 函数如何与 STDIN、STDOUT 和 STDERR 交互？

# 总结

本章讨论了 Fn 项目、其组件和架构。我们开始使用 Fn 及其命令行工具 Fn CLI。

接着我们讨论了 Fn 函数的结构，例如它如何与 STDIN、STDOUT 和 STDERR 交互。我们学习了如何构建和部署 Fn 函数，包括使用 Java 和 Go 运行时。

然后我们在 Docker Swarm 上形成了一个 Fn 集群，并将 Fn 服务器实例与外部数据库存储（MySQL）连接。我们使用 Fn LB（由同一团队专门实现的负载均衡器）对 Fn 实例进行了负载均衡。

使用 Fn UI，我们学会了如何利用它监控 Fn 的调用。通过 MyAdmin，我们能够直接在 MySQL 中浏览调用和错误日志。像 MyAdmin 这样的简单工具可以在不准备复杂工具链的情况下实现相同的分析结果。

下一章将介绍 OpenWhisk，这是 Apache 项目中的另一个无服务器技术栈，以及 IBM 在其云中提供无服务器服务所使用的技术栈。
