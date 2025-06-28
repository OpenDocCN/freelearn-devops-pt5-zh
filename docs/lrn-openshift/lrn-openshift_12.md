# 第十二章：在 OpenShift 中部署简单应用程序

应用程序部署是 OpenShift 最重要且最常用的功能，因为它就是为此而构建的。所有 OpenShift 用户都需要从 Docker 镜像进行应用程序部署。如果有一个知名应用且其镜像已经在 Docker Hub 或其他注册表中可用，OpenShift 用户可以以简单且可重复的方式部署它。在本章中，我们将处理从现有 Docker 镜像部署几个简单应用程序。

完成本章后，你将学到以下内容：

+   手动应用程序部署镜像，包括从 YAML 文件手动创建 Pod 和 Service 对象

+   如何利用 `oc new-app` 工具从现有 Docker 镜像部署应用程序

+   通过路由暴露应用程序

# 技术要求

本章没有严格的环境限制；支持任何 OpenShift 安装和开发环境：MinitShift，`oc cluster up` 或基于 Ansible 的标准生产就绪部署。使用哪种方式由你决定。然而，本章基于在 Vagrant 中运行的 `oc cluster up`。以下 `Vagrantfile` 可用于部署实验环境：

```
$ cat Vagrantfile
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
end
$lab_main = <<SCRIPT
cat <<EOF >> /etc/hosts
172.24.0.11 openshift.example.com openshift
172.24.0.12 storage.example.com storage nfs
EOF
systemctl disable firewalld
systemctl stop firewalld
yum update -y
yum install -y epel-release
yum install -y docker
cat << EOF >/etc/docker/daemon.json
{
   "insecure-registries": [
     "172.30.0.0/16"
   ]
}
EOF
systemctl restart docker
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

部署完之前列出的 Vagrant 虚拟机后，你可以按如下方式连接到它：

```
$ vagrant ssh
```

最后，作为 `developer` 用户登录，以便能够运行大多数命令：

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

我们可以使用任何密码

# 手动应用程序部署

在其他方法中，OpenShift 允许直接从现有 Docker 镜像部署应用程序。假设你的开发团队有一个内部流程，用于从他们的应用程序构建 Docker 镜像——通过这种方式，你可以在 OpenShift 环境中使用这些镜像部署应用程序，无需任何修改，这大大简化了迁移到 OpenShift 的过程。创建所有所需的 OpenShift 实体需要几个步骤。

首先，你需要创建一个 Pod，该 Pod 运行从应用程序的 Docker 镜像部署的容器。一旦 Pod 启动并运行，你可能需要创建一个服务，以便为其关联持久的 IP 地址和内部 DNS 记录。该服务允许你的应用程序通过 OpenShift 内部的一个一致的 **地址:端口** 对外访问。如果是仅供内部使用的应用程序，不需要外部访问（例如数据库或键值存储），这就足够了。

如果你的应用程序需要对外部可用，你需要通过创建 OpenShift 路由来 `expose` 它，使其可以通过外部网络（如互联网）访问。

简而言之，过程如下：

1.  创建一个 Pod

1.  通过暴露 Pod 创建服务

1.  通过暴露服务创建路由

在本章中，我们将使用一个简单的 httpd Docker 容器来演示应用部署过程。我们选择 httpd 是因为它足够简单，同时也能让我们专注于主要目标——演示与 OpenShift 相关的任务。

# 创建一个 Pod

`httpd` Docker 镜像可以在 Docker Hub 上找到。你可能想通过运行以下命令来确认这一点：

```
$ sudo docker search httpd
INDEX NAME DESCRIPTION STARS OFFICIAL AUTOMATED
docker.io docker.io/httpd The Apache HTTP Server Project 1719 [OK]
<OMITTED>
```

根据图片文档([`docs.docker.com/samples/library/httpd/`](https://docs.docker.com/samples/library/httpd/))，它监听 TCP 端口`80`。我们不能简单地使用这个容器，因为它绑定了一个特权端口。OpenShift 的默认安全策略不允许应用程序绑定 1024 以下的端口。为了避免问题，OpenShift 提供了一个名为`httpd`的镜像流，指向一个适用于 OpenShift 的`httpd`镜像构建。例如，在我们版本的 OpenShift 中，`httpd`镜像流指向`docker.io/centos/httpd-24-centos7` Docker 容器。你可能需要通过运行以下命令来验证这一点：

```
$ oc get is -n openshift | egrep "^NAME | ^httpd"
NAME DOCKER REPO TAGS UPDATED
httpd 172.30.1.1:5000/openshift/httpd latest,2.4 3 hours ago

