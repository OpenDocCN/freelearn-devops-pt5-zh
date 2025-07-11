# 前言

本书的标题是《面向网络的 DevOps》。你可能已经非常了解，DevOps 是“开发（Development）”和“运维（Operations）”的缩写，那么它为什么与网络有关系呢？的确，DevOps 的名称中并没有“Net”，尽管可以说，DevOps 的职责范围已经远远超出了最初的目标。

最初的 DevOps 运动旨在消除开发与运营团队之间的“把问题扔过墙”的被动心态，但 DevOps 可以有效地用于促进 IT 所有团队之间的协作，而不仅仅是开发和运营人员。

DevOps 作为一个概念，最初旨在解决开发人员编写代码、做出重大架构更改时，不考虑需要将代码部署到生产环境的运营团队的情况。所以，当运营团队准备将开发人员的代码变更部署到生产环境时，这通常会导致部署失败，意味着关键的软件修复或新产品无法按计划到达客户，部署过程通常需要几天甚至几周才能修复。

这导致了所有相关团队的挫败感，因为开发人员不得不停止编写新功能，而是要帮助运营人员修复部署过程。运营团队也会感到沮丧，因为他们通常没有被告知部署新版本到生产环境时需要对基础设施做出更改。因此，运营团队没有准备好生产环境，以支持架构变更。

这一常见的 IT 场景突出了一个断裂的流程和操作模型，这种情况不断发生，并在开发和运营团队之间引发摩擦。

“DevOps”是一个旨在促进之前有冲突的团队之间的合作与沟通的倡议。它鼓励团队每天进行沟通，彼此了解变更，从而避免可避免的情况发生。因此，恰恰是开发和运营人员是 DevOps 最早试图解决的孤岛。结果，DevOps 被标榜为一种将团队统一为一个流畅的整体功能的方式，但它本来也可以叫别的名字。

DevOps 旨在改善团队之间的工作关系，并创造一个更愉快的工作环境，因为坦白说，没有人喜欢每天处理冲突或修复可以避免的问题。它还旨在促进团队之间的知识共享，以防止开发团队被视为“对基础设施一无所知”，而运营团队则被视为“变更的障碍”和“拖慢开发进度”。这些是各个部门孤立工作时常见的误解，因为他们没有花时间去理解彼此的目标。

DevOps 努力构建一个团队之间彼此欣赏、尊重共同目标的办公环境。DevOps 无疑是当今 IT 行业讨论最多的话题之一。它的流行与敏捷软件开发的兴起并非偶然，因为敏捷软件开发是作为瀑布式方法的替代方案出现的。

瀑布式开发和“V 模型”包括了分析、设计、实现（编码）和测试的不同阶段。这些阶段传统上被划分为不同的孤立团队，并设有正式的项目交接日期，这些日期是固定不变的。

敏捷诞生于意识到在快节奏的软件行业中，长期运行的项目并不是最佳选择，不能为客户提供真正的价值。因此，敏捷推动项目进入更短的迭代周期，将分析、设计、实现和测试纳入两周一轮的周期（冲刺），并采用原型化方法。

原型化方法采用增量开发的概念，使得公司能够在发布周期的早期收集产品反馈，而不是使用“大爆炸”方法，在实施阶段结束时一次性交付完整解决方案。

在瀑布式实施阶段结束时以瀑布方式交付项目，存在交付客户不需要的产品或开发人员误解需求的风险。这些问题通常仅在最终测试阶段被发现，此时项目已经准备好推向市场。这常常导致项目被认为是失败，或者出现巨大的延迟，而此时可以进行昂贵的返工和变更请求。

敏捷软件开发大部分时间推动了团队之间的孤岛现象的打破，这种孤岛现象通常与瀑布式软件开发相关，并且加强了每天进行协作的需求。

随着敏捷软件开发的引入，软件测试的方式也发生了变化，DevOps 的原则同样被应用于测试功能。

