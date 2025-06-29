# 第五章：Docker 插件

在 2014 年 DockerCon Europe 大会期间，举行了一个圆桌讨论，讨论了 Docker 生态系统的现状，识别出了以下问题及可能的解决方案：

> Docker 目前面临的问题是，随着它转变为一个平台，它被认为威胁到了自己的生态系统。提议的解决方案是，Docker 将自己的附加功能作为后期绑定的、可组合的、可选的扩展发布，并允许其他供应商也这样做。Docker 称之为“内置电池但可拆卸”。

在 2015 年 DockerCon Seattle 大会期间，Docker 宣布在实验分支中提供插件功能，公告以博客文章的形式发布，可以在[`blog.docker.com/2015/06/extending-docker-with-plugins/`](https://blog.docker.com/2015/06/extending-docker-with-plugins/)找到。

正如你从这篇文章中看到的，Docker 提供了一个解决方案，第三方可以替换核心功能。现在，用户可以运行`docker volume`和`docker network`命令，并通过驱动程序选项让 Docker 调用外部组件，从而添加核心 Docker 引擎之外的功能，同时保持高度的兼容性。

在本章中，我们将介绍两个不同的 Docker 插件，第一个是名为**REX-Ray**的卷插件，第二个是名为**Weave**的网络插件。

# REX-Ray 卷插件

到目前为止，我们一直在使用本地主机上可用的本地存储，正如第四章中提到的，*Docker Swarm* 在你可能需要在多个主机之间移动存储时并不太有用，无论是因为你托管了一个集群，还是因为主机本身存在问题。

在这个例子中，我们将启动一个在 Amazon Web Services 上的 Docker 实例，安装一个名为 REX-Ray 的卷插件，它由 EMC 编写，然后启动我们的 WordPress 示例，但这次我们将把 Amazon Elastic Block Storage 卷附加到我们的容器上。一旦配置完成，我们将把容器移动到第二台主机上，以展示数据的持久性。

REX-Ray 支持公共云和 EMC 自身系列的几种存储类型，具体如下：

+   Amazon Elastic Block Store - [`aws.amazon.com/ebs/`](https://aws.amazon.com/ebs/)

+   Digital Ocean 块存储 - [`www.digitalocean.com/products/storage`](https://www.digitalocean.com/products/storage)

+   OpenStack Cinder - [`wiki.openstack.org/wiki/Cinder`](https://wiki.openstack.org/wiki/Cinder)

+   Google 计算引擎 - [`cloud.google.com/compute/docs/disks/`](https://cloud.google.com/compute/docs/disks/)

+   EMC Isilon，ScaleIO，VMAX 和 XtremIO - [`www.emc.com/`](https://www.emc.com/)

该驱动程序正在积极开发中，并且支持的存储类型也在不断增加，同时也在进行工作，将该驱动程序迁移到 Docker 的新插件系统中。

在安装 REX-Ray 之前，我们需要在 Amazon Web Services 中有一个 Docker 主机，要启动一个，请使用以下命令。你可以参考 第二章 中的 Amazon Web Services 驱动程序部分，*使用 Docker 启动应用程序*，了解如何生成你的访问密钥和秘密密钥，并找到你的 VPC ID。记得将 `access-key`、`secret-key` 和 `vpc-id` 替换为你自己的：

```
docker-machine create \
 --driver amazonec2 \
 --amazonec2-access-key AKIAJ3GYNKVTEWNMFDHQ \
 --amazonec2-secret-key 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 \
 --amazonec2-vpc-id vpc-35c91750 \
 awstest

```

现在你已经启动了实例，可以在 AWS 控制台中看到它：

![REX-Ray 卷插件](img/B06455_05_01.jpg)

我们需要安装 REX-Ray 插件。由于 REX-Ray 支持 Docker 的新插件格式，这意味着我们需要运行 `docker plugin` 命令。首先，我们需要配置本地 Docker 客户端，通过运行以下命令来连接到 AWS 主机：

```
eval $(docker-machine env awstest)

```

现在我们已经连接上来安装插件，只需要运行以下命令，`EBS_ACCESSKEY` 和 `EBS_SECRETKEY` 变量与我们在 Docker Machine 中使用的相同，替换为你自己的密钥：

```
docker plugin install rexray/ebs \
EBS_ACCESSKEY=AKIAJ3GYNKVTEWNMFDHQ \
EBS_SECRETKEY=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

```

在安装插件之前，系统会要求你确认是否允许插件访问 Docker 安装的各个部分，提示时回答“是”（y），插件将被下载并安装。

插件安装完成后，我们需要创建两个卷，一个用于存放 WordPress 数据，另一个用于存放 MySQL 数据库。要创建卷，运行以下命令：

```
docker volume create --driver rexray/ebs --name dbdata
docker volume create --driver rexray/ebs --name wpdata

```

你可以在以下终端中看到前面提到的命令正在运行：

![REX-Ray 卷插件](img/B06455_05_02.jpg)

你也应该能够通过点击 AWS 控制台中 EC2 部分左侧菜单中的 **Volumes** 来查看你的两个卷：

![REX-Ray 卷插件](img/B06455_05_03.jpg)

现在我们有了两个卷，我们需要启动 WordPress，为此我们将使用可以在仓库中的 `/bootcamp/chapter05/wordpress-rexray/` 找到的 Docker Compose 文件。

从 `docker-compose.yml` 文件中可以看出，我们正在构建一个安装了 `wp-cli` 的 WordPress 镜像：

```
version: "3"

services:
mysql:
     image: mysql
     volumes:
       - dbdata:/var/lib/mysql
     restart: always
     environment:
       MYSQL_ROOT_PASSWORD: wordpress
       MYSQL_DATABASE: wordpress
wordpress:
depends_on:
       - mysql
     build: ./
     volumes:
       - wpdata:/var/www/html
     ports:
       - "80:80"
     restart: always
     environment:
       WORDPRESS_DB_PASSWORD: wordpress

volumes:
dbdata:
      external:
        name: dbdata
wpdata:
      external:
        name: wpdata
```

从文件的末尾你也可以看到，我们告诉 Docker Compose 使用我们已经通过 `docker volume create` 命令创建的两个外部卷。

要构建我们的 WordPress 镜像并启动容器，运行以下命令：

```
docker-compose up -d

```

你可以通过运行以下命令检查你的容器是否正在运行：

```
docker-compose ps

```

现在，我们的两个 WordPress 容器已经启动，你可以通过运行以下命令快速安装 WordPress（根据需要更新变量）：

```
$awshost = "$(docker-machine ip awstest)"
docker-compose exec wordpress wp core install --url=http://$(awshost)/ --title=Testing --admin_user=admin --admin_password=adminpassword --admin_email=russ@mckendrick.io

```

安装完成后，你应该看到一条消息，表示 **成功：WordPress 安装成功**。这意味着你可以通过运行以下命令在浏览器中打开安装页面：

```
open http://$(docker-machine ip awstest)

```

这应该会显示你现在熟悉的 WordPress 网站：

![REX-Ray 卷插件](img/B06455_05_05.jpg)

现在让我们对 WordPress 安装进行一些更改，以确保当我们在主机之间迁移应用程序时，一切正常工作。我们将用烟花替换植物的图片。为了做到这一点，我们需要自定义我们的主题，要进入主题编辑页面，请运行以下命令：

```
open "http://$(docker-machine ip awstest)/wp-admin/customize.php?return=%2Fwp-admin%2Fthemes.php"

```

系统会提示你使用管理员用户名和密码登录，如果你按照安装步骤操作，用户名和密码应该是`admin` / `adminpassword`，如果你输入了自己的用户名和密码，请使用它们。

打开页面后，点击左侧菜单中的“Header Media”（头部媒体）。向下滚动，找到左侧菜单中显示的**Add new image**（添加新图片），按照屏幕上的提示上传、裁剪并设置新的头图，你可以在代码库中找到一张名为`fireworks.jpg`的图片，或者使用你自己的图片。完成后，点击**Save & Publish**（保存并发布）。

返回到你的网站主页，应该可以看到你新的头图：

![REX-Ray volume plugin](img/B06455_05_07.jpg)

在移除 Docker 主机之前，我们需要记下它的 IP 地址，方法是运行以下命令：

```
echo $(docker-machine ip awstest)

```

记下 IP 地址，因为我们稍后会用到，在我的例子中，IP 地址是`54.173.130.142`。

现在让我们通过以下命令移除我们的主机：

```
docker-machine rm awstest

```

主机移除后，我们的两个卷在 AWS 控制台中会显示为**可用**：

![REX-Ray volume plugin](img/B06455_05_09.jpg)

这是我们的 WordPress 和数据库数据，要在新的 Docker 主机上访问它，我们需要首先启动一个主机。为此，请再次运行以下命令，记得将凭证和 VPC ID 替换为你自己的：

```
docker-machine create \
 --driver amazonec2 \
 --amazonec2-access-key AKIAJ3GYNKVTEWNMFDHQ \
 --amazonec2-secret-key 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 \
 --amazonec2-vpc-id vpc-35c91750 \
 awstest2

```

一旦新的 Docker 主机启动并运行，运行以下命令切换我们的本地客户端并安装 REX-Ray：

```
eval $(docker-machine env awstest2)
docker plugin install rexray/ebs \
 EBS_ACCESSKEY=AKIAJ3GYNKVTEWNMFDHQ \
 EBS_SECRETKEY=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

```

一旦 REX-Ray 安装完成，我们需要让它识别我们现有的两个卷，方法是简单地运行以下命令：

```
docker volume create --driver rexray/ebs --name dbdata
docker volume create --driver rexray/ebs --name wpdata

```

不用担心，它不会覆盖我们现有的卷，它只会让 Docker 知道它们的存在，因为 REX-Ray 使用的是你分配给卷的名称，而不是唯一 ID。如果它遇到一个已有名称的卷，它会认为这就是你要使用的卷，所以在命名卷时要小心，因为它们会被附加到运行中的容器上。

你可能会注意到这次命令执行得更快了，这是因为卷已经存在，不需要重新创建。运行：

```
docker volume ls

```

应该显示我们的两个卷仍然存在，和之前一样。

现在我们需要启动 WordPress，方法是运行以下命令：

```
docker-compose up -d

```

如果你现在尝试访问你的 WordPress 网站，你会看到一个非常破损的网站，虽然有内容，但没有样式或图片。

这是因为数据库仍然引用我们已终止的 Docker 主机的 IP 地址，指向数据库。运行以下命令时，请确保将命令中的 IP 地址替换为你之前 Docker 主机的 IP 地址（记住我的 IP 地址是`54.173.130.142`）：

```
docker-compose exec wordpress wp search-replace 54.173.130.142 $(docker-machine ip awstest2)

```

你应该能看到数据库中每个表的列表，并确认它已替换为新 Docker 主机的 IP 地址的实例数量。

通过运行以下命令访问你的新 WordPress 安装：

```
open http://$(docker-machine ip awstest2)

```

应该显示你的封面图像完好无损，并且 WordPress 安装和你离开时一样，除了 IP 地址的变化。

测试完成后，你可以通过运行以下命令来移除你的安装：

```
docker-compose stop
docker-compose rm
docker volume rmdbdata
docker volume rmwpdata
docker-machine rm awstest2

```

### 注释

你可能会注意到，当你运行`docker volume rm`命令时，并没有提示你确认操作，所以要小心。

检查你的 AWS 控制台，应该能确认你的 Docker 主机已经终止，并且你的两个卷已经被删除。

# Weave 网络插件

Weave 是最初的 Docker 插件之一，实际上，他们曾参与关于 Docker 插件功能的圆桌讨论，而 Weave 也包含在本章开头提到的原始插件公告中。

Weave 将他们的网络插件描述为：

> 快速、简单、安全地在任何环境中（本地、云端或混合环境）对容器进行网络连接和集群管理，零代码或配置。

任何曾经与软件定义网络（SDN）工作过的人都知道，这是一个相当大胆的声明，特别是 Weave 正在创建一个网状网络。要全面了解这意味着什么，我推荐阅读 Weave 自己的概述，可以在[`www.weave.works/docs/net/latest/how-it-works/`](https://www.weave.works/docs/net/latest/how-it-works/)找到。

不再详细讨论，让我们动手进行安装。首先，使用 Docker Machine 在 DigitalOcean 上启动两个独立的 Docker 主机。

为了增加趣味性，我们将在纽约市启动一个主机，在伦敦启动另一个主机。由于这两个主机将作为独立主机使用，因此不需要配置 Docker Swarm——这通常是多主机网络配置 Docker 时所需要的。

要启动纽约市的 Docker 主机，运行：

```
docker-machine create \
 --driver digitalocean \
 --digitalocean-access-token 57e4aeaff8d7d1a8a8e46132969c2149117081536d50741191c79d8bc083ae73 \
 --digitalocean-region nyc1\

 weave-nyc

```

然后在伦敦的 Docker 主机上运行：

```
docker-machine create \
 --driver digitalocean \
 --digitalocean-access-token 57e4aeaff8d7d1a8a8e46132969c2149117081536d50741191c79d8bc083ae73 \
 --digitalocean-region lon1 \
 weave-lon

```

现在我们已经有了两个 DigitalOcean 主机，我们需要让 Weave 运行起来。在撰写本文时，Weave 还没有完成过渡到 Docker 原生插件架构，预计很快就会完成，因此我们将使用控制脚本来配置 Weave。

首先，我们需要在纽约市的 Docker 主机上下载控制脚本：

```
docker-machine ssh weave-nyc 'curl -L git.io/weave -o /usr/local/bin/weave; chmoda+x /usr/local/bin/weave'

```

下载完成后，我们可以使用以下命令启动 Weave：

```
docker-machine ssh weave-nyc weave launch --password 3UnFh4jhahFC

```

这将下载并启动多个 Docker 主机上的容器，下载完成后，Weave 将被配置，密码也会设置好，这意味着如果你想将主机添加到网络中，你需要提供一个有效的密码。

如果你没有定义密码，那么任何人都可以连接到你的 Weave 网络。如果你知道你的主机是在一个隔离的封闭网络中运行，那么这样做没问题。然而，由于我们是在公网上发送流量，因此我们设置了一个密码。

你可以通过运行以下命令来检查容器：

```
docker $(docker-machine config weave-nyc) container ps

```

现在我们已经启动了所需的三个容器，是时候在伦敦 Docker 主机上安装 Weave，并将其连接到我们的 NYC Docker 主机了。执行以下命令来完成安装：

```
docker-machine ssh weave-lon 'curl -L git.io/weave -o /usr/local/bin/weave; chmoda+x /usr/local/bin/weave'
docker-machine ssh weave-lon weave launch --password 3UnFh4jhahFC

```

三个容器启动后，只需运行以下命令连接到我们的 NYC Docker 主机：

```
docker-machine ssh weave-lon weave connect "$(docker-machine ip weave-nyc)"

```

一旦我们的第二个主机配置完成，你可以通过运行以下命令来检查 Weave 网状网络的状态：

```
docker-machine ssh weave-nyc weave status

```

![WeaveNetwork Plugin](img/B06455_05_15.jpg)

正如你从上面的终端输出中看到的，我们有五个服务正在运行，除了提供密码外，我们没有配置任何服务。

由于我正在运行 Mac OS 系统，我还将本地安装 Weave，相同的指令也适用于 Linux 系统。

以下命令将安装 Weave 控制脚本，该脚本用于在你的 Docker for Mac 安装中启动容器，并连接到我们的 Weave 网状网络：

```
sudo curl -L git.io/weave -o /usr/local/bin/weave; sudochmoda+x /usr/loca
l/bin/weave
weave launch --password 3 UnFh4jhahFC
weave connect "$(docker-machine ip weave-nyc)"

```

一旦安装并连接，运行`weave status`本地命令应该显示现在有 3 个对等节点和 6 个已建立的连接：

![WeaveNetwork Plugin](img/B06455_05_16.jpg)

现在我们有了三个 Docker 主机：

+   一个位于纽约，由 Digital Ocean 托管

+   一个位于伦敦，由 Digital Ocean 托管

+   我们的本地 Docker 主机运行在 Docker for Mac（或 Linux）上

所有这些操作都使用名为**weave**的网络，且使用`weavemesh`驱动。你可以通过运行以下命令来确认这一点：

```
docker network ls
docker $(docker-machine config weave-nyc) network ls
docker $(docker-machine config weave-lon) network ls

```

你应该看到类似以下终端输出的内容：

![WeaveNetwork Plugin](img/B06455_05_17.jpg)

现在我们已经准备好开始将容器启动到我们的 Weave 网络中，并展示它们如何彼此通信。

### 注意

Netcat 是一个允许你通过 TCP 或 UDP 在网络上进行读写的服务。

让我们从在纽约启动一个运行 Netcat(nc)的容器开始。每次请求发送到 4242 端口时，`nc`都会回应`Hello from NYC!!!`：

```
docker $(docker-machine config weave-nyc) container run -itd \
 --name=nyc \
 --net=weave \
 --hostname="nyc.weave.local" \
 --dns="172.17.0.1" \
 --dns-search="weave.local" \
 alpine nc -p 4242-ll -e echo 'Hello from NYC!!!'

```

正如你从 Docker 命令中看到的，我们传递了相当多的不同选项，我们告诉容器使用哪个网络，并且配置了容器内的 DNS 解析器，并设置了主机名为`nyc.weav` `e.local`。

![WeaveNetwork Plugin](img/B06455_05_18.jpg)

现在我们已经启动了 NYC 容器，首先要做的是检查我们是否能从伦敦 Docker 主机 ping 通，执行以下命令来完成此操作：

```
docker $(docker-machine config weave-lon) container run -it --rm \
 --name=ping \
 --net=weave \
 --dns="172.17.0.1" \
 --dns-search="weave.local" \
 alpine sh -c 'ping -c3 nyc.weave.local'

```

这将向`nyc.weave.local`发送三个 ping 请求，所有请求应当得到回应：

![WeaveNetwork Plugin](img/B06455_05_19.jpg)

现在我们已确认可以 ping 通 NYC 容器，我们需要连接到`4242`端口，并检查是否得到预期的回应：

```
docker $(docker-machine config weave-lon) container run -it --rm \
 --name=conect \
 --net=weave \
 --dns="172.17.0.1" \
 --dns-search="weave.local" \
 alpine sh -c 'echo "Where are you?" | ncnyc.weave.local 4242'

```

你应该收到信息**Hello from** **NYC!!!**：

![WeaveNetwork Plugin](img/B06455_05_20.jpg)

现在让我们使用以下命令在本地 Docker 主机上启动一个容器：

```
docker container run -itd \
 --name=mac \
 --net=weave \
 --hostname="mac.weave.local" \
 --dns="172.17.0.1" \
 --dns-search="weave.local" \
 alpine nc -p 4242 -ll -e echo 'Hello from Docker for Mac!!!'

```

![WeaveNetwork Plugin](img/B06455_05_21.jpg)

如之前所示，我们将对本地容器进行简单的 ping 测试：

```
docker $(docker-machine config weave-nyc) container run -it --rm \
 --name=ping \
 --net=weave \
 --dns="172.17.0.1" \
 --dns-search="weave.local" \
 alpine sh -c 'ping -c3 mac.weave.local'

```

正如预期的那样，我们收到了回应：

![WeaveNetwork Plugin](img/B06455_05_22.jpg)

刚开始时有点慢，但最终会变得更好。现在我们知道可以 ping 我们的本地容器，接下来连接到端口 `4242` 并检查响应。首先，从我们的纽约 Docker 主机：

```
docker $(docker-machine config weave-nyc) container run -it --rm \
 --name=conect \
 --net=weave \
 --dns="172.17.0.1" \
 --dns-search="weave.local" \
 alpine sh -c 'echo "Where are you?" | ncmac.weave.local 4242'

```

然后，从我们的伦敦 Docker 主机：

```
docker $(docker-machine config weave-lon) container run -it --rm \
 --name=conect \
 --net=weave \
 --dns="172.17.0.1" \
 --dns-search="weave.local" \
 alpine sh -c 'echo "Where are you?" | ncmac.weave.local 4242'

```

如以下终端输出所示，我们得到了预期的结果：

![WeaveNetwork 插件](img/B06455_05_23.jpg)

要整理你的本地 Docker 主机，请运行以下命令：

```
docker container stop mac
docker container rm mac
weave stop
sudorm -f /usr/local/bin/weave

```

然后，要终止我们的两个 Digital Ocean 主机，请运行：

```
docker-machine stop weave-lon weave-nyc
docker-machine rm weave-lon weave-nyc

```

虽然这些测试在视觉上没有像 REX-Ray Volume 插件的演示那么引人注目，但正如你所看到的，Weave 是一个非常强大的软件定义网络，非常容易配置。

根据经验，这种组合是很难实现的，因为大多数 SDN 解决方案在安装、配置和维护上都非常复杂。

我们仅仅触及了 Weave 的一些可能性。有关完整的功能列表以及一些高级用例的说明，请参考[`docs.weave.works/weave/latest_release/features.html`](http://docs.weave.works/weave/latest_release/features.html)。

# 总结

希望你现在开始看到不同类型插件的使用场景。例如，一个开发者可以在本地卷上工作，但对于生产流量，你可能希望有某种共享存储或块存储，使得多个 Docker 主机上的容器都能访问。

使用插件，在不对用户工作流进行任何实质性更改的情况下，这一点是可能的，因为你清楚 Docker 如何处理通过 `docker volume create` 命令创建的卷。

如前所述，Docker 正在将遗留插件过渡到新的架构，遗留插件的列表可以在以下网址找到：[`docs.docker.com/engine/extend/legacy_plugins/`](https://docs.docker.com/engine/extend/legacy_plugins/)，而使用新架构的插件可以在[`store.docker.com/search?q=&type=plugin`](https://store.docker.com/search?q=&type=plugin)找到。

在下一章中，我们将讨论如何监控你的容器，以及如果出现问题该怎么办。