$ oc describe is httpd -n openshift | grep "tagged from"
  tagged from centos/httpd-24-centos7:latest
```

每次我们想要使用`httpd`镜像部署 Pod 时，我们需要使用`centos/httpd-24-centos7`镜像。

让我们为实验创建一个单独的项目，如下所示：

```
$ oc new-project simpleappication
Now using project "simpleappication" on server "https://localhost:8443".
<OMITTED>
```

简单的`httpd` Pod 可以从其定义手动部署：

```
$ cat <<EOF > pod_httpd.yml
apiVersion: v1
kind: Pod
metadata:
  name: httpd
  labels:
    name: httpd
spec:
  containers:
    - name: httpd
      image: centos/httpd-24-centos7
      ports:
        - name: web
          containerPort: 8080
EOF
```

`centos/httpd-24-centos7`绑定到端口`8080`，这允许在不调整默认安全策略的情况下在 OpenShift 内部运行容器。

一旦文件创建完成，我们可以通过运行`oc create`来创建 Pod：

```
$ oc create -f pod_httpd.yml
pod "httpd" created
```

OpenShift 需要一些时间来下载 Docker 镜像并部署 Pod。一旦完成，你应该能够看到`httpd` Pod 处于`Running`状态：

```
$ oc get pod
NAME READY STATUS RESTARTS AGE
httpd 1/1 Running 0 2m
```

这个 Pod 提供与更复杂应用程序相同的功能（默认的`httpd`网页）。我们可能想要验证这一点，方法如下。

首先，获取 Pod 的内部 IP 地址：

```
$ oc describe pod httpd | grep IP:
IP: 172.17.0.2
```

然后使用 curl 查询上面输出的 IP 地址：

```
$ curl -s http://172.17.0.2:8080 | head -n 4
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.1//EN" "http://www.w3.org/TR/xhtml11/DTD/xhtml11.dtd">
<html  xml:lang="en">
 <head>
```

注意：这是默认 Apache 欢迎页面的开始。你可能需要在生产环境中替换它。可以通过将一个持久卷挂载到`/var/www/html`来实现这一点。为了演示目的，这个输出表明应用程序本身可以正常工作并且是可访问的。

# 创建一个服务

服务可以通过两种不同的方式创建：

+   使用`oc expose`

+   从 YAML/JSON 定义

我们将描述这两种方法。你不必同时使用两者。

# 使用 oc expose 创建服务

你可以通过以下方式使用`oc expose`创建 Pod：

```
$ oc get pod
NAME READY STATUS RESTARTS AGE
httpd 1/1 Running 0 13m

$ oc expose pod httpd --name httpd
service "httpd" exposed

$ oc get svc
NAME CLUSTER-IP EXTERNAL-IP PORT(S) AGE
httpd 172.30.128.131 <none> 8080/TCP 3s
```

上述命令通过暴露 Pod 创建了一个服务，使用`name=httpd`作为选择器。你可以通过`--name`选项定义一个自定义服务名称。

相同的`httpd`应用程序将从服务 IP 地址提供，假设在我们的例子中是`172.30.128.131`，但你从上一个命令得到的输出很可能会不同：

```
$ curl -s http://172.30.128.131:8080 | head -n4
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.1//EN" "http://www.w3.org/TR/xhtml11/DTD/xhtml11.dtd">

<html  xml:lang="en">
  <head>
```

让我们删除服务，并使用另一种方法重新创建它，如下小节所示：

```
$ oc delete svc/httpd
service "httpd" deleted
```

# 从 YAML 定义创建一个服务

以下 YAML 文件允许你定义一个 `Service` OpenShift 对象：

```
$ cat <<EOF > svc-httpd.yml
apiVersion: v1
kind: Service
metadata:
  labels:
    name: httpd
  name: httpd
spec:
  ports:
  - port: 8080
    protocol: TCP
    targetPort: 8080
  selector:
    name: httpd
EOF
```

一旦文件就位，你可以通过运行以下命令来创建服务：

```
$ oc create -f svc-httpd.yml
service "httpd" created

$ oc get svc
NAME CLUSTER-IP EXTERNAL-IP PORT(S) AGE
httpd 172.30.112.133 <none> 8080/TCP 2s
```

服务将显示与之前描述相同的输出：

```
$ curl -s http://172.30.112.133:8080 | head -n 4
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.1//EN" "http://www.w3.org/TR/xhtml11/DTD/xhtml11.dtd">

