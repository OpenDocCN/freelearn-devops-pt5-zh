# 附录 B – 模拟测试问题

回答以下问题，以测试您对本章信息的理解。您可以在本书末尾的*模拟测试答案*部分找到答案。

**第一章，与 Azure 虚拟机一起工作**

1.  您需要为开发环境推荐合适的策略。您会选择哪个虚拟机系列？

    1.  F 系列

    1.  B 系列

    1.  L 系列

    1.  N 系列

1.  您有一组部署在虚拟机上的应用程序。您需要确保在虚拟机更新时应用程序仍然可以访问。您应该怎么办？

    1.  为每个应用程序创建一个可用性集

    1.  实现自动扩展

    1.  创建资源组

    1.  创建一个具有两个或更多虚拟机的可用性集

1.  当将虚拟机放入可用性集时，Azure 提供多少故障域？

    1.  2

    1.  4

    1.  5

    1.  7

1.  当在 Azure 中配置 Azure 虚拟机时，它是使用托管磁盘创建的吗？

    1.  是

    1.  否

1.  如何部署虚拟机规模集？

    1.  使用 Azure 门户

    1.  使用 CLI

    1.  使用 PowerShell

    1.  使用 ARM 模板

1.  部署后，您是否可以通过公共 IP 地址直接使用 SSH 或 RDP 访问虚拟机规模集中的虚拟机？

    1.  是

    1.  否

1.  您将 Azure 虚拟机部署到东美国区域。您需要设计一个解决方案，以便在数据中心故障时尽可能快速地恢复。您将虚拟机部署到哪个区域？

    1.  东美国 2

    1.  西欧洲

    1.  西美国

    1.  西美国 2

**第二章，配置计算密集型应用程序**

1.  您正在设计一个图形密集型应用程序。您会选择哪个虚拟机系列？

    1.  B 系列

    1.  N 系列

    1.  A 系列

    1.  G 系列

1.  您的组织希望每年只运行一次密集型科学计算。最便宜的解决方案是什么？

    1.  Azure 批量处理

    1.  HPC Pack

    1.  Azure 虚拟机

    1.  Azure HDInsight

1.  您的公司需要运行大规模计算工作负载。实施的核心要求是从一个中心点（例如管理环境）添加和删除计算节点。您建议选择什么方案？

    1.  使用 Azure 批量处理

    1.  使用 Azure 虚拟机

    1.  使用 Azure VM H 系列

    1.  使用 HPC Pack

**第三章，设计 Web 应用程序**

1.  CDN 的好处是什么？

    1.  减少响应时间

    1.  减少流量到原始服务

    1.  启用更快的升级

    1.  提高安全性

1.  以下哪些编程语言是 Azure Web 应用支持的？

    1.  .NET Core

    1.  Java

    1.  Node.js

    1.  Python

1.  Azure 流量管理器的可用路由方法有哪些？

    1.  地理位置

    1.  扩展

    1.  缓存

    1.  优先级

1.  您正在创建一个使用 Redis Cache 的应用程序。您应该推荐哪个定价层？您会推荐哪个？

    1.  基础

    1.  标准

    1.  高级

    1.  免费

1.  您正在创建一个自定义应用程序，要求用户通过 Facebook 帐户登录。您会选择哪种解决方案？

    1.  Azure AD B2C

    1.  Microsoft Graph

    1.  Azure Active Directory Connect

    1.  Azure AD B2B

**第四章，实现无服务器和微服务**

1.  您正在设计一个需要触发 BizTalk 服务器电子数据交换（EDI）服务的应用程序。哪种技术实现最适合该功能？

    1.  Azure API 管理

    1.  Azure 逻辑应用

    1.  Azure Functions

    1.  Azure 容器实例

1.  Azure Service Fabric 为你的微服务应用提供完整的生命周期支持。Azure Service Fabric 提供哪些角色？

    1.  服务开发者

    1.  应用开发者

    1.  操作员

    1.  贡献者

1.  什么是 API 管理？

    1.  Visual Studio 2017 的一个附加组件，允许你将 API 部署到 Azure

    1.  用于自定义 API 的 API 网关

    1.  用于管理 Azure 逻辑应用的工具

    1.  用于容器的编排工具

**第五章，强大的网络实现**

1.  Azure 支持哪些 VPN 类型？

    1.  VNet 对 VNet

    1.  站点到站点

    1.  点对站点

    1.  OpenVPN

