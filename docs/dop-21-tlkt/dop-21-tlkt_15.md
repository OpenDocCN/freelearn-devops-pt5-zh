# 第十五章：在 Docker Swarm 集群中管理机密

*Docker 1.13* 引入了一套功能，使我们能够集中管理机密，并仅将它们传递给需要的服务。这些功能提供了一个亟需的机制，允许我们提供只有指定服务可以访问的信息。

从 Docker 的角度来看，机密是一个数据块。一个典型的使用案例可能是证书、SSH 私钥、密码等等。机密应该保持机密，意味着它们不应以未加密的形式存储或通过网络传输。

既然如此，接下来我们通过实际示例来看它们如何工作，并继续我们的讨论。

本章中的所有命令都可以在 `14-secrets.sh` 文件中找到，文件链接为：[`gist.github.com/vfarcic/906d37d1964255b40af430bb03d2a72e`](https://gist.github.com/vfarcic/906d37d1964255b40af430bb03d2a72e)。

# 创建机密

由于单个节点就足以演示 Docker 机密，我们将从创建一个基于 Docker Machines 的单节点 Swarm 集群开始：

```
docker-machine create \
 -d virtualbox \
 swarm 
eval $(docker-machine env swarm) 
docker swarm init \
 --advertise-addr $(docker-machine ip swarm)

```

**Windows 用户注意事项**

推荐在 *Git Bash*（通过 *Docker Toolbox* 和 *Git* 安装）中运行所有示例。这样，你在书中看到的命令就与应在 *OS X* 或任何 *Linux* 发行版上执行的命令相同。

我们创建了一个名为 swarm 的 Docker Machine 节点，并用它初始化了集群。

现在我们可以创建一个机密了。

**Windows 用户注意事项**

为了使下一个命令中使用的挂载（机密也是一种挂载）正常工作，你必须停止 Git Bash 改变文件系统路径。设置这个环境变量。

`export MSYS_NO_PATHCONV=1`

创建机密的命令格式如下（请不要运行）：

```
docker secret create [OPTIONS] SECRET file|-

```

`secret create` 命令期望一个包含机密的文件。然而，创建一个未加密的机密文件违背了使用机密的初衷。任何人都可以读取该文件。我们可以在推送到 Docker 后删除该文件，但那样只会增加不必要的步骤。相反，我们将使用 `-`，这将允许我们将标准输出传递给管道：

```
echo "I like candy" \
    | docker secret create my_secret -

```

我们刚刚执行的命令创建了一个名为 `my_secret` 的机密。这些信息通过 TLS 连接发送到了远程 Docker 引擎。如果我们有一个更大的集群，包含多个管理节点，机密将会在所有节点之间复制。

我们可以检查新创建的机密：

```
docker secret inspect my_secret

```

输出如下：

```
[
    {
        "ID": "9iqwc8zb7xum7krgm183t4mym",
        "Version": {
            "Index": 11
},
        "CreatedAt": "2017-02-20T23:00:48.983267019Z",
        "UpdatedAt": "2017-02-20T23:00:48.983267019Z",
        "Spec": {
            "Name": "my_secret"
}
}
]

```

机密的值是隐藏的。即使恶意用户获得了 Docker 引擎的访问权限，机密仍然无法被访问。说实话，在这种情况下，我们的担忧将远远大于保护 Docker 机密，但我会把这个话题留到以后再讨论。

现在我们已经加密了机密并将其存储在 Swarm 管理节点中，接下来我们应该探索如何在服务中使用它。

# 消费机密

新增了一个参数 `--secret` 到 `docker service create` 命令中。如果一个秘密被附加，它将作为文件存放在所有构成服务的容器中的 `/run/secrets` 目录内。

让我们看看它的实际应用：

```
docker service create --name test \
    --secret my_secret \
    --restart-condition none \
    alpine cat /run/secrets/my_secret

```

我们创建了一个名为 `test` 的服务，并附加了一个名为 `my_secret` 的秘密。该服务基于 `alpine`，并将输出秘密的内容。由于这是一个快速终止的单次命令，我们将 `--restart-condition` 设置为 `none`。否则，服务会在创建后不久终止，Swarm 会重新调度它，但它会再次终止，如此循环。我们将进入一个无尽的循环。

我们来看一下日志：

```
docker logs $(docker container ps -qa)

```

输出如下：

```
I like candy

```

秘密作为 `/run/secrets/my_secret` 文件在容器内部可用。

在我们开始讨论一个更实际的例子之前，让我们删除我们创建的服务和秘密：

```
docker service rm test

docker secret rm my_secret

```

# 使用 secrets 的实际例子

*Docker Flow Proxy* ([`proxy.dockerflow.com/`](http://proxy.dockerflow.com/)) 项目暴露了应仅供内部使用的统计信息。因此，它需要通过 `用户名` 和 `密码` 来保护。在 *Docker v1.13* 之前，这种情况通常通过允许用户通过环境变量指定用户名和密码来处理。*Docker Flow Proxy* 也不例外，实际上，它有 *环境变量* ([`proxy.dockerflow.com/config/#environment-variables`](http://proxy.dockerflow.com/config/#environment-variables)) `STATS_USER` 和 `STATS_PASS`。

创建具有自定义 `用户名` 和 `密码` 的服务的命令如下：

```
docker network create --driver overlay proxy

docker service create --name proxy \
    -p 80:80 \
    -p 443:443 \
    -p 8080:8080 \
-e STATS_USER=my-user \
-e STATS_PASS=my-pass \
    --network proxy \
-e MODE=swarm \
    vfarcic/docker-flow-proxy

```

虽然这可以保护统计页面免受普通用户的访问，但仍然会让它暴露给任何能够检查服务的人。以下是一个简单的例子：

```
docker service inspect proxy --pretty

```

输出的相关部分如下：

```
...
ContainerSpec:
 Image:     vfarcic/docker-flow-proxy:latest@sha256:b1014afa9706413818903671086e484d98db669576b83727801637d1a3323910
 Env:       STATS_USER=my-user STATS_PASS=my-pass MODE=swarm
...

```

可以通过以下命令实现相同的结果，而不暴露机密信息：

```
echo "secret-user" \
    | docker secret create dfp_stats_user -

echo "secret-pass" \
    | docker secret create dfp_stats_pass -

docker service update \
    --secret-add dfp_stats_user \
    --secret-add dfp_stats_pass \
    proxy

```

我们创建了两个秘密 `dfp_stats_user` 和 `dfp_stats_pass`，并更新了我们的服务。从现在开始，这些 secrets 将作为文件 `/run/secrets/dfp_stats_user` 和 `/run/secrets/dfp_stats_pass` 存在于服务容器内部。如果一个 secret 的名称与环境变量相同，并且是小写且以 `dpf_` 前缀开头，它将被替代使用。

如果你再检查一下容器，你会发现没有任何关于 secrets 的痕迹。

我们可以在这里停止。毕竟，Docker secrets 的内容并不多。然而，我们已经习惯了 Docker stacks，如果 secrets 能在新的 YAML Compose 格式中工作，那就太好了。

在我们继续之前，让我们删除 `proxy` 服务：

```
docker service rm proxy

```

# 使用 Docker Compose 的秘密

为了确保在所有支持的版本中都具备相同的功能，Docker 在 Compose YAML 格式 *版本 3.1* 中引入了 secrets。

我们将继续使用 *Docker Flow Proxy* 来演示 secrets 如何在 Compose 文件中工作：

```
curl -o dfp.yml \
    https://raw.githubusercontent.com/vfarcic/\
docker-flow-stacks/master/proxy/docker-flow-proxy-secrets.yml

```

我们从`vfarcic/docker-flow-stacks`（[`github.com/vfarcic/docker-flow-stacks`](https://github.com/vfarcic/docker-flow-stacks)）仓库下载了`docker-flow-proxy-secrets.yml`（[`github.com/vfarcic/docker-flow-stacks/blob/master/proxy/docker-flow-proxy-secrets.yml`](https://github.com/vfarcic/docker-flow-stacks/blob/master/proxy/docker-flow-proxy-secrets.yml)）堆栈。

堆栈定义的相关部分如下：

```
version: "3.1"

...

services:

  proxy:
    image: vfarcic/docker-flow-proxy:${TAG:-latest}
    ports:
      - 80:80
      - 443:443
    networks:
      - proxy
    environment:
      - LISTENER_ADDRESS=swarm-listener
      - MODE=swarm
    secrets:
      - dfp_stats_user
      - dfp_stats_pass
    deploy:
      replicas: 3

...

secrets:
  dfp_stats_user:
    external: true
  dfp_stats_pass:
    external: true

```

格式版本是`3.1`。`proxy`服务附加了两个密钥。最后，还有一个单独的`secrets`部分，将密钥定义为`external`实体。另一种选择是将密钥内部指定。

一个示例如下：

```
secrets:
    dfp_stats_user:
        external: true
    dfp_stats_pass:
        external: true
secrets:
 dfp_stats_user:
    file: ./dfp_stats_user.txt
 dfp_stats_pass:
    file: ./dfp_stats_pass.txt

```

我更倾向于选择第一种方式，它通过外部指定密钥，因为这种方式没有留下任何痕迹。在某些其他情况下，密钥可能会用于非机密信息（我们会很快讨论），在这种情况下，使用作为文件指定的内部密钥可能是更好的选择。

让我们运行`stack`并检查它是否正常工作：

```
docker stack deploy -c dfp.yml proxy

```

如果没有数据，统计本身是没有意义的，因此我们将部署另一个服务，并在`proxy`中重新配置它，从而开始生成一些统计数据：

```
curl -o go-demo.yml \
    https://raw.githubusercontent.com/vfarcic/\
go-demo/master/docker-compose-stack.yml

docker stack deploy -c go-demo.yml go-demo

```

请稍等片刻，直到`go-demo`堆栈中的服务开始运行。你可以通过执行`docker stack ps go-demo`来检查它们的状态。你可能会看到`go-demo_main`副本处于失败状态。不要惊慌。它们只会在`go-demo_db`启动之前持续失败。

现在我们终于可以确认，`proxy`已配置为使用密钥进行身份验证：

```
curl -u secret-user:secret-pass \
"http://$(docker-machine ip swarm)/admin?stats;csv;norefresh"

```

它成功了！只需额外的一步`docker service create`，我们就让系统更加安全。

# 使用密钥的常见方法

在引入密钥之前，将信息传递给容器的常见方式是通过环境变量。虽然对于非机密信息，这仍然是首选方式，但配置中应该也涉及到密钥。两者应结合使用。问题在于选择哪种方法，以及何时使用。

Docker 密钥的显而易见用法就是存储密钥。这个很明显，对吧？如果某些信息只应对特定容器可见，那么它应该通过 Docker 密钥提供。常见的模式是，允许相同的信息既可以作为环境变量，也可以作为密钥指定。如果两者都设置了，密钥应该优先。在*Docker Flow Proxy*中，你已经看到了这种模式。任何可以通过环境变量指定的信息，都可以通过密钥指定。

在某些情况下，你可能无法修改你的服务代码并将其调整为使用机密信息。也许这不是能力问题，而是你不愿意修改代码。如果你属于后一种情况，我暂时不会解释为什么代码应该不断重构，并假设你有非常好的理由这么做。无论哪种情况，通常的解决方案是创建一个包装脚本，将机密信息转换为你的服务所需的格式，然后调用该服务。将这个脚本作为 CMD 指令放入 *Dockerfile* 中，这样就完成了。机密信息保持为机密，你也不必因为重构代码而被解雇。对于某些人来说，这最后一句话听起来有些傻，但对于很多公司来说，重构代码被认为是浪费时间并不罕见。

什么应该是机密信息？没有人能够真正为你回答这个问题，因为这因组织而异。一些例子包括用户名和密码、SSH 密钥、SSL 证书等等。如果你不希望别人知道，就把它作为机密处理。

我们应该努力实现不可变性，尽力运行在任何地方都完全相同的容器。真正的不可变性意味着即使是配置在所有环境中也始终保持一致。然而，这并不总是容易实现的，有时甚至是无法完成的。在这种情况下，Docker Secrets 可能是一个很好的候选方案。它们不一定只能作为指定机密信息的手段。我们可以使用机密信息作为提供在不同集群之间有所不同的配置信息的方式。在这种情况下，应该在不同环境中有所差异的配置项（例如：预发布集群和生产集群）可以作为机密存储。

我相信还有很多我没有想到的用例。毕竟，机密信息是一个新的功能（距离本文写作时只有几周的历史）。

# 现在怎么办？

移除你的 Docker Machine 虚拟机，并开始将机密信息应用到你自己的 Swarm 集群中。暂时没什么更多要说的了：

```
docker-machine rm -f swarm

```
