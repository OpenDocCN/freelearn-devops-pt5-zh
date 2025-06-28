# 第二十章：DevSecOps - 挑战、提示与常见问题

DevSecOps 的采用是一个持续学习的过程，需要大量利益相关者的参与、流程优化、业务优先级冲突、安全工具的定制，以及安全知识的学习曲线。本章的目的是从功能角色的角度为您提供一些实用的提示、挑战和常见问题解答。

本章将涵盖以下主题：

+   DevSecOps 常见问题（适用于安全管理）

+   DevSecOps 常见问题（适用于开发团队）

+   DevSecOps 常见问题（适用于测试团队）

+   DevSecOps 常见问题（适用于运营团队）

# DevSecOps 适用于安全管理

**问：是否有针对 DevOps 的安全开发和部署的行业最佳实践？**

OWASP SAMM（软件保障成熟度模型）、微软安全开发生命周期（SDL）和 SafeCode 为 DevOps 或敏捷开发提供了实际的安全实践。

+   OWASP SAMM: [`github.com/OWASP/samm`](https://github.com/OWASP/samm)

+   Microsoft SDL for Agile: [`www.microsoft.com/en-us/SDL/Discover/sdlagile.aspx`](https://www.microsoft.com/en-us/SDL/Discover/sdlagile.aspx)

+   SafeCode: [`safecode.org/publications/`](https://safecode.org/publications/)

**问：云服务的安全风险有哪些？**

CSA（云安全联盟）在其网站上定义了云计算的主要威胁（[`cloudsecurityalliance.org/group/top-threats/`](https://cloudsecurityalliance.org/group/top-threats/)），如下所示：

+   数据泄露

+   身份、凭证和访问管理不足

+   不安全的接口和 API

+   系统漏洞

+   账户劫持

+   恶意内部人员

+   高级持续性威胁

+   数据丢失

+   不足的尽职调查

+   云服务的滥用与恶意使用

+   服务拒绝

+   共享技术漏洞

**问：在 GDPR 合规性方面，安全要求是什么？**

下表列出了数据处理者和数据控制者的软件/服务的 GDPR 安全要求：

| **GDPR 要求** | **数据处理者** | **数据控制者** |
| --- | --- | --- |
| 提供数据隐私声明 | 必须 | 必须 |
| 数据收集需要用户的明确同意，允许数据收集并允许用户禁用数据收集 | 必须 | 必须 |
| 为了故障排除，用户必须被告知日志收集是否包括个人信息 | 必须 | 必须 |
| 收集用户的 cookie 需要用户的同意。有关详细信息，请参见 [`www.cookielaw.org/the-cookie-law/`](https://www.cookielaw.org/the-cookie-law/) | 必须 | 必须 |
| 如果数据用于市场分析目的，应用程序必须允许用户禁用分析 | 推荐 | 必须 |
| 提供在数据过期后安全删除数据的能力 | 必须 | 必须 |
| 如果数据将提供给第三方合作伙伴，则必须获得用户的明确同意 | 推荐 | 必须 |
| 提供用户查询和更新数据的能力 | 推荐 | 必须 |
| 删除任何不再使用的临时数据 | 推荐 | 必须 |
| 提供导出数据的能力 | 推荐 | 必须 |
| 安全数据传输 | 必须 | 必须 |
| 使用加密、访问控制和日志安全控制的安全本地数据存储 | 必须 | 必须 |

# DevSecOps 适用于开发团队

**问：推荐的安全架构模式有哪些？**

+   开放安全架构模式: [`www.opensecurityarchitecture.org/cms/library/patternlandscape`](http://www.opensecurityarchitecture.org/cms/library/patternlandscape)

+   安全与隐私参考架构: [`security-and-privacy-reference-architecture.readthedocs.io/en/latest/index.html`](http://security-and-privacy-reference-architecture.readthedocs.io/en/latest/index.html)

+   Shiro: [`shiro.apache.org/`](http://shiro.apache.org/)

+   OWASP 备忘单系列: [`www.owasp.org/index.php/OWASP_Cheat_Sheet_Series`](https://www.owasp.org/index.php/OWASP_Cheat_Sheet_Series)

**问：构建安全软件时，常用的安全框架有哪些？**

| **安全改进领域** | **开源安全与隐私框架** |
| --- | --- |
| 认证 |

+   Gluu 用于多因素认证和社交登录

+   ReCAPTCHA

+   Git-Secret 用于保护源代码中的敏感信息

|

| 授权 |
| --- |

+   Gluu 用于用户同意管理

+   Apache Shiro 会话管理

+   OWASP CSRF 防护

|

| API 管理器 |
| --- |

+   Kong

+   API 伞形架构

+   WSO2 API 管理器

|

| 数据输入/输出 |
| --- |

+   OWASP HTML 消毒项目

+   Commons 验证器

+   ValidateJS

+   OWASP Java 编码器

|

| 隐私 |
| --- |

+   ARX De-Identifier 数据匿名化工具

+   Apache Atlas 用于数据治理

+   PrivacyScore 用于网页隐私评估

+   CookieConsent

|

**问：有很多第三方组件和依赖项与软件包一起发布和部署。是否有推荐的工具来评估这些安全风险？**

+   RetireJS:  [`retirejs.github.io/retire.js/`](https://retirejs.github.io/retire.js/)

+   OWASP 依赖检查: [`www.owasp.org/index.php/OWASP_Dependency_Check`](https://www.owasp.org/index.php/OWASP_Dependency_Check)

+   Cuckoo Sandbox: [`cuckoosandbox.org/`](https://cuckoosandbox.org/)

**问：在设计和编码阶段，推荐的安全交付物有哪些？**

| **阶段** | **交付物** |
| --- | --- |
| 需求 |

+   客户安全需求分析

+   安全标准合规性分析

+   安全行业最佳实践（如 OWASP ASVS、CSA CCM）

|

| 设计 |
| --- |

+   威胁建模分析报告

+   安全设计检查表自评报告

|

| 编码 |
| --- |

+   静态安全编码扫描报告

+   高风险模块安全评估报告

+   安全编译器和链接器标志状态

+   禁止或不安全的 API 使用扫描报告

|

**问：推荐的安全编码最佳实践资源有哪些？**

+   OWASP 安全代码审查：[`www.owasp.org/index.php/Category:OWASP_Code_Review_Project`](https://www.owasp.org/index.php/Category:OWASP_Code_Review_Project)

+   通用弱点枚举 (CWE)：[`cwe.mitre.org/data/index.html`](https://cwe.mitre.org/data/index.html)

+   CERT 安全编码：[`www.securecoding.cert.org/confluence/display/seccode/SEI+CERT+Coding+Standards`](https://www.securecoding.cert.org/confluence/display/seccode/SEI+CERT+Coding+Standards)

+   Android 安全编码：[`www.jssec.org/dl/android_securecoding_en.pdf`](https://www.jssec.org/dl/android_securecoding_en.pdf)

**问：用于缓解缓冲区溢出攻击的安全编译器和链接标志是什么？**

参考第八章，*安全编码最佳实践*中的安全编译表。

# 测试团队的 DevSecOps

**问：建议用于数据隐私评估的测试工具有哪些？**

| **数据生命周期** | **测试关键点** | **建议的测试工具** |
| --- | --- | --- |
| 数据传输 |

+   确保敏感信息不会通过 GET 方法传输

+   安全通信协议，如 TLS v1.2、SSH V2、SFTP、SNMP V3

| SSLyze、NMAP、Wireshark |
| --- |
| 数据存储 |

+   检查敏感信息是否加密

+   检查文件权限是否正确配置

| TruffleHog: [`github.com/dxa4481/truffleHog`](https://github.com/dxa4481/truffleHog) |
| --- |
| 数据加密 | 不使用弱加密算法，如 MD5、RC4、Jackfish 和 Tripple DES | 代码扫描工具：[`github.com/floyd-fuh/crass/blob/master/grep-it.sh`](https://github.com/floyd-fuh/crass/blob/master/grep-it.sh) |
| 数据访问和审计 |

+   记录任何敏感数据查询

+   CL 权限

| AuthMatrix: [`github.com/SecurityInnovation/AuthMatrix`](https://github.com/SecurityInnovation/AuthMatrix) |
| --- |
| 数据移除 |

+   检查临时文件、异常文件和 Cookies 中是否存在敏感信息

+   检查内存和缓存中的任何纯文本敏感信息

| GCOREWinHex: [`www.x-ways.net/winhex/`](https://www.x-ways.net/winhex/) LaZagne: [`github.com/AlessandroZ/LaZagne`](https://github.com/AlessandroZ/LaZagne) |
| --- |

参考第十章，*安全测试计划和实践*，获取更多详情。

**问：每个安全领域的行业安全测试指南是什么？**

| **安全领域** | **安全测试指南** |
| --- | --- |
| Web 安全测试 |

+   OWASP 测试指南：**[`www.owasp.org/index.php/OWASP_Testing_Project`](https://www.owasp.org/index.php/OWASP_Testing_Project)**

|

| 虚拟化安全测试 |
| --- |

+   NIST 800-125 虚拟化技术安全指南：[`csrc.nist.gov/publications/detail/sp/800-125/final`](https://csrc.nist.gov/publications/detail/sp/800-125/final)

+   PCI DSS 虚拟化指南：[`www.pcisecuritystandards.org/documents/Virtualization_InfoSupp_v2.pdf`](https://www.pcisecuritystandards.org/documents/Virtualization_InfoSupp_v2.pdf)

+   Red Hat 虚拟化安全指南：[`access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html-single/virtualization_security_guide/index`](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html-single/virtualization_security_guide/index)

+   SANS 虚拟化安全常见错误：[`www.sans.org/reading-room/whitepapers/analyst/top-virtualization-security-mistakes-and-avoid-them-34800`](https://www.sans.org/reading-room/whitepapers/analyst/top-virtualization-security-mistakes-and-avoid-them-34800)

+   ISCACA 虚拟化安全检查表：[`www.isaca.org/Knowledge-Center/Research/Documents/Virtualization-Security-Checklist_res_Eng_1010.pdf`](http://www.isaca.org/Knowledge-Center/Research/Documents/Virtualization-Security-Checklist_res_Eng_1010.pdf)

|

| 固件安全测试 |
| --- |

+   GitHub 精选固件安全：[`github.com/PreOS-Security/awesome-firmware-security`](https://github.com/PreOS-Security/awesome-firmware-security)

+   GitHub BIOS/UEFI 系统固件的安全性——从攻击者和防御者的视角：[`github.com/rmusser01/Infosec_Reference/blob/master/Draft/BIOS%20UEFI%20Attacks%20Defenses.md`](https://github.com/rmusser01/Infosec_Reference/blob/master/Draft/BIOS%20UEFI%20Attacks%20Defenses.md)

|

| 大数据安全测试 |
| --- |

+   NIST 1500-4 大数据互操作性框架：[`www.nist.gov/publications/nist-big-data-interoperability-framework-volume-4-security-and-privacy`](https://www.nist.gov/publications/nist-big-data-interoperability-framework-volume-4-security-and-privacy)

+   CSA 大数据安全与隐私手册：[`downloads.cloudsecurityalliance.org/assets/research/big-data/BigData_Security_and_Privacy_Handbook.pdf`](https://downloads.cloudsecurityalliance.org/assets/research/big-data/BigData_Security_and_Privacy_Handbook.pdf)

|

| 隐私 |
| --- |

+   GDPR 检查表：[`gdprchecklist.io/`](https://gdprchecklist.io/)

+   NIST SP 800-122 保护**个人身份信息**（**PII**）机密性的指南：[`csrc.nist.gov/publications/detail/sp/800-122/final`](https://csrc.nist.gov/publications/detail/sp/800-122/final)

|

| 物联网安全 |
| --- |

+   ENISA IoT 基准安全建议：[`www.enisa.europa.eu/publications/baseline-security-recommendations-for-iot/`](https://www.enisa.europa.eu/publications/baseline-security-recommendations-for-iot/)

+   GSMA 物联网安全评估：[`www.gsma.com/iot/future-iot-networks/iot-security-guidelines/`](https://www.gsma.com/iot/future-iot-networks/iot-security-guidelines/)

|

| 容器安全 |
| --- |

+   NIST 800-190 应用容器安全指南：[`nvlpubs.nist.gov/nistpubs/specialpublications/nist.sp.800-190.pdf`](https://nvlpubs.nist.gov/nistpubs/specialpublications/nist.sp.800-190.pdf)

|

| 移动安全 |
| --- |

+   OWASP **移动安全测试指南**（**MSTG**）：[`github.com/OWASP/owasp-mstg`](https://github.com/OWASP/owasp-mstg)

|

请参阅第十章，*安全测试计划和实践*，了解更多细节。

**Q: 推荐哪些使用正则表达式或字符串模式搜索高风险源代码的白盒审查工具？**

| **工具** | **参考资料** |
| --- | --- |
| DREK | 工具：[`github.com/chrisallenlane/drek`](https://github.com/chrisallenlane/drek)签名：[`github.com/chrisallenlane/drek-signatures/tree/master/signatures`](https://github.com/chrisallenlane/drek-signatures/tree/master/signatures)（参阅`*.yml`文件） |
| GrAudit | 工具：[`github.com/wireghoul/graudit`](https://github.com/wireghoul/graudit)签名：[`github.com/wireghoul/graudit/tree/master/signatures`](https://github.com/wireghoul/graudit/tree/master/signatures)（参阅`*.db`文件） |
| **Visual Code Grepper**（**VCG**） | 工具：[`github.com/nccgroup/VCG`](https://github.com/nccgroup/VCG)签名：[`github.com/nccgroup/VCG/tree/master/VisualCodeGrepper/bin/Release`](https://github.com/nccgroup/VCG/tree/master/VisualCodeGrepper/bin/Release)（参阅`*.conf`文件） |
| CRASS Grep IT | 推荐使用 CRASS Grep IT 工具，因为它不需要任何依赖项，只需要一个 shell 脚本来执行。工具：[`github.com/floyd-fuh/crass/blob/master/grep-it.sh`](https://github.com/floyd-fuh/crass/blob/master/grep-it.sh)签名：[`github.com/floyd-fuh/crass/blob/master/grep-it.sh`](https://github.com/floyd-fuh/crass/blob/master/grep-it.sh) |

请参阅第十一章中展示的*白盒测试技巧*，了解更多细节。

**Q: 推荐的开源工具有哪些，用于 BDD 安全框架？**

| **BDD 安全框架** | **默认安全工具** |
| --- | --- |
| BDD-Security | OWASP ZAP, SSLyze, NessusBDD-Security 基于 Java 和 Cucumber。BDD-Security：[`www.continuumsecurity.net/bdd-security/`](https://www.continuumsecurity.net/bdd-security/) |
| MITTN | BurpSuite, SSlyze 和 Radamsa API 模糊测试 MITTN 基于 Python 和 Behave。MITTN：[`github.com/F-Secure/mittn`](https://github.com/F-Secure/mittn) |
| Gauntlt | CURL, NMAP, SSLyze, SQLmap, Garmr, Heartbleed, dirb, Arachni Gauntlt：[`gauntlt.org/`](http://gauntlt.org/) |

请参阅第十二章，*安全测试工具包*，了解更多细节。

**Q: 推荐哪些开源 Docker 安全扫描工具？**

| **Docker 安全工具** | **目的和参考** |
| --- | --- |
| Docker Bench | Docker Bench 是一个自动化脚本，用于检查系统是否符合 Docker 安全最佳实践。扫描规则基于 CIS Docker 安全基准。Docker Bench: [`github.com/docker/docker-bench-security/`](https://github.com/docker/docker-bench-security/) CIS Docker 安全基准: [`benchmarks.cisecurity.org/`](https://benchmarks.cisecurity.org/) |
| Actuary | Actuary 与 Docker Bench 的工作原理类似。此外，Actuary 还可以根据 Docker 安全社区提供的用户定义的安全配置文件进行扫描。Actuary: [`github.com/diogomonica/actuary/`](https://github.com/diogomonica/actuary/) |
| Clair | Clair 是一个容器镜像安全静态分析工具，用于检测 CVE 漏洞。Clair: [`github.com/coreos/clair`](https://github.com/coreos/clair) |
| Anchor Engine | Anchor Engine 扫描 Docker 镜像中的已知漏洞 CVE。Anchor Engine: [Https://github.com/anchore/anchore-engine](https://github.com/anchore/anchore-engine) 此外，Anchor 还提供云版本，参考 'Anchor Cloud' |
| Falco | Falco 是一个 Docker 容器运行时安全工具，可以检测异常活动。Falco: [`sysdig.com/opensource/falco/`](https://sysdig.com/opensource/falco/) |
| Dagda | Dagda 是一个集成的 Docker 安全工具，提供运行时异常活动检测（Sysdig Falco）、漏洞（CVE）分析（OWASP 依赖检查，Retire.JS）和恶意软件扫描（CalmAV）。Dagda: [`github.com/eliasgranderubio/dagda/`](https://github.com/eliasgranderubio/dagda/) |

参考 第十二章，*安全测试工具包*，以获取更多细节。

**问：有哪些集成的安全测试工具可以汇总各个测试工具的结果？**

Faraday

Faraday 是一个集成渗透测试环境，并提供一个仪表盘来汇总所有测试结果。它整合了超过 50 种安全工具。

Faraday: [`www.faradaysec.com/#why-faraday`](https://www.faradaysec.com/#why-faraday) 参考 [`github.com/infobyte/faraday/wiki/Plugin-List`](https://github.com/infobyte/faraday/wiki/Plugin-List) 以获取可用插件列表。

| **工具** | **默认包含的工具** |
| --- | --- |
| JackHammer | JackHammer，由 Ola 提供，是一个集成的安全测试工具。它提供一个仪表盘来汇总所有测试结果。其主要区别在于，JackHammer 包含了移动应用安全扫描和源代码静态分析工具。支持的开源安全扫描器包括 Brakeman、Bundler-Audit、Dawnscanner、FindSecurityBugs、PMD、RetireJS、Arachni、Trufflehog、Androbugs、Androguard 和 NMAP。JackHammer: [`github.com/olacabs/jackhammer`](https://github.com/olacabs/jackhammer) [Ola:](https://github.com/olacabs/jackhammer) [`jch.olacabs.com/userguide/`](https://jch.olacabs.com/userguide/) |

| Mozilla Minion | Mozilla Minion 也是一个集成的安全测试工具，默认包含以下插件：

+   ZAP

+   Nmap

+   Skipfish

+   SSLScan

Mozilla Minion: [`github.com/mozilla/minion/`](https://github.com/mozilla/minion/) |

| 渗透测试工具包 | 渗透测试工具包提供一个统一的 Web 界面，供多种 Linux 扫描工具使用，如 Nmap、nikto、WhatWeb、SSLyze、fping、URLCrazy、lynx、mtr、nbtscan、automater 和 shellinabox。渗透测试工具包: [`github.com/veerupandey/Penetration-Testing-Toolkit`](https://github.com/veerupandey/Penetration-Testing-Toolkit) |
| --- | --- |

| Seccubus | 使用 Seccubus 的主要优点是它能够整合各种漏洞扫描工具的测试结果，并比较每次扫描之间的差异。它包括以下扫描工具：

+   Nessus

+   OpenVAS

+   NMAP

+   Nikto

+   Medusa

+   SSLyze

+   SSL Labs

+   TestSSL.sh

+   SkipFish

+   ZAP

Seccubus: [`github.com/schubergphilis/Seccubus`](https://github.com/schubergphilis/Seccubus) |

| OWTF | **攻击性网络测试框架**（**OWTF**）是一个集成的安全测试案例，包含 OWASP 测试指南、PTES 和 NIST 测试标准。OWTF: [`owtf.github.io/`](https://owtf.github.io/) OWTF 指南: [`owtf.github.io/online-passive-scanner/`](https://owtf.github.io/online-passive-scanner/) |
| --- | --- |
| RapidScan | RapidScan 是一个包含网页漏洞扫描工具的多功能工具。它包含的安全扫描工具包括 Nmap、dnsrecon、uniscan、sslyze、fierce、theharvester、golismero 等。 |
| DefectDojo | OWASP DefectDojo 是一个安全工具，可以导入并整合各种安全测试工具的输出，集中在一个管理仪表盘中。DefectDojo: [`github.com/DefectDojo/django-DefectDojo`](https://github.com/DefectDojo/django-DefectDojo) |

请参考第十二章，*安全测试工具包*，了解更多详情

**Q: 常见的安全 Jenkins 插件有哪些？**

| **Jenkins 插件** | **描述** |
| --- | --- |
| ZAP | ZAP 是一个动态网页扫描工具。ZAP: [`plugins.jenkins.io/zap`](https://plugins.jenkins.io/zap) |
| Arachni Scanner | Arachni Scanner 是一个动态网页扫描工具。Arachni Scanner: [`plugins.jenkins.io/arachni-scanner`](https://plugins.jenkins.io/arachni-scanner) |
| 依赖检查插件 | 依赖检查插件检测有漏洞的依赖组件。依赖检查插件: [`plugins.jenkins.io/dependency-check-jenkins-plugin`](https://plugins.jenkins.io/dependency-check-jenkins-plugin) |
| FindBugs | FindBugs 是一个针对 Java 的静态代码分析工具。FindBugs: [`plugins.jenkins.io/findbugs`](https://plugins.jenkins.io/findbugs) |
| SonarQube | SonarQube 是一个代码质量分析工具。SonarQube: [`plugins.jenkins.io/sonar`](https://plugins.jenkins.io/sonar) |
| 360 FireLine | 360 FireLine 是一个 Java 的静态代码扫描工具。360 FireLine：[`plugins.jenkins.io/fireline`](https://plugins.jenkins.io/fireline) |
| HTML 发布插件 | HTML 发布插件生成 HTML 格式的测试结果。HTML 发布插件：[`plugins.jenkins.io/htmlpublisher`](https://plugins.jenkins.io/htmlpublisher) |
| 日志解析插件 | 日志解析插件解析安全测试工具的测试结果，如检测到的 XSS 数量或错误数量。日志解析插件：[`plugins.jenkins.io/log-parser`](https://plugins.jenkins.io/log-parser) |
| 静态分析收集器 | 静态分析收集器插件可以汇总所有其他静态代码分析插件的结果，如 Checkstyle、Dry、FindBugs、PMD 和 Android Lin。静态分析收集器：[`plugins.jenkins.io/analysis-collector`](https://plugins.jenkins.io/analysis-collector) |

参见第十三章，*CI 管道中的安全自动化*，了解更多详细信息。

# DevSecOps 针对运营团队

**Q. 针对 20 个 CIS 关键安全控制，有哪些建议的开源安全监控工具可以有效防御网络攻击？**

| **网络安全控制** | **安全技术示例** |
| --- | --- |
| CSC1：授权与未授权设备清单 | 端点安全，资产管理 |
| CSC2：授权与未授权软件清单 | 端点安全，资产管理 |
| CS3：移动设备、笔记本电脑、工作站和服务器的硬件与软件安全配置 | CIS 安全基准，OpenSCAP |
| CSC4：持续漏洞评估与修复 | OpenVAS：[`www.openvas.org/`](http://www.openvas.org/)Nmap：[`nmap.org/`](https://nmap.org/)OWASP 依赖检查：[`www.owasp.org/index.php/OWASP_Dependency_Check`](https://www.owasp.org/index.php/OWASP_Dependency_Check) |
| CSC 5：控制使用管理员权限 | 强密码复杂性审计日志以监控 root 和管理员活动 |
| CSC 6：审计日志的维护、监控和分析 | Syslog、事件日志、SIEMELK：[`bitnami.com/stack/elk`](https://bitnami.com/stack/elk)GrayLog：[`www.graylog.org/security`](https://www.graylog.org/security)Security Onion：[`github.com/Security-Onion-Solutions`](https://github.com/Security-Onion-Solutions)恶意流量检测：[`github.com/stamparm/`](https://github.com/stamparm/) |
| CSC 7：电子邮件与网页浏览器保护 | 电子邮件保护、反垃圾邮件、Web 应用防火墙 ModSecurity：[`www.modsecurity.org/`](https://www.modsecurity.org/)电子邮件加密 Scramble：[`dcposh.github.io/scramble/`](http://dcposch.github.io/scramble/)Linux 恶意软件检测：[`github.com/rfxn/linux-malware-detect`](https://github.com/rfxn/linux-malware-detect) |
| CSC 8: 恶意软件防御 | 终端保护，杀毒软件，HIDS/HIPSOSSEC: [`github.com/ossec/`](https://github.com/ossec/)ClamAV: [`www.clamav.net/`](https://www.clamav.net/) |
| CSC 9: 网络端口、协议和服务的限制与控制 | NMAP, OpenSCAP |
| CSC 10: 数据恢复能力 | Bacula: [`blog.bacula.org/`](https://blog.bacula.org/) |
| CSC 11: 网络设备如防火墙、路由器和交换机的安全配置 | CIS 安全基准： [`www.cisecurity.org/cis-benchmarks/`](https://www.cisecurity.org/cis-benchmarks/) |
| CSC 12: 边界防御 | 防火墙，IPS，蜜罐安全洋葱： [`github.com/Security-Onion-Solutions`](https://github.com/Security-Onion-Solutions) |
| CSC 13: 数据保护 | OSQuery: [`github.com/facebook/osquery/`](https://github.com/facebook/osquery/) 数据保险库:  [`github.com/hashicorp/vault`](https://github.com/hashicorp/vault) |
| CSC 14: 基于知情需要的受控访问 | 数据分类、防火墙、VLAN、日志记录 |
| CSC 15: 无线接入控制 | VPN，SSL 证书，WAP2 |
| CSC 16: 账户监控与控制 | 日志分析工具 Fail2ban: [`github.com/fail2ban/fail2ban/`](https://github.com/fail2ban/fail2ban/) |
| CSC 17: 安全技能评估和适当培训以弥补差距 | 安全培训和实验室资源 Cybrary: [`www.cybrary.it/`](https://www.cybrary.it/)Git 良好的信息安全资源集合：[`github.com/onlurking/awesome-infosec`](https://github.com/onlurking/awesome-infosec) |
| CSC 18: 应用软件安全 | OWASP: [`www.owasp.org/index.php/Category:OWASP_Project`](https://www.owasp.org/index.php/Category:OWASP_Project) |
| CSC 19: 事件响应与管理 | NIST SP800-61 计算机安全事件处理指南**快速事件响应**（**FIR**）： [`github.com/certsocietegenerale/FIR`](https://github.com/certsocietegenerale/FIR) |
| CSC 20: 渗透测试与红队演练 | 参考我们在第十二章中建议的一些开源工具，*安全测试工具包*。 |

**问：有哪些推荐的开源工具可以模拟黑客攻击，以测试安全监控的有效性？**

| **工具** | **APT 模拟** |
| --- | --- |
| 垃圾桶火灾 | 垃圾桶火灾包括各种模拟攻击场景，如账户攻击、文件下载、文件丢弃、命令执行和 Python 中的 Web 访问。它提供了一个用户友好的菜单，允许用户自定义安全事件，即使是那些不懂 Python 的人也能使用。垃圾桶火灾： [`github.com/TryCatchHCF/DumpsterFire`](https://github.com/TryCatchHCF/DumpsterFire) |
| METTA | METTA 允许安全团队根据 MITRE ATT&CK 自定义 APT 攻击的模拟。YAML 定义的模拟 APT 行为包括凭证访问、规避、发现、执行、外泄、横向移动、持久性和特权提升。METTA：[`github.com/uber-common/metta`](https://github.com/uber-common/metta)MITRE ATT&CK：[`attack.mitre.org/wiki/Main_Page`](https://attack.mitre.org/wiki/Main_Page) |
| **红队自动化**（**RTA**） | RTA 是一组 Python 和 PowerShell 脚本，可基于 ATT&CK 模拟超过 50 种恶意行为。RTA：[`github.com/endgameinc/RAT`](https://github.com/endgameinc/RAT) |
| **原子红队**（**ART**） | ART 提供 Windows、macOS 和 Linux 的 Shell 脚本，用于模拟 MITRE ATT&CK。ART：[`github.com/redcanaryco/atomic-red-team`](https://github.com/redcanaryco/atomic-red-team) |
| APT 模拟器 | APT 模拟器是一组 Windows BAT 脚本，用于模拟 APT 行为。APT 模拟器：[`github.com/NextronSystems/APTSimulator`](https://github.com/NextronSystems/APTSimulator) |
| 网络飞行模拟器 | 网络飞行模拟器可用于生成恶意网络流量，例如 DNS 隧道、C2 通信、DGA 流量和端口扫描。网络飞行模拟器：[`github.com/alphasoc/flightsim`](https://github.com/alphasoc/flightsim) |

**问：针对安全事件响应，推荐的行业参考有哪些？**

+   NIST SP 800-62 计算机安全事件处理指南：[`csrc.nist.gov/publications/detail/sp/800-61/rev-2/final`](https://csrc.nist.gov/publications/detail/sp/800-61/rev-2/final)

+   SANS 事件处理员手册：[`www.sans.org/reading-room/whitepapers/incident/incident-handlers-handbook-33901`](https://www.sans.org/reading-room/whitepapers/incident/incident-handlers-handbook-33901)

+   ENISA 云计算——信息安全的利益、风险和建议：[`resilience.enisa.europa.eu/cloud-security-and-resilience/publications/cloud-computing-benefits-risks-and-recommendations-for-information-security`](https://resilience.enisa.europa.eu/cloud-security-and-resilience/publications/cloud-computing-benefits-risks-and-recommendations-for-information-security)

+   MITRE 世界级网络安全运营中心十大战略：[`www.mitre.org/sites/default/files/publications/pr-13-1028-mitre-10-strategies-cyber-ops-center.pdf`](https://www.mitre.org/sites/default/files/publications/pr-13-1028-mitre-10-strategies-cyber-ops-center.pdf)

+   FIRST：[`www.first.org/education/FIRST_PSIRT_Service_Framework_v1.0`](https://www.first.org/education/FIRST_PSIRT_Service_Framework_v1.0)

**问：安全运营团队结构中的典型职能有哪些？**

| **关键职能** | **描述** |
| --- | --- |
| 安全事件分析与取证分析（呼叫中心） | 安全事件分析和取证分析团队可能包括 24x7 安全监控中心中的 Tier 1 事件处理。Tier 1 事件通常通过遵循预定义的检查单或标准操作程序（SOP）来进行初步根本原因分析或缓解处理。 |

| 安全操作和管理 | 安全操作和管理团队涉及以下常规安全活动。这些是检查生产环境的常规安全活动： |

+   网络扫描（每周一次）

+   漏洞扫描（每周一次）

+   渗透测试（每月一次）

+   安全意识培训（每两个月一次）

+   安全日志趋势分析（每月一次）

+   安全管理和监控（每天一次）

+   补丁或安全签名更新（每天/每周）

|

| 安全工具工程 | 安全工具工程团队为安全呼叫中心或安全操作团队实施安全工具。这些工具可以是安全自动化、可疑行为检测器、取证分析工具、安全配置检查工具、威胁情报集成、威胁签名创建工具等。 |
| --- | --- |

**问：推荐哪些开源工具用于安全取证？**

| **类别** | **工具** | **用途和使用场景** |
| --- | --- | --- |
| 日志收集 | OSX Collector | macOS X 日志收集器是一个用于 macOS X 的自动化取证证据收集器。Python 脚本 `osxcollector.py` 是执行所有收集工作的代码。该工具将生成一个 JSON 文件，用于汇总收集到的信息。OSX Collector: [`github.com/Yelp/osxcollector`](https://github.com/Yelp/osxcollector) |
| 日志收集 | IR Rescue | IR Rescue 是一个用于收集主机取证数据的 Windows 和 Linux 脚本。IR Rescue: [`github.com/diogo-fernan/ir-rescue/`](https://github.com/diogo-fernan/ir-rescue/) |
| 日志收集 | FastIR Collector | FastIR Collector for Linux 仅需要一个 Python 脚本即可收集 Linux 中所有相关日志。FastIR Collector: [`github.com/SekoiaLab/Fastir_Collector_Linux`](https://github.com/SekoiaLab/Fastir_Collector_Linux) 对于 Windows 版本，它需要额外的模块和工具。有关更多信息，请参考 [`github.com/SekoiaLab/Fastir_Collector`](https://github.com/SekoiaLab/Fastir_Collector)。 |
| 恶意软件检测器 | Linux 恶意软件扫描器 | Linux 恶意软件扫描器是一个免费的 Linux 恶意软件扫描工具。CalmAV: [`www.calmav.net/downloads`](https://www.calmav.net/downloads) **Linux Malware Detect** (**LMD**): [`github.com/rfxn/linux-maware-detect`](https://github.com/rfxn/linux-maware-detect) |
| 可疑文件分析 | Cuckoo | Cuckoo 是一个自动化恶意软件分析系统。 Cuckoo: [`cuckoosandbox.org/`](https://cuckoosandbox.org/) |
| 客户端/服务器日志收集与分析 | GRR Rapid Response | Google 的远程实况取证工具用于事件响应，需要在目标主机上安装 Python 代理来收集日志，并通过 Python 服务器进行分析。GRR Rapid Response: [`github.com/google/grr`](https://github.com/google/grr) |
| 客户端/服务器日志收集与分析 | OSQuery | OSQuery 的工作方式与 GRR 类似。主要区别在于，OSQuery 提供 SQL 查询来进行端点分析。OSQuery: [`osquery.io/`](https://osquery.io/)附加信息: [`osquery.readthedocs.io/en/stable/deployment/anomaly-detection/`](https://osquery.readthedocs.io/en/stable/deployment/anomaly-detection/) |

**问：有哪些工具集可以帮助构建威胁情报解决方案？**

| **类别** | **开源安全工具** |
| --- | --- |
| 日志收集器/传感器 | Syslog-NG: [`github.com/balabit/syslog-ng`](https://github.com/balabit/syslog-ng)Rsyslog: [`github.com/rsyslog/rsyslog`](https://github.com/rsyslog/rsyslog)FileBeat: [`www.elastic.co/products/beats/filebeat`](https://www.elastic.co/products/beats/filebeat)LogStash: [`www.elastic.co/products/logstash`](https://www.elastic.co/products/logstash) |
| SIEM/可视化 | Kibana: [`www.elastic.co/products/kibana`](https://www.elastic.co/products/kibana)ElasticSearch: [`www.elastic.co/`](https://www.elastic.co/)AlienValut OSSIM: [`www.alienvault.com/products/ossim`](https://www.alienvault.com/products/ossim)Grafana: [`grafana.com/`](https://grafana.com/)GrayLog: [`www.graylog.org/`](https://www.graylog.org/) |
| 威胁情报平台 | MISP - 开源威胁情报平台 MISP: [`www.misp-project.org`](http://www.misp-project.org)附加信息: [`csirtgadgets.org/collective-intelligence-framework/`](http://csirtgadgets.org/collective-intelligence-framework/) |
| 威胁情报源 | 外部威胁源提供了黑名单 IP 和防火墙规则建议：[`rules.emergingthreats.net/fwrules/`](https://rules.emergingthreats.net/fwrules/)[`www.spamhaus.org/drop/`](https://www.spamhaus.org/drop/)[`rules.emergingthreats.net/fwrules/emerging-Block-IPs.txt`](https://rules.emergingthreats.net/fwrules/emerging-Block-IPs.txt)[`check.torproject.org/exit-addresses`](https://check.torproject.org/exit-addresses)[`iplists.firehol.org/`](http://iplists.firehol.org/) |

**问：有哪些开源工具可以帮助我们进行安全扫描？**

| **类别** | **开源安全工具** |
| --- | --- |
| 一体化安全扫描（主机、网络、可视化） | Security Onion 包括了多个开源安全工具，如 Elasticsearch、Logstash、Kibana、Snort、Suricata、Bro、OSSEC、Sguil、Squert 和 NetworkMiner。Security Onion: [`github.com/Security-Onion-Solutions`](https://github.com/Security-Onion-Solutions) |
| 一体化主机 IDS、安全配置和可视化 | Wazuh 集成了 OSSEC（主机 IDS）、OpenSCAP（安全配置扫描）和 Elastic Stack（威胁可视化）。Wazuh: [`github.com/wazuh/wazuh`](https://github.com/wazuh/wazuh)规则: [`github.com/wazuh/wazuh-ruleset/tree/master/rules`](https://github.com/wazuh/wazuh-ruleset/tree/master/rules) |
| 安全配置 | OpenSCAP: [`www.open-scap.org/`](https://www.open-scap.org/) |
| 漏洞 | OpenVAS: [`www.openvas.org/`](http://www.openvas.org/) |
| 防病毒软件 | CalmAV: [`www.clamav.net/`](https://www.clamav.net/)LMD: [`github.com/rfxn/linux-malware-detect`](https://github.com/rfxn/linux-malware-detect) |
| 主机 IDS/IPS | OSSEC: [`github.com/ossec/ossec-hids`](https://github.com/ossec/ossec-hids)OSSEC 主机 IDS 规则: [`github.com/ossec/ossec-hids/tree/master/etc/rules`](https://github.com/ossec/ossec-hids/tree/master/etc/rules)Samhain: [`www.la-samhna.de/samhain/`](https://www.la-samhna.de/samhain/) |
| **Web 应用防火墙**（**WAF**） | ModSecurity: [`github.com/SpiderLabs/ModSecurity`](https://github.com/SpiderLabs/ModSecurity)规则: [`github.com/SpiderLabs/owasp-modsecurity-crs/tree/v3.0/master/rules`](https://github.com/SpiderLabs/owasp-modsecurity-crs/tree/v3.0/master/rules) |
| 网络 IDS/IPS | Snort: [`www.snort.org/`](https://www.snort.org/)Snort 规则: [`snort.org/advisories/talos-rules-2018-06-05`](https://snort.org/advisories/talos-rules-2018-06-05)Suricata: [`suricata-ids.org/`](https://suricata-ids.org/)Suricata 规则: [`github.com/OISF/suricata/tree/master/rules`](https://github.com/OISF/suricata/tree/master/rules) |
| MySQL 审计 | MySQL 审计插件: [`github.com/mcafee/mysql-audit`](https://github.com/mcafee/mysql-audit)MySQL 安全插件: [`dev.mysql.com/doc/mysql-security-excerpt/5.7/en/security-plugins.html`](https://dev.mysql.com/doc/mysql-security-excerpt/5.7/en/security-plugins.html) |

**问：每个新版本需要哪些安全检查清单和工具？**

| **安全类别** | **安全测试方法** | **建议的安全测试工具** |
| --- | --- | --- |
| 隐藏的通信端口或通道 |

+   确保没有隐藏的通信端口或后门。

+   确保没有隐藏的硬编码密钥、密码或硬件密钥。

+   确保没有不必要的系统维护工具。

+   启动源代码审查，检查网络通信，如 Java 相关的 API `connect()`、`getPort()`、`getLocalPort()`、`Socket()`、`bind()`、`accept()` 和 `ServerSocket()`。

+   禁止监听 0.0.0.0。

| NMAPGrauditTruffleHogSnallygasterHpingmasscan |
| --- |
| 隐私信息 |

+   在源代码中搜索明文密码和密钥。

+   搜索个人信息以确保符合 GDPR 合规性。

+   检查用户是否可以修改或删除个人信息。

+   检查个人信息是否能在定义的时间内删除。

| TruffleHogBlueflowerYARAPrivacyScoreSnallygaster |
| --- |
| 安全通信 |

+   使用 SSH v2 代替 Telnet

+   使用 SFTP 代替 FTP

+   使用 TLS 1.2 代替 SSL，TLS 1.1

| NMAPWireSharkSSLyzeSSL/TLS 测试工具 |
| --- |
| 第三方组件。 |

+   CVE 检查

+   已知漏洞检查

+   隐藏恶意代码或机密检查

| OWASP 依赖检查 LMD（Linux 恶意软件检测）OpenVASNMAPCVE 检查器 |
| --- |
| 加密技术 |

+   确保没有弱加密算法

+   确保公共 Web 接口上没有机密文件

| GrauditSSLyzeSnallygaster |
| --- |

| 审计日志 | 确保操作和安全团队能够记录以下场景：

+   非查询操作，包括成功和失败操作

+   非查询计划任务

+   执行管理任务的 API 访问或工具连接

| GREP |
| --- |

| DoS 攻击 | DoS 测试旨在确保应用程序故障是否按预期发生。DoS 场景可能包括以下内容：

+   TCP 同步洪水攻击

+   HTTP 慢请求

+   HTTP Post 洪水攻击

+   NTP DOS

+   SSL DoS 攻击

| PwnlorisSlowlorisSynfloodThc-sll-dosWreckuestsntpDOS |
| --- |

| Web 安全 | 这可以参考 OWASP 测试指南和 OWASP 前十名：

+   注入

+   破坏的身份验证

+   敏感数据泄露

+   XXE

+   访问控制漏洞

+   安全配置错误

+   XSS

+   不安全的反序列化

+   已知漏洞

+   日志记录和监控不足

| 请参考 OWASP 测试指南 v4.OWASP ZAPBurpSuiteArachni 扫描器 SQLMap |
| --- |
| 安全配置 | 确保应用程序、Web 服务、数据库和操作系统的配置是安全的。安全配置基于 CIS 安全基准和 OpenSCAP。 | OpenSCAPDocker Bench SecurityClair |
| 模糊测试 | 模糊测试的目的是生成动态测试数据作为输入，以检查应用程序是否会出现意外失败。 | API FuzzerRadamsaAmerican Fuzzy lopFuzzDBWfuzz |
| 移动应用安全 | 请参考 OWASP 移动应用安全测试指南。 | 移动安全框架 |
| 常见问题 | 基于项目历史数据的最常见安全问题列表。 | CWE/SANS Top 25 最危险的软件错误 |
| 安全合规 | 基于业务需求的安全合规性也可能包括，例如 GDPR 或 PCI DSS。 | 请参考具体的安全合规性要求。 |

**Q. 用于构建大数据框架安全分析的开源工具有哪些？**

| **项目** | **关键特性** |
| --- | --- |
| TheHive 项目 | TheHive 提供威胁事件响应案例管理，允许安全分析员标记 IOCs。Cortex 可以通过 VirtusTotal、MaxMind 和 DomainTools 等威胁情报服务分析问题。它支持超过 80 种威胁情报服务。Hippocampe 提供通过 REST API 或 Web UI 的查询接口。TheHive：[`thehive-project.org/`](https://thehive-project.org/) |
| MISP | MISP 主要是一个威胁情报平台，用于共享恶意软件的 IoC 和指示器。关联引擎有助于识别恶意软件属性和指示器之间的关系。MISP：[`www.misp-project.org/`](https://www.misp-project.org/documentation/) MISP 提供超过 40 个威胁情报数据源。有关更多信息，请参考 [`www.misp-project.org/feeds/`](https://www.misp-project.org/feeds/) |

| Apache Metron | Apache Metron 是一个 SIEM（包含威胁情报、安全数据解析器、警报、仪表盘）以及一个基于 Hadoop 大数据框架的安全分析（异常检测和机器学习）框架。Apache Metron：[`metron.apache.org/`](https://metron.apache.org/) 构建大数据框架所需的典型技术组件包括以下内容：

+   Apache Flume

+   Apache Kafka

+   Apache Storm 或 Spark

+   Apache Hadoop

+   Apache Hive

+   Apache Hbase

+   Elastic Search

+   MySQL

|

参考第十八章，《*商业欺诈与服务滥用*》，了解更多详细信息。

**问：常见的入侵指标和用于识别它们的检测技术是什么？**

| **异常主机行为** | **潜在威胁** |
| --- | --- |
| 多个被攻陷主机与外部主机的数据通信。 | 被攻陷的主机正在将数据发送到外部 C&C 服务器。 |
| 主机连接到外部已知 APT IP 地址或 URL。主机下载了已知的恶意文件。 | 主机显示出来自 APT 或恶意软件攻击的被攻陷迹象。 |
| 多次登录失败尝试 | 内部被攻陷的主机正在尝试登录以访问关键信息。 |
| 包含危险 URL 或恶意文件的电子邮件消息 | 攻击者可能利用社会工程学发送电子邮件进行定向攻击。将电子邮件发送者添加到观察列表中。 |

| 稀有且不寻常的文件名 | 恶意软件在启动时安装自己，即使重启后也能继续运行。以下是恶意软件实现持久性的常见方式。

+   程序启动

+   服务

+   进程注入

+   登录脚本

对于 Windows，建议使用 AutoRuns 检查主机是否被可疑恶意软件感染。AutoRuns：[`docs.microsoft.com/en-us/sysinternals/downloads/autoruns`](https://docs.microsoft.com/en-us/sysinternals/downloads/autoruns) |

| 异常事件和审计日志警报 | 以下系统事件或审计日志可能需要进一步分析：

+   帐户锁定

+   用户已添加到特权组

+   用户帐户登录失败

+   应用程序错误

+   Windows 错误报告

+   蓝屏死机日志（BSOD）

+   事件日志已被清除

+   审计日志已被清除

+   防火墙规则更改

|

以下表格列出了 Web 访问日志中的检测异常：

| **Web 访问分析** | **检测技术** |
| --- | --- |

| 外部来源客户端 IP | IP 地址的来源有助于识别以下内容：

+   已知的坏 IP 或 Tor 出口节点。

+   异常的地理位置变化。

+   来自不同地理位置的并发连接。

MaxMind GeoIP2 数据库可用于将 IP 地址转换为地理位置。 MaxMind GeoIP2: [`dev.maxmind.com/geoip/geoip2/geolite2/#Downloads`](https://dev.maxmind.com/geoip/geoip2/geolite2/#Downloads) |

| 客户端指纹（操作系统、浏览器、用户代理、设备等） | 客户端指纹可以用来识别是否存在任何异常的客户端或非浏览器连接。开源的 `clientJS` 是一个纯 JavaScript 库，可以用于收集客户端指纹信息。Salesforce 提供的 JA3 工具使用 SSL/TLS 连接分析来识别恶意客户端。ClientJS: [`clientjs.org/`](https://clientjs.org/)JA3: [`github.com/salesforce/ja3`](https://github.com/salesforce/ja3) |
| --- | --- |
| 网站信誉 | 当存在到外部网站的出站连接时，我们可以检查该特定网站的威胁信誉。这可以通过 Web 应用防火墙或 Web 网关安全解决方案完成。 VirusTotal: [`www.virustotal.com/`](https://www.virustotal.com/) |
| 通过 DGA（域名生成算法）生成的随机域名 | C&C 服务器的域名可以通过 DGA 生成。DGA 域名的关键特征可能是高熵、高辅音数和长域名长度。根据这些指标，我们可以分析域名是否由 DGA 生成，从而成为潜在的 C&C 服务器。DGA 检测器: [`github.com/exp0se/dga_detector/`](https://github.com/exp0se/dga_detector/)此外，为了减少误报，我们还可以使用 Alexa 前 1,000,000 个网站作为网站白名单。有关更多信息，请参阅 [`s3.amazonaws.com/alexa-static/top-1m.csv.zip`](https://s3.amazonaws.com/alexa-static/top-1m.csv.zip) |
| 可疑文件下载 | Cuckoo Sandbox 对于可疑文件分析非常有用。 Cuckoo Sandbox: [`cuckoosandbox.org/`](https://cuckoosandbox.org/) |

| DNS 查询 | 对于 DNS 查询的分析，以下是妥协的关键指标：

+   DNS 查询到未经授权的 DNS 服务器。

+   不匹配的 DNS 回复可能是 DNS 欺骗的指示。

+   客户端连接到多个 DNS 服务器。

+   长 DNS 查询（例如，超过 150 个字符），这是 DNS 隧道的指示。

+   高熵的域名。这是 DNS 隧道或 C&C 服务器的指示。

|

请参阅 第十八章，*商业欺诈与服务滥用*，以获取更多详细信息。

**问：在商业场景中，常见的网络犯罪活动有哪些？**

| **业务场景** | **网络犯罪活动** |
| --- | --- |
| 在新用户注册的推广中，电商网站可能会赠送 10 美元优惠券或某些折扣。 | **账户作弊：** 网络犯罪分子可能会注册大量账户以获得优惠券和折扣，然后转售这些优惠券。 |
| 购物网站可能会出售数量有限的特别版商品。 | **黄牛党：**网络犯罪分子可能会注册大量账户购买商品，然后以更高的价格转售。 |
| 购物搜索查询结果按在线卖家的评分和销量排序。 | **虚假订单：**在线卖家可能与网络犯罪分子合作，操纵大量虚假订单和评分，以便在查询结果的排名中位列前茅。 |
| 购物网站账户通常需要使用电子邮件地址、电话号码和身份证号注册。 | **账户接管：**网络犯罪分子冒充真实用户，控制账户进行未经授权的交易。此外，网络犯罪分子可能会对账户进行暴力破解，并重新注册其他电子邮件或电话号码，以获取经济利益。 |

参考第十九章，*GDPR 合规案例分析*，获取更多详细信息。

**问：**“分析”如何帮助检测商业欺诈和滥用行为？

| **分析** | **描述** |
| --- | --- |

| IP 分析 | IP 分析用于识别账户和设备的 IP 行为。IP 分析包括以下属性：

+   地理位置

+   VPN、代理、网关或 Tor（这些 IP 地址需要用户进一步验证）

+   已知的黑名单 IP 地址

|

| 设备指纹 | 设备指纹是关于远程客户端设备或浏览器的收集信息，用于身份识别。我们使用设备指纹来判断远程连接的设备是否为用户/账户常用设备。例如，对于同一账户，如果每天用不同的手机登录电子商务服务，这肯定是异常的标志。以下是一些常见的设备指纹：

+   机器类型、CPU、虚拟化

+   操作系统版本、软件插件、字体

+   同一设备指纹的并发连接

+   同一设备在同一天的地理定位

+   多个不同账户使用相同的设备指纹

+   同一用户账户使用多个不同的设备指纹

|

| 机器与人工行为 | 行为分析的目的是识别请求的来源是否被恶意程序操控，或者是真实用户的行为。以下是分析用户行为的几个线索，用于判断其是人类还是机器：

+   键盘使用

+   鼠标移动

+   用户代理 HTTPS 指纹

|

| 账户分析 | 以下属性与账户相关。如果某一属性被识别为可疑，例如电子邮件地址，那么与该电子邮件地址相关的其他账户也很可能是可疑的。因此，我们将建立一个关注名单，包含以下隐私信息：

+   电子邮件地址

+   收货地址

+   银行账户号码

+   电话号码

+   社交网络朋友

+   支付

|

| 使用情况分析 | 基于历史使用情况，我们还可以识别是正常用户，还是仅仅是滥用服务或业务推广代码的单次用户：

+   页面访问历史记录

+   历史与卖家的沟通记录

+   购买历史与习惯

|

请参见 第十九章，*GDPR 合规性案例研究*，以获取更多详细信息。

# 总结

在本章最后，我们总结了不同角色（如安全管理、开发、测试、IT 和运维团队）在 DevSecOps 实践中的关键常见问题解答（FAQ）。

安全管理识别安全需求，并确定支持业务成功的安全合规性需求。为了实现这一目标，安全经理可能会定义安全意识计划、安全保障计划、安全指南，以及供开发、测试和安全监控团队使用的流程或工具。

开发团队的目标是以快速交付构建安全的软件和服务。安全和隐私设计原则将贯穿整个开发周期，从安全需求、设计安全架构框架、加固编译选项、安全编码，到安全的第三方依赖关系。我们列出了许多行业最佳实践、建议框架和安全代码扫描工具供开发团队参考。开发团队可以应用这些安全实践。

安全测试团队通过多种方法确保安全质量，如白盒代码审查、渗透测试、安全配置、安全通信、敏感信息审查等。我们介绍了几种开源工具和测试方法来进行安全测试。这些安全测试工具也可以被其他团队使用。例如，开发团队也可以使用代码扫描工具确保在构建阶段的安全编码，运维团队也可以在部署阶段使用安全配置扫描工具。安全不仅仅是测试团队的责任，还需要开发与运维团队之间的合作。

安全运维团队需要确保云服务的安全性 24/7。安全运维团队的安全活动包括安全监控、安全事件响应、安全环境管理、漏洞管理和服务滥用监控。此外，我们还介绍了威胁情报，它提供威胁信息流，以检测已知的恶意 IP、文件哈希值或 DNS。安全运维团队是对抗威胁的前沿防线，可以为开发和测试团队提供最有价值的反馈，以提高安全性。同样，一个有效的反馈环路也需要每个职能团队之间的高度协作。

向 DevSecOps 过渡需要开发、测试、IT、运维和安全监控团队之间的高度协作。例如，基础设施运维团队可能会使用 Docker 容器技术进行部署。安全测试团队将提供安全配置评估工具，开发团队可能会定义 Dockerfile 的安全配置。安全管理团队会定义 Docker 采用全生命周期的安全要求。将安全“左移”——即将安全融入过程的早期阶段——需要每个角色的协作文化和安全意识。一个明确定义的角色、责任矩阵以及标准操作程序（SOP）可以帮助提高任务执行的效率。然而，职能团队之间的壁垒以及各个角色的关键绩效指标（KPI）可能对推动 DevSecOps 进展产生负面影响。每个职能团队对安全和隐私重要性的理解以及团队之间的协作是 DevSecOps 成功的关键。

# 进一步阅读

请访问以下网址了解更多信息：

+   **SANS DevSecOps 行动手册**: [`www.sans.org/reading-room/whitepapers/analyst/devsecops-playbook-36792`](https://www.sans.org/reading-room/whitepapers/analyst/devsecops-playbook-36792)

+   **CSA 云计算关键领域的安全指南 v4.0**: [`cloudsecurityalliance.org/guidance/#_overview`](https://cloudsecurityalliance.org/guidance/#_overview)

+   **超棒的信息安全课程和培训资源**: [`github.com/onlurking/awesome-infosec`](https://github.com/onlurking/awesome-infosec)

+   **Cybrary 安全培训课程**: [`www.cybrary.it/`](https://www.cybrary.it/)

+   **开放安全培训**: [`opensecuritytraining.info/`](http://opensecuritytraining.info/)

+   **SaaS 初创企业的安全 101**: [`github.com/forter/security-101-for-saas-startups/blob/english/security.md`](https://github.com/forter/security-101-for-saas-startups/blob/english/security.md)

+   **固件安全培训**: [`github.com/advanced-threat-research/firmware-security-training`](https://github.com/advanced-threat-research/firmware-security-training)

+   **超棒的事件响应**: [`github.com/meirwah/awesome-incident-response`](https://github.com/meirwah/awesome-incident-response)

+   **超棒的 AI 安全**: [`github.com/RandomAdversary/Awesome-AI-Security`](https://github.com/RandomAdversary/Awesome-AI-Security)

+   **超棒的渗透测试**: [`github.com/wtsxDev/Penetration-Testing`](https://github.com/wtsxDev/Penetration-Testing)

+   **超棒的渗透测试**: [`github.com/enaqx/awesome-pentest`](https://github.com/enaqx/awesome-pentest)
