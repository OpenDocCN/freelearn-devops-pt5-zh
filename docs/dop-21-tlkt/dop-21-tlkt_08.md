# 第八章：使用 Docker Stack 和 Compose YAML 文件来部署 Swarm 服务

复制和粘贴是一个设计错误。

–David Parnas

在我进行与 Docker 相关的讲座和研讨会时，最常见的问题通常与 Swarm 和 Compose 有关。

*某人*: 我如何在 Docker Swarm 中使用 Docker Compose？

*我*: 你不能！你可以将你的 Compose 文件转换为一个不支持所有 Swarm 特性的 Bundle。如果你想充分利用 Swarm，请准备好使用包含无尽参数列表的 `docker service create` 命令。

这样的回答通常会伴随着失望。Docker Compose 向我们展示了将所有内容指定在 YAML 文件中的优点，而不是试图记住我们需要传递给 Docker 命令的所有参数。它使我们能够将服务定义存储在仓库中，从而提供了一个可重复且文档化的管理流程。Docker Compose 替代了 bash 脚本，我们喜欢它。然后，Docker v1.12 发布了，并且给我们提出了一个艰难的选择。我们应该采用 Swarm 并抛弃 Compose 吗？自 2016 年夏天以来，Swarm 和 Compose 已经不再亲密。那是一场痛苦的离婚。

但是，经过近半年的分离后，它们重新走到了一起，我们可以见证它们的第二次蜜月。算是吧……我们不再需要 Docker Compose 二进制文件来管理 Swarm 服务，但我们仍然可以使用它的 YAML 文件。

*Docker Engine v1.13* 引入了在 stack 命令中支持 Compose YAML 文件。与此同时，*Docker Compose v1.10* 引入了其格式的新 *版本 3*。它们使我们能够使用已熟悉的 Docker Compose YAML 格式来管理我们的 Swarm 服务。

我假设你已经熟悉 Docker Compose，并且不会详细讲解我们可以用它做的所有事情。相反，我们将通过一个创建几个 Swarm 服务的示例来进行讲解。