1.  ExpressRoute 最多能提供多少带宽？

    1.  100 Mbps

    1.  1 Gbps

    1.  10 GBps

    1.  2 Gbps

1.  你正在设计一个需要面向全球成千上万用户的应用。对于位于远程地区的用户，你有性能和响应时间的担忧。你应该怎么做？

    1.  设置 VPN 网关

    1.  在多个区域部署应用

    1.  部署流量管理器

    1.  部署负载均衡器

1.  你有两台虚拟机，VM1 和 VM2。所有外部流量必须通过 VM1 路由。你应该使用什么解决方案？

    1.  DNS

    1.  系统路由

    1.  用户定义路由

    1.  网络安全组

**第六章，连接混合应用**

1.  你想将本地应用连接到云，但不允许打开防火墙端口。你应该使用哪个 Azure 服务？

    1.  Azure Active Directory 域服务

    1.  Azure 数据工厂

    1.  Azure 数据管理网关

    1.  Azure Relay 服务

1.  你的应用部署在一个本地虚拟机上，你希望在应用中使用 Azure AD 凭证。实现这一目标的最简单方法是什么？

    1.  设置混合环境并使用 Azure AD Connect 将用户账户和凭证与云同步

    1.  配置 Azure AD 域服务

    1.  使用 Azure 本地数据网关

    1.  使用 Azure VNet 集成

**第七章，使用存储解决方案**

1.  你正在开发一个应用，需要存储 Microsoft Word 和 Excel 文档。你选择哪种存储类型？

    1.  Blob 存储

    1.  StorageV2

    1.  存储

    1.  表存储

1.  你正在为一个跨国公司开发应用，需要存储数据集。你选择哪种复制类型？

    1.  地理冗余

    1.  区域冗余

    1.  本地冗余

1.  你的组织使用一个本地的 SharePoint 2013 农场，存储了大量归档数据，这些数据存储在 SAN 存储上，并且达到了当前设备的存储限制。他们需要一个更具成本效益的存储选项来存储未来的归档数据。你会推荐什么？

    1.  在本地数据中心添加新的 SAN 存储设备

    1.  使用 StorSimple 虚拟阵列

    1.  使用 Office 365 和 SharePoint Online 存储云中的数据

    1.  使用 StorSimple 8000 系列

1.  你想为一个金融应用使用 CosmosDB。哪个 API 最适合？

    1.  图形 API

    1.  SQL API

    1.  Cassandra API

    1.  DocumentDB API

1.  您计划在组织内部署 Azure 搜索，搜索公司网站、移动应用程序和其他应用程序中的内容。您有以下需求：存储最多 1 亿个文档，支持多租户场景，并支持超过 1,000 个索引。您选择哪个服务层级？

    1.  标准 S1

    1.  标准 S2

    1.  标准 S3

    1.  标准 HD

**第八章，可扩展的数据实现**

1.  您有一个本地 SQL Server 2012 数据库，并希望尽快进行迁移。您需要推荐一个迁移解决方案。您选择什么？

    1.  Stretch Database

    1.  Azure MySQL 数据库

    1.  Mongo DB

    1.  Azure SQL 数据库

1.  您想为 Azure SQL 数据库实施地理复制。您可以在哪里配置此项？

    1.  PowerShell

    1.  Transact SQL

    1.  REST API

    1.  Azure 门户

1.  Azure SQL 数据库备份有哪些恢复场景类型？

    1.  时间点恢复

    1.  标准恢复

    1.  已删除数据库恢复

    1.  地理恢复

1.  您有五个使用标准服务层级的 Azure SQL 数据库。您正在为这些数据库设计备份策略，备份必须保存至少 5 年，以符合法律要求。数据库必须能够恢复到特定时间点。您应该怎么做？

    1.  将服务层级从标准更改为高级

    1.  配置 Azure 恢复服务并添加长期备份保留策略

    1.  配置地理恢复

    1.  导出自动化备份并将其存储为 Blob

1.  您正在将本地 SQL Server 数据库迁移到 Azure SQL 数据库，数据库大约有 100 GB 的数据。您有以下需求：99.99% 的正常运行时间保证，支持多个并发查询，并尽量减少成本。您应该选择哪个服务层级和级别？

    1.  标准 S1

    1.  标准 S2

    1.  高级 P1

    1.  高级 P2