质量保证测试团队再也不能只是被动应对，就像之前的运营团队一样。因此，这促使测试团队需要更高效地工作，不再拖延产品进入市场的时间。然而，这不能以牺牲产品质量为代价，因此他们需要找到一种方式，确保应用程序得到充分测试，并通过所有质量保证检查，同时以更智能的方式工作。

已广泛接受质量保证测试团队不能再与开发团队分开孤立运作；相反，敏捷软件开发推动了测试用例与软件开发并行编写，因此它们不再是一个独立的活动。这与将代码部署到测试环境中，并交给测试团队执行手动测试或运行一组测试包来被动处理问题的做法截然不同。

敏捷方法促进了开发人员和质量保证测试人员每天在敏捷团队中共同工作，在软件打包部署之前进行测试，之后这些相同的测试将得到维护、更新，并用于生成回归测试包。

这已被用来减轻由于开发人员提交的代码更改破坏了质量保证测试团队的回归测试包所引起的摩擦。在独立的测试团队中，一个常见的摩擦场景是开发人员突然更改了图形用户界面（GUI），导致部分回归测试失败。这个更改没有通知测试团队。测试失败是因为它们是为旧版 GUI 编写的，而突然变得过时，而不是因为开发人员实际上引入了软件故障或 bug。

这种反应式的测试方法没有建立对自动化测试包报告的测试失败有效性的信心，因为这些失败并不总是完全由于软件故障，而且由于 IT 结构的次优，导致了不必要的延迟。

如果开发和测试团队之间的沟通更好，运用 DevOps 提倡的原则，那么这些延迟和次优的工作方式是可以避免的。

最近，我们看到了 DevSecOps 的兴起，它通过将安全性和合规性集成到软件交付过程中，取代了手动操作和分离的反应性举措。DevSecOps 借助 DevOps 和敏捷哲学，并采用了将安全工程师嵌入到敏捷团队中的想法，以确保在项目开始时就满足安全要求。

这意味着安全性和合规性可以作为自动化阶段集成到持续交付管道中，在每次软件发布时运行安全合规检查，并且不会延缓开发人员的软件开发生命周期，同时生成必要的反馈循环。

因此，网络团队也可以从 DevOps 方法论中学习，就像开发、运维、质量保证和安全团队一样。这些团队已经利用敏捷流程来改善与软件开发过程的互动，并从使用反馈循环中受益。

网络工程师有多少次不得不推迟软件发布，因为需要实施网络变更，而这些变更通过与其他部门流程不对接的票务系统低效地进行？网络工程师手动实施的网络变更又有多少次导致生产服务中断？这并不是对网络团队或网络工程师能力的批评，而是认识到运营模型需要改变，并且他们有能力做到这一点。

# 本书内容概览

本书将探讨如何提高网络变更的效率，以免影响软件开发生命周期。它将帮助概述网络工程师可以采用的自动化网络操作策略。我们将重点讨论如何建立成功的网络团队，使其能够在一个自动化驱动的环境中协作工作，从而提高效率。

本书还将展示网络团队需要建立新的技能，学习如 Ansible 这样的配置管理工具，以帮助他们实现这一目标。书中将展示这些工具带来的优势，使用它们提供的网络模块，并将帮助简化自动化过程，成为一个自我启动的指南。

我们将重点讨论需要克服的一些文化挑战，以便影响和实施网络功能的自动化流程，并说服网络团队充分利用目前供应商提供的可信网络 API。

本书将讨论公有云和私有云，如 AWS 和 OpenStack，以及它们如何为用户提供网络服务。还将讨论软件定义网络解决方案的出现，如 Juniper Contrail、VMWare NSX、CISCO ACI，并重点介绍诺基亚 Nuage VSP 解决方案，旨在将网络功能变成自服务商品。

本书还将重点讲解如何将持续集成和交付流程以及部署管道应用于网络变更的管理。还将展示如何将单元测试应用于自动化网络变更，以便将其与软件交付生命周期集成。

