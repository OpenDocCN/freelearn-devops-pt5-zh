# 第十三章：编排器

在上一章中，我们介绍了 Docker Compose，它是一个允许我们在单一 Docker 主机上以声明式方式处理多服务应用的工具。

本章介绍了编排器的概念。它讲解了为什么需要编排器以及它们的概念性工作原理。本章还将概述最流行的编排器，并列出它们的一些优缺点。

在本章中，我们将涵盖以下主题：

+   什么是编排器，我们为什么需要它们？

+   编排器的任务

+   常见编排器概览

完成本章后，您将能够完成以下任务：

+   请列出三到四个由编排器负责的任务

+   列举两到三种最流行的编排器

+   用你自己的话，并通过适当的类比，向一个感兴趣的外行解释为什么我们需要容器编排器

# 什么是编排器，我们为什么需要它们？

在第九章*，分布式应用架构*中，我们学习了构建、部署和运行高分布式应用程序时常用的模式和最佳实践。现在，如果我们的分布式应用程序是容器化的，那么我们就面临着与非容器化分布式应用程序相同的问题或挑战。这些挑战正是我们在第九章*，分布式应用架构*中讨论过的——服务发现、负载均衡、扩展等。

类似于 Docker 对容器的做法——通过引入容器来标准化软件的打包和运输——我们希望有一个工具或基础设施软件来处理我们刚才提到的所有或大多数挑战。这些软件就是我们所说的容器编排器，或者也可以称它们为编排引擎。

如果我刚才说的这些还没有让你完全理解，那我们换个角度来看这个问题。想象一个演奏乐器的艺术家，他们能够独自为观众演奏美妙的音乐——仅仅是艺术家和他们的乐器。但是，如果你把一群音乐家组成一个管弦乐队，安排他们在同一个房间里，给他们一份交响曲的乐谱，让他们去演奏，然后离开房间。如果没有指挥，这些非常有天赋的音乐家可能无法和谐地演奏这首曲子；最终的效果可能听起来就像一阵杂乱无章的噪音。只有当乐团有了一位指挥，来指挥这些音乐家，乐团演奏出的音乐才会让我们的耳朵感到愉悦：

![](img/a0425d38-20e7-4cc7-b2f5-5680c5ae5b2a.jpg)

容器编排器就像管弦乐队的指挥