<html  xml:lang="en">
  <head>
```

# 创建路由

服务使应用程序能够通过一致的地址：端口对在内部访问。为了能够从集群外部访问它，我们需要确保创建了 OpenShift `Route`。一旦路由创建完成，OpenShift 将通过集群的路由器（默认通过 HAProxy Docker 容器实现）将服务暴露到外部。

和服务一样，路由可以通过两种方式创建：

+   使用 `oc expose`

+   从 YAML/JSON 定义

本节展示了两种方法，但只需使用其中一种即可。

假设你之前创建了一个名为 `httpd` 的服务。

# 使用 `oc expose` 创建路由

你可以通过以下方式使用 `oc expose` 创建路由：

```
$ oc expose svc httpd
route "httpd" exposed
$ oc get route
NAME HOST/PORT PATH SERVICES PORT TERMINATION WILDCARD
httpd httpd-simpleappication.127.0.0.1.nip.io httpd 8080 None
$ curl -s http://httpd-simpleappication.127.0.0.1.nip.io | head -n 4
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.1//EN" "http://www.w3.org/TR/xhtml11/DTD/xhtml11.dtd">
```

```
<html  xml:lang="en">
  <head> $ oc delete route httpd route "httpd" deleted
```

默认情况下，`oc cluster up` 方法使用 `127.0.0.1.nip.io` DNS 区域。

你可能希望通过 `--hostname` 选项创建一个具有备用 URL 的路由：

```
$ oc expose svc httpd --name httpd1 --hostname httpd.example.com
route "httpd1" exposed
$ oc get route
NAME HOST/PORT PATH SERVICES PORT TERMINATION WILDCARD
httpd httpd-simpleappication.127.0.0.1.nip.io httpd 8080 None
httpd1 httpd.example.com httpd 8080 None
 $ sudo bash -c  'echo "127.0.0.1 httpd.example.com" >>/etc/hosts'
 $ curl -s http://httpd.example.com | head -n 4
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.1//EN" "http://www.w3.org/TR/xhtml11/DTD/xhtml11.dtd">

<html  xml:lang="en">
  <head>
```

OpenShift 允许为单个应用程序创建多个路由。

如果你使用了备用名称，请确保 DNS 记录指向托管路由器 pod 的 OpenShift 节点的 IP 地址。

一旦路由被创建，你就可以通过这个外部路由访问你的应用程序。

# 从 YAML 定义创建路由

让我们为名为 `httpd2` 的应用程序创建一个备用路由。该路由将具有 `myhttpd.example.com` URL：

```
$ cat <<EOF > route-httpd2.yml
apiVersion: v1
kind: Route
metadata:
 labels:
 name: httpd2
 name: httpd2
spec:
 host: myhttpd.example.com
 port:
 targetPort: 8080
 to:
 kind: Service
 name: httpd
 weight: 100
EOF
```

可以通过 `oc create` 来创建路由：

```
$ oc create -f route-httpd2.yml route "httpd2" created $ oc get route
NAME HOST/PORT PATH SERVICES PORT TERMINATION WILDCARD
httpd httpd-simpleappication.127.0.0.1.nip.io httpd 8080 None
httpd1 httpd.example.com httpd 8080 None
httpd2 myhttpd.example.com httpd 8080 None
```

你可以看到新路由已成功添加。现在，如果存在相应的 DNS 记录，你将能够通过该备用路由访问你的应用程序。

# 使用 `oc new-app`

`oc` 工具允许你以用户友好的方式部署简单的应用程序。通常，你只需传递一个或多个选项给 `oc new-app` 命令，命令会创建应用程序所需的所有资源，包括 pod(s) 和 service(s)。此外，该命令还会创建 `ReplicationController` 和 `DeploymentConfig` API 对象，用于控制应用程序的部署方式。

# `oc new-app` 命令

因此，`oc new-app` 在应用程序部署过程中会从现有的 Docker 镜像创建以下资源：

| **资源** | **缩写** | **描述** |
| --- | --- | --- |
| Pod | pod | 表示你容器的 Pod |
| 服务 | svc | 包含内部应用程序端点的服务 |
| ReplicationController | rc | 复制控制器是 OpenShift 对象，用于控制应用程序的副本数量 |
| DeploymentConfig | dc | 部署配置是你部署的定义 |

`oc new-app` 是一个非常简单的工具，但它足够强大，可以满足大多数简单的部署需求。

`oc new-app` 在从 Docker 镜像部署应用程序时不会创建路由！

`oc new-app` 提供的功能也可以通过 Web 控制台访问，开发人员通常更倾向于使用该方式。

# 使用默认选项的 `oc new-app`

让我们删除之前创建的资源：

```
$ oc delete all --all
route "httpd" deleted
route "httpd1" deleted
route "httpd2" deleted
pod "httpd" deleted
service "httpd" deleted
```

删除所有内容的另一种方法是删除项目并重新创建它。

最近，我们展示了 OpenShift 自带一个镜像流，包含指向 OpenShift 准备好的 `httpd` 镜像的路径。默认情况下，`oc new-app` 工具使用镜像流引用的 Docker 镜像。

这是创建基础 `httpd` 应用程序的示例：

```
$ oc new-app httpd
--> Found image cc641a9 (5 days old) in image stream "openshift/httpd" under tag "2.4" for "httpd"

    Apache httpd 2.4
    ----------------
    Apache httpd 2.4 available as container, is a powerful, efficient, and extensible web server. Apache supports a variety of features, many implemented as compiled modules which extend the core functionality. These can range from server-side programming language support to authentication schemes. Virtual hosting allows one Apache installation to serve many different Web sites.

    Tags: builder, httpd, httpd24

    * This image will be deployed in deployment config "httpd"
    * Ports 8080/tcp, 8443/tcp will be load balanced by service "httpd"
      * Other containers can access this service through the hostname "httpd"