我们将探讨如何通过 *Docker Compose* 文件和 `docker stack deploy` 命令来创建 *Docker Flow Proxy* 服务 ([`proxy.dockerflow.com/`](http://proxy.dockerflow.com/))。

# Swarm 集群设置

要使用 Docker Machine 设置一个示例 Swarm 集群，请运行以下命令。

本章中的所有命令都可以在 `07-docker-stack.sh` 文件中找到 ([`gist.github.com/vfarcic/57422c77223d40e97320900fcf76a550`](https://gist.github.com/vfarcic/57422c77223d40e97320900fcf76a550))：

```
cd cloud-provisioning

git pull

scripts/dm-swarm.sh

```

现在我们准备部署 `docker-flow-proxy` 服务。

# 通过 Docker stack 命令创建 Swarm 服务

我们将从创建网络开始：

**Windows 用户注意**

你可能会遇到卷未正确映射的问题。如果你看到 `Invalid volume specification` 错误，请导出环境变量 `COMPOSE_CONVERT_WINDOWS_PATHS` 并将其设置为 `0`：

`export COMPOSE_CONVERT_WINDOWS_PATHS=0`

请确保在运行 `docker-compose` 或 `docker stack deploy` 之前导出变量。

```
eval $(docker-machine env swarm-1)

docker network create --driver overlay proxy

```

`proxy` 网络将专用于 `proxy` 容器及将要连接到它的服务。

我们将使用 `vfarcic/docker-flow-proxy`（[`github.com/vfarcic/docker-flow-proxy/blob/master/docker-compose-stack.yml`](https://github.com/vfarcic/docker-flow-proxy/blob/master/docker-compose-stack.yml)） 仓库中的 `docker-compose-stack.yml` 文件来创建 `docker-flow-proxy` 和 `docker-flow-swarm-listener` 服务。

`docker-compose-stack.yml` 文件的内容如下：

```
version: "3"

services:

  proxy:
    image: vfarcic/docker-flow-proxy
    ports:
      - 80:80
      - 443:443
    networks:
      - proxy
    environment:
      - LISTENER_ADDRESS=swarm-listener
      - MODE=swarm
    deploy:
      replicas: 2

  swarm-listener:
    image: vfarcic/docker-flow-swarm-listener
    networks:
      - proxy
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    environment:
      - DF_NOTIFY_CREATE_SERVICE_URL=http://proxy:8080/v1/\
docker-flow-proxy/reconfigure
      - DF_NOTIFY_REMOVE_SERVICE_URL=http://proxy:8080/v1/\
docker-flow-proxy/remove
    deploy:
      placement:
        constraints: [node.role == manager]

networks:
  proxy:
    external: true

```

该格式采用 *version 3*（这是 `docker stack deploy` 的必需格式）。

它包含了两个服务：`proxy` 和 `swarm-listener`。由于你已经熟悉 `proxy`，我就不再详细解释每个参数的含义了。

与之前的 Compose 版本相比，大多数新参数都定义在 deploy 部分。你可以把这一部分看作是为 Swarm 特定参数保留的占位符。在这种情况下，我们指定 `proxy` 服务应有两个副本，而 `swarm-listener` 服务应限制为管理节点角色。对于这两个服务的其他定义，采用与早期 Compose 版本相同的格式。

在 YAML 文件的底部是网络列表，这些网络会在服务中引用。如果某个服务没有指定任何网络，则会自动创建默认网络。在这种情况下，我们选择手动创建一个网络，因为其他堆栈中的服务需要与 `proxy` 进行通信。因此，我们手动创建了一个网络，并在 YAML 文件中将其定义为外部网络。

让我们根据我们探讨的 YAML 文件来创建堆栈：

```
curl -o docker-compose-stack.yml \
    https://raw.githubusercontent.com/\
vfarcic/docker-flow-proxy/master/docker-compose-stack.yml

docker stack deploy \
    -c docker-compose-stack.yml proxy

```

第一个命令从 `vfarcic/docker-flow-proxy`（[`github.com/vfarcic/docker-flow-proxy`](https://github.com/vfarcic/docker-flow-proxy)） 仓库中下载了 Compose 文件 `docker-compose-stack.yml`（[`github.com/vfarcic/docker-flow-proxy/blob/master/docker-compose-stack.yml`](https://github.com/vfarcic/docker-flow-proxy/blob/master/docker-compose-stack.yml)）。第二个命令创建了形成该堆栈的服务。

可以通过 `stack ps` 命令查看堆栈的任务：

```
docker stack ps proxy

```

输出结果如下（为了简洁，已去除 ID）：

```
NAME                   IMAGE                                     NODE    
proxy_proxy.1          vfarcic/docker-flow-proxy:latest          node-2        
proxy_swarm-listener.1 vfarcic/docker-flow-swarm-listener:latest node-1       
proxy_proxy.2          vfarcic/docker-flow-proxy:latest          node-3       
------------------------------------------------------------
DESIRED STATE CURRENT STATE         ERROR PORTS
Running       Running 2 minutes ago
Running       Running 2 minutes ago
Running       Running 2 minutes ago 

```

我们运行了两个 `proxy` 的副本（以确保在故障发生时具有高可用性），以及一个 `swarm-listener`。

# 部署更多的堆栈

我们来部署另一个堆栈。

这次我们将使用 Compose 文件 `docker-compose-stack.yml` 中定义的 Docker stack（[`github.com/vfarcic/go-demo/blob/master/docker-compose-stack.yml`](https://github.com/vfarcic/go-demo/blob/master/docker-compose-stack.yml)），该文件位于 `vfarcic/go-demo`（[`github.com/vfarcic/go-demo/`](https://github.com/vfarcic/go-demo/)） 仓库中。具体内容如下：

```
version: '3'

services:

  main:
    image: vfarcic/go-demo
    environment:
      - DB=db
    networks:
      - proxy
      - default
    deploy:
      replicas: 3
      labels:
        - com.df.notify=true
        - com.df.distribute=true
        - com.df.servicePath=/demo
        - com.df.port=8080

  db:
    image: mongo
    networks:
      - default

networks:
  default:
    external: false
  proxy:
    external: true

```

这个 stack 定义了两个服务（`main` 和 `db`）。它们将通过 stack 自动创建的默认网络进行通信（无需使用 `docker network create` 命令）。由于主服务是 API，因此它应该能够通过 `proxy` 访问，所以我们也将 `proxy` 网络附加上。

需要注意的重要一点是，我们在部署部分使用了 `Swarm-specific` 参数。在本例中，主服务定义了应该有三个副本以及一些标签。和之前的 stack 一样，我们不打算详细讨论每个服务。如果您想更深入了解与主服务相关的标签，请访问 *Running Docker Flow Proxy In Swarm Mode With Automatic Reconfiguration* ([`proxy.dockerflow.com/swarm-mode-auto/`](http://proxy.dockerflow.com/swarm-mode-auto/)) 教程。

让我们部署这个 stack：

```
curl -o docker-compose-go-demo.yml \
    https://raw.githubusercontent.com/\
vfarcic/go-demo/master/docker-compose-stack.yml

docker stack deploy \
    -c docker-compose-go-demo.yml go-demo

docker stack ps go-demo

```

我们下载了 stack 定义文件，执行了 `stack deploy` 命令，创建了服务，并运行了 `stack ps` 命令列出了属于 `go-demo` stack 的任务。输出如下（为了简洁，ID 和错误端口列已删除）：

```
NAME            IMAGE                   NODE    DESIRED STATE           
go-demo_main.1  vfarcic/go-demo:latest  node-2  Running       
go-demo_db.1    mongo:latest            node-2  Running       
go-demo_main.2  vfarcic/go-demo:latest  node-2  Running       
go-demo_main.3  vfarcic/go-demo:latest  node-2  Running       
--------------------------------------------------------
CURRENT STATE
Running 7 seconds ago
Running 21 seconds ago
Running 19 seconds ago
Running 20 seconds ago 

```

由于 Mongo 数据库比主服务大得多，所以拉取它需要更多时间，导致了一些失败。`go-demo` 服务设计为如果无法连接到数据库则会失败。一旦 `db` 服务启动，主服务应该停止失败，我们会看到三个副本处于 `Running` 状态。

几秒钟后，`swarm-listener` 服务将检测到 `go-demo` stack 的主服务，并发送请求给 `proxy` 重新配置自己。我们可以通过发送 HTTP 请求到 `proxy` 来查看结果：

```
curl -i "http://$(docker-machine ip swarm-1)/demo/hello"

```

输出如下：

```
HTTP/1.1 200 OK
Date: Thu, 19 Jan 2017 23:57:05 GMT
Content-Length: 14
Content-Type: text/plain; charset=utf-8

hello, world!

```

`proxy` 被重新配置，并将所有带有基础路径 `/demo` 的请求转发到 `go-demo` stack 的主服务。

# 堆叠与不堆叠

Docker stack 是 Swarm 模式的一个重要补充。我们不再需要处理那些通常有着无穷参数列表的 `docker service create` 命令。通过在 Compose YAML 文件中指定服务，我们可以用一个简单的 `docker stack deploy` 来替代这些长长的命令。如果这些 YAML 文件存储在代码仓库中，我们可以将相同的实践应用到服务部署中，就像处理其他软件工程领域一样。我们可以追踪更改，进行代码审查，与他人共享等等。

Docker `stack` 命令的加入以及其使用 Compose 文件的能力，极大地丰富了 Docker 生态系统，值得欢迎。

在本书的其余部分，我们将在探索新服务时使用 `docker service create` 命令，而在创建我们已经熟悉的服务时使用 `docker stack deploy`。如果你在将 `docker service create` 命令转换为堆栈时遇到困难，请查看 `vfarcic/docker-flow-stacks` ([`github.com/vfarcic/docker-flow-stacks`](https://github.com/vfarcic/docker-flow-stacks)) 仓库。它包含了一些我们将使用的服务的堆栈。我希望你能贡献你所使用的堆栈。请 fork 仓库并提交 pull request。如果你在创建堆栈时遇到困难，请提出 issue。

# 清理

请移除我们创建的 Docker Machine 虚拟机。你可能需要这些资源来完成其他任务：

```
exit

docker-machine rm -f swarm-1 swarm-2 swarm-3

```