本书的详细章节概述如下：

第一章，*云对网络的影响*，将讨论公有云的 AWS 和私有云的 OpenStack 的出现如何改变了开发者对网络的需求。它将探讨 AWS 和 OpenStack 提供的一些即插即用的网络服务，并展示它们提供的部分网络功能。书中将举例说明这些云平台如何将网络变成像基础设施一样的商品。

第二章，*软件定义网络的出现*，将讨论软件定义网络如何出现。它将探讨方法论，并重点介绍 AWS 和 OpenStack 提供的开箱即用体验之上，软件定义网络所带来的一些扩展性优势和特性。它将展示市场领先的 SDN 解决方案之一 Nuage 如何应用这些概念和原理，并讨论市场上其他 SDN 解决方案。

第三章，*将 DevOps 引入网络运营*，将详细说明自上而下和自下而上的 DevOps 网络倡议的利弊。它将为读者提供一些成功的策略和通常失败的策略的思考。此章节将帮助首席技术官（CTO）、高级经理和工程师，尤其是那些试图在公司网络部门启动 DevOps 模型的人，概述他们可以使用的不同策略，以实现他们期望的文化变革。

第四章，*使用 Ansible 配置网络设备*，将概述使用配置管理工具安装和推送配置到网络设备的好处，并讨论目前可用的一些开源网络模块及其工作原理。它将举例说明可以采用的一些流程，以维持设备配置。

第五章，*使用 Ansible 编排负载均衡器*，将描述使用 Ansible 编排负载均衡器的好处，以及在无需停机或人工干预的情况下将新软件版本推送到服务中的方法。它将举例说明可以采用的一些流程，以便编排不可变和静态服务器，探讨不同的负载均衡技术。

第六章，*使用 Ansible 编排 SDN 控制器*，将概述使用 Ansible 编排 SDN 控制器的好处。它将概述软件定义网络的好处，以及为什么自动化 SDN 控制器暴露的网络功能至关重要。这包括动态设置 ACL 规则，允许网络工程师提供网络即服务（NaaS），使开发人员能够自助满足其网络需求。它将讨论部署策略，如蓝绿网络，并探索一些可以用来实现 NaaS 方法的流程。

第七章，*使用持续集成构建进行网络配置*，将讨论如何转向一种将网络配置存储在源代码管理系统中的模式，以便于审计、版本控制和回滚变更。

它将探讨使用如 Jenkins 和 Git 等工具设置网络配置 CI 构建的工作流。

第八章，*测试网络变更*，将概述在将网络变更应用到生产环境之前，使用测试环境进行测试的重要性。它将探讨一些可用的开源工具，并详细介绍一些可以应用的测试策略，确保网络变更在应用到生产环境之前经过充分测试。

第九章，*使用持续交付流水线部署网络变更*，将展示读者如何使用持续集成和持续交付流水线将网络变更交付到生产环境，并通过相关测试环境进行验证。它将给出一些可以采用的流程示例，说明如何将网络变更轻松地与基础设施和代码变更一起放入部署流水线。

第十章，*容器对网络的影响*，专用容器技术，如 Docker，以及容器编排引擎，如 Kubernetes 和 Swarm，越来越受到迁移到微服务架构的公司的青睐。因此，这改变了网络需求。本章将探讨容器的运作方式及其对网络的影响。

第十一章，*网络安全*，将探讨这种方法如何使安全工程师在审计网络时的工作变得更加轻松。它将分析软件定义网络中的可能攻击路径，以及如何将安全检查集成到 DevOps 模型中。

# 本书所需的内容

本书假设读者具备中级的网络知识、基础的 Linux 知识、基础的云计算技术知识以及广泛的 IT 知识。重点主要放在可以实施的特定流程工作流上，而非基础技术，因此这些思想和内容可以应用于任何组织，无论使用何种技术。

然而，尽管如此，在消化某些章节内容时，读者访问以下技术可能会有所帮助：

