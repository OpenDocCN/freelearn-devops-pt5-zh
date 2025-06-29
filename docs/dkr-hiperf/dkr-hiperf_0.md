# 序言

Docker 是构建和部署应用程序的伟大工具。它的可移植容器格式使我们能够在任何地方运行代码，从开发者的工作站到流行的云计算提供商。围绕 Docker 的工作流程使开发、测试和部署变得更容易、更快速。然而，Docker 的内部机制以及持续改进的最佳实践对充分发挥其潜力至关重要。

# 本书涵盖的内容

具有基础 Docker 知识的工程师可以按顺序阅读本书，从章节到章节。对于已经深入了解 Docker 或曾在生产环境中部署过应用的技术负责人，可以先阅读第八章，*上生产环境*，以了解 Docker 如何与现有应用程序兼容。本书涵盖的主题如下：

第一章，*准备 Docker 主机*，快速回顾了如何设置和运行 Docker。本章记录了你将在本书中使用的设置。

第二章，*优化 Docker 镜像*，展示了调优 Docker 镜像的重要性。将展示一些调优技巧，以提高 Docker 容器的可部署性和性能。

第三章，*使用 Chef 自动化 Docker 部署*，介绍了如何自动化 Docker 主机的供应和设置。本章将讨论自动化投资的重要性以及它如何促进 Docker 容器的可扩展部署。

第四章，*监控 Docker 主机和容器*，介绍了如何使用 Graphite 和 Elasticsearch-Logstash-Kibana (ELK)堆栈搭建监控系统和日志系统。

第五章，*性能基准测试*，是关于如何使用 Apache JMeter 创建工作负载来基准测试 Docker 容器性能的教程。本章回顾了你在第四章，*监控 Docker 主机和容器*，中设置的监控系统，分析一些 Docker 应用的基准测试结果，如响应时间和吞吐量。

第六章，*负载均衡*，将教你如何配置和部署基于 Nginx 的负载均衡器 Docker 容器。本章还提供了如何使用你所设置的负载均衡器来扩展 Docker 应用程序的性能和可部署性的教程。

第七章，*容器故障排除*，展示了如何使用典型 Linux 系统中的常见调试工具来排查 Docker 容器的问题。它们描述了每个工具的工作原理以及如何读取运行中 Docker 容器的诊断信息。

第八章，*进入生产环境*，总结了你在上一章中所做的所有性能优化，并讲解了在生产环境中使用 Docker 运营任何 Web 应用程序的含义。

# 本书所需的内容

需要一台运行最新内核的 Linux 工作站作为 Docker 1.10.0 的主机。本书使用 Debian Jessie 8.2 作为其基础操作系统来安装和设置 Docker。

如何让 Docker 启动并运行的更多细节，可以在 第一章，*准备 Docker 主机* 中找到。

# 本书适合谁阅读

本书是为那些希望将其 Docker 应用程序和基础设施部署到生产环境中的开发人员和运维人员编写的。如果你已经学习了 Docker 的基础知识，但希望更进一步，那么本书适合你。

# 约定

在本书中，你会看到多种文本样式，用来区分不同类型的信息。以下是这些样式的一些示例及其含义的解释。

文章中的代码词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 账号等都会按如下方式显示：“我们将使用 `--link` `<source>:<alias>` 来创建一个从源容器 `source` 到别名 `webapp` 的链接。”

一块代码的显示方式如下：

```
FROM ubuntu:14.04
MAINTAINER Docker Education Team <education@docker.com>
RUN apt-get update
RUN DEBIAN_FRONTEND=noninteractive apt-get \
        install -y -q python-all python-pip 
ADD ./webapp/requirements.txt /tmp/requirements.txt
RUN pip install -qr /tmp/requirements.txt
ADD ./webapp /opt/webapp/
WORKDIR /opt/webapp
EXPOSE 5000
CMD ["python", "app.py"]
```

当我们希望引起你对某个代码块中特定部分的注意时，相关的行或项目会以粗体显示：

```
import os
from flask import Flask
app = Flask(__name__)
@app.route('/')
def hello():
    provider = str(os.environ.get('PROVIDER', 'world'))
    return 'Hello '+provider+'!'
if __name__ == '__main__':
    # Bind to PORT if defined, otherwise default to 5000.
 port = int(os.environ.get('PORT', 5000))
    app.run(host='0.0.0.0', port=port)
```

任何命令行输入或输出会按如下方式显示：

```
dockerhost$ docker inspect -f "{{ .NetworkSettings.IPAddress }}" \
 source
172.17.0.15
dockerhost$ docker inspect -f "{{ .NetworkSettings.IPAddress }}" \
 destination
172.17.0.28
dockerhost$ iptables -L DOCKER
Chain DOCKER (1 references)
target     prot opt source         destination 
ACCEPT     tcp  --  172.17.0.28    172.17.0.15       tcp dpt:5000
ACCEPT     tcp  --  172.17.0.15    172.17.0.28       tcp spt:5000

```

**新术语**和**重要单词**会以粗体显示。你在屏幕上看到的词语，例如菜单或对话框中的内容，会像这样出现在文本中：“最后，点击 **下载入门套件**。”

### 注意

警告或重要提示会以类似这样的框框显示。

### 提示

小贴士和技巧会像这样显示。

# 读者反馈

我们欢迎读者的反馈。请告诉我们你对本书的看法——你喜欢或不喜欢的内容。读者的反馈对我们非常重要，因为它帮助我们开发出你真正能从中受益的书籍。

要向我们发送一般反馈，只需通过电子邮件发送至`<feedback@packtpub.com>`，并在邮件的主题中提及书籍的标题。

如果您在某个领域拥有专业知识并且有兴趣撰写或为书籍做贡献，请查看我们的作者指南，网址为[www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

既然您已经成为了 Packt 书籍的骄傲拥有者，我们为您提供了许多资源，帮助您最大化地利用您的购买。

## 下载示例代码

您可以通过您的帐户从[`www.packtpub.com`](http://www.packtpub.com)下载所有您购买的 Packt 出版书籍的示例代码文件。如果您是在其他地方购买了此书，可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)并注册，以便将文件直接通过电子邮件发送给您。

## 勘误

尽管我们已尽力确保内容的准确性，但难免会出现错误。如果您在我们的书籍中发现错误——例如文本或代码中的错误——我们将非常感激您能向我们报告。这样，您可以避免其他读者的困扰，并帮助我们改进后续版本的书籍。如果您发现任何勘误，请访问[`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)，选择您的书籍，点击**勘误提交表单**链接，并填写勘误的详细信息。一旦您的勘误被验证，您的提交将被接受，勘误将上传至我们的网站或添加到该书籍的勘误列表中。

要查看之前提交的勘误，请访问[`www.packtpub.com/books/content/support`](https://www.packtpub.com/books/content/support)，并在搜索框中输入书籍的名称。所需的信息将在**勘误**部分显示。

## 盗版

网络上版权材料的盗版是一个持续存在的问题，涉及所有媒体。在 Packt，我们非常重视保护我们的版权和许可证。如果您在网络上发现我们的作品的任何非法副本，请立即向我们提供该副本的地址或网站名称，以便我们采取措施。

请通过`<copyright@packtpub.com>`联系我们，提供涉嫌盗版材料的链接。

我们感谢您为保护我们的作者以及我们提供有价值内容的能力所做的帮助。

## 问题

如果您在本书的任何方面遇到问题，可以通过`<questions@packtpub.com>`与我们联系，我们将尽力解决问题。