**第九章，保护您的资源**

1.  您正在创建一个托管在 Azure 上的移动应用程序，外部用户可以使用他们的 LinkedIn 凭据登录。您使用哪个 Azure 服务？

    1.  Microsoft Graph

    1.  Azure B2B

    1.  Azure AD REST API

    1.  Azure B2C

1.  如何在您的移动应用程序中实现这一点？

    1.  通过添加第三方认证 SDK

    1.  通过将内置策略添加到 web.config

    1.  通过在 Azure Active Directory 中注册应用程序

    1.  通过在 Visual Studio 2017 中创建新解决方案

1.  您为一家希望将本地 AD 与 Azure AD 集成的组织工作。一个要求是他们可以继续使用当前的登录页面，该页面具有自定义通知和帮助台信息。您建议使用什么解决方案？

    1.  部署 Azure AD Connect

    1.  使用 Azure AD 密码同步

    1.  部署 ADFS

    1.  将当前的活动目录迁移到 Azure 虚拟机

1.  您的组织希望在 Office 365 中向外部用户提供受控访问，并希望让外部管理员管理这些用户。最佳解决方案是什么？

    1.  在 Office 365 中启用外部共享，让用户决定与谁共享文档

    1.  部署 ADFS

    1.  使用 Azure B2B

    1.  使用 Azure B2C

**第十章，保护您的数据**

1.  你想在存储帐户上启用 Azure 存储服务加密。哪个存储类型将自动加密？

    1.  Blob

    1.  文件

    1.  队列

    1.  表格

1.  你想加密虚拟机的数据磁盘。你会使用哪项技术？

    1.  Azure 存储服务加密

    1.  始终加密

    1.  Azure 密钥保管库

    1.  Azure 磁盘加密

1.  什么是始终加密？

    1.  存储证书的位置

    1.  数据传输中的数据库设置

    1.  数据使用中的数据库设置

    1.  加密和解密数据的密钥

1.  你正在设计一个新应用程序，出于安全原因，你不希望在代码和 web.config 文件中存储任何凭据。你会选择哪项技术？

    1.  Azure Active Directory 托管服务身份

    1.  Azure 密钥保管库

    1.  透明数据加密

    1.  应用服务主体

**第十一章，治理和政策**

1.  你需要授予一组管理员有限的访问权限。其中两个人需要管理 Azure 订阅的权限。你会将管理员添加到哪个角色？

    1.  贡献者

    1.  管理员

    1.  阅读者

    1.  拥有者

1.  你正在 Azure 安全中心监控你的计算资源。显示了一个安装端点保护的建议。这个保护提供了什么类型的防护？

    1.  针对 API 端点的反恶意软件保护

    1.  针对虚拟机的反恶意软件保护

    1.  针对 API 端点的身份保护

    1.  针对虚拟机的身份保护

1.  你组织中有几个用户需要临时访问 Exchange Online 的管理员资源。然而，从安全角度考虑，你不希望他们被添加到 Exchange 管理员角色中。那么你建议使用什么解决方案？

    1.  Azure Active Directory 托管服务身份

    1.  将他们添加到 RBAC 中的贡献者角色

    1.  使用 Azure AD 特权身份管理

    1.  将他们添加到拥有者角色，并在用户完成后手动移除他们

1.  你在一家全球性公司工作，该公司正在结合来自不同云提供商的解决方案，并拥有多个本地数据中心。你需要从一个解决方案中监控和保护所有这些环境。你建议使用哪种解决方案？

    1.  Azure 安全中心

    1.  操作管理套件 安全与合规性

    1.  Azure AD 身份保护

    1.  高级威胁防护

1.  你需要授予一组管理员有限的访问权限。其中一个管理员只需要重新启动虚拟机的权限。你会将该管理员添加到哪个角色？

    1.  创建自定义角色

    1.  阅读者

    1.  拥有者

    1.  贡献者

**第十二章，人工智能、物联网和媒体服务**

1.  你正在创建一个网站，用户可以上传文化地标的照片。这些图片需要根据图像自动分类。你会使用哪个认知服务 API？

    1.  面部 API

    1.  计算机视觉 API

    1.  AMS 媒体分析

    1.  LUIS API