+   AWS [`aws.amazon.com/free/`](https://aws.amazon.com/free/)

+   OpenStack [`trystack.org/`](http://trystack.org/)

+   Nuage VSP [`nuagex.io/`](http://nuagex.io/)

# 本书适合的人群

本书的目标读者是希望自动化其工作中手动和重复部分的网络工程师，或者希望自动化所有网络功能的开发者和系统管理员。

本书还将为 CTO 或经理提供有价值的见解，帮助他们了解如何使其网络部门更加敏捷，并在其组织内部发起真正的文化变革。

本书还将帮助那些希望了解更多关于 DevOps、持续集成和持续交付的人，并了解它们如何应用于现实世界场景，以及一些可用的工具，以便于自动化。

# 约定

本书中有多种文本样式，用于区分不同类型的信息。以下是这些样式的几个例子及其含义的解释。

文本中的代码词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 用户名将如下所示：“这些服务随后将绑定到`lbvserver`实体。”

任何命令行输入或输出都将如下所示：

```
ansible-playbook  -I inevntories/openstack.py  -l qa  -e environment=qa  -e current_build=9 playbooks/add_hosts_to_netscaler.yml

```

**新术语**和**重要词汇**以粗体显示。你在屏幕上看到的词语，例如在菜单或对话框中，出现在文本中如下：“点击 Google 上的**搜索**按钮。”

### 注意

警告或重要说明将以这样的框显示。

### 提示

提示和技巧如下所示。

# 读者反馈

我们始终欢迎读者的反馈。让我们知道你对本书的看法——你喜欢或不喜欢什么。读者反馈对我们非常重要，因为它帮助我们开发出你能从中获得最大收益的书籍。

如需发送一般反馈，请直接发送电子邮件至`<feedback@packtpub.com>`，并在邮件主题中注明书名。如果你对某个主题有专长，并有兴趣撰写或参与编写书籍，请参考我们的作者指南：[www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

既然你已经成为 Packt 书籍的自豪拥有者，我们提供了多项帮助你最大化利用购买的内容。

## 下载本书的彩色图片

我们还为你提供了一个 PDF 文件，其中包含本书中使用的截图/图表的彩色图像。这些彩色图像将帮助你更好地理解输出的变化。你可以从[`www.packtpub.com/sites/default/files/downloads/DevOpsforNetworking_ColorImages.pdf`](http://www.packtpub.com/sites/default/files/downloads/DevOpsforNetworking_ColorImages.pdf)下载该文件。

## 勘误

尽管我们已尽力确保内容的准确性，但错误仍然可能发生。如果您在我们的书籍中发现错误——可能是文本或代码中的错误——我们将非常感激您向我们报告。通过这样做，您不仅能为其他读者避免困惑，还能帮助我们改进后续版本的书籍。如果您发现任何勘误，请通过访问[`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)，选择您的书籍，点击**勘误提交表格**链接，并填写勘误的详细信息。一旦您的勘误得到验证，我们将接受您的提交，并将勘误上传到我们的网站，或将其添加到该书标题下的现有勘误列表中。

若要查看之前提交的勘误，请访问[`www.packtpub.com/books/content/support`](https://www.packtpub.com/books/content/support)并在搜索框中输入书名。所需信息将显示在**勘误**部分。

## 盗版

网络上的版权盗版问题在各类媒体中普遍存在。在 Packt，我们非常重视版权和许可证的保护。如果您在互联网上发现我们作品的任何非法复制品，请立即向我们提供该位置地址或网站名称，以便我们采取相应措施。

如果您发现涉嫌盗版的内容，请通过`<copyright@packtpub.com>`与我们联系，并提供盗版内容的链接。

我们感谢您在保护我们的作者和帮助我们为您提供有价值内容方面的帮助。

## 问题

如果您对本书的任何部分有问题，可以通过`<questions@packtpub.com>`与我们联系，我们将尽最大努力解决问题。
