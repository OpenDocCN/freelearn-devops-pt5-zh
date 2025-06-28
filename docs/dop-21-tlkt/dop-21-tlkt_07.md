# 第七章：探索 Docker 远程 API

你可以大规模生产硬件；你不能大规模生产软件——你不能大规模生产人类的思维。

—— Michio Kaku

直到现在，我们一直通过 Docker 客户端来使用 Docker。每当我们需要某个功能时，唯一需要做的就是执行一个`docker`命令（例如：`docker service create`）。在大多数情况下，当我们仅限于从命令行操作集群时，这已经足够了。

如果我们想实现一些客户端提供的功能之外的操作会发生什么呢？如果我们希望从我们的应用程序内部操作 Docker 呢？我们能从整个集群中获取所有正在运行的容器的统计信息吗？

对这些问题以及其他许多问题的一个可能答案在于采用 Docker 公司之外的工具。我们将在接下来的章节中探讨这些工具。

另一种方法是使用 Docker 远程 API。毕竟，如果我们选择了 Docker 生态系统中的某个产品，它很可能会使用 API。Docker Compose 使用它向 Docker 引擎发送命令。即使是客户端，也使用它与远程引擎进行通信。你也许会发现它很有用。

默认情况下，Docker 守护进程监听 `unix:///var/run/docker.sock`，并且客户端必须拥有 root 权限才能与守护进程交互。如果你的系统上存在名为 `docker` 的组，Docker 会将套接字的所有权赋给该组。这并不意味着套接字是访问 API 的唯一方式。事实上，还有很多其他方式，我鼓励你尝试不同的组合。为了本章的目的，我们将坚持使用套接字，因为它是发送 API 请求最简单的方式。

# 环境设置

就像前几章一样，我们将从创建一个我们将用来实验的集群开始。

