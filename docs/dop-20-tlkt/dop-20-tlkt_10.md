# 第十章. 部署管道的实施 – 后期阶段

我们不得不中断部署管道的实施并探索服务发现和代理服务。如果没有代理服务，我们的容器将无法以简单和可靠的方式访问。为了提供所有数据代理服务需要的数据，我们花了一些时间探索不同的选项，并提出了几种可能作为服务发现解决方案的组合。

带着服务发现和代理服务的工具，我们可以继续上次离开的地方，并完成部署管道的手动执行：

1.  检查代码 – 完成

1.  运行部署前测试 – 完成

1.  编译和/或打包代码 – 完成

1.  构建容器 – 完成

1.  将容器推送到注册表 – 完成

1.  部署容器到生产服务器 – 完成

1.  集成容器 – 待完成

1.  运行部署后测试 – 待完成

1.  推送测试容器到注册表 – 待完成

    图 10-1 – 使用 Docker 的部署管道的中间阶段

我们的部署管道缺少三个步骤。我们应该集成我们的容器，并且一旦完成，运行部署后测试。最后，我们应该将我们的测试容器推送到注册表，以便所有人都可以使用它。

我们将从启动我们用于部署管道的两个节点开始：

```
vagrant up cd prod

```

我们将使用 `prod2.y` `ml` Ansible playbook 来配置 `prod` 节点。它包含我们在前一章中已经讨论过的服务发现和代理角色：

```
- hosts: prod
 remote_user: vagrant
 serial: 1
 sudo: yes
 roles:
 - common
 - docker
 - docker-compose
 - consul
 - registrator
 - consul-template
 - nginx

```

一旦运行，我们的 `prod` 节点将会运行 Consul、Registrator、Consul Template 和 nginx。它们将允许我们将所有请求代理到它们目标服务（目前只有 `books-ms`）。让我们从 `cd` 节点运行 playbook。

```
vagrant ssh cd
ansible-playbook /vagrant/ansible/prod2.yml \
 -i /vagrant/ansible/hosts/pro
d

```

# 启动容器

在我们继续集成之前，我们应该运行这些容器：

```
wget https://raw.githubusercontent.com/vfarcic\
/books-ms/master/docker-compose.yml
export DOCKER_HOST=tcp://prod:2375
docker-compose up -d app

```

由于我们使用 Consul 和 Registrator 配置了这个节点，这两个容器中的 IP 和端口应该在注册表中可用。我们可以通过在浏览器中打开 `http://10.100.198.201:8500/ui` 访问 Consul UI 来确认这一点。

如果我们点击 **Nodes** 按钮，我们可以看到 `prod` 节点已注册。更进一步，点击 `prod` 节点按钮应该会显示它包含两个服务；`consul` 和 `books-ms`。我们启动的 `mongo` 容器没有注册，因为它没有暴露任何端口：

启动容器 – 完成

图 10-2 – 带有生产节点和运行在其上的服务的 Consul 截图

发送请求到 Consul，可以看到相同的信息：

```
curl prod:8500/v1/catalog/services | jq '.'
curl prod:8500/v1/catalog/service/books-ms | jq '.'

```

第一个命令列出了 Consul 中注册的所有服务。输出如下：

```
{
  "dockerui": [],
  "consul": [],
  "books-ms": []
}
```

第二个命令输出与 `books-ms` 服务相关的所有信息：

```
[
 {
 "ModifyIndex": 27,
 "CreateIndex": 27,
 "Node": "prod",
 "Address": "10.100.198.201",
 "ServiceID": "prod:vagrant_app_1:8080",
 "ServiceName": "books-ms",
 "ServiceTags": [],
 "ServiceAddress": "10.100.198.201",
 "ServicePort": 32768,
 "ServiceEnableTagOverride": false
 }
]

```

在容器启动并运行，且其信息已存储在服务注册表中后，我们可以重新配置 nginx，使得 `books-ms` 服务可以通过标准 HTTP 端口 `80` 进行访问。

# 集成服务

我们将从确认 nginx 并不认识我们的服务开始：

```
curl http://prod/api/v1/books

```

发送请求后，nginx 返回了 `404 Not Found` 消息。我们来修改一下：

```
Exit
vagrant ssh prod
wget https://raw.githubusercontent.com/vfarcic\
/books-ms/master/nginx-includes.conf \
 -O /data/nginx/includes/books-ms.conf
wget https://raw.githubusercontent.com/vfarcic\
/books-ms/master/nginx-upstreams.ctmpl \
 -O /data/nginx/upstreams/books-ms.ctmpl
consul-template \
 -consul localhost:8500 \
 -template "/data/nginx/upstreams/books-ms.ctmpl:\
/data/nginx/upstreams/books-ms.conf:\
docker kill -s HUP nginx" \
 -once

```

