# 使用 Docker 容器的持续集成

这是一种矛盾但真实的说法：我们知道的越多，在绝对意义上我们变得越无知，因为只有通过启蒙，我们才能意识到自己的局限性。正是知识演化最令人欣慰的成果之一，就是不断开辟新的、更广阔的前景。

—尼古拉·特斯拉

要充分理解 Docker Swarm 所带来的挑战和好处，我们需要从头开始。我们需要回到代码库，决定如何构建、测试、运行、更新以及监控我们正在开发的服务。尽管目标是实现对 Swarm 集群的持续部署，但我们需要退后一步，首先探讨 **持续集成** (**CI***)*。我们为 CI 过程定义的步骤将决定我们如何朝着 **持续交付** (**CD***)* 迈进，再从那里迈向 **持续部署** (**CDP***)*，最后确保我们的服务得到监控并能够自我修复。本章将探讨持续集成，作为更高级过程的前提条件。

**致《DevOps 2.0 工具包》读者的说明** 以下内容与 *《DevOps 2.0 工具包》* 中发布的文本完全相同。如果你记得这些内容，随时可以跳到子章节 *定义完全 Docker 化的手动持续集成流程*。自从我写了 *2.0* 版后，我发现了一些更好的方式来实现 CI 过程。即使你已经是 CI 方面的老手，我希望你仍能从本章中受益。

要理解持续部署，我们首先应该定义它的前身——持续集成和持续交付。

项目开发的集成阶段往往是 **软件开发生命周期** (**SDLC**) 中最痛苦的阶段之一。我们会花费数周、数月甚至数年时间，分别在不同团队中为各自的应用程序和服务工作。每个团队都有自己的一组需求，并尽力满足这些需求。尽管定期独立验证这些应用程序和服务并不困难，但我们都害怕那一刻的到来——团队负责人决定是时候将它们集成到一个统一的交付中。凭借之前项目的经验，我们知道集成将会很有问题。我们知道会发现问题、未满足的依赖、接口之间的通讯不畅，而管理人员将会感到失望、沮丧和紧张。花费数周甚至数个月时间来处理这个阶段并不罕见。最糟糕的是，在集成阶段发现的一个 bug 可能意味着需要回头重做几天或几周的工作。如果当时有人问我对集成的看法，我会说，这是我离永久抑郁症最接近的状态。那是不同的时代，我们曾认为那是开发应用程序的 *正确* 方式。

自那以后，发生了许多变化。**极限编程**（**XP**）和其他敏捷方法变得普及，自动化测试变得频繁，持续集成开始占据主导地位。今天我们知道，当时我们开发软件的方式是错误的。从那些日子起，行业已经走了很长的路。

持续集成通常指的是在开发环境中集成、构建和测试代码。它要求开发人员经常将代码集成到共享代码库中。多频繁才算“经常”可以有多种解释，这取决于团队的规模、项目的大小以及我们投入编码的时间。在大多数情况下，这意味着开发人员要么直接推送代码到共享代码库，要么将代码与共享代码库进行合并。不管是推送还是合并，在大多数情况下，这些操作应当至少每天进行几次。将代码推送到共享代码库并不够，我们需要一个流水线，至少要检查代码并运行所有与代码相关的测试（无论是直接还是间接的）。流水线执行的结果可以是*红色*或者*绿色*。即要么某些东西失败了，要么所有的测试都没有问题。在前者的情况下，最基本的行动是通知提交代码的人。

持续集成流水线应该在每次提交或推送时运行。与持续交付不同，持续集成并没有明确的流水线目标。说一个应用与其他应用集成并没有告诉我们它的生产准备情况。我们并不知道还需要做多少工作才能将代码交付到生产环境。我们唯一追求的目标是确保一次提交没有破坏任何现有的测试。然而，当做得对时，CI 是一个巨大的改进。在许多情况下，它是一个非常难以实施的实践，但一旦每个人都适应了它，通常结果会非常令人印象深刻。