1.  你正在设计一个用于收集传感器设备生成的大量数据的解决方案。数据需要在本地网络内进行处理。你会选择哪种解决方案？

    1.  Azure 时间序列分析

    1.  Azure IoT Hub

    1.  Azure Event Hubs

    1.  Azure IoT Edge

1.  你正在设计一个解决方案，用于处理由各种 IoT 设备生成的每秒数百万个事件。你会选择哪种解决方案？

    1.  Azure Time Series Analytics

    1.  Azure IoT Hub

    1.  Azure Event Hubs

    1.  Azure IoT Edge

1.  你正在设计一个使用 Azure IoT Hub 的解决方案，每天处理超过 1 亿个事件。你会选择哪个定价层？

    1.  Standard S1

    1.  Standard S2

    1.  Standard S3

    1.  Standard S4

1.  你正在设计一个解决方案，需要将数据从 IoT 设备传输到 Azure。你需要选择最具成本效益的解决方案。你会选择哪种解决方案？

    1.  Azure IoT Hub

    1.  Azure IoT Edge

    1.  Azure Functions

    1.  Azure Event Hubs

**第十三章，实现消息传递解决方案**

1.  你正在设计一个应用程序，具有以下要求：用户界面层需要与多个处理层解耦。消息大小约为 1 MB。这是一个具有未来扩展要求的应用程序，其中一些处理层只会接收消息的子集。你会选择哪种解决方案？

    1.  Azure Service Bus Queues

    1.  Azure Service Bus Topics

    1.  Azure Queue Storage

    1.  Azure Functions

1.  你正在设计一个解决方案，可以监控并响应设备事件。你希望能够将设备事件广播到多个端点，并创建自定义主题和 Azure Functions。你会选择哪种解决方案？

    1.  Azure Event Grid

    1.  Azure IoT Hub

    1.  Azure Event Hubs

    1.  Azure IoT Edge

1.  你正在设计一个应用程序，具有以下要求：用户界面层需要与处理层解耦。消息大小不超过 50 KB。这是一个小型应用程序，只有一个处理层，且未来无需升级。你会选择哪种解决方案？

    1.  Azure Service Bus Queues

    1.  Azure Service Bus Topics

    1.  Azure Queue Storage

    1.  Azure Functions

**第十四章，应用监控和告警策略**

1.  你正在创建一个监控和告警解决方案，监控你的 Azure 资源。你有以下要求：当虚拟机被删除时发送告警，并将事件写入 Event Hubs 和 Log Analytics。你会使用哪种解决方案？

    1.  Azure Security Center

    1.  Azure Health

    1.  Azure Activity Log

    1.  Azure Application Insights

1.  你正在创建一个监控和告警解决方案，管理员希望查询 Azure 订阅中的所有可用日志。你会使用哪种解决方案？

    1.  Azure Security Center

    1.  Azure Log Analytics

    1.  Azure Health

    1.  Azure Network Watcher

1.  你正在创建一个 Web 应用程序，可以托管在本地或 Azure 的 Docker 容器中。你希望获得更多关于页面查看和 API 调用的洞察。你会使用哪种监控解决方案？

    1.  Azure Application Insights

    1.  Azure Service Health

    1.  Azure Activity Log

    1.  Azure Network Watcher

1.  你想了解微软在 Azure 资源上进行维护活动的时间。哪种监控解决方案能提供此类信息？

    1.  Azure Application Insights

    1.  Azure Service Health

    1.  Azure Activity Log

    1.  Azure Network Watcher

**第十五章，探索运维自动化策略**

1.  你正在设计标准化的虚拟机部署和配置。你需要发布一个配置库，用于自动设置新虚拟机上的角色和功能安装。你应该选择哪种解决方案作为自动化策略？

    1.  Chef

    1.  Puppet

    1.  图形化运行簿

    1.  Azure PowerShell Desired State Configuration (DSC)

1.  你需要设计一个策略，在晚上 7 点关闭所有虚拟机，并在早上 8 点重新启动它们。解决方案应完全在 Azure 中运行。你会选择哪种解决方案？

    1.  Chef

    1.  Azure Automation

    1.  Puppet

    1.  作为虚拟机上计划任务配置的 PowerShell 脚本。

1.  你的公司使用 Windows、Linux 和 Mac 设备。你正在设计一种自动化策略。你会选择哪种解决方案？

    1.  PowerShell

    1.  Azure Automation

    1.  Chef

    1.  Puppet