我们已经在上一章中完成了大部分步骤，因此这次我们将简要介绍。我们进入了 `prod` 节点，并从代码仓库下载了包含文件和上游模板。然后，我们运行了 `consul-template`，它从 Consul 获取数据并应用到模板中，结果就是 nginx 上游配置文件。请注意，这一次，我们添加了第三个参数 `docker kill -s HUP nginx`。不仅 `consul-template` 从模板中创建了配置文件，它还重新加载了 nginx。之所以从 `prod` 服务器运行这些命令，而不是像上一章那样远程操作，是因为自动化的原因。我们刚刚执行的步骤更接近于我们将在下一章中自动化这一部分过程的方式。

现在我们可以测试我们的服务是否确实可以通过端口 `80` 进行访问：

```
exit
vagrant ssh cd
curl -H 'Content-Type: application/json' -X PUT -d \
 "{\"_id\": 1,
 \"title\": \"My First Book\",
 \"author\": \"John Doe\",
 \"description\": \"Not a very good book\"}" \
 http://prod/api/v1/books | jq '.'
curl http://prod/api/v1/books | jq
 '.'

```

# 运行部署后测试

尽管我们确实通过发送请求并观察正确的响应来确认服务可以从 nginx 访问，但如果我们试图实现过程的完全自动化，这种验证方式并不可靠。相反，我们应该重复执行集成测试，但这次使用端口 `80`（或者不指定端口，因为 `80` 是标准的 `HTTP` 端口）：

```
git clone https://github.com/vfarcic/books-ms.git
cd books-ms
docker-compose \
 -f docker-compose-dev.yml \
 run --rm \
 -e DOMAIN=http://10.100.198.201 \
 integ

```

输出如下：

```
[info] Loading project definition from /source/project
[info] Set current project to books-ms (in build file:/source/)
[info] Compiling 2 Scala sources to /source/target/scala-2.10/classes...
[info] Compiling 2 Scala sources to /source/target/scala-2.10/test-classes...
[info] ServiceInteg
[info]
[info] GET http://10.100.198.201/api/v1/books should
[info] + return OK
[info]
[info] Total for specification ServiceInteg
[info] Finished in 23 ms
[info] 1 example, 0 failure, 0 error
[info] Passed: Total 1, Failed 0, Errors 0, Passed 1
[success] Total time: 27 s, completed Sep 17, 2015 7:49:28 PM

```

正如预期的那样，输出显示集成测试成功通过。事实上，我们只有一个测试，它发出了与之前运行的 `curl` 命令相同的请求。然而，在“真实世界”情况下，测试的数量会增加，使用适当的测试框架比运行 `curl` 请求要可靠得多。

# 将测试容器推送到注册表

说实话，我们已经将这个容器推送到了注册表，以避免每次需要它时重新构建，从而节省等待时间。然而，这一次，我们应该将它作为部署管道过程的一部分进行推送。我们尝试按任务的重要性顺序运行，以便尽快获得反馈。将包含测试的容器推送是我们优先级列表中的低项，因此我们将其留到最后。现在，其他一切都顺利运行后，我们可以推送容器，让其他人从注册表中拉取并按需使用。

```
docker push 10.100.198.200:5000/books-
ms-tests

```

# 检查清单

我们成功完成了整个部署流程。由于我们需要休息几次并探索不同的方式，所以花费了相当多的时间。我们无法在不探索配置管理概念和工具的情况下部署到生产环境。后来，我们再次遇到瓶颈，必须学习服务发现和代理，才能成功集成服务容器：

1.  检出代码 – 完成

1.  运行部署前的测试 – 完成

1.  编译和/或打包代码 – 完成

1.  构建容器 – 完成

1.  将容器推送到注册表 – 完成

1.  将容器部署到生产服务器 – 完成

1.  运行部署后的测试 – 完成

1.  将测试容器推送到注册表 – 完成！检查清单

    图 10-3 – 使用 Docker 部署流程的后期阶段

现在我们已经准备好。我们能够手动执行部署过程。下一步是将所有这些命令自动化，并开始从头到尾自动运行整个流程。我们将销毁当前使用的节点，以便重新开始并确认自动化过程是否确实有效：

```
exit
vagrant destroy -f

```
