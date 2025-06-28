# 从 Dockerfile 构建应用程序镜像

在上一章中，您学习了 OpenShift 模板的概念，以及如何编写自己的模板并从模板部署应用程序。

本章将介绍 OpenShift 如何通过提供自动化应用程序从源代码部署的 Docker 构建策略来简化 Docker 镜像生命周期。本章是一个实践实验室，描述了如何使用 Docker **源代码到镜像**（**S2I**）策略进行应用程序交付。

完成本章后，您将学会如何从 Dockerfile 构建和部署应用程序。

本章将涵盖以下主题：

+   OpenShift 的 Dockerfile 开发

+   从 Dockerfile 构建应用程序

+   Dockerfile 构建自定义

# 技术要求

本章没有严格的环境限制；支持任何 OpenShift 安装和开发环境——MiniShift、`oc cluster up`、标准的基于 Ansible 的生产就绪部署。您可以根据自己的需要选择使用哪个版本。然而，本章基于 `oc cluster up` 方法。以下 Vagrantfile 可用于部署开发环境：

```
Vagrant.configure(2) do |config| 
  config.vm.define "openshift" do |conf| 
    conf.vm.box = "centos/7" 
    conf.vm.network "private_network", ip: "172.24.0.11" 
    conf.vm.hostname = 'openshift.example.com' 
    conf.vm.network "forwarded_port", guest: 80, host: 980
    conf.vm.network "forwarded_port", guest: 443, host: 9443
    conf.vm.network "forwarded_port", guest: 8080, host: 8080
    conf.vm.network "forwarded_port", guest: 8443, host: 8443
    conf.vm.provider "virtualbox" do |v| 
      v.memory = 4096 
      v.cpus = 2 
    end
    conf.vm.provision "shell", inline: $lab_main 
  end 

$lab_main = <<SCRIPT
cat <<EOF >> /etc/hosts
172.24.0.11 openshift.example.com openshift
172.24.0.12 storage.example.com storage nfs
EOF
systemctl disable firewalld
systemctl stop firewalld
yum update -y
yum install -y epel-release git
yum install -y docker tree
cat << EOF >/etc/docker/daemon.json
{
   "insecure-registries": [
     "172.30.0.0/16"
   ]
}
EOF
systemctl start docker
systemctl enable docker
yum -y install centos-release-openshift-origin39
yum -y install origin-clients
oc cluster up
SCRIPT
```

环境可以按如下方式部署：

```
$ vagrant up
```

部署前面列出的 vagrant 机器后，您可以按如下方式连接到它：

```
$ vagrant ssh
```

最后，登录为开发者用户，以便能够执行操作：

```
$ oc login -u developer
Server [https://localhost:8443]:
The server uses a certificate signed by an unknown authority.
You can bypass the certificate check, but any data you send to the server could be intercepted by others.
Use insecure connections? (y/n): y

Authentication required for https://localhost:8443 (openshift)
Username: developer
Password: <ANY PASSWORD>
Login successful.

You have one project on this server: "myproject"

Using project "myproject".
Welcome! See 'oc help' to get started.

$ sudo -i
# 
```

# OpenShift 的 Dockerfile 开发

本书的早期章节中，我们解释了如何通过 Dockerfile 开发将应用程序容器化。这涉及了 `docker build` 工具，它通过遵循 Dockerfile 指令创建一个可用的容器镜像。

一般来说，OpenShift 支持现有的应用程序 Dockerfile，但它有一些特殊的默认安全要求，需要您修改或调整应用程序的 Dockerfile 以符合 OpenShift 安全标准。

默认安全策略会以随机的**用户 ID**（**UID**）运行任何容器，并忽略 `USER` Dockerfile 指令。应用程序总是以 root 用户组运行。

如果应用程序需要读写访问权限，您需要为 root 组配置 RW 访问权限，通常可以通过以下 Dockerfile 代码段来实现：

```
...
RUN chown -R 1001:0 /var/lib/myaplication /var/log/myapplication &amp;&amp; \
    chmod -R g=u /var/lib/myaplication /var/log/myapplication
...
USER 1001
```

上面的示例将目录和文件所有者更改为`1001`，并将组的权限设置为与所有者相同。这使得应用程序在任何 UID 下都能具有读写权限。

OpenShift 不允许应用程序绑定到小于 `1024` 的端口。因此，可能需要调整 Dockerfile 中的 `EXPOSE` 指令，以便能够在 OpenShift 基础设施中运行应用程序。

以下 Dockerfile 代码段提供了如何修改 HTTPD 镜像端口的示例：

```
EXPOSE 8080
```

