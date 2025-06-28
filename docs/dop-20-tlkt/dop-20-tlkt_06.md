# 第六章：Docker 世界中的配置管理

任何管理超过几个服务器的人都可以确认，手动完成这种任务既浪费时间又存在风险。**配置管理**（**CM**）已经存在很长时间了，我无法想到不使用某种工具的理由。问题不是是否要采纳其中一个工具，而是选择哪个工具。那些已经采用其中一个工具并投入了大量时间和金钱的人，可能会争辩说，最好的工具就是他们选择的那个。正如事情通常发展的那样，选择随着时间的推移而变化，选择一个工具的理由可能与昨天不同。在大多数情况下，决策并不是基于可用的选项，而是基于我们誓言维护的遗留系统的架构。如果这些系统可以被忽视，或者有足够勇气和资金的人愿意对其进行现代化，那么今天的现实将被容器和微服务主导。在这种情况下，我们昨天做出的选择与今天可以做出的选择是不同的。

# CFEngine

CFEngine 可以被认为是配置管理的奠基者。它于 1993 年创建，彻底改变了我们处理服务器设置和配置的方式。它最初是一个开源项目，2008 年推出了第一个企业版本，开始商业化。

CFEngine 是用 C 语言编写的，只有少数依赖项，并且运行速度极快。事实上，据我所知，没有其他工具能够超越 CFEngine 的速度。这曾经是，并且至今仍然是它的主要优势。然而，它也有一些弱点，要求具备编程技能可能是其中最大的弱点。在许多情况下，一名普通操作员无法使用 CFEngine。它需要一名 C 开发者来管理。尽管如此，它依然在一些大型企业中得到了广泛采用。然而，随着年轻工具的崛起，新的工具相继问世，今天，除了因为公司在其上的投资而被迫选择外，很少有人会选择 CFEngine。

## Puppet

后来，Puppet 应运而生。它也最初作为一个开源项目开始，然后推出了企业版本。与 CFEngine 相比，它因其基于模型的方式和较小的学习曲线被认为更“操作友好”。最终，出现了一款操作部门可以使用的配置管理工具。与 CFEngine 使用的 C 语言不同，Ruby 被证明更容易理解并且更容易被操作部门接受。CFEngine 的学习曲线可能是 Puppet 能够立足配置管理市场并逐渐取代 CFEngine 的主要原因。这并不意味着 CFEngine 已经不再使用。它仍然在使用，而且似乎不会像 Cobol 那样在银行和其他金融相关业务中消失。尽管如此，它失去了作为首选工具的声誉。

## 厨师

然后，Chef 应运而生，承诺解决 Puppet 的一些细节问题。它确实做到了，一段时间内也很有效。后来，随着 Puppet 和 Chef 的普及，它们进入了零和博弈。当其中一个工具推出新功能或做出改进时，另一个工具也会采纳。两者都有越来越多的工具，往往使它们的学习曲线和复杂度不断增加。Chef 对开发者更友好，而 Puppet 更倾向于面向运维和系统管理员类型的任务。两者没有明显的优劣之分，选择通常更多是基于个人经验。Puppet 和 Chef 都是成熟的、广泛采用的（尤其在企业环境中），并且有着大量的开源贡献。唯一的问题是，它们对于我们试图实现的目标来说，过于复杂。它们都没有考虑到容器的出现。它们在设计时并不知道 Docker 会改变游戏规则，因为在它们设计时，Docker 并不存在。

到目前为止，我们提到的所有配置管理工具都试图解决一些问题，这些问题在我们采用容器和不可变部署后就不再存在了。我们以前遇到的服务器混乱已经不复存在。现在，我们不再面对成百上千的包、配置文件、用户、日志等，而是需要处理大量的容器和非常有限的其他资源。这并不意味着我们不再需要配置管理，我们仍然需要！然而，选择的工具应当执行的范围要小得多。在大多数情况下，我们只需要一两个用户、运行中的 Docker 服务以及一些其他组件。其他的基本都是容器。部署已经成为另一类工具的范畴，并重新定义了配置管理应做的事情。Docker Compose、Mesos、Kubernetes 和 Docker Swarm，仅是我们今天可能使用的快速增长的部署工具中的一部分。在这种情况下，我们选择的配置管理工具应当更注重简洁性和不可变性，而不是其他因素。语法应当简单、易读，即使是那些从未使用过该工具的人也能理解。不可变性可以通过强制推送模型来实现，这种模型不需要在目标服务器上安装任何东西。

### Ansible