来源： [`it.wikipedia.org/wiki/Giuseppe_Lanzetta#/media/File:UMB_5945.JPG`](https://it.wikipedia.org/wiki/Giuseppe_Lanzetta#/media/File:UMB_5945.JPG)

许可证： [`creativecommons.org/licenses/by-sa/3.0/deed.en`](https://creativecommons.org/licenses/by-sa/3.0/deed.en)

现在我们不再有音乐家，而是有容器；不再有不同的乐器，而是有对容器主机有不同要求的容器。与音乐以不同的节奏演奏不同，我们现在有容器以特定的方式相互通信，并且需要进行扩展或缩减。在这方面，容器编排器的角色与交响乐团的指挥非常相似。它确保集群中的容器和其他资源能够和谐地共同工作。

我希望你现在能更清楚地看到容器编排器是什么，以及为什么我们需要它。假设你确认了这个问题，我们现在可以问自己，编排器如何实现预期的结果，也就是确保集群中的所有容器能够和谐地协同工作。那么，答案是，编排器必须执行非常具体的任务，类似于交响乐团指挥执行一系列任务，以驯服并同时提升乐团的表现。

# 编排器的任务

*那么，**我们期望一个值得信赖的编排器为我们执行哪些任务呢？*让我们详细看一下它们。以下列表展示了在写这篇文章时，企业用户通常期望他们的编排器完成的最重要的任务。

# 协调期望状态

使用编排器时，你以声明式的方式告诉它如何运行特定的应用或应用服务。我们在第十一章*，Docker Compose*中学习了声明式与命令式的区别。这种描述我们想要运行的应用服务的声明式方式包括元素，例如使用哪个容器镜像、运行多少个服务实例、开放哪些端口等等。我们所说的应用服务的属性声明就是我们所称之为的*期望状态*。

所以，当我们第一次告诉编排器根据声明创建一个新的应用服务时，编排器会确保按要求调度集群中尽可能多的容器。如果容器镜像尚未在容器应运行的目标节点上可用，调度器会确保先从镜像注册表下载它们。接下来，容器会启动，并配置所有设置，例如要连接的网络或需要暴露的端口。编排器会尽力确保集群的实际状态与声明完全匹配。

一旦我们的服务按要求启动并运行，即处于期望的状态，编排器会继续监控它。每次编排器发现服务的实际状态与期望状态之间存在差异时，它会再次尽最大努力去协调期望状态。

应用服务的实际状态和期望状态之间可能出现什么样的不一致呢？假设服务的某个副本，也就是其中一个容器，由于一个 bug 崩溃了，那么调度器会发现实际状态与期望状态在副本数量上不同：少了一个副本。调度器会立即将一个新的实例调度到集群的另一个节点，替代崩溃的实例。另一个不一致可能是应用服务运行的实例过多，如果该服务已经被缩减。在这种情况下，调度器会随机杀掉多余的实例，以便使实际实例数与期望实例数保持一致。还有一种不一致可能是调度器发现有某个实例正在运行错误（可能是旧版）的底层容器镜像。到现在，你应该明白了吧？

因此，代替我们主动监控在集群中运行的应用服务，并纠正任何偏离期望状态的情况，我们将这项繁琐的任务委托给了调度器。前提是我们使用声明式而非命令式的方式来描述应用服务的期望状态，这样的方式效果非常好。

# 复制型和全局型服务

我们可能想要在由调度器管理的集群中运行两种截然不同类型的服务。它们是 *复制型* 和 *全局型* 服务。复制型服务是指需要以特定数量的实例运行的服务，例如 10 个实例。全局型服务则是要求在集群的每个工作节点上都运行恰好一个实例的服务。我在这里使用了 *工作节点* 这一术语。在由调度器管理的集群中，我们通常有两种类型的节点，*管理节点* 和 *工作节点*。管理节点通常由调度器专门用于管理集群，并不运行其他工作负载。而工作节点则用来运行实际的应用程序。

因此，调度器确保对于全局服务，它在每个工作节点上都有一个实例在运行，无论节点数量是多少。我们不需要关心实例的数量，只需要确保每个节点上都保证运行一个服务实例。

再次强调，我们可以完全依赖调度器来处理这一切。在复制型服务中，我们将始终保证找到精确的期望实例数；而对于全局型服务，我们可以确保在每个工作节点上总是会运行恰好一个服务实例。调度器将始终尽全力保证这一期望状态的实现。

在 Kubernetes 中，全局服务也被称为 **DaemonSet**。

# 服务发现

当我们以声明性方式描述应用服务时，我们永远不应该告诉编排器不同实例必须运行在哪些集群节点上。我们将这项决策交给编排器来决定，哪个节点最适合执行这个任务。

当然，从技术上讲，可以指示编排器使用非常确定性的部署规则，但这将是一个反模式，除非在非常特殊的边缘情况下，否则绝对不推荐这样做。

所以，如果我们现在假设编排引擎对应用服务的单个实例的部署地点具有完全的自由意志，而且，进一步假设实例可能崩溃并被编排器重新调度到不同的节点，那么我们会意识到，追踪单个实例在任何给定时间运行的位置是徒劳的任务。更好的是，我们甚至不应该尝试了解这个，因为它并不重要。

好吧，你可能会说，但如果我有两个服务，A 和 B，而服务 A 依赖于服务 B；*难道服务 A 的任何给定实例不应该知道它能在哪里找到服务 B 的实例吗？*

在这里，我必须大声清楚地说——不，它不应该。 在高度分布式和可扩展的应用程序中，这种知识是不希望出现的。相反，我们应该依赖编排器提供我们所需的信息，以便到达我们依赖的其他服务实例。这有点像电话时代的情况，当时我们不能直接拨打朋友的电话，而是必须拨打电话公司的中央办公室，在那里某个接线员会将我们路由到正确的目的地。在我们的例子中，编排器充当接线员的角色，将来自服务 A 实例的请求路由到可用的服务 B 实例。这个整个过程叫做**服务发现**。

# 路由

到目前为止，我们已经了解到，在分布式应用中，我们有许多相互交互的服务。当服务 A 与服务 B 交互时，它是通过数据包的交换来实现的。这些数据包需要通过某种方式从服务 A 传递到服务 B。这个将数据包从源传送到目标的过程也叫做**路由**。作为应用的开发者或运营者，我们确实期望调度器承担起路由的任务。正如我们在后续章节中所看到的，路由可以在不同的层级上进行。就像在现实生活中一样。假设你在一家大公司的一座办公楼里工作。现在，你有一份文件需要转交给公司里的另一位员工。内部邮政服务会从你的发件箱取走文件，并将其送到同一座大楼内的邮局。如果目标员工在同一座大楼里，文件可以直接转交给他/她。如果目标员工在同一区块的另一栋大楼里，文件将被转发到该目标大楼的邮局，然后通过内部邮政服务将其分发给收件人。第三种情况是，如果文件的目标员工在公司的另一个分支，且该分支位于不同的城市，甚至是不同的国家，那么文件将被转交给外部邮政服务，如 UPS，它将把文件运送到目标位置，然后再次由内部邮政服务接手并将文件送达收件人。

在容器中运行的应用服务之间路由数据包时，也会发生类似的情况。源容器和目标容器可能位于同一个集群节点上，这对应着两个员工在同一座大楼里工作的情况。目标容器也可能运行在不同的集群节点上，这对应着两个员工在同一区块的不同大楼里工作的情况。最后，第三种情况是当数据包来自集群外部，需要路由到集群内部运行的目标容器时。

所有这些情况，甚至更多，都需要由调度器来处理。

# 负载均衡

在一个高可用的分布式应用中，所有组件都必须具备冗余性。这意味着每个应用服务都必须运行多个实例，这样，如果一个实例出现故障，整个服务仍然能够正常运行。

为了确保所有服务实例实际上都在处理工作，而不是闲置，我们必须确保服务请求均匀地分配到所有实例上。将工作负载分配到各个服务实例的过程叫做**负载均衡**。存在多种算法用于分配工作负载。通常，负载均衡器使用所谓的轮询算法，确保工作负载均匀地分配给各个实例，使用的是一个循环算法。

再次强调，我们期望调度器处理来自一个服务到另一个服务，或从外部源到内部服务的负载均衡请求。

# 扩展

当我们在由调度器管理的集群中运行容器化的分布式应用时，我们还需要一种简便的方式来处理预期的或意外的工作负载增加。为了处理增加的工作负载，我们通常只需要调度更多的服务实例来应对这种负载增加。负载均衡器会自动配置，将工作负载分配到更多可用的目标实例上。

但在现实场景中，工作负载是随着时间变化的。如果我们看一个像亚马逊这样的购物网站，可能在晚上的高峰时段流量很大，因为大家都在家里购物；在黑色星期五等特殊日子，流量可能极为庞大；而在清晨，流量则可能非常少。因此，服务不仅需要能够扩展，还需要在工作负载减少时能够缩减。

我们还期望调度器在扩展服务时，能够以有意义的方式分布服务的实例。当进行扩展或缩减时，将所有服务实例调度到同一集群节点上并不是明智的做法，因为如果该节点宕机，整个服务都会宕机。调度器负责容器放置的调度器，也需要考虑不要将所有实例放在同一台计算机机架上，因为如果机架的电源出现故障，同样会影响整个服务。此外，关键服务的实例应该分布在不同的数据中心，以避免服务中断。所有这些决策，甚至更多，都是调度器的责任。

在云环境中，通常使用“**可用区**”这一术语，而不是计算机机架。

# 自愈

现在，调度器非常复杂，能够为我们维护一个健康的系统做很多工作。调度器监控集群中所有正在运行的容器，并自动替换崩溃或无响应的容器，换成新的实例。调度器还监控集群节点的健康状况，如果某个节点变得不健康或宕机，会将其从调度器循环中移除。原本位于这些节点上的工作负载会自动重新调度到其他可用节点。

所有这些活动，其中协调器监控当前状态并自动修复损坏或协调期望状态，最终形成了所谓的**自愈**系统。在大多数情况下，我们不需要主动介入修复损坏。协调器会自动为我们完成这项工作。

然而，协调器有一些情况是无法在没有我们帮助的情况下处理的。想象一下我们有一个在容器中运行的服务实例。容器正常运行，从外部看似健康无虞。但是，容器内运行的应用程序却处于不健康状态。应用程序并没有崩溃，它只是无法像最初设计的那样工作了。*协调器怎么可能在没有我们提示的情况下知道这一点呢？* 它不知道！处于不健康或无效状态对每个应用服务来说意味着完全不同的事情。换句话说，健康状态是服务依赖的。只有服务的作者或其操作员知道在服务的上下文中“健康”意味着什么。

现在，协调器定义了缝隙或探针，应用服务可以通过它们向协调器传达其当前状态。有两种基本的探针类型：

+   服务可以告诉协调器它是健康的还是不健康的

+   服务可以告诉协调器它已准备好或暂时不可用

服务如何确定前述的答案完全取决于服务本身。协调器仅定义它将如何提问，例如，通过`HTTP GET`请求，或它期望得到什么类型的答案，例如`OK`或`NOT OK`。

如果我们的服务实现了逻辑来回答前述的健康或可用性问题，那么我们就拥有一个真正的自愈系统，因为协调器可以杀死不健康的服务实例，并用新的健康实例替换它们，且可以将暂时不可用的服务实例从负载均衡器的轮询中移除。

# 零停机部署

如今，完全停机进行任务关键型应用程序更新的理由变得越来越难以站得住脚。这不仅意味着错失机会，还可能导致公司的声誉受损。使用该应用的客户已不再接受这样的不便，并会迅速转身离开。此外，我们的发布周期越来越短。过去我们每年只有一到两个新版本发布，而现在，很多公司每周甚至每天更新多次他们的应用程序。

解决这个问题的办法是制定一个零停机的应用程序更新策略。调度器需要能够批量更新各个应用程序服务。这也被称为**滚动更新**。在任何给定时刻，只有一个或少数几个实例会被下线，并被新版本的服务替换。只有当新实例正常运行，并且没有产生任何意外错误或表现出任何异常行为时，下一批实例才会被更新。这个过程会一直重复，直到所有实例都被替换为新版本。如果由于某些原因更新失败，那么我们期望调度器会自动将更新的实例回滚到之前的版本。

其他可能的零停机部署方式包括蓝绿部署和金丝雀发布。在这两种方式中，服务的新版本与当前的活跃版本并行安装。但最初，新版本仅对内部可用。操作团队可以对新版本进行冒烟测试，当新版本运行正常时，在蓝绿部署的情况下，路由器会从当前的蓝色版本切换到新的绿色版本。在此期间，新绿色版本的服务会受到密切监控，如果一切正常，则可以停用旧的蓝色版本。另一方面，如果新的绿色版本未按预期运行，那么只需要将路由器切换回旧的蓝色版本，即可实现完全回滚。

在金丝雀发布的情况下，路由器的配置方式是将整体流量的一个小比例（比如 1%）引导到新版本的服务上，而 99%的流量仍然通过旧版本的服务。新版本的行为会被密切监控，并与旧版本的行为进行比较。如果一切正常，便会稍微增加通过新服务的流量比例。这个过程会一直重复，直到 100%的流量都通过新服务。如果新服务运行一段时间且一切正常，则可以停用旧服务。

大多数调度器至少支持开箱即用的滚动更新类型零停机部署。蓝绿部署和金丝雀发布通常也非常容易实现。

# 亲和性和位置感知

有时，某些应用服务需要在运行它们的节点上具备专用硬件。例如，I/O 密集型服务需要配备高性能**固态硬盘**（**SSD**）的集群节点，或者一些用于机器学习等的服务需要配备**加速处理单元**（**APU**）。调度器允许我们为每个应用服务定义节点亲和性。然后，调度器会确保它的调度器只会在满足要求条件的集群节点上调度容器。

应避免将亲和性定义到特定节点上；这样做会引入单点故障，从而危及高可用性。始终将一组多个集群节点定义为应用服务的目标。

一些调度引擎还支持所谓的**位置感知**或**地理位置感知**。这意味着您可以要求调度器将服务的实例平均分布到不同的地点。例如，您可以定义一个`datacenter`标签，可能的值包括`west`、`center`和`east`，并将标签应用于所有集群节点，值对应于各自节点所在的地理区域。然后，您指示调度器使用这个标签来实现某个应用服务的地理位置感知。在这种情况下，如果您请求九个副本的服务，那么调度器会确保在三个数据中心——西部、中心和东部——的每个节点上部署三个实例。

地理位置感知甚至可以按层次定义；例如，您可以将数据中心作为顶级区分器，然后是可用性区域。

地理位置感知或位置感知用于减少由于电力供应故障或数据中心故障引起的停机概率。如果应用实例分布在多个节点、可用性区域甚至数据中心中，那么所有的实例同时宕机的可能性非常低。总会有一个区域是可用的。

# 安全

如今，IT 安全是一个非常热门的话题。网络战处于历史最高水平。大多数高知名度的公司都曾是黑客攻击的受害者，且付出了高昂的代价。每位**首席信息官**（**CIO**）或**首席技术官**（**CTO**）最糟糕的噩梦之一就是早上醒来，听到新闻说他们的公司成为了黑客攻击的受害者，并且敏感信息被窃取或泄露。

为了应对大多数安全威胁，我们需要建立一个安全的软件供应链，并加强深度安全防御。让我们来看看在企业级调度器中可以期待的一些任务。

# 安全通信和加密节点身份

首先，我们要确保由协调器管理的集群是安全的。只有受信任的节点才能加入集群。每个加入集群的节点都会获得一个加密的节点身份，节点之间的所有通信必须加密。为此，节点可以使用**相互传输层安全性**（**MTLS**）。为了验证集群节点之间的身份，使用证书。这些证书会定期自动轮换，或者按需轮换，以防止证书泄露时保护系统。

集群中的通信可以分为三种类型。你可以谈论通信平面——管理平面、控制平面和数据平面：

+   管理平面是集群管理员或主节点使用的平面，例如，调度服务实例、执行健康检查，或创建和修改集群中的其他资源，如数据卷、机密或网络。

+   控制平面用于在集群的所有节点之间交换重要的状态信息。例如，这些信息用于更新集群中的本地 IP 表，这些表用于路由。

+   数据平面是实际的应用服务相互通信和交换数据的地方。

通常，协调器主要关心保护管理平面和控制平面。数据平面的安全由用户负责，尽管协调器可以协助完成这一任务。

# 安全网络和网络策略

在运行应用服务时，并不是每个服务都需要与集群中的其他每个服务进行通信。因此，我们希望能够将服务相互隔离，只在那些必须互相通信的服务之间，才将它们放置在同一个网络沙箱中。所有其他服务和所有来自集群外部的网络流量都不应有机会访问这些沙箱服务。

这种基于网络的沙箱隔离可以通过至少两种方式实现。我们可以使用**软件定义网络**（**SDN**）将应用服务进行分组，或者可以使用一个扁平网络，结合网络策略控制谁有权限访问特定的服务或服务组。

# 基于角色的访问控制（RBAC）

在使集群适合企业使用之前，**协调器**必须完成的一项最重要任务（仅次于安全性）是提供基于角色的访问控制。RBAC 定义了系统的主体、用户或用户组（如团队等）如何访问和操作系统资源。它确保未经授权的人员无法对系统造成任何损害，也无法看到他们不应该知道或看到的系统资源。

一个典型的企业可能会有一些用户组，如开发组（Development）、质量保障组（QA）和生产组（Prod），每个组可以有一个或多个与之关联的用户。开发者约翰·多伊（John Doe）是开发组的成员，因此他可以访问专门为开发团队提供的资源，但他无法访问生产团队的资源，例如安·哈伯（Ann Harbor）是生产组的成员。反过来，她也不能干扰开发团队的资源。

实现基于角色的访问控制（RBAC）的一种方法是通过**授权**的定义。授权是主题、角色和资源集合之间的关联。在这里，角色由一组对资源的访问权限组成。这些权限可以是创建、停止、删除、列出或查看容器；部署新应用服务；列出集群节点或查看集群节点的详细信息等。

资源集合是集群中一组逻辑相关的资源，如应用服务、机密信息、数据卷或容器。

# 机密

在我们的日常生活中，有很多机密信息。机密信息是指那些不应该公开的内容，例如你用来访问网上银行账户的用户名和密码，或是你手机的密码，或者是健身房储物柜的密码。

在编写软件时，我们常常需要使用机密信息。例如，我们需要一个证书来验证我们的应用服务与我们想要访问的外部服务，或者我们需要一个令牌来验证和授权我们的服务在访问其他 API 时的权限。在过去，开发者为了方便，通常会将这些值硬编码，或者将它们以明文形式放在某些外部配置文件中。这样，这些非常敏感的信息就被广泛的群体所访问，而实际上，这些人不应该有机会看到这些机密。

幸运的是，现在的调度器提供了所谓的机密信息管理功能，以一种高度安全的方式处理这些敏感信息。机密可以由授权或可信的人员创建。之后，这些机密的值会被加密并存储在高度可用的集群状态数据库中。由于机密信息已经被加密，因此在静止状态下是安全的。一旦授权的应用服务请求机密信息，该机密仅会被转发到实际运行该服务实例的集群节点，而该机密的值从不存储在节点上，而是以`tmpfs`基于内存的卷挂载到容器中。只有在各自的容器内部，机密的值才以明文形式可用。

我们之前提到过，秘密在静态时是安全的。一旦服务请求了这些秘密，集群管理器或主节点会解密秘密，并通过网络将其发送到目标节点。*那么，秘密在传输过程中安全吗？* 好吧，我们之前了解到，集群节点在通信时使用 MTLS，因此尽管秘密以明文传输，但仍然是安全的，因为数据包会通过 MTLS 加密。因此，秘密在静态和传输过程中都能得到保障。只有获得授权的服务才能访问这些秘密值。

# 内容信任

为了提高安全性，我们希望确保在我们的生产集群中只运行受信任的镜像。某些调度器允许我们配置集群，使其只能运行签名镜像。内容信任和签名镜像的目的是确保镜像的作者是我们期望的，即我们的可信开发者，甚至更好的是，我们的可信 CI 服务器。此外，借助内容信任，我们还希望确保镜像是最新的，而不是过时的、可能存在漏洞的镜像。最后，我们还希望确保镜像在传输过程中不会被恶意黑客篡改。后者通常被称为**中间人攻击**（**MITM**）。

通过在源头签名镜像，并在目标上验证签名，我们可以确保我们想要运行的镜像没有被篡改。

# 反向正常运行时间

我在安全性方面想讨论的最后一点是反向正常运行时间。*这是什么意思？* 想象一下，你已经配置并保护了一个生产集群。在这个集群中，你运行着一些对公司至关重要的应用程序。现在，一个黑客成功找到了你某个软件栈的安全漏洞，并获得了你集群某个节点的 root 权限。这本身已经很糟糕了，但更糟糕的是，这个黑客现在可以在他已获得 root 访问权限的节点上掩盖自己的存在，然后利用它作为攻击集群中其他节点的基地。

在 Linux 或任何 Unix 类操作系统中，root 访问意味着你可以在系统上做任何事情。这是用户可以拥有的最高访问级别。在 Windows 中，等同的角色是管理员。

但是*如果我们利用容器是短暂的、集群节点通常在几分钟内完成自动化部署的事实会怎么样？* 我们可以在每个集群节点达到一定正常运行时间后（比如 1 天）将其终止。调度器会指示将节点排空，然后将其从集群中排除。一旦节点被移出集群，它就会被销毁并由一个新节点替代。

通过这种方式，黑客已经失去了它们的基础，问题也已经被解决。尽管如此，这个概念目前还没有广泛应用，但对我来说，这似乎是朝着增强安全性迈出的一大步，至少在我与在这个领域工作的工程师们讨论时，实施起来并不困难。

# 内省

到目前为止，我们已经讨论了许多编排器负责的任务，它可以完全自主执行。但是，操作人员也需要能够查看和分析集群上当前正在运行的内容，以及单个应用程序的状态或健康状况。为此，我们需要进行内省。编排器需要以易于消化和理解的方式展示关键信息。

编排器应收集所有集群节点的系统指标，并使操作人员能够访问这些指标。指标包括 CPU、内存和磁盘使用情况、网络带宽消耗等。这些信息应该可以按节点和聚合形式轻松获取。

我们还希望编排器为我们提供由服务实例或容器生成的日志访问权限。更进一步，如果有正确的授权，编排器应为每个容器提供 `exec` 访问权限。有了对容器的 `exec` 访问权限，您可以调试行为不端的容器。

在高度分布式应用中，每个应用请求经过多个服务处理，直到完全处理完成，跟踪请求是一个非常重要的任务。理想情况下，编排器支持我们实施跟踪策略，或者给出一些良好的遵循指南。

最后，当操作人员使用所有收集的指标、日志和跟踪信息的图形表示时，他们最能有效地监视系统。在这里，我们谈论的是仪表板。每个体面的编排器应至少提供一些基本的仪表板，显示最关键的系统参数的图形表示。

然而，人工操作人员不是唯一关心内省的人。我们还需要能够将外部系统连接到编排器，以便消费这些信息。需要提供一个 API，外部系统可以通过该 API 访问集群状态、指标和日志，并利用这些信息做出自动化决策，如创建警报或电话警报、发送电子邮件或在系统超出阈值时触发警报。

# 流行编排器概述

在撰写本文时，有许多编排引擎在使用中，但明显有几个领先者。排名第一的显然是 Kubernetes，其统治地位无可争议。遥远的第二名是 Docker 自家的 SwarmKit，其后是诸如 Apache Mesos、AWS **弹性容器服务** (**ECS**) 或 Microsoft **Azure 容器服务** (**ACS**) 等其他引擎。

# Kubernetes

Kubernetes 最初由 Google 设计，后来捐赠给了**Cloud Native Computing Foundation**（**CNCF**）。Kubernetes 的模型来源于 Google 的专有 Borg 系统，Borg 多年来一直在超大规模运行容器。Kubernetes 是 Google 重新审视并完全重新设计系统的尝试，目的是结合 Borg 的所有经验教训。

与专有技术 Borg 不同，Kubernetes 早期就开源了。这是 Google 非常明智的选择，因为它吸引了大量来自公司外部的贡献者，仅仅几年时间，围绕 Kubernetes 已经发展出一个更加庞大的生态系统。你可以毫不夸张地说，Kubernetes 是容器编排领域社区的宠儿。没有其他编排工具能够像它一样，激起如此大的关注，并吸引这么多愿意以有意义的方式贡献成功的项目成员或早期采用者。

在这方面，我觉得 Kubernetes 在容器编排领域，和 Linux 在服务器操作系统领域的地位非常相似。Linux 已经成为了服务器操作系统的*事实标准*。所有相关公司，如微软、IBM、亚马逊、红帽，甚至 Docker，都已经拥抱了 Kubernetes。

有一件事是不可否认的：Kubernetes 从一开始就被设计为具有大规模可扩展性。毕竟，它是以 Google 的 Borg 为基础设计的。

你可能对 Kubernetes 提出的一个负面意见是，至少在写这篇文章时，它的设置和管理仍然复杂。对于新手来说，存在一个显著的障碍。第一步陡峭，但一旦你与这个编排工具工作了一段时间，一切都会变得清晰。整体设计经过深思熟虑，并且执行得非常好。

在 Kubernetes 的 1.10 版本中，**正式发布版**（**GA**）于 2018 年 3 月发布，相较于 Docker Swarm 等其他编排工具，最初的一些不足之处已经被消除。例如，安全性和保密性现在不仅是事后的考虑，而是系统的一个不可或缺的部分。

新功能的实现速度极快。新的版本大约每三个月发布一次，准确来说，大约每 100 天发布一次。大部分新功能都是根据需求驱动的，也就是说，使用 Kubernetes 编排其关键任务应用程序的公司可以提出他们的需求。这使得 Kubernetes 变得适合企业级应用。认为这个编排工具只适用于初创企业，而不适用于风险规避的企业是错误的。恰恰相反，*我为什么这么说？* 其实，我的说法是有根据的，因为像微软、Docker 和 Red Hat 这样的公司，它们的客户大多是大型企业，已经完全拥抱了 Kubernetes，并为其提供企业级的支持，无论它是否被集成到他们的企业产品中。

Kubernetes 支持 Linux 和 Windows 容器。

# Docker Swarm

众所周知，Docker 使得软件容器成为大众化和商品化的产品。Docker 并没有发明容器，但它标准化了容器并使其广泛可用，特别是通过提供免费的镜像仓库——Docker Hub。最初，Docker 主要关注开发者和开发生命周期。然而，开始使用并喜爱容器的公司很快也希望在开发或测试新应用程序时使用它们，同时也希望将这些应用程序用于生产环境中。

最初，Docker 在这一领域并没有提供什么解决方案，因此其他公司进入了这一空白市场，并为用户提供帮助。但很快，Docker 意识到市场上对于一个简单而强大的编排器有着巨大的需求。Docker 的第一次尝试是推出了一个名为经典 Swarm 的产品。它是一个独立的产品，允许用户创建 Docker 主机集群，以便在高度可用和自愈的方式下运行和扩展他们的容器化应用程序。

然而，经典 Docker Swarm 的设置非常困难，涉及很多复杂的手动步骤。客户喜欢这个产品，但却因其复杂性而苦恼。因此，Docker 决定做得更好。它回到设计图纸，提出了 SwarmKit。SwarmKit 在 2016 年 DockerCon 大会上首次亮相，并成为 Docker 引擎最新版本的核心部分。没错，你没听错；SwarmKit 曾是，也是现在 Docker 引擎不可或缺的一部分。因此，如果你安装了 Docker 主机，它会自动带有 SwarmKit。

SwarmKit 的设计理念是简洁和安全。其口号是，至今依然如此，设置一个 Swarm 应该几乎是微不足道的，并且这个 Swarm 在开箱即用时就必须是高度安全的。Docker Swarm 的运行假设是最小权限原则。

安装一个完整的、高可用的 Docker Swarm，实际上就像在集群的第一个节点上执行`docker swarm init`，该节点将成为所谓的领导节点，然后在所有其他节点上执行`docker swarm join <join-token>`。`join-token`是在初始化过程中由领导节点生成的。整个过程在最多 10 个节点的 Swarm 上不到 5 分钟就能完成。如果是自动化的，时间会更少。

正如我之前提到的，安全性是 Docker 设计和开发 SwarmKit 时的首要考虑事项。容器通过依赖 Linux 内核命名空间和 cgroups，以及 Linux 系统调用白名单（seccomp），并支持 Linux 能力和**Linux 安全模块**（**LSM**）来提供安全性。除此之外，SwarmKit 增加了 MTLS 和在静态存储和传输过程中都加密的秘密管理功能。此外，Swarm 还定义了所谓的**容器网络模型**（**CNM**），该模型支持软件定义网络（SDN），为在 Swarm 上运行的应用服务提供沙箱隔离功能。

Docker SwarmKit 支持 Linux 和 Windows 容器。

# Apache Mesos 和 Marathon

Apache Mesos 是一个开源项目，最初的设计目的是使一组服务器或节点从外部看起来像一个单一的大服务器。Mesos 是一种使计算机集群管理变得简单的软件。Mesos 的用户不需要关心单个服务器，而是可以假设他们有一个巨大的资源池可以使用，这个资源池对应集群中所有节点的资源总和。

在 IT 术语中，Mesos 已经是相当古老的了，至少与其他容器编排工具相比是如此。它最早于 2009 年公开发布，但那个时候，当然并没有设计用来运行容器，因为当时 Docker 还没有出现。与 Docker 使用容器的方式类似，Mesos 使用 Linux cgroups 来隔离诸如 CPU、内存或磁盘 I/O 等资源，供单个应用程序或服务使用。

Mesos 其实是其他有趣服务的基础设施，这些服务是建立在它之上的。从容器的角度来看，Marathon 是一个重要的组成部分。Marathon 是一个容器编排工具，运行在 Mesos 之上，能够扩展到数千个节点。

Marathon 支持多种容器运行时，如 Docker 或它自己的 Mesos 容器。它不仅支持无状态应用程序，还支持有状态应用程序服务，例如 PostgreSQL 或 MongoDB 等数据库。类似于 Kubernetes 和 Docker SwarmKit，它支持本章早些时候提到的许多功能，如高可用性、健康检查、服务发现、负载均衡以及位置感知等几个最重要的功能。

尽管 Mesos 和在某种程度上 Marathon 已经是相对成熟的项目，但它们的应用范围相对有限。它们似乎在大数据领域最为流行，也就是说，主要用于运行数据处理服务，如 Spark 或 Hadoop。

# 亚马逊 ECS

如果你正在寻找一个简单的编排工具，并且已经深度融入了 AWS 生态系统，那么亚马逊的 ECS 可能是一个合适的选择。需要指出的是 ECS 的一个非常重要的限制：如果你选择了这个容器编排工具，那么你就将自己锁定在 AWS 环境中。你将无法轻松地将运行在 ECS 上的应用程序迁移到其他平台或云环境。

亚马逊将其 ECS 服务宣传为一个高度可扩展、快速的容器管理服务，使得在集群上运行、停止和管理 Docker 容器变得非常简单。除了运行容器，ECS 还可以直接访问许多 AWS 的其他服务，这些服务在容器内部的应用服务中运行。这种与众多流行 AWS 服务的紧密和无缝集成，使得 ECS 对于那些希望以简便的方式在一个强大且高度可扩展的环境中运行容器化应用的用户非常有吸引力。亚马逊还提供了自己的私有镜像注册中心。

使用 AWS ECS，你可以利用 Fargate 完全管理底层基础设施，从而让你可以专注于部署容器化应用程序，而不需要担心如何创建和管理节点集群。ECS 支持 Linux 和 Windows 容器。

总结来说，ECS 易于使用、具有高度可扩展性，并且与其他流行的 AWS 服务集成良好；但它不如 Kubernetes 或 Docker SwarmKit 那样强大，并且仅可在亚马逊 AWS 上使用。

# 微软 ACS

类似于我们对 ECS 的评价，我们也可以对微软的 ACS 做出相同的评价。它是一个简单的容器编排服务，如果你已经深度融入 Azure 生态系统，那么它是有意义的。我应该说出和我对 Amazon ECS 所指出的相同的内容：如果你选择了 ACS，你就将自己锁定在微软的服务中。从 ACS 将你的容器化应用程序迁移到任何其他平台或云环境将会非常困难。

ACS 是微软的容器服务，支持多个编排工具，如 Kubernetes、Docker Swarm 和 Mesos DC/OS。随着 Kubernetes 越来越受欢迎，微软的重点显然已经转向这个编排工具。微软甚至重新命名了它的服务，并称之为 **Azure Kubernetes Service**（**AKS**），以便将焦点放在 Kubernetes 上。

AKS 在 Azure 上为你管理托管的 Kubernetes、Docker Swarm 或 DC/OS 环境，这样你可以专注于你想要部署的应用程序，而无需关注基础设施的配置。微软用自己的话说，声明如下：

“AKS 使得快速且轻松地部署和管理容器化应用程序变得更加简单，即使没有容器编排的专业知识。它还通过按需提供、升级和扩展资源，消除了持续运维和维护的负担，而无需将你的应用程序下线。”

# 总结

本章展示了为什么首先需要编排器，以及它们的概念性工作原理。指出了在撰写时最为突出的编排器，并讨论了不同编排器之间的主要共同点和差异。

下一章将介绍 Docker 的原生编排器 SwarmKit。它将详细阐述 SwarmKit 使用的所有概念和对象，这些概念和对象用于在集群中（无论是本地还是云端）部署和运行一个分布式、弹性、稳健且高可用的应用程序。

# 问题

回答以下问题，以评估你的学习进度：

1.  我们为什么需要一个编排器？提供两到三个理由。

1.  列举三到四个编排器的典型职责。

1.  至少列举两个容器编排器，以及它们背后的主要赞助商。

# 深入阅读

以下链接提供了一些关于编排相关主题的更深入见解：

+   Kubernetes—生产级编排：[`kubernetes.io/.`](https://kubernetes.io/)

+   Docker Swarm 模式概述：[`docs.docker.com/engine/swarm/.`](https://docs.docker.com/engine/swarm/)

+   Marathon，一个为 Mesos 和 DC/OS 提供容器编排的平台：[https://](https://mesosphere.github.io/marathon/)[mesosphere.github.io/marathon/](https://mesosphere.github.io/marathon/)

+   容器和编排的解释：[`bit.ly/2DFoQgx.`](https://bit.ly/2npjrEl)
