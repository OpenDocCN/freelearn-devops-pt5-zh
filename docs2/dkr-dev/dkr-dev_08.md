# *第六章*：使用 Docker Compose 部署应用

使用 Docker 部署应用的最简单实用场景是通过在单主机上运行 Docker Compose。你作为开发人员使用的许多命令，例如 `docker-compose up -d`，同样适用于在单主机上部署 Docker 应用。

在单主机上运行 Docker 应用比使用更复杂的容器编排系统运行它们更容易理解，因为你可以使用许多与运行非 Docker 应用相同的技术；然而，在性能和可用性方面，它存在一些显著的缺点。

在本章中，你将发现为何这是最简单的实用选项，学习如何为单主机生产环境配置 Docker，并掌握一些高效管理和监控简单部署的技巧。此外，你还将更好地理解在单主机上运行 Docker 的缺点，包括可能面临的问题。

本章将涵盖以下主要主题：

+   为单主机部署选择主机和操作系统

+   为 Docker 和 Docker Compose 准备主机

+   使用配置文件和支持脚本进行部署

+   监控小型部署——日志记录和警报

+   单主机部署的限制

# 技术要求

要完成本章的练习，你需要在本地工作站上安装 Git 和 Docker，并且需要一台能够运行 Linux 和 Docker 的单主机作为生产服务器，该主机需连接到一个可以通过 SSH 访问且用户能够访问的网络。

