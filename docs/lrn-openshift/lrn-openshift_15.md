# 从源代码构建 PHP 应用程序

在上一章中，你学会了如何从 Dockerfile 构建应用程序，如何使用`oc new-app`启动 Docker 构建，最后如何使用`oc start-build`从现有的构建配置启动新构建。

在本章中，我们将讨论 Docker Hub 上已有的最常用的应用程序镜像。偶尔需要构建一个自定义镜像，包含定制的软件或符合公司安全政策/标准。你将学习 OpenShift 如何通过**Source-to-Image**（**S2I**）构建策略自动化构建过程，这是其主要优势之一。你还将学习如何从应用程序的源代码构建镜像，并将其作为容器运行。

本章将涵盖以下主题：

+   PHP S2I

+   构建一个简单的 PHP 应用程序

+   理解 PHP 构建过程

# 技术要求

本章在实验环境方面没有严格要求，因此任何 OpenShift 安装和开发环境都是支持的——Minishift、`oc cluster up`或基于 Ansible 的标准生产就绪部署。选择使用哪种方式由你决定。不过，本章将重点介绍`oc cluster up`方法。以下的 Vagrantfile 可以用来部署我们的虚拟实验室：

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
yum install -y docker
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

该环境可以通过运行单个命令来部署：

```
$ vagrant up
```

一旦虚拟机部署完成，你可以按以下方式连接到它：

```
$ vagrant ssh
```

最后，作为`developer`用户登录，以便能够执行没有特权的操作：

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
```

# PHP S2I

OpenShift 支持 PHP 的 S2I 构建，也支持许多其他运行时。S2I 过程通过将应用程序的源代码与基础构建器镜像结合，生成一个可运行的镜像，该镜像准备好应用程序。构建器是一个特殊的镜像，能够处理特定编程语言/框架的应用程序安装和配置。例如，PHP 构建器只能处理 PHP 源代码，并且默认不支持 Java。大多数常用的编程语言，如 Python、Ruby、Java 和 Node.js，已经由 OpenShift 内置的构建器覆盖。S2I 过程包括以下步骤：

1.  确定正确的基础构建器镜像。此过程依赖于复杂的启发式算法，主要通过查找特定的文件和文件扩展名来实现，比如 Ruby on Rails 的`Gemfile`，或 Python 的`requirements.txt`。构建器镜像的运行时环境也可以通过 CLI 由用户覆盖

1.  创建一个指向应用程序源代码仓库和构建器镜像的 ImageStream 的`BuildConfig`

1.  从构建器镜像启动`build` pod

1.  下载应用程序的源代码

1.  使用`tar`工具将脚本和应用程序的源代码流式传输到构建器镜像容器中，适用于支持该工具的镜像

1.  运行 `assemble` 脚本（由构建器镜像提供的脚本优先级最高）

1.  将最终的镜像保存到内部注册表

1.  创建支持应用程序所需的资源，包括但不限于 `DeploymentConfig`、`Service` 和 `Pod`。

PHP 构建器支持通过使用多个环境变量来更改默认的 PHP 配置。每个特定构建器的网页上有详细描述。

构建过程可以通过运行 `oc new-app` 命令来启动。此命令将仓库 URL 或本地路径作为参数，并同时创建所有必需的 OpenShift 实体来支持 S2I 构建和应用程序部署。

默认情况下，创建以下 OpenShift 实体：

| **Type** | **Name** | **Description** |
| --- | --- | --- |
| Pod | `<application name>-<build sequential number>-build` | 构建您的应用程序的构建器 Pod，生成应用程序镜像并可能生成一些供未来构建使用的工件 |
| Pod | `<name>-<build sequential number>-<id>` | 构建过程中生成的应用程序 Pod |
| Service | `<name>` | 应用程序服务 |
| Replication controller | `<name>-<build sequential number>` | 应用程序的复制控制器，默认情况下仅维持一个副本 |
| Deployment config | `<name>` | 包含有关如何部署应用程序的所有信息的部署配置 |
| Image stream | `<name>` | 构建的镜像路径 |
| Build config | `<name>` | 包含如何构建应用程序的所有必要信息的构建配置 |
| Build | `<name>-<build sequential number>` | 特定构建；可以多次运行 |

OpenShift 不会自动创建路由。如果需要暴露应用程序，应该通过运行 `oc expose svc <name>` 来创建路由。

# 构建一个简单的 PHP 应用程序

第一个 S2I 实验将使用一个非常简单的 PHP 应用程序，使用标准的 PHP 函数 `phpinfo()` 显示 PHP 配置信息。它由一个 `index.php` 文件组成，内容如下：

```
$ cat index.php
<?php
phpinfo();
?>
```

这个应用程序足以展示基本的 S2I 构建。我们已经在 GitHub 上准备好了一个包含我们应用程序代码的 Git 仓库。该仓库位于 [`github.com/neoncyrex/phpinfo.git`](https://github.com/neoncyrex/phpinfo.git)，并将在本实验中使用。

首先，我们需要为我们的新实验创建一个单独的项目，如下所示：

```
$ oc new-project phpinfo
Now using project "phpinfo" on server "https://localhost:8443".
<OMITTED>
```

`oc new-app` 命令可以从源代码构建应用程序，使用本地或远程 Git 仓库。

让我们将仓库克隆到本地，以便能够对其进行更改。这将创建 `phpinfo` 目录，其中包含仓库文件：

```
$ git clone https://github.com/neoncyrex/phpinfo.git
Cloning into 'phpinfo'...
remote: Counting objects: 6, done.
remote: Compressing objects: 100% (3/3), done.
remote: Total 6 (delta 0), reused 3 (delta 0), pack-reused 0
Unpacking objects: 100% (6/6), done.
```

构建和应用程序部署过程可以通过运行 `oc new-app` 命令来启动。此基本 S2I 策略可以如下触发：

```
$ oc new-app phpinfo
--> Found image 23e49b6 (17 hours old) in image stream "openshift/php" under tag "7.0" for "php"
...
<output omitted>
...
--> Success
    Build scheduled, use 'oc logs -f bc/phpinfo' to track its progress.
    Application is not exposed. You can expose services to the outside world by executing one or more of the commands below:
     'oc expose svc/phpinfo'
    Run 'oc status' to view your app.