Ansible 尝试解决与其他配置管理工具相同的问题，但采用了完全不同的方式。一个显著的区别是，它通过 SSH 执行所有操作。CFEngine 和 Puppet 要求在所有需要管理的服务器上安装客户端。而 Chef 声称它不需要，但其对无代理运行的支持功能有限。与此相比，Ansible 的一个巨大优势是，它不需要服务器具备任何特殊的配置，因为 SSH（几乎）总是存在的。它利用了一个定义良好且广泛使用的协议来执行需要执行的命令，以确保目标服务器符合我们的规格。唯一的要求是 Python，而大多数 Linux 发行版上已经预装了 Python。换句话说，与那些试图强迫你以特定方式设置服务器的竞争者不同，Ansible 利用现有的现实条件，不需要任何额外配置。由于其架构，你只需要在一台 Linux 或 OS X 计算机上运行一个实例。例如，我们可以从笔记本电脑管理所有服务器。虽然这样做并不建议，Ansible 应该运行在一台真正的服务器上（最好是与其他持续集成和部署工具一起安装的服务器），但笔记本电脑的例子说明了它的简便性。根据我的经验，像 Ansible 这样的推送式系统比我们之前讨论的拉取式工具更容易理解。

与掌握其他工具所需的复杂性相比，学习 Ansible 所花费的时间极少。它的语法基于 YAML，只需简单浏览一个 playbook，即使是从未使用过该工具的人也能明白发生了什么。与 Chef、Puppet，尤其是 CFEngine 这些由开发者为开发者编写的工具不同，Ansible 是由开发者为那些有更重要事情要做的人编写的，他们不想再学习另一种语言和/或领域特定语言（DSL）。

有人会指出，Ansible 的一个主要缺点是对 Windows 的支持有限。客户端甚至无法在 Windows 上运行，且可以在 playbooks 中使用并在其上运行的模块数量非常有限。就我而言，在使用容器的情况下，这个缺点反而是一个优势。Ansible 的开发者没有浪费时间去创建一个通用工具，而是专注于最有效的方式（在 Linux 上通过 SSH 执行命令）。无论如何，Docker 目前还不能在 Windows 上运行容器。未来可能会，但在我写这篇文章时（或者至少是在那个时刻），这还在路线图上。即使忽略容器及其在 Windows 上的未来问题，其他工具在 Windows 上的表现也远不如在 Linux 上好。简而言之，Windows 的架构并不像 Linux 那样适合配置管理的目标。

我可能走得太远了，不应该过于苛刻地评价 Windows 或质疑你的选择。如果你更喜欢 Windows 服务器而非某些 Linux 发行版，那么我对 Ansible 的所有赞扬就变得毫无意义。你应该选择 Chef 或 Puppet，除非你已经在使用它，否则忽略 CFEngine。

## 最后的思考

如果几年前有人问我应该使用哪个工具，我会很难回答。今天，如果有机会切换到容器（无论是 Docker 还是其他类型）和不可变部署，那么选择是显而易见的（至少在我提到的工具中是这样）。Ansible（结合 Docker 和 Docker 部署工具）随时都能脱颖而出。我们甚至可以争论是否根本不需要配置管理工具。有些例子中，人们完全依赖，比如说 CoreOS、容器和 Docker Swarm 或 Kubernetes 这样的部署工具。我并没有这么激进的观点（至少现在还没有），我认为配置管理仍然是工具库中的宝贵工具。由于配置管理工具需要执行的任务范围，Ansible 正是我们需要的工具。任何更复杂或者更难学习的工具都会是浪费。我还没见过哪个人曾在维护 Ansible playbook 时遇到困难。因此，配置管理可以迅速成为整个团队的责任。我并不是在说基础设施应该轻视（它绝对不应该）。然而，项目中整个团队的贡献是任何类型任务的重大优势，配置管理也不应例外。CFEngine、Chef 和 Puppet 因其复杂的架构和陡峭的学习曲线而显得过于繁琐，至少在与 Ansible 相比时是这样。

我们简要介绍的四个工具绝不是唯一可供选择的工具。你可能会轻易地争辩说这些工具都不是最好的，而选择其他工具。没问题，这完全取决于我们尝试实现的目标和个人偏好。然而，与其他工具不同，Ansible 几乎不可能浪费时间。它非常容易学习，即使你选择不使用它，也无法说浪费了大量宝贵的时间。此外，我们学习的每一样东西都会带来新的收获，并让我们成为更优秀的专业人士。

你现在可能已经猜到，Ansible 将是我们用来进行配置管理的工具。

## 配置生产环境

让我们看看 Ansible 的实际操作，然后讨论它是如何配置的。我们需要两台虚拟机正常运行；`cd` 将用作一个服务器，我们将从这个服务器配置生产节点。

```
vagrant up cd prod --provision
vagrant ssh cd
ansible-playbook /vagrant/ansible/prod.yml -i /vagrant/ansible/hosts/prod

```

输出应该类似于以下内容：