本章中的所有命令都可以在 `07-api.sh` 中找到（[`gist.github.com/vfarcic/bab7f89f1cbd14f9895a9e0dc7293102`](https://gist.github.com/vfarcic/bab7f89f1cbd14f9895a9e0dc7293102)）Gist。

请进入我们拉取仓库的 `cloud-provisioning` 目录。由于我可能自上次你使用它之后更新过它，我们将执行一个 pull。最后，我们将运行已经熟悉的 `script/ dm-swarm.sh`，它将创建一个新的 Swarm 集群：

```
cd cloud-provisioning

git pull

scripts/dm-swarm.sh

```

集群已经启动并运行。

使用 *Docker Machine* 创建的虚拟机基于 *Boot2Docker*。它是一个专门为运行 Docker 容器而设计的轻量级 Linux 发行版。它完全运行在 RAM 中，下载大小仅为 38 MB，并且大约 5 秒内即可启动。它基于 *Tiny Core Linux*（[`tinycorelinux.net/`](http://tinycorelinux.net/)）。与更流行的 Linux 发行版相比，它的区别在于体积。它被精简到最基本的程度。这种做法对我们来说非常适用。如果我们采用容器，大多数通常在像 Ubuntu 和 RedHat 这样的发行版中看到的内核模块其实并没有必要。

这与我们在使用容器时追求的简约方法一致。我之前讨论了使用 `Alpine` 作为容器基础镜像的原因。最主要的原因是它的体积（仅几个 MB）。毕竟，为什么我们要将不需要的东西打包到容器中呢？主机操作系统也可以这样说。少即是多，只要满足我们的所有需求即可。

有一个警告。*Boot2Docker* 当前设计和优化用于开发环境。在此时，强烈不建议将其用于任何生产负载。这并不削弱其价值，但明确区分了它适合做什么，不适合做什么。

对 *Boot2Docker* 和 Tiny Core Linux 的简短介绍是为了后续步骤做准备。我们即将安装一些程序，而我们需要了解所使用的发行版的包管理工具。Tiny Core Linux 使用 `tce-load`。

在前几章中，我们从操作系统（MacOS、Linux 或 Windows）执行了大多数命令。这一次，我们将在 Docker Machine 虚拟机中执行这些命令。原因在于我们将使用 `jq` ([`stedolan.github.io/jq/`](https://stedolan.github.io/jq/)) 来格式化从 API 接收到的 JSON 输出。它在大多数平台上都可用，但我认为最好将你放入虚拟机中，以避免可能出现的问题。第二个更重要的原因是，我们选择通过机器上的 Docker 套接字向 API 发送请求。

不再多说，我们将开始安装 `curl` 和 `jq`：

```
docker-machine ssh swarm-1

tce-load -wi curl wget

wget https://github.com/stedolan/jq/releases/download/jq-1.5/jq-linux64

sudo mv jq-linux64 /usr/local/bin/jq

sudo chmod +x /usr/local/bin/jq

```

我们进入了 `swarm-1` 机器，并使用 `tce-load` 安装了 `curl` 和 `wget`。由于 `jq` 无法通过 `tce-load` 获取，我们使用 `wget` 下载了二进制文件。最后，我们将 `jq` 移动到 bin 目录，并添加了执行权限。

现在我们准备好开始探索 Docker 远程 API 了。

# 通过 Docker 远程 API 操作 Docker Swarm

我们不会详细讲解整个 API。官方的 *文档* ([`docs.docker.com/engine/reference/api/docker_remote_api_v1.24/`](https://docs.docker.com/engine/reference/api/docker_remote_api_v1.24/)) 编写得非常清晰，并提供了足够的细节。相反，我们将通过一些围绕 Docker Swarm 的基本示例来学习。我们将通过重复之前练习过的一些客户端命令来查看如何使用 API。本章的目标是获取足够的知识，以便在应用程序中使用 API，并将其作为连接不同服务的粘合剂，我们将在后续章节中探索这些服务。之后，我们将尝试利用这些知识创建一个监控系统，将有关集群的信息存储在数据库中，并执行一些操作。

让我们讨论一个非常简单的示例，展示 API 的可能使用场景。

如果某个节点出现故障，Swarm 会确保该节点内部运行的容器被重新调度。然而，这并不意味着这样就足够了。我们可能想要发送一封邮件，通知某个节点故障。收到这样的邮件后，有人将调查故障节点的原因，并可能采取一些纠正措施。这些任务并不紧急，因为 Swarm 已经缓解了问题。然而，事情不紧急并不意味着它不应当被完成。

接下来的章节将尝试让我们的集群更加强大，而 API 在其中将发挥关键作用。现在，让我们先做一个简要的概述。

我们将从一个简单的示例开始。让我们看看哪些节点构成了我们的集群：

```
curl \ 
    --unix-socket /var/run/docker.sock \
    http:/nodes | jq '.'

```

输出显示了三节点的详细信息。展示所有节点的所有信息对本书来说太多了，所以我们限制输出为一个节点。我们只需要将节点的名称附加到之前的命令后面：

```
curl \
    --unix-socket /var/run/docker.sock \
    http:/nodes/swarm-1 | jq '.'

```

输出如下：

```
[
  {
    "ID": "2vxiqun3wvh1l1g43utk2v5a7",
    "Version": {
      "Index": 23
    },
    "CreatedAt": "2017-01-23T20:30:00.402618571Z",
    "UpdatedAt": "2017-01-23T20:30:04.026051022Z",
    "Spec": {
      "Labels": {
        "env": "prod"
      },
      "Role": "manager",
      "Availability": "active"
    },
    "Description": {
      "Hostname": "swarm-1",
      "Platform": {
        "Architecture": "x86_64",
        "OS": "linux"
       },
      "Resources": {
        "NanoCPUs": 1000000000,
        "MemoryBytes": 1044131840
      },
      "Engine": {
        "EngineVersion": "1.13.0",
        "Labels": {
          "provider": "virtualbox"
        },
        "Plugins": [
          {
            "Type": "Network",
            "Name": "bridge"
          },
          {
            "Type": "Network",
            "Name": "host"
          },
          {
            "Type": "Network",
            "Name": "macvlan"
          },
          {
            "Type": "Network",
            "Name": "null"
          },
          {
            "Type": "Network",
            "Name": "overlay"
          },
          {
            "Type": "Volume",
            "Name": "local"
          }
        ]
     }
  },
  "Status": {
    "State": "ready",
    "Addr": "127.0.0.1"
},
"ManagerStatus": {
  "Leader": true,
   "Reachability": "reachable",
      "Addr": "192.168.99.100:2377"
}
},

```

上面的输出被截断，仅包括“Leader”节点。你的输出将包含以 `ID` 开头的三个节点集合。我们不会详细讨论每个字段的含义，你应该已经熟悉大部分字段。

更多信息，请参考 *Docker Remote API v1.24: 节点* ([`docs.docker.com/engine/reference/api/docker_remote_api_v1.24/#/nodes`](https://docs.docker.com/engine/reference/api/docker_remote_api_v1.24/#/nodes))。

API 不仅限于查询操作。我们还可以利用它执行 Docker 客户端提供的任何操作（还有一些其他操作）。例如，我们可以创建一个新服务：

```
curl -XPOST \
      -d '{
  "Name": "go-demo-db",
  "TaskTemplate": {
    "ContainerSpec": {
      "Image": "mongo:3.2.10"
    }
  }
}' \
    --unix-socket /var/run/docker.sock \
    http:/services/create | jq '.'

```

我们发送了一个 `POST` 请求来创建一个名为 `go-demo-db` 的服务。该服务的镜像是 `mongo:3.2.10`。结果，API 返回了该服务的 `ID`：

```
{
  "ID": "7157kfo9cp2vhed4bidrc8hfi"
}

```

更多信息，请参考 *Docker Remote API v1.24: 创建服务* ([`docs.docker.com/engine/reference/api/docker_remote_api_v1.24/#create-a-service`](https://docs.docker.com/engine/reference/api/docker_remote_api_v1.24/#create-a-service))。

我们可以通过列出所有服务来确认操作确实成功：

```
curl \
    --unix-socket /var/run/docker.sock \
    http:/services | jq '.'

```

输出如下：

```
[
  {
    "ID": "s73ez21cmshwu8okipejehvo0",
    "Version": {
      "Index": 26
    },
    "CreatedAt": "2017-01-23T20:47:27.247329291Z",
    "UpdatedAt": "2017-01-23T20:47:27.247329291Z",
    "Spec": {
      "Name": "go-demo-db",
      "TaskTemplate": {
        "ContainerSpec": {
          "Image":  "mongo:3.2.10@sha256:532a19da83ee0e4e2a2ec6bc4212fc4af26357c040675d5c2\
629a4e4c4563cef"
         },
        "ForceUpdate": 0
      },
      "Mode": {
         "Replicated": {
           "Replicas": 1
      }
    }
  },
  "Endpoint": {
    "Spec": {}
  },
  "UpdateStatus": {
      "StartedAt": "0001-01-01T00:00:00Z",
      "CompletedAt": "0001-01-01T00:00:00Z"
   }
  }
] 

```

我们获得了一个服务列表（目前只有一个），其中包含一些服务的属性。我们可以看到该服务的创建时间、副本数等信息。更多信息，请参考 *Docker Remote API v1.24: 列出服务* ([`docs.docker.com/engine/reference/api/docker_remote_api_v1.24/#/list-services`](https://docs.docker.com/engine/reference/api/docker_remote_api_v1.24/#/list-services))。

类似地，我们可以检索单个实例的信息：

```
curl \
    --unix-socket /var/run/docker.sock \
    http:/services/go-demo-db | jq '.'

```

输出几乎与我们列出所有服务时相同。唯一的显著区别是，这次我们得到了一个单一的结果，而列出服务时返回的是一个用`[和]`括起来的数组：

```
{
  "ID": "s73ez21cmshwu8okipejehvo0",
  "Version": {
    "Index": 26
},
  "CreatedAt": "2017-01-23T20:47:27.247329291Z",
  "UpdatedAt": "2017-01-23T20:47:27.247329291Z",
  "Spec": {
    "Name": "go-demo-db",
    "TaskTemplate": {
      "ContainerSpec": {
        "Image":  "mongo:3.2.10@sha256:532a19da83ee0e4e2a2ec6bc4212fc4af26357c040675d5c2
629a4e4c4563cef"
      },
      "ForceUpdate": 0
    },
    "Mode": {
      "Replicated": {
        "Replicas": 1
      }
    }
  },
  "Endpoint": {
    "Spec": {}
  },
  "UpdateStatus": {
    "StartedAt": "0001-01-01T00:00:00Z",
    "CompletedAt": "0001-01-01T00:00:00Z"
  }
}

```

请查阅*Docker 远程 API v1.24: 检查一个或多个服务*（[`docs.docker.com/engine/reference/api/docker_remote_api_v1.24/#inspect-one-or-more-services`](https://docs.docker.com/engine/reference/api/docker_remote_api_v1.24/#inspect-one-or-more-services)）以获取更多信息。

让我们稍微增加点挑战，将副本数扩展到三个。我们可以通过更新服务来实现这一点。然而，在发送更新请求之前，我们需要知道服务的版本和 ID。这些信息可以从我们之前发送的服务请求的输出中获得。不过，由于我们希望以便于自动化的方式执行操作，可能更好将这些值放入环境变量中。

我们可以使用`jq`来过滤输出并返回特定的值。

返回服务版本的命令如下：

```
VERSION=$(curl \
    --unix-socket /var/run/docker.sock \
    http:/services/go-demo-db | \
    jq '.Version.Index')

echo $VERSION

```

`$VERSION`变量的输出如下：

```
27

```

类似地，我们还应该获取服务的`ID`：

```
ID=$(curl \
    --unix-socket /var/run/docker.sock \
    http:/services/go-demo-db | \
    jq --raw-output '.ID')

echo $ID

```

输出如下：

```
7157kfo9cp2vhed4bidrc8hfi

```

现在我们已经拥有更新服务所需的所有信息。我们将副本数更改为三个：

```
curl -XPOST \
    -d '{
  "Name": "go-demo-db",
  "TaskTemplate": {
    "ContainerSpec": {
      "Image": "mongo:3.2.10"
    }
  },
  "Mode": {
    "Replicated": {
      "Replicas": 3
    }
  }
}' \
    --unix-socket /var/run/docker.sock \
    http:/services/$ID/update?version=$VERSION

```

请查阅*Docker 远程 API v1.24: 更新服务*（[`docs.docker.com/engine/reference/api/docker_remote_api_v1.24/#update-a-service`](https://docs.docker.com/engine/reference/api/docker_remote_api_v1.24/#update-a-service)）以获取更多信息。

接下来，我们可以列出`tasks`并确认服务是否真的扩展到三个实例：

```
curl \
    --unix-socket /var/run/docker.sock \
    http:/tasks | jq '.'

```

输出如下：

```
[
  {
    "ID": "c0nev776zyuoaul51n5w9xc85",
    "Version": {
      "Index": 32
    },
    "CreatedAt": "2017-01-23T20:47:27.255828694Z",
    "UpdatedAt": "2017-01-23T20:47:51.329755266Z",
    "Spec": {
      "ContainerSpec": {
        "Image":  "mongo:3.2.10@sha256:532a19da83ee0e4e2a2ec6bc4212fc4af26357c040675d5c2\
629a4e4c4563cef"
       },
       "ForceUpdate": 0
    },
    "ServiceID": "s73ez21cmshwu8okipejehvo0",
    "Slot": 1,
    "NodeID": "2vxiqun3wvh1l1g43utk2v5a7",
    "Status": {
      "Timestamp": "2017-01-23T20:47:51.295994806Z",
      "State": "running",
      "Message": "started",
      "ContainerStatus": {
        "ContainerID": 
"20112c904386733c6748bc186e84255640c9dc279fd3530b771616a1ef767957",
        "PID": 4238
       },
     "PortStatus": {}
     },
    "DesiredState": "running"
 },
 {
    "ID": "ptbp594sh5k2qey6lexafr31d",
    "Version": {
      "Index": 37
 },
    "CreatedAt": "2017-01-23T21:16:55.680872244Z",
    "UpdatedAt": "2017-01-23T21:16:55.960585822Z",
    "Spec": {
      "ContainerSpec": {
        "Image":  "mongo:3.2.10@sha256:532a19da83ee0e4e2a2ec6bc4212fc4af26357c040675d5c2\
629a4e4c4563cef"
     },
        "ForceUpdate": 0
    },
    "ServiceID": "s73ez21cmshwu8okipejehvo0",
         "Slot": 2,
     "NodeID": "skj5sjemrvnusdop3ovcv6q7h",
     "Status": {
       "Timestamp": "2017-01-23T21:16:55.882919942Z",
       "State": "preparing",
       "Message": "preparing",
       "ContainerStatus": {},
       "PortStatus": {}
    },
    "DesiredState": "running"
},
{
    "ID": "rqa04bpvdcddkeia2x1d6r95r",
    "Version": {
      "Index": 37
  },
  "CreatedAt": "2017-01-23T21:16:55.6812298Z",
  "UpdatedAt": "2017-01-23T21:16:55.960175605Z",
    "Spec": {
      "ContainerSpec": {
        "Image":  "mongo:3.2.10@sha256:532a19da83ee0e4e2a2ec6bc4212fc4af26357c040675d5c2\
629a4e4c4563cef"
    },
    "ForceUpdate": 0
  },
     "ServiceID": "s73ez21cmshwu8okipejehvo0",
     "Slot": 3,
     "NodeID": "lbgy0xih6n0w3nmzih0gnfvhd",
     "Status": {
      "Timestamp": "2017-01-23T21:16:55.881446699Z",
      "State": "preparing",
      "Message": "preparing",
      "ContainerStatus": {},
      "PortStatus": {}
    },
    "DesiredState": "running"
  }
]

```

如你所见，返回了三个任务，每个任务代表`go-demo-db`服务的一个副本。

请查阅*Docker 远程 API v1.24: 列出任务*（[`docs.docker.com/engine/reference/api/docker_remote_api_v1.24/#list-tasks`](https://docs.docker.com/engine/reference/api/docker_remote_api_v1.24/#list-tasks)）以获取更多信息。

到目前为止，我们所做的所有 API 请求都与节点和服务有关。在某些情况下，我们可能需要更深入地使用容器级别的 API。例如，我们可能想获取与单个容器相关的统计信息。

在继续之前，请确保所有任务的状态都是“正在运行”。可以随时重复`http:/tasks`请求来确认状态。如果它没有运行，请稍等片刻再检查。

要获取容器的统计信息，首先我们需要找出它运行在哪个节点上：

```
exit

eval $(docker-machine env swarm-1)

NODE=$(docker service ps \
    -f desired-state=running \
    go-demo-db \
    | tail -n 1 \
    | awk '{print $4}')

echo $NODE

```

我们退出了`swarm-1`机器，并使用 eval 命令创建了环境变量，指示主机上运行的 Docker 客户端使用在`swarm-1`上运行的引擎。请注意，这些环境变量告诉客户端使用我们在本章中探讨的相同 API。

进一步地，我们检索了一个容器所在的节点，这个容器是`go-demo-db`服务的一部分。我们已经使用过类似的命令几次，所以无需再详细解释。

`$NODE`变量的输出如下：

```
swarm-2

```

在我的笔记本电脑上，我们正在寻找的容器运行在 `swarm-2` 节点内。在你的情况下，它可能是另一个节点。

现在我们可以进入节点并获取容器的 `ID`：

```
docker-machine ssh $NODE

ID=$(docker ps -qa | tail -n 1)

echo $ID

```

`ID` 变量的输出如下：

```
f8f345042cf7

```

最后，我们准备获取统计信息。命令如下：

```
curl \
    --unix-socket /var/run/docker.sock \
    http:/containers/$ID/stats

```

一旦请求发送，你将看到不断流动的统计信息。当你看腻了时，请按 *CTRL* + *C* 停止流式传输。

如果我们想实现自己的监控解决方案，流式统计可能是一个非常有用的功能。在许多其他情况下，我们可能希望禁用流式传输，只检索单个记录集。我们可以通过将 `stream` 参数设置为 `false` 来实现这一点。

返回单个统计记录集的命令如下：

```
curl \
    --unix-socket /var/run/docker.sock \
    http:/containers/$ID/stats?stream=false

```

输出仍然太大，无法在书中展示，因此你需要从自己的屏幕上查看。

我们不会详细探讨每个统计字段的含义。你需要等到我们进入监控章节时才能深入探讨。目前，重要的是要注意，你可以为集群中的每个容器检索这些统计信息。

作为练习，创建一个脚本，检索节点上运行的所有容器。遍历每个容器以获取该虚拟机内所有容器的统计信息。

请参阅 *Docker Remote API v1.24: 根据资源使用情况获取容器统计信息* ([`docs.docker.com/engine/reference/api/docker_remote_api_v1.24/#get-container-stats-based-on-resource-usage`](https://docs.docker.com/engine/reference/api/docker_remote_api_v1.24/#get-container-stats-based-on-resource-usage)) 了解更多信息。

我们已经快完成与 Swarm 服务相关的基本 API 请求的探索了，现在让我们删除我们创建的服务：

```
curl -XDELETE \
    --unix-socket /var/run/docker.sock \
    http:/services/go-demo-db

curl \
    --unix-socket /var/run/docker.sock \
    http:/services

```

我们发送了 `DELETE` 请求以删除 `go-demo-db` 服务，随后发送了请求以检索所有服务。后者的输出如下：

```
[]

```

我们的服务已经不存在了。我们将其从集群中删除，且由于这是我们创建的唯一服务，请求返回了一个空的数组 `[]`。

请参阅 *Docker Remote API v1.24: 删除服务* ([`docs.docker.com/engine/reference/api/docker_remote_api_v1.24/#remove-a-service`](https://docs.docker.com/engine/reference/api/docker_remote_api_v1.24/#remove-a-service)) 了解更多信息。

最后，让我们退出机器：

```
exit

```

现在你对 API 有了基本了解，我们可以探索一个可能的用例。

# 使用 Docker Remote API 自动化代理配置

到目前为止，我们一直向代理发送重新配置和删除请求。这大大简化了配置。我们不再自己修改 HAProxy 配置，而是让服务自行重新配置。我们使用 Consul 来持久化代理的状态。我们能否通过利用 Docker Remote API 改进现有设计？我认为我们可以。

不再发送重新配置和删除请求，我们可以有一个服务来通过 API 监视集群状态。这样的工具可以检测新创建和删除的服务，并向`proxy`发送与我们手动发送的相同请求。

我们可以走得更远。由于 API 允许我们检索与集群相关的任何信息，我们不再需要将其存储在 Consul 中。每当创建服务的新实例时，它可以从 API 中检索所需的所有信息。

总之，我们可以利用 API 完全自动化对`proxy`配置及其状态的更改。我们可以创建一个新的服务来监视集群状态。我们还可以修改`proxy`以在其初始化过程中查询该服务。

我考虑节省您的一些时间，因此擅自创建了这样一个服务。它背后的项目称为*Docker Flow Swarm Listener* ([`github.com/vfarcic/docker-flow-swarm-listener`](https://github.com/vfarcic/docker-flow-swarm-listener))。

让我们看看它的工作情况。

# 将 Swarm 监听器与代理结合使用

*Docker Flow Swarm Listener* ([`github.com/vfarcic/docker-flow-swarm-listener`](https://github.com/vfarcic/docker-flow-swarm-listener)) 项目利用了 Docker Remote API。它有很多用途，但现在我们将限制在可以帮助我们完全无需手动操作来配置代理的功能上。

我们将首先创建两个网络：

```
eval $(docker-machine env swarm-1)

docker network create --driver overlay proxy

docker network create --driver overlay go-demo

```

我们已经创建了这两个网络很多次了，没有理由再去讨论它们的有用性。唯一的区别是，这次我们将有一个更多的服务附加到`proxy`网络。

接下来，我们将创建`swarm-listener` ([`github.com/vfarcic/docker-flow-swarm-listener`](https://github.com/vfarcic/docker-flow-swarm-listener)) 服务。它将作为 Docker Flow Proxy 的伴侣。其目的是监视 Swarm 服务并在创建或销毁服务时向代理发送请求。

**Windows 用户请注意**

Git Bash 有改变文件系统路径的习惯。要停止这个行为，请执行以下操作：

`export MSYS_NO_PATHCONV=1`

让我们创建`swarm-listener`服务：

```
docker service create --name swarm-listener \
    --network proxy \
    --mount \
    "type=bind,source=/var/run/docker.sock,target=/var/run/\
    docker.sock" \
-e DF_NOTIF_CREATE_SERVICE_URL=http://proxy:8080/v1/
    docker-flow-proxy/reconfigure \
-e DF_NOTIF_REMOVE_SERVICE_URL=http://proxy:8080/v1/
    docker-flow-proxy/remove \
    --constraint 'node.role==manager' \
    vfarcic/docker-flow-swarm-listener

```

该服务附加到`proxy`网络，挂载 Docker 套接字，并声明环境变量`DF_NOTIF_CREATE_SERVICE_URL`和`DF_NOTIF_REMOVE_SERVICE_URL`。我们很快将看到这些变量的用途。该服务受限于管理节点。

下一步是创建`proxy`服务：

```
docker service create --name proxy \
    -p 80:80 \
    -p 443:443  \
    --network proxy \
    -e MODE=swarm \
    -e LISTENER_ADDRESS=swarm-listener \
    vfarcic/docker-flow-proxy

```

我们打开了`*80*`和`*443*`端口。外部请求将通过它们路由到目标服务。请注意，这次我们没有打开`8080`端口。由于`proxy`将从`swarm-listener`接收通知，因此无需为手动通知保留`8080`端口。

`proxy`连接到`proxy`网络，并将模式设置为 swarm。`proxy`必须与监听器属于同一网络。它们将在每次创建或删除服务时以及每次创建新的`proxy`实例时交换信息。

# 自动重新配置代理

让我们创建已经熟悉的示例服务：

```
docker service create --name go-demo-db \
  --network go-demo \
  mongo:3.2.10

docker service create --name go-demo \
  -e DB=go-demo-db \
  --network go-demo \
  --network proxy \
  --label com.df.notify=true \
  --label com.df.distribute=true \
  --label com.df.servicePath=/demo  \
  --label com.df.port=8080 \
  vfarcic/go-demo:1.0

```

请注意标签。在前面的章节中我们没有使用它们。情况发生了变化，现在它们已成为服务定义中的关键部分。`com.df.notify=true`告诉`swarm-listener`服务是否在服务被创建或删除时发送通知。由于我们不想将`go-demo-db`服务添加到`proxy`中，因此该标签仅定义在`go-demo`服务上。其余的标签与我们手动重新配置代理时使用的查询参数相匹配。唯一的区别是，这些标签前缀为`com.df`。有关查询参数的列表，请参阅项目中的*重新配置* ([`github.com/vfarcic/docker-flow-proxy#reconfigure`](https://github.com/vfarcic/docker-flow-proxy#reconfigure))部分。

现在我们应该等待所有服务都在运行。您可以通过执行以下命令来查看它们的状态：

```
docker service ls

```

一旦所有副本都设置为`1/1`，我们可以通过向`proxy`发送请求来看到`com.df`标签的效果，该请求会通过`proxy`访问`go-demo`服务：

```
curl -i "$(docker-machine ip swarm-1)/demo/hello"

```

输出如下：

```
HTTP/1.1 200 OK
Date: Thu, 13 Oct 2016 18:26:18 GMT
Content-Length: 14
Content-Type: text/plain; charset=utf-8

hello, world!

```

我们向`proxy`发送了一个请求（唯一监听端口`80`的服务），并收到了来自`go-demo`服务的响应。`proxy`在`go-demo`服务创建时自动进行了配置。

该过程的工作方式如下：

*Docker Flow Swarm Listener*运行在一个 Swarm 管理节点内，并查询 Docker API 以寻找新创建的服务。一旦找到新服务，它会查找其标签。如果服务包含`com.df.notify`标签（它可以包含任何值），则会获取所有其他以`com.df`开头的标签。这些标签将用于形成请求参数。这些参数会附加到作为`DF_NOTIF_CREATE_SERVICE_URL`环境变量定义在`swarm-listener`服务中的地址上。最后，发送请求。在这个特定的例子中，请求是为了使用`go-demo`（服务名称）重新配置`proxy`，路径为`/demo`，并且运行在端口`8080`上。分发标签在这个例子中不是必须的，因为我们只运行了一个`proxy`实例。然而，在生产环境中，我们应该至少运行两个`proxy`实例（以实现容错），而分发参数意味着该重新配置应应用于所有实例。

请参见*重新配置* ([`github.com/vfarcic/docker-flow-proxy#reconfigure`](https://github.com/vfarcic/docker-flow-proxy#reconfigure))部分，查看可以与代理一起使用的所有参数列表。

# 从代理中移除服务

由于`swarm-listener`正在监控`docker`服务，如果删除了一个服务，相关的`proxy`配置条目也将被删除：

```
docker service rm go-demo

```

如果您像检查任何其他容器服务的日志一样检查 Swarm Listener 的日志，您将看到类似以下条目的一个条目：

```
Sending service removed notification to http://proxy:8080/v1/
docker-flow-proxy/remove?serviceName=go-demo

```

不久之后，`proxy`日志中会出现一个新条目：

```
Processing request /v1/docker-flow-proxy/remove?serviceName=go-demo
Processing remove request /v1/docker-flow-proxy/remove
Removing go-demo configuration
Removing the go-demo configuration files
Reloading the proxy

```

从现在开始，服务`go-demo`将无法通过代理访问。

Swarm Listener 检测到服务已被移除，向`proxy`发送了一个通知，`proxy`进而改变其配置并重新加载底层的 HAProxy。

# 现在怎么办？

除了在每次创建或删除服务时自动配置代理的潜在实用性之外，`swarm-listener`显示了利用 Docker 远程 API 的有用性。如果您有自己的需求，而 Docker 或其生态系统中的工具尚未完全满足您的需求，编写自己的服务并不是很困难。事实上，在撰写本章时，Swarm 模式只有几个月的历史，并且没有太多第三方工具可以用来微调或扩展其行为。即使您找到了几乎满足您需求的所有工具，编写一些自定义代码以确切地满足您的需求仍然是个好主意。

我鼓励您启动您最喜欢的编辑器，并用您选择的编程语言编写一个服务。您可以监控服务，并在您的团队成员创建或删除服务时发送电子邮件给自己。或者您可以将统计数据集成到您最喜欢的监控工具中。

如果你对自己的服务已经没有了新的想法，并且你不怕*Go*（[`golang.org/`](https://golang.org/)），那么你可以尝试扩展*Docker Flow Swarm Listener*（[`github.com/vfarcic/docker-flow-swarm-listener`](https://github.com/vfarcic/docker-flow-swarm-listener)）。分叉它，添加一个新功能，并提交一个拉取请求。

记住，学习是金贵的。如果您学到了一些东西，那已经非常不错了。如果证明是有用的，那就更好了。

我们已经到了本章的结尾，您已经了解了程序。我们将销毁我们创建的机器并重新开始。

```
docker-machine rm -f swarm-1 swarm-2 swarm-3

```