```

前面的命令执行以下操作：

+   使用 `phpinfo` 作为应用程序源代码的路径

+   自动检测编程语言——PHP

+   启动构建过程

+   创建多个 OpenShift 资源

构建过程需要一些时间。在第一阶段，你可以看到一个名称中带有 `-build` 的容器。这个容器是从 PHP 构建器镜像部署的，负责执行构建操作：

```
$ oc get pod
NAME             READY  STATUS    RESTARTS  AGE
phpinfo-1-build  1/1    Running   0         23s
```

经过一段时间后，应用程序将可用。这意味着应用程序的 pod 应该处于 `Running` 状态：

```
$ oc get pod
NAME              READY  STATUS     RESTARTS  AGE
phpinfo-1-build   0/1    Completed  0         39s
phpinfo-1-h9xt5   1/1    Running    0         4s
```

OpenShift 已构建并部署了 `phpinfo` 应用程序，现在可以通过其服务 IP 进行访问。让我们尝试使用 `curl` 命令访问我们的应用程序：

```
$ oc get svc
NAME    CLUSTER-IP    EXTERNAL-IP PORT(S)            AGE
phpinfo 172.30.54.195 <none>      8080/TCP,8443/TCP  1h

$ curl -s http://172.30.54.195:8080 | head -n 10
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "DTD/xhtml1-transitional.dtd">
<html ><head>
<style type="text/css">
body {background-color: #fff; color: #222; font-family: sans-serif;}
pre {margin: 0; font-family: monospace;}
a:link {color: #009; text-decoration: none; background-color: #fff;}
a:hover {text-decoration: underline;}
table {border-collapse: collapse; border: 0; width: 934px; box-shadow: 1px 2px 3px #ccc;}
.center {text-align: center;}
.center table {margin: 1em auto; text-align: left;}
```

`phpinfo()` 函数以 HTML 表格的形式显示 PHP 配置信息。

可以通过运行 `oc status` 或 `oc status -v` 命令显示构建过程的摘要：

```
$ oc status -v
In project phpinfo on server https://localhost:8443

svc/phpinfo - 172.30.54.195 ports 8080, 8443
  dc/phpinfo deploys istag/phpinfo:latest <-
    bc/phpinfo source builds https://github.com/neoncyrex/phpinfo.git#master on openshift/php:7.0
    deployment #1 deployed about an hour ago - 1 pod

Info:
  * pod/phpinfo-1-build has no liveness probe to verify pods are still running.
    try: oc set probe pod/phpinfo-1-build --liveness ...
  * dc/phpinfo has no readiness probe to verify pods are ready to accept traffic or ensure deployment is successful.
    try: oc set probe dc/phpinfo --readiness ...
  * dc/phpinfo has no liveness probe to verify pods are still running.
    try: oc set probe dc/phpinfo --liveness ...
View details with 'oc describe <resource>/<name>' or list everything with 'oc get all'.
```

上述命令显示了部署 `#1` 已经完成。它还可能包含一些有用的信息，用于在构建出现问题时进行故障排除。

还有另一种显示详细构建日志的方法——使用 `oc logs` 命令。我们需要显示 `buildconfig`（或简称 `bc`）实体的日志，可以通过如下命令显示：`oc logs bc/phpinfo`：

```
$ oc logs bc/phpinfo
Cloning "https://github.com/neoncyrex/phpinfo.git" ...
  Commit: 638030df45052ad1d9300248babe0b141cf5dbed (initial commit)
  Author: vagrant <vagrant@openshift.example.com>
  Date: Sat Jun 2 04:22:59 2018 +0000
---> Installing application source...
=> sourcing 20-copy-config.sh ...
---> 05:00:11 Processing additional arbitrary httpd configuration provided by s2i ...
=> sourcing 00-documentroot.conf ...
=> sourcing 50-mpm-tuning.conf ...
=> sourcing 40-ssl-certs.sh ...
Pushing image 172.30.1.1:5000/phpinfo/phpinfo:latest ...
Pushed 0/10 layers, 23% complete
Pushed 1/10 layers, 30% complete
Pushed 2/10 layers, 21% complete
Pushed 3/10 layers, 30% complete
Pushed 4/10 layers, 40% complete
Pushed 5/10 layers, 51% complete
Pushed 6/10 layers, 60% complete
Pushed 7/10 layers, 76% complete
Pushed 8/10 layers, 87% complete
Pushed 9/10 layers, 95% complete
Pushed 10/10 layers, 100% complete
Push successful
```

上述输出给了我们一些关于构建如何工作的线索。

# 理解 PHP 构建过程

现在我们知道 `phpinfo` 应用程序按预期工作，让我们专注于理解构建过程所需的低级细节。OpenShift 创建了多个 API 资源来使构建成为可能。其中一些与部署过程相关，这部分内容我们在前面的章节已经学习过了。我们可以通过如下命令显示所有实体：

```
$ oc get all
NAME                 TYPE   FROM       LATEST
buildconfigs/phpinfo Source Git@master 1

NAME            TYPE    FROM       STATUS    STARTED           DURATION
builds/phpinfo-1 Source Git@638030d Complete About an hour ago 34s

NAME                 DOCKER REPO                     TAGS                      UPDATED
imagestreams/phpinfo 172.30.1.1:5000/phpinfo/phpinfo latest 
About an hour ago

NAME                     REVISION DESIRED CURRENT TRIGGERED BY
deploymentconfigs/phpinfo 1       1       1 config,image(phpinfo:latest)

NAME                READY  STATUS     RESTARTS  AGE
po/phpinfo-1-build  0/1    Completed  0         1h
po/phpinfo-1-h9xt5  1/1    Running    0         1h

NAME         DESIRED  CURRENT READY  AGE
rc/phpinfo-1 1        1       1      1h

NAME        CLUSTER-IP    EXTERNAL-IP PORT(S)           AGE
svc/phpinfo 172.30.54.195 <none>      8080/TCP,8443/TCP 1h
```

上述输出中的大多数实体（如 pod、服务、复制控制器和部署配置）在前面的章节中你应该已经熟悉了。

S2I 构建策略使用以下附加实体——**构建配置、构建** 和 **镜像流**。构建配置包含构建应用程序所需的所有信息。与 OpenShift 其他 API 资源一样，其配置可以通过 `oc get` 命令获取：

```
$ oc get bc phpinfo -o yaml
apiVersion: v1
kind: BuildConfig
...
<output omitted>
...
spec:
...
<output omitted>
...
  source:
    git:
      ref: master
      uri: https://github.com/neoncyrex/phpinfo.git
    type: Git
  strategy:
    sourceStrategy:
      from:
        kind: ImageStreamTag
        name: php:7.0
        namespace: openshift
    type: Source
  successfulBuildsHistoryLimit: 5
  triggers:
  - github:
      secret: 5qFv8z-J1me1Mj7Q27rY
    type: GitHub
  - generic:
      secret: -g6nTMasd6TRCMxBvKWz
    type: Generic
  - type: ConfigChange
  - imageChange:
      lastTriggeredImageID: centos/php-70-centos7@sha256:eb2631e76762e7c489561488ac1eee966bf55601b6ab31d4fbf60315d99dc740
    type: ImageChange
status:
  lastVersion: 1
```

以下字段尤为重要：

+   `spec.source.git`: 应用程序源代码的仓库 URL

+   `spec.strategy.sourceStrategy`: 包含将使用哪个构建器的信息。

在我们的例子中，OpenShift 使用来自 `openshift` 命名空间中的内建构建器，基于镜像流 `php:7.0`。让我们看看它的配置：

```
$ oc get is php -o yaml -n openshift
apiVersion: v1
kind: ImageStream
<OMITTED>
spec:
  lookupPolicy:
    local: false
  tags:
<OMITTED>
  - annotations:
<OMITTED>
      openshift.io/display-name: PHP 7.0
      openshift.io/provider-display-name: Red Hat, Inc.
      sampleRepo: https://github.com/openshift/cakephp-ex.git
      supports: php:7.0,php
      tags: builder,php
      version: "7.0"
    from:
 kind: DockerImage
 name: centos/php-70-centos7:latest
<OMITTED>
```

用于构建我们应用程序的 PHP 构建器镜像是 `centos/php-70-centos7:latest`。

我们现在已经掌握了从源代码构建应用程序的所有信息。OpenShift 将所有信息（包括你提供的和从源代码推断出的信息）组合起来，并启动新的构建。每个构建都有一个顺序编号，从 `1` 开始。你可以通过运行以下命令显示所有构建：

```
$ oc get build
NAME      TYPE   FROM        STATUS   STARTED     DURATION
phpinfo-1 Source Git@638030d Complete 2 hours ago 34s
```

该构建报告为 `Complete`，因为我们的应用程序已经启动并运行。

# 启动新构建

如果应用程序的源代码已更新，您可以通过运行`oc start-build`命令来触发重新构建过程。构建本身由构建配置管理。

首先，我们需要收集所有可用的构建配置的信息：

```
$ oc get bc
NAME    TYPE   FROM       LATEST
phpinfo Source Git@master 1
```

如您所见，我们只有一个构建，`phpinfo`，并且它只部署了一次；因此，版本号是`1`。

让我们开始一个新构建，如下所示：

```
$ oc start-build phpinfo
build "phpinfo-2" started

$ oc get pod
NAME             READY  STATUS    RESTARTS AGE
phpinfo-1-build  0/1    Completed 0        2h
phpinfo-1-h9xt5  1/1    Running   0        2h
phpinfo-2-build  0/1    Init:0/2  0        3s
```

OpenShift 启动了一个新构建，版本号为`2`，该版本在新构建生成的 Pod 名称中出现。完成后，您的应用程序将重新部署：

```
$ oc get pod
NAME             READY  STATUS     RESTARTS AGE
phpinfo-1-build  0/1    Completed  0        2h
phpinfo-2-build  0/1    Completed  0        32s
phpinfo-2-zqtj6  1/1    Running    0        23s
```

最新版本的构建现在是`2`：

```
$ oc get bc
NAME    TYPE   FROM       LATEST
phpinfo Source Git@master 2
```

OpenShift 会保存所有构建的列表，以便将来检查：

```
$ oc get build
NAME      TYPE   FROM        STATUS   STARTED            DURATION
phpinfo-1 Source Git@638030d Complete 2 hours ago        34s
phpinfo-2 Source Git@638030d Complete About a minute ago 7s
```

这个示例并不接近生产环境，因为构建是手动触发的，而不是由源代码的更改触发的。然而，它可以让您大致了解如何使用构建。

使用以下代码清理所有内容，为下一个实验做准备：

```
$ oc delete all --all deploymentconfig "phpinfo" deleted
buildconfig "phpinfo" deleted
imagestream "phpinfo" deleted
pod "phpinfo-2-fbd8l" deleted
service "phpinfo" deleted

$ oc delete project phpinfo
project "phpinfo" deleted

$ oc project myproject
Now using project "myproject" on server "https://127.0.0.1:8443".
```

# 概要

在本章中，您了解了 OpenShift 创建的构建实体，以及如何从源代码部署一个简单的 PHP 应用程序。我们向您展示了如何启动新构建以及如何自定义构建过程。

在接下来的章节中，您将从自定义模板构建并部署一个 WordPress 应用程序。您还将学习如何创建和部署 OpenShift 模板，以及如何从 OpenShift 模板部署应用程序。

# 问题

1.  以下哪个 OpenShift 实体控制如何从源代码构建特定应用程序？选择两个：

+   1.  Pod

    1.  路由

    1.  复制控制器

    1.  构建配置

    1.  构建

    1.  部署配置

1.  以下哪个命令启动一个新构建？选择一个：

+   1.  `oc new-app`

    1.  `oc new build`

    1.  `oc start-build`

    1.  `oc get build`

1.  以下哪个命令显示构建的信息？选择三个：

+   1.  `oc status -v`

    1.  `oc status`

    1.  `oc logs build/phpdemo-2`

    1.  `oc show logs`

# 进一步阅读

S2I 是 OpenShift 最重要的功能之一。您可能希望深入了解这一过程。以下链接提供了更多信息：

+   构建与镜像流：[`docs.openshift.org/latest/architecture/core_concepts/builds_and_image_streams.html#builds`](https://docs.openshift.org/latest/architecture/core_concepts/builds_and_image_streams.html#builds)。

+   PHP：[`docs.openshift.com/online/using_images/s2i_images/php.html/`](https://docs.openshift.com/online/using_images/s2i_images/php.html)。