在大多数情况下，您需要调整应用程序配置，以便监听新的端口。例如，对于 HTTPD，必须更改 `Listen` 指令选项。

# 从 Dockerfile 构建应用程序

在不同的命名空间中部署应用程序是一种良好的实践：

```
$ oc new-project dockerfile
Now using project "dockerfile" on server "https://localhost:8443".
```

对于这个实验，我们将使用`redis`容器。首先，我们需要一个包含额外文件的 Dockerfile，位于[`github.com/docker-library/redis.git`](https://github.com/docker-library/redis.git)。让我们克隆这个仓库到本地，以了解其结构：

```
$ git clone https://github.com/docker-library/redis.git
Cloning into 'redis'...
remote: Counting objects: 738, done.
remote: Compressing objects: 100% (15/15), done.
remote: Total 738 (delta 7), reused 13 (delta 4), pack-reused 719
Receiving objects: 100% (738/738), 108.56 KiB | 0 bytes/s, done.
Resolving deltas: 100% (323/323), done.
```

该仓库包含多个目录，表示 Redis 的不同版本：

```
$ tree redis/
redis/
├── 3.2
│   ├── 32bit
│   │   ├── docker-entrypoint.sh
│   │   └── Dockerfile
│   ├── alpine
│   │   ├── docker-entrypoint.sh
│   │   └── Dockerfile
│   ├── docker-entrypoint.sh
│   └── Dockerfile
├── 4.0
│   ├── 32bit
│   │   ├── docker-entrypoint.sh
│   │   └── Dockerfile
│   ├── alpine
│   │   ├── docker-entrypoint.sh
│   │   └── Dockerfile
│   ├── docker-entrypoint.sh
│   └── Dockerfile
├── 5.0-rc
│   ├── 32bit
│   │   ├── docker-entrypoint.sh
│   │   └── Dockerfile
│   ├── alpine
│   │   ├── docker-entrypoint.sh
│   │   └── Dockerfile
│   ├── docker-entrypoint.sh
│   └── Dockerfile
├── generate-stackbrew-library.sh
├── LICENSE
├── README.md
└── update.sh
```

仓库结构包含多个目录，表示特定版本。要构建应用程序，我们需要指定一个包含所需 Dockerfile 的目录。可以通过使用`oc new-app`的`--context-dir`选项来实现。稍后会详细描述。

# 一个简单的 Dockerfile 构建

好的，我们了解了目录结构，并希望从现有的 Dockerfile 构建和部署 Redis 应用程序。我们专注于版本 3.2。`oc new-app`可以使用子目录从源代码启动构建。我们准备好启动一个简单的 Dockerfile 构建：

```
$ oc new-app https://github.com/docker-library/redis.git --context-dir=3.2
...
<OUTPUT OMITTED>
...
Run 'oc status' to view your app.
```

如我们所见，OpenShift 创建了以下多个对象：

+   名为`debian`的`imagestream`

+   名为`redis`的`buildconfig`

+   名为`redis`的`deploymentconfig`

+   名为`redis`的`service`

你可以运行`oc get all`命令，以确保所有对象都已创建：

```
$ oc get all
NAME                  TYPE        FROM        LATEST
buildconfigs/redis    Docker      Git         1

NAME                TYPE    FROM        STATUS    STARTED        DURATION
builds/redis-1      Docker  Git@d24f2be Complete  39 seconds ago 6s

NAME                DOCKER REPO                       TAGS        UPDATED
imagestreams/debian 172.30.1.1:5000/dockerfile/debian jessie-slim 39 seconds ago
imagestreams/redis  172.30.1.1:5000/dockerfile/redis  latest      34 seconds ago

NAME                    REVISION DESIRED CURRENT TRIGGERED BY
deploymentconfigs/redis 1        1        1      config,image(redis:latest)

NAME                    READY      STATUS      RESTARTS    AGE
po/redis-1-build        0/1        Completed   0           40s
po/redis-1-js789        1/1        Running     0           33s

```

```
NAME                    DESIRED    CURRENT     READY       AGE
rc/redis-1              1           1          1           35s

NAME                    CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
svc/redis               172.30.165.223  <none>        6379/TCP   41s
```

上一个命令启动了一个构建。OpenShift 启动了一个`-build`的 Pod 来进行构建。你可能会暂时看到一个名字中带有`-build`的 Pod 处于`Running`状态：

```
$ oc get pod
NAME            READY    STATUS             RESTARTS    AGE
redis-1-build   1/1      Running            0           6s
redis-1-deploy  0/1      ContainerCreating  0           1s
```

Docker 构建由`build`配置对象控制。可以使用`oc logs bc/<NAME>`命令显示构建状态：

```
$ oc logs bc/redis|tail -n10
Successfully built b3b7a77f988a
Pushing image 172.30.1.1:5000/dockerfile/redis:latest ...
Pushed 0/6 layers, 7% complete
Pushed 1/6 layers, 30% complete
Pushed 2/6 layers, 43% complete
Pushed 3/6 layers, 57% complete
Pushed 4/6 layers, 75% complete
Pushed 5/6 layers, 87% complete
Pushed 6/6 layers, 100% complete
Push successful
```

解决构建过程故障的另一种方法是使用`oc status`。

OpenShift 从 Dockerfile 构建了 Redis 镜像并上传到本地注册表。让我们确保容器正常工作：

```
$ oc get pod
NAME             READY    STATUS      RESTARTS  AGE
redis-1-build    0/1      Completed   0         2m
redis-1-js789    1/1      Running     0         1m

$ oc exec redis-1-8lf8h /usr/local/bin/redis-cli ping
PONG
```

```
$ oc rsh redis-1-js789 /usr/local/bin/redis-server --version
Redis server v=3.2.11 sha=00000000:0 malloc=jemalloc-4.0.3 bits=64 build=994283e2d09fba41
```

在这个实验中，我们使用一个简单的`ping`测试来确保基本的可达性。我们使用容器提供的`redis-cli`命令。

所以，看起来我们的 Redis 应用程序工作正常，版本是 3.2.11。

# Dockerfile 构建定制

如前所述，OpenShift 可以从 Dockerfile 构建应用程序。有时，应用程序的源代码会更新，需要使用新源代码重新启动构建过程。OpenShift 通过`oc start-build`命令支持这一功能。

在这一部分中，我们将使用`oc new-app`命令最近创建的镜像流，启动一个构建过程，使用应用程序的新源代码。

我们使用特定目录 3.0 的源代码构建了 Redis 应用程序。

源代码包含另一个 Dockerfile，使用的是更新的版本 4.0：

```
$ tree redis/4.0/
redis/4.0/
├── 32bit
│   ├── docker-entrypoint.sh
│   └── Dockerfile
├── alpine
│   ├── docker-entrypoint.sh
│   └── Dockerfile
├── docker-entrypoint.sh
└── Dockerfile
```

现在，假设我们需要使用另一个仓库中的新代码，或现有仓库中的另一个上下文目录来更新应用程序。对于我们的特定情况，看起来我们需要将上下文目录从`3.2`改为`4.0`。

`oc new-app`命令创建了多个控制应用程序构建和部署的实体。构建过程由`build config`对象控制。我们需要显示此对象以了解如何更改以指向仓库中的另一个目录：

```
$ oc get bc
NAME             TYPE         FROM     LATEST
redis            Docker       Git      1

$ oc get bc redis -o yaml
apiVersion: v1
kind: BuildConfig
metadata:
  annotations:
    openshift.io/generated-by: OpenShiftNewApp
  creationTimestamp: 2018-06-04T19:54:26Z
  labels:
    app: redis
  name: redis
  namespace: dockerfile
  resourceVersion: "256886"
  selfLink: /oapi/v1/namespaces/dockerfile/buildconfigs/redis
  uid: 16116413-6831-11e8-91ff-5254005f9478
spec:
  failedBuildsHistoryLimit: 5
  nodeSelector: null
  output:
    to:
      kind: ImageStreamTag
      name: redis:latest
  postCommit: {}
  resources: {}
  runPolicy: Serial
 source:
 contextDir: "3.2"
 git:
 uri: https://github.com/docker-library/redis.git
 type: Git <OMITTED>
```

我们突出显示了一个源元素，指定了构建过程中使用的目录。现在，如果有任务需要指向另一个上下文目录，我们只需要通过修改对象定义中的`spec.source.contextDir`来更新`bc/redis`对象。这可以通过几种方式实现：

+   手动使用`oc edit`

+   在使用`oc patch`的脚本中

`oc edit bc/redis`将启动一个文本编辑器来修改对象。以下是如何使用`oc patch`命令更新对象内容的示例：

```
$ oc patch bc/redis --patch '{"spec":{"source":{"contextDir":"4.0"}}}'
buildconfig "redis" patched

$ oc get bc/redis -o yaml|grep contextDir:
    contextDir: 4.0
```

好吧，我们更新了构建配置，但没有任何变化。我们的 Pod 没有改变。这表明构建过程没有被触发。你可以通过`oc get pod`来验证这一点。

如果需要启动应用程序重建过程，必须运行`oc start-build`。此命令将从现有的构建配置启动一个新构建。

让我们列出所有当前的构建：

```
$ oc get build
NAME         TYPE     FROM         STATUS     STARTED         DURATION
redis-1      Docker   Git@d24f2be  Complete   12 minutes ago  6s
```

所以，最近我们启动了一个构建，那个构建在一段时间前完成了。让我们尝试再次运行构建：

```
$ oc start-build bc/redis
build "redis-2" started
```

构建会创建多个表示新版本（版本 2）的新 Pod。过一段时间后，Pod 状态将会改变：

```
$ oc get pod
NAME              READY      STATUS             RESTARTS   AGE
redis-1-build     0/1        Completed          0          13m
redis-1-js789     1/1        Running            0          12m
redis-2-build     0/1        Completed          0          8s
redis-2-deploy    1/1        Running            0          2s
redis-2-l6bbl     0/1        ContainerCreating  0          0s
```

你可以看到`redis`现在正在经过构建和部署阶段，然后新的`redis`容器将会启动并运行。如果你等待一分钟左右，你应该能看到以下内容：

```
$ oc get pod
NAME              READY      STATUS            RESTARTS    AGE
```

```
redis-1-build     0/1        Completed         0           14m
redis-2-build     0/1        Completed         0           1m
redis-2-l6bbl     1/1        Running           0           57s
```

好吧，看起来这个构建已经完成：

```
$ oc get build
NAME      TYPE       FROM            STATUS     STARTED         DURATION
redis-1   Docker     Git@d24f2be     Complete   15 minutes ago  6s
redis-2   Docker     Git@d24f2be     Complete   2 minutes ago   6s
```

现在我们可以检查应用程序的版本：

```
$ oc rsh redis-2-l6bbl /usr/local/bin/redis-server --version
Redis server v=4.0.9 sha=00000000:0 malloc=jemalloc-4.0.3 bits=64 build=40ca48d6a92db598

```

最后，让我们确保应用程序已启动并运行：

```
$ oc rsh redis-2-l6bbl /usr/local/bin/redis-cli ping
PONG
```

我们能够从更新后的源代码启动构建。

清理你的实验环境。

```
$ oc delete all --all
deploymentconfig "redis" deleted
buildconfig "redis" deleted
imagestream "debian" deleted
imagestream "redis" deleted
pod "redis-2-vw92x" deleted
service "redis" deleted

$ oc delete project dockerfile
project "dockerfile" deleted

$ oc project myproject
Now using project "myproject" on server "https://localhost:8443".
```

如果你打算继续下一章，你可以保持你的 OpenShift 集群运行，否则你可以关闭或删除 vagrant 虚拟机。

# 概述

在本章中，你已经学习了如何调整 Dockerfile，以便在 OpenShift 中运行。我们还解释了如何从 Dockerfile 构建应用程序，如何使用`oc new-app`启动 Docker 构建，以及如何使用`oc start-build`从现有构建配置启动新构建。

在下一章中，我们将讨论已经在 Docker Hub 上可用的最常用应用程序镜像。但时不时地，需要构建一个包含自定义软件或符合公司安全政策/标准的自定义镜像。我们将学习 OpenShift 如何通过 S2I 构建策略自动化构建过程，这是 OpenShift 的主要优势之一，以及它如何允许你从应用程序的源代码构建镜像，然后将其作为容器运行。

# 问题

1.  以下哪些 OpenShift 命令可以更新现有的 API 对象？请选择两个：

+   1.  `oc edit bc/redis`

    1.  `oc get bc redis`

    1.  `oc patch bc/redis --patch ...`

    1.  `oc update bc/redis`

    1.  `oc build bc/redis`

1.  以下哪个命令可以启动新的构建？（选择一个）：

+   1.  `oc new-app`

    1.  `oc new build`

    1.  `oc start-build`

    1.  `oc get build`

1.  在仓库中必须存在哪个文件才能执行 docker build（选择一个）？

+   1.  `Jenkinsfile`

    1.  `Dockerfile`

    1.  `README.md`

    1.  `index.php`

    1.  `docker.info`

# 深入阅读

以下是与本章节相关的主题链接，您可能希望深入了解：

+   **在** [`docs.openshift.com/container-platform/3.9/dev_guide/application_lifecycle/new_app.html`](https://docs.openshift.com/container-platform/3.9/dev_guide/application_lifecycle/new_app.html) 中创建新应用程序

+   **构建在** [`docs.openshift.org/latest/dev_guide/builds/index.html`](https://docs.openshift.org/latest/dev_guide/builds/index.html) 中的工作原理