集成测试需要与实现代码一起提交，或者至少在实现代码之前提交。为了获得最大效益，我们应当采用**测试驱动开发**（**TDD**）的方式来编写测试。这样，不仅测试能够和实现代码一起提交，而且我们知道测试没有问题，无论我们做什么都不会失败。TDD 带来了许多其他好处，如果你还没有采用它，我强烈建议你采纳。你可能想参考*测试驱动开发*（[`technologyconversations.com/category/test-driven-development/`](http://technologyconversations.com/category/test-driven-development/)）部分，位于*技术对话*（[`technologyconversations.com/`](http://technologyconversations.com/)）博客中。

测试并不是唯一的 CI 前提条件。最重要的规则之一是，当流水线失败时，修复问题的优先级高于其他任何任务。如果这项工作被推迟，那么下一次流水线执行也会失败。人们将开始忽略失败通知，渐渐地，CI 流程将失去其意义。我们越早解决在 CI 流水线执行过程中发现的问题，我们就越好。如果立即采取纠正措施，那么关于问题潜在原因的知识仍然是新鲜的（毕竟，从提交到失败通知只有几分钟的时间），修复起来应该是微不足道的。

# 定义一个完全 Docker 化的手动持续集成流程

每个持续集成过程都从从仓库中检出的代码开始。本书中，我们将使用 GitHub 仓库 `vfarcic/go-demo` ([`github.com/vfarcic/go-demo`](https://github.com/vfarcic/go-demo))。它包含了本书中将使用的服务的代码。该服务是用 *Go* ([`golang.org/`](https://golang.org/)) 编写的。别担心！尽管我认为它是目前最好的编程语言之一，你并不需要学习 Go。本书将使用 go-demo 服务仅作为展示整个流程的示范。虽然我强烈推荐学习 Go，但本书不假设你有任何关于该语言的知识。所有的示例都将是与编程语言无关的。

本章节中的所有命令都可以在 `01-continuous-integration.sh` ([`gist.github.com/vfarcic/886ae97fe7a98864239e9c61929a3c7c`](https://gist.github.com/vfarcic/886ae97fe7a98864239e9c61929a3c7c)) Gist 中找到。

**Windows 用户提示**

请确保你的 Git 客户端已配置为按原样检出代码 *AS-IS*。否则，Windows 可能会将回车符转换为 Windows 格式。

让我们开始吧，首先检出 `go-demo` 代码：

```
git clone https://github.com/vfarcic/go-demo.git

cd go-demo

```

有些文件将在主机文件系统和我们即将创建的 Docker Machines 之间共享。Docker Machine 使当前用户所属的整个目录可以在虚拟机内访问。因此，请确保代码已检出到用户的某个子文件夹内。

既然我们已经从仓库中检出了代码，接下来我们需要一台服务器来构建和运行测试。暂时，我们将使用 Docker Machine，因为它提供了一种在我们的笔记本上轻松创建“Docker 就绪”虚拟机的方式。

*Docker Machine* ([`docs.docker.com/machine/overview/`](https://docs.docker.com/machine/overview/)) 是一款工具，允许你在虚拟主机上安装 Docker 引擎，并使用 `docker-machine` 命令来管理主机。你可以使用 Docker Machine 在本地的 Mac 或 Windows 主机、公司网络、数据中心，或者像 AWS 或 DigitalOcean 这样的云服务提供商上创建 Docker 主机。

使用`docker-machine`命令，你可以启动、检查、停止和重启托管主机，升级 Docker 客户端和守护进程，并配置 Docker 客户端与主机通信。

在*Docker v1.12*之前，Machine 是唯一在 Mac 或 Windows 上运行 Docker 的方法。从 Beta 版本和*Docker v1.12*开始，Docker for Mac 和 Docker for Windows 作为本地应用程序发布，是更新桌面和笔记本电脑上的更好选择。我鼓励你尝试这些新应用。Docker for Mac 和 Docker for Windows 的安装程序包括 Docker Machine 以及 Docker Compose。

以下示例假设你使用的是*Docker Machine v0.9*（[`www.docker.com/products/docker-machine`](https://www.docker.com/products/docker-machine)），其中包括*Docker Engine v1.13+*（[`www.docker.com/products/docker-engine`](https://www.docker.com/products/docker-engine)）。安装说明可以在*Install Docker Machine*（[`docs.docker.com/machine/install-machine/`](https://docs.docker.com/machine/install-machine/)）页面找到。

**Windows 用户注意**

推荐使用*Git Bash*运行所有示例（通过*Docker Toolbox*以及 Git 安装）。这样，你将在本书中看到的命令与在*OS X*或任何*Linux*发行版上执行的命令相同。**Linux 用户注意**

在 Linux 上，Docker Machine 可能无法将主机卷挂载到虚拟机中。问题与主机和 Docker Machine 操作系统都使用`/home`目录有关。挂载主机的`/home`目录会覆盖一些必要的文件。如果你遇到挂载主机卷的问题，请导出`VIRTUALBOX_SHARE_FOLDER`变量：

`export VIRTUALBOX_SHARE_FOLDER="$PWD:$PWD"`

如果机器已经创建，你需要销毁它们并重新创建。

请注意，这个问题应该在更新版本的 Docker Machine 中得到解决，所以只有在你注意到卷没有被挂载（主机中的文件在虚拟机内不可用）时才使用这个解决方法。

让我们创建第一个名为`go-demo`的服务器，使用以下命令：

```
docker-machine create -d virtualbox go-demo

```

**Windows 用户注意**

如果你使用的是*Docker for Windows*而非*Docker Toolbox*，你需要将驱动程序从 virtualbox 更改为 Hyper-V。问题在于，Hyper-V 不允许挂载主机卷，因此在使用*Docker Machine*时，仍然强烈建议使用*Docker Toolbox*。选择在*Docker Machines*内运行*Docker*而不是直接在主机上运行的原因在于需要运行集群（将在下一章介绍）。*Docker Machine*是模拟多节点集群的最简单方法。

该命令应该是自解释的。我们指定了 virtualbox 作为驱动程序（或者如果你使用*Docker for Windows*则是 Hyper-V），并将机器命名为`go-demo`：

**Windows 用户注意**

在某些情况下，Git Bash 可能会认为它仍然以 BAT 模式运行。如果你在运行 `docker-machine env` 命令时遇到问题，请导出 `SHELL` 变量：

`export SHELL=bash`

现在机器已经在运行，我们应该指示本地的 Docker 引擎使用它，使用以下命令：

```
docker-machine env go-demo

```

`docker-machine env go-demo`命令输出本地引擎需要的环境变量，以便找到我们想要使用的服务器。在这种情况下，远程引擎位于我们通过`docker-machine create`命令创建的虚拟机中。

输出如下：

```
export DOCKER_TLS_VERIFY="1"
export DOCKER_HOST="tcp://192.168.99.100:2376"
export DOCKER_CERT_PATH="/Users/vfarcic/.docker/machine/machines/go-demo"
export DOCKER_MACHINE_NAME="go-demo"

```

我们可以将`env`命令封装进一个`eval`命令中，该命令会评估输出，并在这种情况下，使用以下命令创建环境变量：

```
eval $(docker-machine env go-demo)

```

从现在开始，我们在本地执行的所有 Docker 命令都会通过 `go-demo` 机器内的引擎执行。

现在我们准备好运行 CI 流程中的前两步。我们将执行单元测试并构建服务二进制文件。

# 运行单元测试并构建服务二进制文件

我们将使用 Docker Compose 来执行 CI 流程。正如你很快会看到的，Docker Compose 在操作集群时几乎没有任何价值。然而，对于应该在单一机器上执行的操作，Docker Compose 仍然是最简单且最可靠的方式。

Compose 是一个定义和运行多容器 Docker 应用程序的工具。使用 Compose，你可以通过 Compose 文件来配置应用程序的服务。然后，使用一个命令，你就可以从配置中创建并启动所有服务。Compose 非常适合开发、测试、暂存环境以及 CI 工作流。

我们之前克隆的仓库已经在`docker-compose-test-local.yml`文件中定义了我们需要的所有服务([`github.com/vfarcic/go-demo/blob/master/docker-compose-test-loc`](https://github.com/vfarcic/go-demo/blob/master/docker-compose-test-loc))。

让我们来看看`docker-compose-test-local.yml`文件的内容：([`github.com/vfarcic/go-demo/blob/master/docker-compose-test-local.yml`](https://github.com/vfarcic/go-demo/blob/master/docker-compose-test-local.yml))

```
cat docker-compose-test-local.yml

```

我们将用于单元测试的服务名为`unit`，它如下所示：

```
unit:
    image: golang:1.6
    volumes:
      - .:/usr/src/myapp
      - /tmp/go:/go
    working_dir: /usr/src/myapp
    command: bash -c "go get -d -v -t && go test --cover -v \
./... && go build -v-o go-demo"

```

这是一个相对简单的定义。由于服务是用*Go*编写的，我们使用的是`golang:1.6`镜像。

接下来，我们将暴露一些卷。卷是指在这种情况下挂载到主机上的目录。它们通过两个参数定义。第一个参数是主机目录的路径，第二个表示容器内部的目录。任何已经存在于主机目录中的文件都将可以在容器内访问，反之亦然。

第一个卷用于源文件。我们将当前的主机目录 `.` 与容器目录 `/usr/src/myapp` 共享。第二个卷用于*Go*库。由于我们希望避免每次运行单元测试时都下载所有依赖项，因此它们将存储在主机目录 `/tmp/go` 中。这样，依赖项只有在第一次运行服务时才会被下载。

卷后面跟着 `working_dir` 指令。当容器运行时，它将使用指定的值作为起始目录。

最后，我们指定了要在容器内运行的命令。我不会详细介绍这些命令，因为它们特定于*Go*。简而言之，我们下载所有依赖项 `go get -d -v -t`，运行 `unit` 测试 `go test --cover -v ./...`，并构建 go-demo 二进制文件 `go build -v -o go-demo`。由于包含源代码的目录已作为卷挂载，因此二进制文件将存储在主机上并可供后续使用。

通过这个单一的 Compose 服务，我们定义了 CI 流程的两个步骤。它包含单元测试和二进制文件的构建。

请注意，尽管我们运行名为 `unit` 的服务，但该 CI 步骤的实际目的是运行任何不需要部署的测试。这些测试是在构建二进制文件之前，我们可以执行的测试，稍后还可以运行 Docker 镜像。

让我们运行以下代码：

```
docker-compose \ 
    -f docker-compose-test-local.yml \ 
    run --rm unit

```

**Windows 用户注意**

你可能会遇到卷未正确映射的问题。如果你看到 `Invalid volume specification error` 错误，请将环境变量 `COMPOSE_CONVERT_WINDOWS_PATHS` 导出并设置为 `0`：

`export COMPOSE_CONVERT_WINDOWS_PATHS=0`

如果这解决了卷的问题，请确保每次运行 `docker-compose` 时都导出该变量。

我们指定 Compose 使用 `docker-compose-test-local.yml` 文件（默认是 `docker-compose.yml`）并运行名为 `unit` 的服务。`--rm` 参数表示容器停止后应被移除。该命令适用于那些不打算长期运行的服务。它非常适合批处理任务，以及在本例中用于运行测试。

从输出中可以看到，我们拉取了 `golang` 镜像，下载了服务依赖项，成功运行了测试，并构建了二进制文件。

我们可以通过列出当前目录中的文件来确认二进制文件确实已构建并且可以在主机上使用。为了简便起见，我们将过滤结果：

```
ls -l *go-demo*

```

现在我们通过了第一轮测试并且有了二进制文件，我们可以继续构建 Docker 镜像。

# 构建服务镜像

Docker 镜像是通过存储在 Dockerfile 中的定义来构建的。除少数例外，它采用的方式类似于定义一个简单的脚本。我们不会探索定义 Dockerfile 时可以使用的所有选项，而仅会讲解 `go-demo` 服务使用的选项。有关更多信息，请参考 *Dockerfile 参考*（[`docs.docker.com/engine/reference/builder/`](https://docs.docker.com/engine/reference/builder/)）页面。

[`go-demo`](https://github.com/vfarcic/go-demo/blob/master/Dockerfile) Dockerfile 如下所示：

```
FROM alpine:3.4
MAINTAINER Viktor Farcic <viktor@farcic.com>

RUN mkdir /lib64 && ln -s /lib/libc.musl-x86_64.so.1 /lib64/ld-linux-x86-64.so.2

EXPOSE 8080
ENV DB db
CMD ["go-demo"]
HEALTHCHECK --interval=10s CMD wget -qO- localhost:8080/demo/hello

COPY go-demo /usr/local/bin/go-demo
RUN chmod +x /usr/local/bin/go-demo

```

每个语句将构建为一个独立的镜像。容器是多个镜像层叠在一起的集合。

每个 Dockerfile 都以 `FROM` 语句开始。它定义了应使用的基础镜像。在大多数情况下，我更倾向于使用 `alpine` Linux。由于它的大小约为 2MB，可能是我们可以使用的最小的发行版。这与容器应该只包含所需的内容，并避免任何额外开销的理念一致。

`MAINTAINER` 仅用于提供信息。

`RUN` 语句执行作为其参数设置的任何命令。由于它非常具体于我们正在构建的服务，我就不再解释了。

`EXPOSE` 语句定义了服务监听的端口。接下来是环境变量 `DB` 的定义，它告诉服务数据库的地址。默认值为 `db`，如你将很快看到的，它可以在运行时进行更改。`CMD` 语句表示容器启动时将执行的命令。

`HEALTHCHECK` 指令告诉 Docker 如何测试容器，以确保它仍然正常工作。这可以检测出诸如 Web 服务器卡在无限循环中，无法处理新连接的情况，尽管服务器进程仍在运行。当容器指定了健康检查时，它将拥有健康状态，除了正常状态外。该状态最初为“启动中”。每当健康检查通过时，它就变为“健康”（无论之前处于什么状态）。在连续失败达到一定次数后，它会变为“不健康”。

在我们的例子中，健康检查将每十秒执行一次。该命令向 API 端点之一发送一个简单的请求。如果服务返回状态 `200`，`wget` 命令将返回 `0`，Docker 会认为服务是健康的。任何其他响应都将被视为不健康，Docker 引擎会采取某些措施来修复该情况。

最后，我们将 `go-demo` 二进制文件从主机复制到镜像内的 `/usr/local/bin/` 目录，并使用 `chmod` 命令赋予其可执行权限。

对某些人来说，语句的顺序可能看起来不太合逻辑。然而，这些声明及其顺序背后是有充分理由的。那些不太可能更改的声明会先于那些容易更改的声明定义。由于 `go-demo` 每次构建镜像时都会是一个新的二进制文件，所以它被定义在最后。

这种顺序的原因在于 Docker 引擎创建镜像的方式。它从最上面的定义开始，检查自上次构建以来是否有变化。如果没有变化，它将跳到下一个语句。一旦找到可能会生成新镜像的语句，它和所有后续的语句都将构建成 Docker 镜像。通过将那些不太可能改变的部分放置在顶部，我们可以减少构建时间、磁盘使用量和带宽消耗。

现在我们理解了`go-demo`服务背后的 Dockerfile，我们可以构建镜像。

该命令非常简单，如下所示：

```
docker build -t go-demo .

```

作为替代，我们可以在 Docker Compose 文件中定义构建参数。`docker-compose-test-local.yml`文件中定义的服务如下：[`github.com/vfarcic/go-demo/blob/master/docker-compose-test-local.yml`](https://github.com/vfarcic/go-demo/blob/master/docker-compose-test-local.yml)

```
 app:
    build: .
    image: go-demo

```

在这两种情况下，我们都指定了当前目录应作为构建过程的工作目录`。`，并且镜像的名称为`go-demo`。

我们可以通过以下命令使用 Docker Compose 运行构建：

```
docker-compose \ 
    -f docker-compose-test-local.yml \ 
    build app

```

我们将在本书的其余部分中使用后一种方法。

我们可以通过执行`docker images`命令来确认镜像确实已经构建，如下所示：

```
docker images

```

输出如下：

```
REPOSITORY TAG    IMAGE ID     CREATED        SIZE
go-demo    latest 5e90126bebf1 49 seconds ago 23.61 MB
golang     1.6    08a89f0a4ee5 11 hours ago   744.2 MB
alpine     latest 4e38e38c8ce0 9 weeks ago    4.799 MB

```

如你所见，`go-demo`是我们服务器中存储的镜像之一。

现在镜像已经构建完成，我们可以运行依赖于该服务及其依赖项在服务器上部署的预发布测试。

# 运行预发布测试

请注意，此步骤在 CI 流程中的真正目的是运行那些需要服务及其依赖项正在运行的测试。这些还不是需要生产环境或类似生产环境的集成测试。这些测试的目的是将服务与其直接依赖项一起运行，执行测试，并在测试完成后移除所有内容并释放资源以进行其他任务。由于这些还不是集成测试，所以某些（如果不是全部）依赖项可以是模拟对象。

由于这些测试的性质，我们需要将任务分为三个步骤：

1.  运行服务及所有依赖项。

1.  运行测试。

1.  销毁服务及所有依赖项。

依赖项在`docker-compose-test-local.yml`文件中被定义为`staging-dep`服务。该定义如下：[`github.com/vfarcic/go-demo/blob/master/docker-compose-test-local.yml`](https://github.com/vfarcic/go-demo/blob/master/docker-compose-test-local.yml)

```
 staging-dep:
    image: go-demo
    ports:
      - 8080:8080
    depends_on:
      - db

  db:
    image: mongo:3.2.10

```

该镜像为`go-demo`，并且暴露了`8080`端口（在主机和容器内均为该端口）。它依赖于名为`db`的`mongo`镜像。定义为`depends_on`的服务将在定义该依赖关系的服务之前运行。换句话说，如果我们运行`staging-dep`目标，Compose 会首先运行`db`。

让我们按照以下代码运行依赖项：

```
docker-compose \ 
    -f docker-compose-test-local.yml \
    up -d staging-dep

```

一旦命令完成，我们将有两个容器在运行（`go-demo`和`db`）。我们可以通过列出所有进程来确认这一点：

```
docker-compose \ 
    -f docker-compose-test-local.yml \
    ps

```

输出如下：

```
Name                 Command               State Ports
---------------------------------------------------------------------
godemo_db_1          /entrypoint.sh mongod Up    27017/tcp
godemo_staging-dep_1 go-demo               Up    0.0.0.0:8080->8080/tcp

```

现在，服务及其依赖的数据库正在运行，我们可以执行测试。它们被定义为服务暂存。其定义如下：

```
 staging:
    extends:
      service: unit
    environment:
      - HOST_IP=localhost:8080
    network_mode: host
command: bash -c "go get -d -v -t && go test --tags integration -v"

```

由于暂存测试的定义与我们作为单元测试运行的测试非常相似，暂存服务扩展了单元测试。通过扩展服务，我们继承了它的完整定义。接下来，我们定义了一个环境变量`HOST_IP`。测试代码使用该变量来确定待测试服务的位置。在此情况下，由于`go-demo`服务与测试在同一服务器上运行，因此 IP 是服务器的本地主机。由于默认情况下，容器内的 localhost 与主机上的 localhost 不相同，我们不得不将`network_mode`定义为`host`。最后，我们定义了应该执行的命令。它将下载测试依赖项`go get -d -v -t`并运行测试`go test --tags integration -v`。

让我们运行以下命令：

```
docker-compose \ 
    -f docker-compose-test-local.yml \ 
    run --rm staging

```

所有测试都通过了，我们离实现完全确信服务确实可以安全部署到生产环境的目标又近了一步。

我们不再需要保持服务和数据库运行，因此让我们将它们移除，释放资源以供其他任务使用：

```
docker-compose \ 
    -f docker-compose-test-local.yml \ 
    down

```

`down`命令停止并移除在该 Compose 文件中定义的所有服务。我们可以通过运行以下`ps`命令来验证这一点：

```
docker-compose \ 
    -f docker-compose-test-local.yml \ 
    ps

```

输出如下：

```
Name   Command   State   Ports
------------------------------

```

只有一件事还缺少，才能完成 CI 流程。目前，我们有一个只在 go-demo 服务器内部可用的`go-demo`镜像。我们应该将其存储在一个注册表中，以便其他服务器也能访问它。

# 将镜像推送到注册表

在推送我们的`go-demo`镜像之前，我们需要一个目标位置来推送。Docker 提供了多种作为注册表的解决方案。我们可以使用*Docker Hub*（[`hub.docker.com/`](https://hub.docker.com/)）、*Docker Registry*（[`docs.docker.com/registry/`](https://docs.docker.com/registry/)）和*Docker Trusted Registry*（[`docs.docker.com/docker-trusted-registry/`](https://docs.docker.com/docker-trusted-registry/)）。除此之外，还有许多第三方供应商提供的其他解决方案。

我们应该使用哪个注册表？Docker Hub 需要用户名和密码，而我不够信任你来提供我的密码。开始编写本书之前，我定义的一个目标是只使用开源工具，因此 Docker Trusted Registry，尽管在不同的情况下是一个很好的选择，但也不适合。剩下的唯一选择（不包括第三方解决方案）是*Docker Registry*（[`docs.docker.com/registry/`](https://docs.docker.com/registry/)）。

注册表在 `docker-compose-local.yml` ([`github.com/vfarcic/go-demo/blob/master/docker-compose-local.yml`](https://github.com/vfarcic/go-demo/blob/master/docker-compose-local.yml)) 这个 Compose 文件中被定义为其中的一个服务。定义如下：

```
 registry:
    container_name: registry
    image: registry:2.5.0
    ports:
      - 5000:5000
    volumes:
      - .:/var/lib/registry
    restart: always

```

我们将注册表设置为一个明确的容器名称，指定了镜像，并打开了 `5000` 端口（主机和容器内都开放）。

注册表将镜像存储在 `/var/lib/registry` 目录下，因此我们将其挂载为主机上的一个卷。这样，如果容器失败，数据就不会丢失。由于这是一个生产环境服务，可能会被许多人使用，我们定义了在失败时它应始终重启。

让我们运行以下命令：

```
docker-compose \ 
    -f docker-compose-local.yml \ 
    up -d registry

```

现在我们已经有了注册表，可以进行一次干运行。让我们确认我们能否从中拉取并 `push` 镜像：

```
docker pull alpine

docker tag alpine localhost:5000/alpine

docker push localhost:5000/alpine

```

Docker 使用命名约定来决定从哪里拉取和推送镜像。如果名称前面有地址，Docker 引擎将利用这个地址来确定注册表的位置。否则，系统会默认我们要使用 Docker Hub。因此，第一条命令从 Docker Hub 拉取了 alpine 镜像。

第二条命令创建了 alpine 镜像的标签。这个标签是我们注册表地址 `localhost:5000` 和镜像名称的组合。最后，我们将 `alpine` 镜像推送到了同一台服务器上的注册表。

在我们开始更正式地使用注册表之前，先确认镜像确实已经在主机上持久化：

```
ls -1 docker/registry/v2/repositories/alpine/

```

输出如下：

```
_layers
_manifests
_uploads

```

我不会详细说明这些子目录各自包含的内容。需要注意的重要事项是，注册表在主机上持久化存储镜像，因此如果它失败，或者在这种情况下，即使我们销毁了虚拟机，由于该机器目录映射到我们笔记本上的相同目录，数据也不会丢失。

我们在声明这个注册表应该用于生产时有点匆忙。即使数据得到了持久化，如果整个虚拟机崩溃，仍会有停机时间，直到有人重新启动它或创建一个新的。由于我们的目标之一是尽可能避免停机，因此稍后我们应该寻找更可靠的解决方案。目前的设置暂时可以使用。

现在我们准备将 `go-demo` 镜像推送到注册表：

```
docker tag go-demo localhost:5000/go-demo:1.0

docker push localhost:5000/go-demo:1.0

```

和 Alpine 示例一样，我们使用注册表前缀标记了镜像并将其推送到注册表。我们还添加了版本号 `1.0`。

推送是持续集成流程中的最后一步。我们运行了单元测试，构建了二进制文件，构建了 Docker 镜像，运行了预发布测试，并将镜像推送到了注册表。尽管我们完成了所有这些步骤，但我们仍然不确定服务是否已准备好进入生产环境。我们从未测试过它在部署到生产（或类似生产环境的）集群后会如何表现。我们做了很多，但还不够。

如果持续集成是我们的最终目标，那么此时应该进行手动验证。虽然手动工作中有很多需要创造力和批判性思维的价值，但我们不能对重复性任务说同样的话。将这个持续集成流程转化为持续交付，并最终实现部署的任务，确实是重复性的。

我们已经完成了 CI 流程，现在是时候再多做一步，将其转换为持续交付（Continuous Delivery）。

在进入将持续集成（Continuous Integration）过程转化为持续交付（Continuous Delivery）的步骤之前，我们需要退后一步，探索集群管理。毕竟，在大多数情况下，没有集群就没有生产环境。

我们将在每一章结束时销毁虚拟机。这样，你可以随时回到书中的任何部分并进行练习，而不必担心可能需要执行前面章节中的某些步骤。此外，这样的操作会迫使我们重复一些内容，熟能生巧。为了减少你的等待时间，我尽力将事情做到尽可能小，并将下载时间降到最低。执行以下命令：

```
docker-machine rm -f go-demo

```

下一章将专门讲解 Swarm 集群的设置和操作。