```
PPLAY [prod] *******************************************************************
 GATHERING FACTS ***************************************************************
 The authenticity of host '10.100.198.201 (10.100.198.201)' can't be established.
 ECDSA key fingerprint is 2c:05:06:9f:a1:53:2a:82:2a:ff:93:24:d0:94:f8:82.
 Are you sure you want to continue connecting (yes/no)? yes
 ok: [10.100.198.201]
 TASK: [common | JQ is present] ************************************************
 changed: [10.100.198.201]
 TASK: [docker | Debian add Docker repository and update apt cache] ************
 changed: [10.100.198.201]
 TASK: [docker | Debian Docker is present] *************************************
 changed: [10.100.198.201]
 TASK: [docker | Debian python-pip is present] *********************************
 changed: [10.100.198.201]
 TASK: [docker | Debian docker-py is present] **********************************
 changed: [10.100.198.201]
 TASK: [docker | Debian files are present] *************************************
 changed: [10.100.198.201]
 TASK: [docker | Debian Daemon is reloaded] ************************************
 skipping: [10.100.198.201]
 TASK: [docker | vagrant user is added to the docker group] ********************
 changed: [10.100.198.201]
 TASK: [docker | Debian Docker service is restarted] ***************************
 changed: [10.100.198.201]
 TASK: [docker-compose | Executable is present] ********************************
 changed: [10.100.198.201]
 PLAY RECAP ********************************************************************
 10.100.198.201             : ok=11   changed=9    unreachable=0    failed=0
```

Ansible（以及一般的配置管理）中最重要的一点是，我们在大多数情况下指定的是某个事物的期望状态，而不是我们想要执行的命令。Ansible 将尽力确保服务器处于该状态。从上面的输出中我们可以看到所有任务的状态要么是*已更改*，要么是*跳过*。例如，我们指定了需要 Docker 服务，Ansible 注意到目标服务器（*prod*）上没有安装 Docker，于是安装了它。

如果我们再次运行 playbook，会发生什么？

```
ansible-playbook prod.yml -i hosts/prod

```

你会注意到所有任务的状态都是`ok`：

```
PLAY [prod] *******************************************************************
GATHERING FACTS ***************************************************************
ok: [10.100.198.201]
TASK: [common | JQ is present] ************************************************
ok: [10.100.198.201]
TASK: [docker | Debian add Docker repository and update apt cache] ************
ok: [10.100.198.201]
TASK: [docker | Debian Docker is present] *************************************
ok: [10.100.198.201]
TASK: [docker | Debian python-pip is present] *********************************
ok: [10.100.198.201]
TASK: [docker | Debian docker-py is present] **********************************
ok: [10.100.198.201]
TASK: [docker | Debian files are present] *************************************
ok: [10.100.198.201]
TASK: [docker | Debian Daemon is reloaded] ************************************
skipping: [10.100.198.201]
TASK: [docker | vagrant user is added to the docker group] ********************
ok: [10.100.198.201]
TASK: [docker | Debian Docker service is restarted] ***************************
skipping: [10.100.198.201]
TASK: [docker-compose | Executable is present] ********************************
ok: [10.100.198.201]
PLAY RECAP ********************************************************************
10.100.198.201             : ok=10   changed=0    unreachable=0    failed=0
```

Ansible 访问了服务器并逐一检查了所有任务的状态。由于这是第二次运行，并且我们没有修改服务器上的任何内容，Ansible 认为无需执行任何操作，当前状态与预期一致。

我们刚刚运行的命令（`ansible-playbook prod.yml -i hosts/prod`）非常简单。第一个参数是 playbook 的路径，第二个参数的值表示包含应该运行此 playbook 的服务器列表的清单文件路径。

这是一个非常简单的例子。我们需要设置生产环境，此时所需的只是 Docker、Docker Compose 和一些配置文件。稍后，我们将看到更复杂的例子。

现在我们已经看到 Ansible 的实际运行，让我们来看一下我们刚刚运行的（两次）*playbook* 的配置。

## 设置 Ansible Playbook

`prod.yml` Ansible playbook 的内容如下：

```
- hosts: prod
  remote_user: vagrant
  serial: 1
  sudo: yes
  roles: - common - docker
```

仅通过阅读 playbook，应该能够理解它的内容。它在名为*prod*的主机上以用户*vagrant*身份运行，并以 `sudo` 权限执行命令。底部是角色列表，在我们的案例中只有两个角色：`common` 和 `docker`。角色是一组通常围绕某一功能、产品或操作类型等组织的任务。Ansible playbook 的组织结构基于将任务分组到角色中，角色可以组合成 playbook。

在我们查看之前，让我们先讨论一下 *docker* 角色的目标。我们希望确保 Docker Debian 仓库存在，并且安装了最新的 *docker-engine* 包。稍后，我们需要安装 `docker-py`（Docker 的 Python API 客户端），可以通过 `pip` 安装，因此我们确保系统中同时存在这两者。接下来，我们需要将标准的 Docker 配置替换为我们位于 *files* 目录下的文件。Docker 配置要求重新启动 Docker 服务，所以每次 `files/docker` 文件发生更改时，我们都会执行此操作。最后，我们确保用户 *vagrant* 被添加到 *docker* 组中，从而能够运行 Docker 命令。