--> Creating resources ...
    deploymentconfig "httpd" created
    service "httpd" created
--> Success
    Application is not exposed. You can expose services to the outside world by executing one or more of the commands below:
     'oc expose svc/httpd'
    Run 'oc status' to view your app.
```

部署过程需要一些时间。一旦一切准备就绪，你可以检查是否已创建所有资源：

```
$ oc get all
NAME REVISION DESIRED CURRENT TRIGGERED BY
deploymentconfigs/httpd 1 1 1 config,image(httpd:2.4)

NAME READY STATUS RESTARTS AGE
po/httpd-1-n7st4 1/1 Running 0 31s

NAME DESIRED CURRENT READY AGE
rc/httpd-1 1 1 1 32s

NAME CLUSTER-IP EXTERNAL-IP PORT(S) AGE
svc/httpd 172.30.222.179 <none> 8080/TCP,8443/TCP 33s
```

让我们确认已使用正确的镜像：

```
$ oc describe pod httpd-1-n7st4 | grep Image:
    Image: centos/httpd-24-centos7@sha256:6da9085c5e7963efaae3929895b9730d7e76e937a7a0109a23015622f3e7156b
```

剩下的工作是暴露服务，使应用可以在外部访问：

```
$ oc expose svc httpd
route "httpd" exposed

$ curl -s http://httpd-simpleappication.127.0.0.1.nip.io | head -n 4
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.1//EN" "http://www.w3.org/TR/xhtml11/DTD/xhtml11.dtd">

<html  xml:lang="en">
  <head>
```

# 高级部署

`oc new-app` 命令接受多个参数，允许你根据需求修改部署过程。例如，你可能需要修改名称、指定环境变量等。

高级选项始终可以通过使用内置的帮助功能显示，帮助功能可以通过 `oc new-app --help` 显示：

```
$ oc new-app --help
Create a new application by specifying source code, templates, and/or images

...
<OMITTED>
...

If you provide source code, a new build will be automatically triggered. You can use 'oc status' to
check the progress.

Usage:
  oc new-app (IMAGE | IMAGESTREAM | TEMPLATE | PATH | URL ...) [options]

Examples:
  # List all local templates and image streams that can be used to create an app
  oc new-app --list

  # Create an application based on the source code in the current git repository (with a public
remote)
  # and a Docker image
  oc new-app . --docker-image=repo/langimage
...
<OMITTED>
...
```

在接下来的章节中，我们将大量使用 `oc new-app`。你现在不需要了解所有选项。

# 部署 MariaDB

本节中，我们将部署一个带有额外配置选项的数据库容器。容器需要将一些参数传递给 `oc new-app`。让我们按照下面的示例创建一个简单的 `mariadb` 容器。

首先删除之前创建的对象：

```
$ oc delete all --all ...
<OUTPUT OMITTED>
...
```

现在我们想创建一个数据库容器，允许名为 `openshift` 的数据库用户连接到名为 `openshift` 的数据库。为了简化，我们将使用 `openshift` 作为数据库密码。以下示例展示了如何启动一个 MariaDB 容器：

```
$ oc new-app -e MYSQL_USER=openshift -e MYSQL_PASSWORD=openshift \
-e MYSQL_DATABASE=openshift mariadb 
--> Found image 1b0e3a6 (5 days old) in image stream "openshift/mariadb" under tag "10.2" for "mariadb"