本章的 GitHub 仓库可以在 [`github.com/PacktPublishing/Docker-for-Developers`](https://github.com/PacktPublishing/Docker-for-Developers) 找到——请参阅 `chapter6` 文件夹。

查看以下视频，查看代码实际应用：

[`bit.ly/31OSi1H`](https://bit.ly/31OSi1H)

## 示例应用 – ShipIt Clicker v2

本章中的 ShipIt Clicker 版本比我们在*第五章*中使用的版本更加完善，*部署和运行生产环境中的容器的替代方案*。它具有以下功能：

+   改进的 Dockerfile 和适合基本生产环境使用的 `docker-compose.yml` 文件

+   在 Redis 中存储游戏状态并与服务器会话绑定，导致不同客户端设备的游戏状态各异

+   改进的视觉和音频资源

我们将使用这个增强版本的 ShipIt Clicker 作为应用，通过 Docker Compose 在单主机上进行部署。

# 为单主机部署选择主机和操作系统

在单一主机上部署应用是生产环境中最简单的运行方式。从许多方面来看，它类似于使用 Docker 和 Docker Compose 进行本地开发的用户体验。如果你能够通过 `docker-compose.yml` 文件打包应用的各个部分，那么你已经完成了 70% 的工作。如果你已经具备基本的 UNIX 或 Linux 系统管理技能，那么这将非常简单——这个策略所需的努力最少，你可以在一到两个小时内掌握要点。

## 单主机部署的要求

为了继续进行部署，你需要一台运行现代 Linux 操作系统且架构与开发系统相同的计算机，且该计算机应具备足够的内存、处理器和存储容量来运行你的应用。如果你是在使用 Docker Community Edition 的 Windows 10 64 位桌面上进行开发，你需要一台使用 x86_64 架构的 Linux 系统。如果你是在运行 Raspbian 的 Raspberry Pi 4 上使用 Docker，你需要一台 ARM 架构的服务器。实际上，你可以使用任何裸金属服务器或虚拟机服务器，无论是在本地还是云端，只要它支持 Docker。

一些云服务提供商，如 **Amazon Web Services** (**AWS**)，为其最小虚拟机部署提供免费层，至少是第一年。在本章中的示例可以在类似的主机上运行，但如果你的应用较大，你可能需要使用更大、更昂贵的系统。

生产环境中的应用通常需要 *24*7* 不间断运行，且这些应用的用户可能有可靠性方面的担忧。虽然在单一主机上运行 Docker 应用可能是最不可靠的方式，但对于某些应用来说，这可能已经足够。像 HP、Dell 和 IBM 这样的厂商提供的单主机可靠性措施在很多情况下已经足够确保应用的可靠性，前提是你的应用需要这种级别的可靠性。

你需要以下支持 Docker 的 Linux 操作系统发行版之一：

+   Red Hat Enterprise Linux（或 CentOS）7 或 8

+   Ubuntu 16.04 或 18.04 或更新版本

+   Amazon Linux 2

+   Debian Stretch 9

+   Buster 10

为了最小化生产时间并最大化便捷性，选择你已经熟悉的操作系统，或者使用 CentOS 7，后续示例中将使用此版本。

仅当你希望采用更慢且更高级的生产路径时，才选择专注于 Docker 的发行版，如 Container Linux 或 CoreOS，因为在这些环境下，你的系统管理技能可能效果不佳。例如，CoreOS 中的用户管理与主流发行版的方式大不相同。

因为这个策略仅依赖于用户能够访问的主机，你将拥有极大的灵活性。

# 为 Docker 和 Docker Compose 准备主机

在你配置主机上的软件之前，你应确保它具有一个稳定的 IP 地址。有时这些被称为静态 IP 地址，或者在 AWS 中称为弹性 IP 地址。你可能需要通过提供商特别分配这些 IP 地址，通常可以通过提供商的控制台进行操作，例如 AWS Lightsail 中的**网络**选项卡，或者 AWS EC2 控制台中的**弹性 IP**设置。

此外，你应该映射一个地址（例如使用`shipitclicker.example.com`而不是原始 IP 地址，如`192.2.0.10`）。所有公共云系统都能够管理 DNS 条目——例如，[AWS Route 53](https://docs.aws.amazon.com/route53/index.html)，大多数虚拟主机系统也具备此功能。

## 使用操作系统软件包安装 Docker 和 Git

你需要在主机上安装 Docker。对于生产环境，避免使用操作系统发行版中附带的过时 Docker 版本，尽量使用 Docker 为 Docker 社区版发布的操作系统软件包。你可以在 Docker 官网上找到针对各种操作系统的 Docker 社区版安装说明，具体如下：[链接](https://docs.docker.com/install/linux/docker-ce/centos/)

+   [**CentOS**: https://docs.docker.com/install/linu](https://docs.docker.com/install/linux/docker-ce/centos/)x/docker-[ce/centos/](https://docs.docker.com/install/linux/docker-ce/debian/)

+   [**Debian**: https://docs.docker.com/install/linu](https://docs.docker.com/install/linux/docker-ce/debian/)x/docker-[ce/debian/](https://docs.docker.com/install/linux/docker-ce/fedora/)

+   [**Fedora**: https://docs.docker.com/install/linu](https://docs.docker.com/install/linux/docker-ce/fedora/)x/docker-[ce/fedora/](https://docs.docker.com/install/linux/docker-ce/ubuntu/)

+   [**Ubuntu**: https://docs.docker.com/install/linu](https://docs.docker.com/install/linux/docker-ce/ubuntu/)x/docker-ce[/ubuntu/](https://docs.docker.com/install/linux/docker-ce/binaries/)

+   [**二进制文件**: https://docs.docker.com/install/linux/](https://docs.docker.com/install/linux/docker-ce/binaries/)docker-ce/binaries/

对于 CentOS 7 的全新安装，使用以下命令：

```
$ sudo yum install -y yum-utils
$ sudo yum install -y device-mapper-persistent-data lvm2 
$ sudo yum-config-manager --add-repo \
https://download.docker.com/linux/centos/docker-ce.repo 
$ sudo yum install -y docker-ce docker-ce-cli containerd.io
```

将你常用的非 root 用户添加到 Docker 用户组，并在当前终端会话中成为该组的成员：

```
$ sudo usermod -aG docker $USER
$ newgrp docker 
```

确保 Docker 服务已启用，这样它会在启动时自动启动，并且 Docker 服务已经启动：

```
$ sudo systemctl enable docker
$ sudo systemctl restart docker
```

按照[`docs.do`](https://docs.docker.com/compose/install/)cker.com/compose/install/上的说明安装`docker-compose`。截至 2020 年 1 月，`1.25.3`是最新版本，但请检查该页面上的版本号，使用最新版本填入以下命令，这应该是单行命令：

```
$ sudo curl -L "https://github.com/docker/compose/releases/download/1.25.3/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
$ sudo chmod +x /usr/local/bin/docker-compose
```

现在你已经启动并启用了 Docker 守护进程，并且安装了`docker-compose`，你可以部署你的应用程序了。

接下来，通过操作系统的包管理器安装 `git`。对于 Red Hat 系列的发行版（如 RHEL、CentOS、Fedora 和 Amazon Linux），请使用以下命令：

```
$ sudo yum install -y git
```

对于 Debian 系列的发行版（包括 Ubuntu），请运行以下命令：

```
$ sudo apt-get update && apt-get install -y git
```

到这时，主机已准备好部署 Docker 应用程序。为了完成部署，我们将使用一种依赖于 shell 脚本和 Docker 环境配置文件的策略。

# 使用配置文件和支持脚本进行部署

为了将我们的应用程序部署到生产服务器，我们将使用简单命令和支持脚本的组合来启动或更新正在运行的容器集。让我们首先仔细看看部署所需的两个最重要的文件：`Dockerfile` 和 `docker-compose.yml`。

## 重新审视初始 Dockerfile

来自 *第五章*的 Dockerfile，*在生产环境中部署和运行容器的替代方案*，具有良好的分层，并且在 `RUN npm -s install` 执行之前和将应用程序的主要部分复制到镜像之前，已经将 `package.json` 和 `package.json.lock` 复制到镜像中。然而，它也有一些不足之处，我们将在本章中进行改进，以便为生产部署做好充分准备。首先，让我们看看初始的 Dockerfile：

```
FROM ubuntu:bionic
RUN apt-get -qq update && \
    apt-get -qq install -y nodejs npm > /dev/null
RUN mkdir -p /app/public /app/server
COPY src/package.json* /app
WORKDIR /app
RUN npm -s install
COPY src/.babelrc \
     src/.env \
     src/.nodemonrc.json \
     /app/
COPY src/public/ /app/public/
COPY src/server/ /app/server/
EXPOSE 3000
ENTRYPOINT DEBUG='shipit-clicker:*' npm run dev
```

前面的 ShipIt Clicker 游戏原型的 Dockerfile 从本地开发的角度来看有很多地方是正确的，但也有一些限制，我们将在本章的 Dockerfile 中解决这些问题。

很多时候，开发人员会从一个基础镜像开始（例如 `FROM ubuntu:bionic`），这个镜像通常是他们最熟悉的：传统的 Linux 发行版，通常在工作站上运行。这可能有助于最初调试 Dockerfile，但代价很高，因为基础镜像和生成的镜像都非常大，包含数百兆字节。此外，Ubuntu 的软件包安装过程相当冗长，因此 `apt-get install` 命令必须将 `stdout` 重定向到 `/dev/null` 以防止冗长的输出 [占用终端](https://askubuntu.com/a/1134785)（参见 [`askubuntu.com/a/1134785`](https://askubuntu.com/a/1134785)）。

初始 Dockerfile 的其余部分有一些常见的缺陷，您应该避免在生产环境中使用，比如复制所有开发工具的配置文件（参见 `COPY` 命令，它复制点文件）。初始 Dockerfile 有一个入口点（`ENTRYPOINT`），它指向一个更适合开发的服务器，而非生产环境，因为这种方式定义起来快速而简便。真正的生产设置需要一个构建步骤来创建适合分发的资源集，还需要一个不同的 `npm` 命令来使用这些资源启动应用程序。

本章的 Dockerfile 修正了所有这些问题：

```
FROM alpine:20191114
RUN apk update && \
    apk add nodejs nodejs-npm
RUN addgroup -S app && adduser -S -G app app
RUN mkdir -p /app/public /app/server
ADD src/package.json* /app/
WORKDIR /app
RUN npm -s install
COPY src/public/ /app/public/
COPY src/server/ /app/server/
COPY src/.babelrc /app/
RUN npm run compile
USER app
EXPOSE 3000
ENTRYPOINT npm start
```

在这个修改后的 Dockerfile 中，我们使用了 Alpine Linux 而不是 Ubuntu，目的是生成更小的镜像，同时我们固定了 Alpine 的版本，以确保构建的一致性。基于 Alpine Linux 的容器镜像体积比 Ubuntu 小 71%：

```
$ docker images | awk '/chapter._ship/{ print $1 " " $7}'
chapter6_shipit-clicker-web-v2 154MB
chapter5_shipit-clicker-web 524MB
```

在修改后的 Dockerfile 中，我们还创建了一个`app`用户，这样 Docker 就可以以普通的 UNIX 用户身份运行应用程序，而不是`root`用户，因为后者可能会加剧安全问题。

在尽可能无声地安装操作系统包和`npm`包后，我们可以将应用程序文件和`.babelrc`配置文件复制到`/app`目录，然后运行`RUN npm run compile`来准备生产版本的节点应用程序，我们以`app`用户身份运行它，并使用`ENTRYPOINT npm start`。

## 重新审视初始的 docker-compose.yml 文件

前一章中的初始`docker-compose.yml`文件完成了启动 Web 和 Redis 容器的任务，但存在一些不足。初始的`docker-compose.yml`文件是从 Docker 文档中的简单示例[e[xample](https://docs.docker.com/compose/)改编而来，位于[`docs.docker.com/compose/`](https://docs.docker.com/compose/)，因此它在生产使用时还有一些缺陷。许多开发者在没有考虑到某些生产部署时需要注意的细微差异的情况下，直接适应这些示例。你可以将它视为起点，而非终点。初始的`docker-compose.yml`文件如下：

```
---
version: '3'
services:
  shipit-clicker-web:
    build: .
    environment:
      REDIS_HOST: redis
    ports:
    - "3005:3000"
    links:
    - redis
  redis:
    image: redis
    ports:
    - "6379:6379"
```

本章修改后的`docker-compose.yml`文件更加健壮。这个文件[部分灵感来自于 https://gith](https://github.com/docker-library/redis/issues/111)ub.com/docker-library/redis/issues/111，尤其是 GitHub 用户`@lagden`提供的示例，其中展示了一个支持 Redis 的`docker-compose.yml`文件的优秀示例：

```
---
version: '3'
services:
  shipit-clicker-web-v2:
    build: .
    environment:
        - APP_ID=shipit-clicker-v2
        - OPENAPI_SPEC=/api/v1/spec
        - OPENAPI_ENABLE_RESPONSE_VALIDATION=false
        - PORT=3000
        - LOG_LEVEL=${LOG_LEVEL:-debug}
        - REQUEST_LIMIT=100kb
        - REDIS_HOST=${REDIS_HOST:-redis}
        - REDIS_PORT=${REDIS_PORT:-6379}
        - SESSION_SECRET=${SESSION_SECRET:-mySecret-v2}
```

请注意，我们显式地为应用程序定义了所有环境变量，其中有几个变量使用`${VARIABLE_NAME:-default_value}`语法，该语法利用环境变量的值。可以在命令行、常见的配置文件（如：`$HOME/.profile`、`$HOME/.bashrc`）或者与`docker-compose.yml`文件在同一目录中的`.env`文件中指定这些变量：

```
 ports:
      - "${PORT:-3006}:3000"
    networks:
      - private-redis-shipit-clicker-v2
    links:
      - redis
    depends_on:
      - redis
```

上述的`ports`部分定义了主容器的网络配置；它定义了一个名为`private-redis-shipit-clicker-v2`的私有网络，该网络将两个容器连接起来。请注意，这部分使用了`depends_on`。这意味着 ShipIt Clicker 容器将在 Redis 容器启动后才会启动。接下来，让我们看看 Redis 容器的定义：

```
  redis:
    command: ["redis-server", "--appendonly", "yes"]
    image: redis:5-alpine3.10
    volumes:
      - redis-data-shipit-clicker:/data
    networks:
      - private-redis-shipit-clicker-v2
volumes:
  redis-data-shipit-clicker: {}
networks:
  private-redis-shipit-clicker-v2:
```

这个文件包含许多环境变量条目——例如 `LOG_LEVEL`、`REDIS_HOST` 和 `REDIS_PORT`——允许轻松覆盖。这使得重写 Redis 主机设置变得容易，无论是为了更方便的调试，还是为了便于连接到云端 Redis 服务。它通过命令行参数启动 Redis，并启用持久化，同时分配一个 Docker 持久卷来存储 Redis 的仅附加日志文件。否则，每次 Redis 容器重启时，数据都会丢失。它使得 Redis 和 Web 服务器之间的网络通信变为私密。这一点尤其重要，因为 Redis 的默认配置下，Redis 服务器在没有任何身份验证或授权的情况下运行——这意味着任何能够连接的人都可以访问！

在这个简约的、适用于生产环境的 `docker-compose.yml` 文件中，我们将 Web 服务器直接暴露在 `80` 端口上供全世界访问。这是可行的，但现代浏览器会对纯 HTTP 内容显示安全警告。虽然这能让你顺利进入生产环境，但许多生产应用程序需要比纯 HTTP 更强的安全保护。你可以通过使用代理或外部负载均衡器，在 `443` 端口终止 HTTPS，或者通过配置 SSL 证书来绕过这个问题。我们将在后续章节中详细介绍这一点。

`docker-compose` v3 配置的一个特点是它设置了容器失败时的默认行为为*始终重启*。即使主机重新启动，这也应该发生，如果进程由于未处理的异常退出，这种行为也一定会发生。如果你需要更直接地配置应用程序的重启行为，可以参考文档中列出的设置，链接为 [`docs.docker.com/compose/compose-file/#restart_policy`](https://docs.docker.com/compose/compose-file/#restart_policy)。

## 准备生产环境的 .env 文件

克隆仓库并准备配置 `docker-compose`：

```
$ git clone https://github.com/PacktPublishing/Docker-for-Developers.git
$ cd Docker-for-Developers/chapter6
```

为了配置生产环境中的应用程序，你应该在 `docker-compose.yml` 文件所在的目录中创建一个名为 `.env` 的文件。如果你想更改任何默认设置——例如，将生产环境中的调试级别从 `info` 改为 `debug`——你应该通过创建和编辑与生产部署相关的 `.env` 文件来进行修改。将 `env.sample` 文件复制为 `.env` 并根据你的生产需求进行编辑。

### 处理密钥

这个示例应用程序使用环境变量和 `.env` 文件来存储密钥。这符合[12 因子应用](https://12factor.net/config)原则（详见 https://12factor.net/config），但这并不是处理密钥的唯一方法，也不是最安全的方法。你可以使用密钥管理系统，例如 HashiCorp Vault 或 Amazon Secrets Manager，来存储和检索密钥。我们将在*第八章*《将 Docker 应用部署到 Kubernetes》和*第十四章*《高级 Docker 安全性 - 密钥、密钥命令、标签和标签》详细介绍这一点；但目前，我们先使用环境变量来处理密钥。

你应该用随机的密钥替换环境变量中的 `SESSION_SECRET` 密钥，并确认是否要将端口 `80` 暴露给外界。使用你熟悉的编辑器，不论是 `vi`、`emacs` 还是 `nano`：

```
cp env.sample .env
vi .env
```

设置好环境变量覆盖后，你可以部署应用程序。

### 首次部署

一旦你把 `.env` 文件放置好，后台启动服务以部署应用程序：

```
$ docker-compose up -d
```

按照以下步骤验证服务是否正在运行：

```
$ docker-compose ps
    Name           Command      State       Ports
-----------------------------------------------------
chapter6_redi   docker-         Up      6379/tcp
s_1             entrypoint.sh
                redis ...
chapter6_ship   /bin/sh -c      Up      0.0.0.0:80-
it-clicker-     npm start               >3000/tcp
web-v2_1
```

检查系统日志是否显示任何错误：

```
$ docker-compose logs
```

只要在日志中没有看到一连串的错误信息，你应该能够通过服务器的 [IP 地址](http://192.0.2.10)访问该网站，例如 `http://192.0.2.10`，用你的 IP 地址替代。如果你使用 DNS 分配了主机名，你应该可以[通过主机名访问它—例如](http://shiptclicker.example.com)，例如，[`shiptclicker.example.com`](http://shiptclicker.example.com)，用这个域名替代原始域名。

### 排查常见错误

如果你遇到类似的错误，你需要确保主机上没有运行其他的 Web 服务器，例如 Apache HTTPD 或 NGINX：

```
docker.errors.APIError: 500 Server Error: Internal Server Error ("b'Ports are not available: listen tcp 0.0.0.0:80: bind: address already in use'")
```

如果你遇到这个问题，你应该卸载主机上运行的 Web 服务器，或者更改它监听请求的端口。你也可以通过更改 `.env` 文件中的 `PORT` 变量，改变 ShipIt Clicker 运行的端口。对于 Red Hat 系列系统，监听端口 `80` 的服务器很可能是 Apache HTTPd，你可以使用以下命令来移除它：

```
$ yum remove -y httpd
```

对于 Debian 系列的系统，通常也可能是 Apache，你需要使用以下命令来移除它：

```
$ apt-get remove -y apache2
```

可能你的系统上运行着其他 Web 服务器。你可以使用 `netstat` 查找你 Web 服务器的进程名称：

```
$ sudo netstat -nap | grep :80
tcp6       0      0 :::80                   :::*                    LISTEN      12037/httpd 
```

你可能不需要进行故障排除就能让应用程序在 Docker 中运行，但在单主机部署场景下，你可以使用系统管理员的故障排除技能来找出可能出错的地方。

一旦应用程序运行起来，您可能会发现您经常运行一些相同的操作，例如在进行更改后重建应用程序。这就是支持脚本派上用场的地方。

## 支持脚本

在生产环境中运行站点时，您可能经常需要执行一些操作。记住重新启动和更新运行系统或连接到数据库所需的确切 Docker 命令序列变得很烦人。

您应该继续在本地工作站上开发您的应用程序，并使用生产系统将更改部署给您的用户。

在本章节中改进的网络设置中，不再可能通过直接 TCP 端口直接连接到 Redis 容器，因此我们将使用脚本内的`docker exec`来执行此操作。

如果你在`Docker-for-Developers``/chapter6`目录中，可以通过以下命令将该目录永久添加到`PATH`中，以便更方便地运行这些脚本：

```
$ echo "PATH=$PWD:$PATH" | tee -a "$HOME/.bash_profile"
$ . "$HOME/.bash_profile"
```

对于此应用程序来说，最常见的操作可能是重新启动应用程序，部署更改和连接到 Redis 进行故障排除。对于这些操作，我们将使用`restart.sh`脚本，`deploy.sh`脚本和`redis-cli.sh`脚本。

### 重新启动

`restart.sh`脚本将重新启动所有容器。在修改配置文件`.env`后，您应该运行此命令。您可以简单地运行`docker-compose up -d`，但仅此不足以告诉您更改是否生效。此命令还将为您运行`docker-compose ps`，以显示更改后您的容器是否正确运行，包括端口映射。在以下示例会话中，我们完全删除`.env`文件，然后仅使用`PORT=80`设置重新创建它：

```
[centos@ip-172-26-0-237 chapter6]$ rm .env
[centos@ip-172-26-0-237 chapter6]$ deploy.sh
chapter6_redis_1 is up-to-date
Recreating chapter6_shipit-clicker-web-v2_1 ... done
              Name                            Command               State           Ports         
--------------------------------------------------------------------------------------------------
chapter6_redis_1                   docker-entrypoint.sh redis ...   Up      6379/tcp              
chapter6_shipit-clicker-web-v2_1   npm start                        Up      0.0.0.0:3006->3000/tcp
[centos@ip-172-26-0-237 chapter6]$ echo 'PORT=80' > .env
[centos@ip-172-26-0-237 chapter6]$ restart.sh
chapter6_redis_1 is up-to-date
Recreating chapter6_shipit-clicker-web-v2_1 ... done
              Name                            Command               State          Ports        
------------------------------------------------------------------------------------------------
chapter6_redis_1                   docker-entrypoint.sh redis ...   Up      6379/tcp            
chapter6_shipit-clicker-web-v2_1   npm start                        Up      0.0.0.0:80->3000/tcp
[centos@ip-172-26-0-237 chapter6]$
```

您可以看到第二次运行`restart.sh`时重新创建了`chapter6_shipit-clicker-web-v2_1`应用程序，并且服务器现在通过通配符 IPv4 地址`0.0.0.0`连接到端口`80`。这将允许服务器在 URL 中没有特殊端口号的情况下响应 HTTP 请求。

### 部署

`deploy.sh`脚本从`git`上游仓库拉取更改，构建容器，并重新启动需要更新的所有容器。在本地测试完代码并对其进行了测试后，请使用此选项。

### Redis

`redis-cli.sh`脚本将允许您在命令行中连接到运行的 Redis 服务器。它使用`docker exec`命令，在运行的容器中附加并启动新的`redis-cli`命令。由于现在 Redis 在一个隔离的网络中运行，甚至从生产主机也不应该能够通过 TCP 套接字访问它。这将帮助您排除后端服务器的任何问题。

这里是展示`redis-cli.sh`运行的示例会话：

```
[centos@ip-172-26-0-237 chapter6]$ ./redis-cli.sh
127.0.0.1:6379> help
redis-cli 5.0.7
To get help about Redis commands type:
      "help @<group>" to get a list of commands in <group>
      "help <command>" for help on <command>
      "help <tab>" to get a list of possible help topics
      "quit" to exit
To set redis-cli preferences:
      ":set hints" enable online hints
      ":set nohints" disable online hints
Set your preferences in ~/.redisclirc
127.0.0.1:6379> keys *
1) "example/deploys"
2) "example/nextPurchase"
3) "example/score"
127.0.0.1:6379> get example/score
"209"
127.0.0.1:6379> quit
```

请注意，即使`redis-cli.sh`脚本位于一个私有虚拟网络中（如果您在宿主机上安装了标准的`redis-cli`程序，您将无法访问该网络），您仍然可以使用它连接到 Redis 服务器。依赖容器中的工具可以让您深入到应用程序的配置中，即便该应用程序被保护，不直接暴露在互联网上。

## 练习 – 避免在生产服务器上进行构建

本章的部署脚本执行最简单的更新方式：它在生产服务器上重建容器。然而，这可能导致资源耗尽并使生产服务器宕机。

在*第四章*《使用容器构建系统》中学到的关于 Docker Hub 的知识基础上，您如何修改应用程序开发的工作流程，以便修改`docker-compose.yml`文件和`deploy.sh`脚本，避免在生产服务器上构建 Docker 容器？

写下您将使用的工作流程的一两句话，并描述`docker-compose.yml`配置文件需要做出哪些更改。

注意：

有多种方法可以实现这些目标，并且没有单一的答案来实现它们。您可以将您的答案与下一章中的`docker-compose.yml`文件进行对比，看看您的想法与该章中构建容器的解决方案有何不同。

## 练习 – 规划如何保护生产站点

假设您从老板那里听到消息，ShipIt Squirrel 代码和生产系统将受到公司首席信息安全官的关注，他将全面审查并寻找潜在的安全漏洞。他担心在急于上线的过程中，采取了太多的快捷方式，他希望您提供更多信息。请回答以下三个问题：

1.  如何通过 SSL 加密来保护客户端与服务器之间的通信？以下哪些是您应该做的？

    a. 在程序内部终止 SSL。

    b. 使用外部负载均衡器终止 SSL。

    c. 在宿主机上使用 Web 服务器，但不在 Docker 中，来终止 SSL。

    d. 使用 Docker 和 Web 服务器容器终止 SSL。

1.  您计划如何定期更新 SSL 证书？

1.  您能否找到当前系统中其他的安全弱点，可能存在于 Docker 层或 API 层？

部署应用程序并考虑一些安全增强措施后，您应该学习如何监控部署，以便在用户发现问题之前，先发现故障。

关于如何保护生产站点的答案：

*问题 1*的四个选项中任何一个都可以使用，但实践中选项*b*和*d*最为稳健和稳定。选项*a*难以正确实现，选项*c*需要对应用环境进行单独更新。

关于*问题 2*，您可以从供应商购买 SSL 证书，每年必须更新和重新安装，或者依赖负载均衡器的供应商自动更新您的证书（如果他们提供此选项），或者使用 Let's Encrypt 自动更新证书。有关使用 Let's Encrypt 更新证书以及使用一组 Docker 容器终止 SSL 的更多信息，请参阅下一章的*进一步阅读*部分。

*问题 3*是开放式的，但您应该注意的第一件事是`chapter6`代码库中的 Web 服务没有内置认证或授权。

# 监控小型部署 - 日志记录和警报

一个好处是，从小开始，您可能可以依赖于非常简单的机制来处理日志记录和警报。对于在单个主机上使用 Docker 和`Docker Compose`部署的任何部署（例如，ShipIt Clicker 的部署），您可以使用一些基本的工具和命令来处理日志记录，并使用第三方提供的各种简单的警报服务来处理警报。

### 记录

对于日志记录，在许多情况下，只需使用 Docker 内置的日志即可。Docker 捕获其启动的每个进程的标准输出和标准错误文件句柄，并将它们作为每个容器的日志提供。您可以使用以下命令查看自上次容器重启以来启动的所有服务的汇总日志，假设您在包含您的`docker-compose.yml`文件的目录中（`less -R`将解释`logs`命令生成的 ANSI 颜色转义）：

```
$ docker-compose logs 2>&1 | less -R
```

您还可以执行`docker ps`以查找正在运行的容器的名称，以便检索其日志流：

```
 [centos@ip-172-26-0-237 ~]$ docker ps
CONTAINER ID        IMAGE                            COMMAND                  CREATED      
       STATUS              PORTS                  NAMES
e947e7de33ef        chapter6_shipit-clicker-web-v2   "npm start"              4 hours ago  
       Up 4 hours          0.0.0.0:80->3000/tcp   chapter6_shipit-clicker-web-v2_1
3f91820e097b        redis:5-alpine3.10               „docker-entrypoint.s…"   4 hours ago  
       Up 4 hours          6379/tcp               chapter6_redis_1
```

一旦您获得了容器的名称，您可以分别检索每个运行容器的单独日志文件。您可以将它们管道传输到`less`，或者将日志输出重定向到文件，例如：

```
 [centos@ip-172-26-0-237 ~]$ docker logs chapter6_shipit-clicker-web-v2_1 > shipit.log
[centos@ip-172-26-0-237 ~]$ tail shipit.log 
> shipit-clicker@1.0.0 start /app
> node dist/index.js
{"level":30,"time":1580087119723,"pid":16,"hostname":"e947e7de33ef","name":"shipit-clicker-
v2","msg":"Redis connection established","redis_url":"redis://redis:6379","v":1}
{"level":30,"time":1580087119934,"pid":16,"hostname":"e947e7de33ef","name":"shipit-clicker-
v2","msg":"up and running in development @: e947e7de33ef on port: 3000}","v":1}
[centos@ip-172-26-0-237 ~]$
```

此过程确实要求您登录到生产服务器并在那里运行一些命令，但实际操作中，这是检查运行在单个主机上的应用程序日志的好方法。

### 警报

要开始监视生产服务器上端口`80`上的 HTTP 服务器以确保其保持运行，这就足够了。如果您可以访问公司的网络监控系统，例如 Nagios 或 Icinga 服务器，可以使用该系统。如果系统可以通过互联网访问，您可以使用免费的监控服务，例如[`uptimerobot.com`](https://uptimerobot.com)，监控服务器。

为了进一步扩展监控，您可能还需要监控内部服务，如 Redis。然而，在像这样简单的设置中，这更加具有挑战性。我们将在*第十章*中深入讨论高级监控系统，*使用 Prometheus、Grafana 和 Jaeger 监控 Docker*。

这里的基本思路是，如果系统出现故障，您希望收到电子邮件、短信或两者兼有的通知。

# 单主机部署的局限性

将 Docker 应用程序部署到单一主机时可能出现什么问题？很多！虽然单主机部署提供了操作上的简便性，但也存在一些重大局限性。接下来我们将详细讨论这些局限性。

## 没有自动故障转移

如果数据库服务器容器或 Web 服务容器发生故障且无法自动重启，站点将无法访问，并需要人工干预。这可能只是简单地发现您的监控系统显示站点已停机，然后您需要 SSH 进入并重启服务器。但有时，单个服务器的内存会非常低，以至于必须从更高层次的控制台手动重启，甚至手动断电再重启。这通常会导致应用程序长时间停机，无法响应请求。

## 无法横向扩展以接受更多负载

如果系统的流量超出了当前的容量，会发生什么？在单主机部署中，您可能能够将主机切换到更大的计算机，增加更多内存和处理器，这称为*纵向扩展*。在云环境中，这要比在需要处理物理硬件（如本地环境或数据中心环境）中要容易得多。而将这些简单的部署技术应用到整个服务器实例群组中——这被称为*横向扩展*——则要困难得多。

## 基于错误的主机调优跟踪不稳定行为

根据您的托管提供商、您所使用的基础操作系统以及 Docker 容器的配置，您可能会遇到很难追踪的不稳定性。也许由于提供商的网络检测到不稳定的硬件或网络条件，您的主机会频繁重启。也许您已将操作系统配置为安装自动更新，应用这些更新会导致一段时间的中断。也许应用程序的内存增长到触发某种故障。

为了简化起见，本章中的示例没有在应用程序或容器级别指定内存限制。这意味着，由于缺少在应用程序级主配置文件中的`max_memory`设置，Redis 容器可能会消耗主机上所有可用内存。这也意味着，运行 Express Web 应用程序的节点容器可能会泄漏内存，直到操作系统的**内存溢出** (**OOM**)杀手终止它，或者 Docker 守护进程终止。

缓解此问题的一种方法是通过使用交换文件或交换分区来配置主机的虚拟内存，这样系统看起来就像拥有比实际更多的物理内存。如果未在主机上配置交换文件，则可能会发现运行 `deploy.sh` 脚本时失败。发生这种情况时，控制台可能没有任何信息，但如果检查 `/var/log/messages`，你会发现 Linux 内核的 OOM 杀手终止了 `npm` 安装程序或 Docker 容器构建过程的其他部分。

请参阅 Docker 文档，了解未为容器和操作系统适当配置内存的风险：

[`docs.docker.com/config/containers/resource_constraints/`](https://docs.docker.com/config/containers/resource_constraints/)

## 单台主机的丢失可能会造成灾难性后果 – 定期备份至关重要

如果你的应用托管在单台物理服务器或虚拟服务器上，应该确保定期备份系统。许多提供商提供镜像备份服务，你可以配置该服务进行每日备份，并将备份保留一段时间，通常需要额外付费。你也可以使用传统方法脚本化备份关键卷，例如使用 [TAR 和 SSH 或使用现代备份系统](https://restic.readthedocs.io/en/latest/)，比如 `restic`（参见 [`restic.readthedocs.io/en/latest/`](https://restic.readthedocs.io/en/latest/)），将文件和卷备份到云存储系统。

## 案例研究 – 从 CoreOS 和 Digital Ocean 迁移到 CentOS 7 和 AWS

作者之一 Richard Bul[lington-McGuire，维护了](https://freezingsaddles.org/)一个冬季骑行比赛网站，[`freezingsaddles.org/`](https://freezingsaddles.org/)，该网站托管在 Digital Ocean 的一个虚拟机上，使用 CoreOS 运行超过一年。这个系统经常在重启后掉线，很难追踪导致周期性宕机的具体问题。由于无法访问 Digital Ocean 控制面板的控制台，且对 CoreOS 不熟悉，系统故障排查变得更加困难。为了确保系统有备份，安装并配置了 `restic`，将备份发送到 Amazon S3。经过多次令人沮丧的系统管理经历后，系统迁移到了 AWS，使用 Lightsail，并运行 CentOS 7 作为主机操作系统。为了防止 OOM（内存不足）情况发生，新系统配置了一个与内存大小相等的交换文件。此后，系统停止了每隔几天随机宕机的问题，操作也变得更加顺畅。此外，新系统启用了每日自动快照备份，减少了使用像 `restic` 这样的应用级工具进行备份的需求。尽管如此，如果系统重启，Web 服务器有时仍无法平稳启动，需要手动干预才能恢复服务。

# 摘要

将基于 Docker 的应用程序推向生产环境的最简单方式是通过 Docker Compose 将其部署到单个主机上。如果你已正确准备好主机，安装了合适的软件，包括 Docker Compose，那么你可以在生产环境配置下将应用程序部署到该主机。这可以在几小时内完成，并且能高效地服务于低到中等性能和可用性需求的应用。如果你对配置文件做出正确调整，应用程序就可以准备好进行生产部署。通过使用封装长而繁琐命令的 shell 脚本，你可以更轻松地处理应用程序的常规维护和更新。在最简单的情况下，你可以使用外部监控和警报来处理此类应用，并以低成本解决此类问题。

你可以将本章所学的内容应用于提升支持你应用程序的 Dockerfile 和 `docker-compose.yml` 文件的复杂度。你可以编写简单的 shell 脚本来自动化最常见的应用。你将了解到，你可以依赖外部监控服务（例如 [`uptimerobot.com`](https://uptimerobot.com)）来提供简单的可用性监控，并且可以使用内建的 Docker 日志功能来深入了解你应用的运行情况。

一旦你部署了应用程序，最好能提高与之相关的自动化水平，特别是在如何构建和部署应用程序方面。在下一章中，我们将看到如何使用 Jenkins —— 一个常见的持续集成系统 —— 来[自动化部署和测试](https://www.packtpub.com/free-ebooks/virtualization-and-cloud/docker-cookbook-second-edition/9781788626866)。

# [进一步阅读](https://www.packtpub.com/free-ebooks/virtualization-and-cloud/docker-cookbook-second-edition/9781788626866)

+   [*Docker 食谱*](https://www.packtpub.com/free-ebooks/vi)(https://www.packtpub.com/free-ebooks/virtualization-and-cloud/docker-cookbook-second-edition/9781788626866)虚拟化与云端/Docker 食谱第二版

+   [*在生产环境中使用*](https://docs.docker.com/compose/production/) *Compose*： https:/[/docs.docker.com/compose/production/](https://geekflare.com/best-open-source-monitoring-software/)

+   [开源监控](https://geekflare.com/best-open-source-monitoring-software/)工具: https://geekflar[e.com/best-open-source-monitoring-software/](https://www.dnsstuff.com/free-network-monitoring-software)

+   [免费的监控工具](https://www.dnsstuff.com/free-network-monitoring-software): https://www.dnsstuff.com/free-ne[twork-monitoring-software](https://vsupalov.com/docker-compose-production/)

+   [`docker-compose` 是否适用于生产环境？](https://vsupalov.com/docker-compose-production/) [`vsupalov.com/docker-compose-production/`](https://vsupalov.com/docker-compose-production/)

+   Docker 提示 2：`COPY` 和 `ADD` 在 Dockerfile 中的区别：[`nickjanetakis.com/blog/docker-tip-2-the-difference-between-copy-and-add-in-a-dockerile`](https://nickjanetakis.com/blog/docker-tip-2-the-difference-between-copy-and-add-in-a-dockerile)

如果您在单台主机上运行真实的生产应用并使用 docker-compose，您应当强烈考虑使用 SSL 来保护您的网站。您可以使用 Let's Encrypt 和一组 Docker sidecar 容器来实现这一点：

+   如何使用 Let's Encrypt、NGINX 和 Docker 为您的网站启用 SSL：[`github.com/nginx-proxy/docker-letsencrypt-nginxproxy-companion`](https://github.com/nginx-proxy/docker-letsencrypt-nginxproxy-companion)

+   使用 `docker-compose.yml` 配置 Let's Encrypt 与 NGINX 和 Docker：[`github.com/nginx-proxy/docker-letsencryptnginx-proxy-companion/blob/master/docs/Docker-Compose.md`](https://github.com/nginx-proxy/docker-letsencryptnginx-proxy-companion/blob/master/docs/Docker-Compose.md)