让我们看一下定义我们正在使用的角色的 `roles/docker` 目录。它由两个子目录组成：`files` 和 `tasks`。任务是任何角色的核心，默认情况下，任务需要在 `main.yml` 文件中定义：

```
The content of the roles/docker/tasks/main.yml file is as follows.
- include: debian.yml
  when: ansible_distribution == 'Debian' or ansible_distribution == 'Ubuntu'
- include: centos.yml
  when: ansible_distribution == 'CentOS' or ansible_distribution == 'Red Hat Enterprise Linux'
```

由于我们将在 Debian（Ubuntu）和 CentOS 或 Red Hat 上运行 Docker，因此角色被拆分为 `debian.yml` 和 `centos.yml` 文件。目前，我们将使用 Ubuntu，所以让我们看看 `roles/docker/tasks/debian.yml` 角色。

```
- name: Debian add Docker repository and update apt cache
  apt_repository:
    repo: deb https://apt.dockerproject.org/repo ubuntu-{{ debian_version }} main
    update_cache: yes
    state: present
  tags: [docker]
- name: Debian Docker is present
  apt:
    name: docker-engine
    state: latest
    force: yes
  tags: [docker]
- name: Debian python-pip is present
  apt: name=python-pip state=present
  tags: [docker]
- name: Debian docker-py is present
  pip: name=docker-py version=0.4.0 state=present
  tags: [docker]
- name: Debian files are present
  template:
    src: "{{ docker_cfg }}"
    dest: "{{ docker_cfg_dest }}"
  register: copy_result
  tags: [docker]
- name: Debian Daemon is reloaded
  command: systemctl daemon-reload
  when: copy_result|changed and is_systemd is defined
  tags: [docker]
- name: vagrant user is added to the docker group
  user:
    name: vagrant
    group: docker  register: user_resulttags: [docker]- name: Debian Docker service is restarted
  service:
    name: docker
    state: restarted
  when: copy_result|changed or user_result|changed
  tags: [docker]
```

如果这是一个不同的框架或工具，我会逐一讲解每个任务，你也会非常感谢获得更多的智慧。然而，我认为没有必要这样做。Ansible 非常直接。假设你有基本的 Linux 知识，我敢打赌你可以在没有任何进一步解释的情况下理解每个任务。如果我错了，你确实需要解释，请查看 Ansible 文档中的[所有模块列表](http://docs.ansible.com/ansible/list_of_all_modules.html)。例如，如果你想知道第二个任务做了什么，你可以打开 apt 模块。现在唯一需要知道的是缩进的工作原理。YAML 基于 `key: value` 和 `parent/child` 结构。例如，最后一个任务有 `name` 和 `state` 键，它们是 `service` 的子项，而 `service` 又是 Ansible 模块之一。

还有一件事我们在 `prod.yml` playbook 中使用过。我们执行的命令带有 `-i hosts/prod` 参数，我们用它来指定 inventory 文件，其中列出了 playbook 应该运行的主机。`hosts/prod` inventory 相当大，因为它在整个书中都有使用。目前，我们只关心 `prod` 部分，因为那是我们在 playbook 中指定的 `hosts` 参数的值：

```
...
[prod]
10.100.198.201
...
```

如果我们想将相同的配置应用到多个服务器，只需添加另一个 IP 地址。

我们稍后会看到更复杂的示例。我故意说“更复杂”，因为在 Ansible 中没有什么是非常复杂的，但根据一些任务及其相互依赖性，某些角色可能更复杂或更简单。我希望我们刚刚运行的 playbook 能给你一个 Ansible 工具类型的大致了解，也希望你喜欢它。我们将依赖它来完成所有配置管理任务以及更多工作。

你可能已经注意到，我们从未进入过`prod`环境，而是通过`cd`服务器远程运行所有操作。本书中将继续采用这种做法。借助 Ansible 和后面我们将介绍的其他工具，我们无需通过 ssh 进入服务器并手动执行任务。依我看，我们的知识和创造力应该用于编程，其他所有的事情都应该是自动化的；例如测试、构建、部署、扩展、日志记录、监控等等。这是本书的一大收获。成功的关键在于大规模的自动化，它让我们有更多时间去做令人兴奋和更具生产力的任务。

和以前一样，我们将在本章结束时销毁所有虚拟机。下一章将创建我们所需的虚拟机：

```
exit
vagrant destroy -f

```

随着第一台生产服务器（目前仅运行 Ubuntu 操作系统、Docker 和 Docker Compose）上线，我们可以继续进行部署管道的基本实现工作。