...
<OUTPUT OMITTED>
...

    Run 'oc status' to view your app.
```

验证 mariadb 是否启动并运行：

```
$ oc get all
NAME REVISION DESIRED CURRENT TRIGGERED BY
deploymentconfigs/mariadb 1 1 1 config,image(mariadb:10.1)

NAME READY STATUS RESTARTS AGE
po/mariadb-1-54h6x 1/1 Running 0 2m

NAME DESIRED CURRENT READY AGE
rc/mariadb-1 1 1 1 2m

NAME CLUSTER-IP EXTERNAL-IP PORT(S) AGE
svc/mariadb 172.30.233.119 <none> 3306/TCP 2m
```

现在你可以使用容器名称 "po/mariadb-1-54h6x" 来访问数据库，针对我们的案例是这样。

现在使用 'oc exec' 登录到容器：

```
$ oc exec -it mariadb-1-54h6x /bin/bash
bash-4.2$
```

连接到 mariadb 数据库，并验证名为 'openshift' 的数据库已创建，并且你可以通过运行 'show databases' 命令访问它。

```
$ mysql -uopenshift -popenshift -h127.0.0.1 openshift
...
<OUTPUT OMITTED>
...

MariaDB [openshift]> show databases;
+--------------------+
| Database |
+--------------------+
| information_schema |
| openshift |
| test |
+--------------------+
3 rows in set (0.00 sec)
```

之前的输出显示数据库服务已启动并运行，准备从应用端使用。我们将在接下来的章节中进一步探讨这个话题。

你可以退出数据库，为下一章节做好准备。

```
MariaDB [openshift]> exit
Bye
bash-4.2$ exit
exit
[vagrant@openshift ~]
```

清理你的实验环境。

```
$ oc delete all --all
deploymentconfig "mariadb" deleted
imagestream "mariadb" deleted
pod "mariadb-1-9qcsp" deleted
service "mariadb" deleted

$ oc delete project simpleappication
project "simpleappication" deleted

$ oc project myproject
Now using project "myproject" on server "https://localhost:8443".
```

如果你打算继续进行下一章节，可以保持 OpenShift 集群运行，否则可以关闭或删除 vagrant 虚拟机。

# 总结

在本章中，我们向您展示了如何从 Docker 镜像部署多个简单应用程序，如何手动创建 pod，以及如何手动创建服务。我们详细讲解了创建服务的过程，如何手动创建路由，以及如何使用 `oc expose` 创建路由。最后，我们向您展示了如何使用 `oc new-app` 命令从 Docker 镜像部署应用程序。

在下一章，我们将使用 OpenShift 模板部署多层应用程序。

# 问题

1.  以下哪个 OpenShift 实体不是 `oc new-app` 自动创建的？

    1.  Pod

    1.  路由

    1.  副本控制器

    1.  部署配置

    1.  服务

1.  以下哪些实体应创建，以便在最小配置下将应用程序公开到外部？（选择三个）

    1.  Pod

    1.  服务

    1.  路由

    1.  副本控制器

    1.  部署配置

    1.  镜像流

1.  哪个命令创建了自定义 URL `myservice.example.com` 的路由？

    1.  oc expose svc httpd

    1.  oc expose svc httpd --host myservice.example.com

    1.  oc expose svc httpd --hostname myservice.example.com

    1.  oc create svc httpd --hostname myservice.example.com

1.  哪些命令显示所有 OpenShift 路由？（选择两个）

    1.  oc get all

    1.  oc get pod

    1.  oc get application

    1.  oc get route

    1.  docker ps

    1.  ip route

# 进一步阅读

我们已经向您介绍了 OpenShift 中的应用程序部署基础知识。如果您想进一步了解，以下链接会对您有所帮助：

+   **Pods 和 Services**：[`docs.openshift.org/latest/architecture/core_concepts/pods_and_services.html`](https://docs.openshift.org/latest/architecture/core_concepts/pods_and_services.html)

+   **从镜像创建应用程序**：[`docs.openshift.org/latest/dev_guide/application_lifecycle/new_app.html#specifying-an-image`](https://docs.openshift.org/latest/dev_guide/application_lifecycle/new_app.html#specifying-an-image)
