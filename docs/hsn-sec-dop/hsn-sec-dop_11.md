# 白盒测试技巧

测试计划概述了测试方法、风险评估和建议的测试工具。在本章中，我们将重点介绍白盒测试技巧。

白盒代码审查最有效的方式是识别特定的安全问题，如 XXE、反序列化和 SQL 注入。然而，如果没有合适的工具或策略，白盒审查可能会耗时。如果要进行有效的白盒测试，我们需要关注特定的编码模式和高风险模块。本章将提供识别高风险安全问题的技巧、工具和关键编码模式。

本章将涵盖以下主题：

+   白盒审查准备

+   整个项目的鸟瞰图

+   高风险模块

+   白盒审查检查清单

+   常见的主要问题

+   安全编码模式和关键字

+   案例研究——Java Struts 安全审查

# 白盒审查准备

白盒测试或源代码审查最有效的方式是识别源代码中的隐藏安全问题。在开始白盒源代码审查之前，一些准备工作和输入将帮助我们判断该如何（方法、工具）以及做什么（哪些模块）来进行安全源代码审查。

以下是我们在进行源代码审查之前可能会检查的列表；请查看此表：

| **白盒测试输入** | **注意事项** |
| --- | --- |
| 源代码 |

+   我们需要完整的可构建源代码吗？

+   源代码是否包含相关的导入模块或头文件？

+   这些依赖源代码在我们希望追溯某些 API 定义时非常有用。然而，如果没有完整的源代码，可能需要进行逆向工程。

|

| 威胁建模文档 | 威胁建模为我们提供了一个很好的参考，帮助识别我们应重点关注的高风险模块和接口。 |
| --- | --- |
| 架构和设计文档 | 架构和设计文档为我们提供了设计流程和模块关系的良好视图。 |
| 自动化静态代码分析结果 | 在进行白盒审查之前，最好先进行一次自动化安全代码扫描。扫描结果不仅使工作更容易，而且还会告诉我们应该关注哪些部分。 |
| 应用程序相关配置 | 一些安全框架可能在配置中定义安全策略，这些配置也应进行审查。例如，Spring MVC 或 Spring Security 框架中的 `web.xml` 文件对于访问控制非常关键。 |
| 通信接口或端口 | 列出外部 API 接口和通信端口的目的是了解它们如何与来自不可信源的外部输入、不安全的通信协议或错误暴露的 API 进行交互。 |

对于一些外部依赖或第三方组件，在没有源代码的情况下，我们可能需要对这些组件进行某些分析，以确定是否存在后门、弱加密、硬编码或密码。这将需要逆向工程和动态运行时分析。下表提供了一些工具供进一步参考：

| **描述** | **工具** |
| --- | --- |
| Cuckoo | Cuckoo Sandbox 是一个开源虚拟化环境，用于对任何二进制文件进行静态和动态分析。欲了解更多信息，请参阅 [`cuckoosandbox.org`](https://cuckoosandbox.org)。 |
| REMnux | REMnux 包含了许多用于逆向工程的 Linux 工具包。欲了解更多信息，请参阅 [`remnux.org/`](https://remnux.org/)。 |

# 查看整个项目

自上而下的方法意味着我们使用源代码分析工具来查看编程流程图，如类图、调用图或依赖图。下表列出了一些推荐的工具，帮助你更容易地分析源代码：

| **工具** | **描述** |
| --- | --- |

| Doxygen | 它可以从源代码生成文档，并且还可以通过使用 Graphviz 的 dot 工具，自动可视化模块之间的关系、依赖图和继承图。请参阅网站 [www.doxygen.org](http://www.doxygen.org)。要能够从源代码生成文档，需要在源代码中添加适当的注释和标签。以下是一些可能值得阅读的提示。请记住，通过 doxygen 生成文档可能需要较长时间。不要将 doxygen 与编译器的部分任务绑定。有关更多信息，请查看以下链接：

+   [`www.rosettacommons.org/docs/latest/development_documentation/tutorials/doxygen-tips`](https://www.rosettacommons.org/docs/latest/development_documentation/tutorials/doxygen-tips)。

+   [`www.stack.nl/~dimitri/doxygen/manual/commands.html`](http://www.stack.nl/~dimitri/doxygen/manual/commands.html)。

|

| Graphviz | 它不是一个代码分析工具，但它帮助 doxygen 生成图表。欲了解更多信息，请参阅 [www.graphviz.org/Download.php](http://www.graphviz.org/Download.php)。 |
| --- | --- |
| HTML Help Workshop | 用于将 doxygen 生成的 HTML 文件转换为 CHM 文档。请查看 [`msdn.microsoft.com/en-us/library/windows/desktop/ms669985(v=vs.85).aspx`](https://msdn.microsoft.com/en-us/library/windows/desktop/ms669985(v=vs.85).aspx)。 |
| phpDocumentor | 如果项目的编程语言是 PHP，phpDocumentor 能够很好地直接从 PHP 源代码生成 API 文档和类继承图。请查看 [`www.phpdoc.org/`](https://www.phpdoc.org/)。 |

| Natural docs | 它支持超过 20 种编程语言，并允许开发者以非常直接的方式记录源代码。请记住，源文档仍然需要开发团队正确地注释源代码。请查看 [`www.naturaldocs.org/`](http://www.naturaldocs.org/)。以下是源代码中注释的示例：

```
// Function: Sum
// Sum up two integers and returns the result.
int Sum (int a, int b)
{ return a + b; }
```

|

| Pandoc | 它是一个通用的文档格式转换器。请查看以下链接以获取更多信息：

+   [`pandoc.org/try/`](http://pandoc.org/try/)

|

| Sphinx  | 它主要用于 Python 文档。请查看 [`www.sphinx-doc.org/`](http://www.sphinx-doc.org/)。 |
| --- | --- |

总结来说，为了直接从源代码生成文档，我们将使用以下工具——Natural docs 和 doxygen 用于一般编程语言，phpDocumentor 用于 PHP，Sphinx 用于 Python。这些文档生成工具并非魔法。如果开发团队没有遵循某些编码注释规范，生成的信息也会受到限制。对于白盒审查，我们使用源代码文档生成工具来更高效地识别安全问题。然而，如果生成的文档在这方面帮助不大，可以转向以下审查方法。请仔细考虑以下各节内容。

# 高风险模块

一旦我们对整个项目有了清晰的了解，我们将需要识别那些需要进一步手动代码审查的模块或功能。我们不仅仅对高风险模块进行手动代码审查；我们会对所有模块进行自动化代码扫描，并对那些可能隐藏安全问题的高风险模块进行进一步的手动代码审查，这些问题可能无法通过自动化扫描工具轻易识别。

当我们识别高风险模块，以优先考虑白盒源代码审查模块时，尽量像黑客一样思考。*哪些模块会吸引黑客？* *哪些信息对黑客最有价值？* *所有应用中最薄弱的环节是什么？* 以下表格列出了应考虑进行进一步白盒审查的典型高风险模块：

| **高风险模块** | **业务功能** |
| --- | --- |
| 身份验证 |

+   账户注册。

+   登录与验证码。

+   密码恢复或重置。

+   密码更改。

+   身份和密码存储与访问控制。

+   多次失败后的账户锁定控制。

|

| 授权 |
| --- |

+   敏感资源访问。

+   管理管理。

|

| 配置管理 | 配置审查有两种类型。一种是配置值，另一种是应用程序如何安装或更新配置。一般来说，网页、数据库和服务配置需要特别注意。 |
| --- | --- |
| 财务 |

+   支付功能。

+   订单和购物车。

|

| 文件处理 |
| --- |

+   文件上传。

+   文件下载。

+   文件处理。

|

| 数据库 |
| --- |

+   数据库查询操作。

+   数据库创建、添加、更新和删除选项。

|

| API 接口 |
| --- |

+   Restful API 接口或其他通信接口。

+   第三方集成接口。

|

| 遗留 |
| --- |

+   不支持安全通信的模块。

+   可能仍在使用弱加密算法的模块。

+   使用禁用或危险的 API。

|

| 加密 |
| --- |

+   使用禁用的加密算法。

+   在开发过程中，源代码中硬编码的敏感信息或注释，例如 IP、电子邮件、密码或隐藏的快捷键。

|

# 白盒审查检查表

建议制定检查表进行白盒审查。在白盒源代码审查中使用安全检查表可以帮助团队决定应该关注的重点内容。典型的代码审查安全检查表可能包括关键安全控制项，如身份验证、数据验证、授权、会话管理、错误处理、加密、日志记录、安全配置、管理功能、支付、与金钱相关的功能以及私人数据处理。

安全检查表的参考来源可以来自行业最佳实践或历史项目经验。检查表的内容可以根据审查的目标不同而有所不同。

请查看下表：

| **安全检查表类别** | **目标和参考** |
| --- | --- |

| 一般安全代码审查检查表 | 目标是为项目团队提供一个安全代码审查检查表模板。项目团队可以根据项目概况进一步添加或自定义列表。以下是行业参考链接：

+   *OWASP 安全编码实践* 在 [`www.owasp.org/index.php/OWASP_Secure_Coding_Practices_-_Quick_Reference_Guide`](https://www.owasp.org/index.php/OWASP_Secure_Coding_Practices_-_Quick_Reference_Guide)。

+   *OWASP 代码审查指南* 在 [`www.owasp.org/index.php/Category:OWASP_Code_Review_Project`](https://www.owasp.org/index.php/Category:OWASP_Code_Review_Project)。

|

| 常见问题 | 理想的常见问题检查表是根据历史项目记录、编程语言或项目类型总结的。如果没有足够的项目数据来生成该列表，可以参考 CWE 或 OWASP。以下是行业参考链接：

+   *CWE Top 25 最危险的软件错误* 在 [`cwe.mitre.org/top25/`](http://cwe.mitre.org/top25/)。

+   [*OWASP Top Ten 项目*](http://cwe.mitre.org/top25/) [`www.owasp.org/index.php/Category:OWASP_Top_Ten_Project`](https://www.owasp.org/index.php/Category:OWASP_Top_Ten_Project)[.](http://cwe.mitre.org/top25/)

|

| 特定安全问题（Struts，反序列化） | 目标是专注于某一特定安全问题的安全审查。在某些情况下，我们可能会发现这些类型的安全审查很有帮助。例如，Java Struts 框架正在遭受攻击，团队可能希望检查与 Struts 相关的实现是否存在漏洞。项目 A 中已经发现了一个重大安全问题，组织希望了解其他项目是否也有类似的安全问题。检查特定安全问题的驱动因素可能来自最近发布的 CVE、重大安全事件新闻，或是客户报告的安全问题，我们希望检查所有其他项目是否存在同样的问题。以下是这一类别的示例列表：

+   Struts 安全问题。

+   Java 反序列化安全问题。

+   REST API 安全。

|

# 常见安全问题

一个常见问题的清单对项目团队在进行安全代码审查时非常有效。为了构建一个常见的安全问题清单，建议参考 CWE Top 25。安全团队和项目团队可以基于历史项目数据，以 CWE Top 25 为基础，结合公司内部的安全问题，共同达成关于前五大安全问题的共识。

总结公司内部的主要安全问题至关重要，因为 CWE Top 25 可能不完全适用于公司内部项目，这与业务背景、技术栈和实现方式密切相关。一旦识别出公司内部的安全问题列表，应该同时列出相应的缓解方法。请参阅下表，了解它可能的样式。表格的目的是给出一个示例，您也可以根据自己的组织定义适合的列表，而不是直接复制 CWE Top 25 中的全部内容。请注意，以下仅为示例，而非全面列表。让我们看看这个表格：

| **主要问题示例** | **主要安全问题的缓解方法** |
| --- | --- |
| CWE-89 SQL 注入 | 通过工具可以有效检测特定源代码模式中的 SQL 注入问题。重点检查那些没有使用预编译语句的 SQL 语句，或者在 iBATIS 框架中使用`$`作为 SQL 参数的情况。 |
| CWE-78 操作系统命令注入 | 由于代码扫描工具可以检测操作系统命令注入问题，团队决定列出可能导致命令注入的高风险 API，并开发一个工具进行源代码搜索。 |

| CWE-120 缓冲区溢出 | 根据历史记录，缓冲区溢出问题曾是常见问题之一。团队进一步识别了可能导致缓冲区溢出的常见 API。例如，列出了 C/C++相关的 API：

+   `strcpy`, `strncpy_s`, `strncpy`

+   `strncat`, `strcat`

+   `sprint`, `snprintf`

+   `memcpy`

+   `memmove_s`, `memset`, `memset_s`

+   `scanf_s`, `gets`, `vscanf`

|

| CWE-79 跨站脚本攻击 (XSS) | 团队还发现 XSS 是最主要的问题之一。为了审查 XSS 问题，团队决定列出所有可能导致 XSS 的 API。以下是一些例子——在 JS/JSP/HTML 中，查找以下相关函数：

+   `document.location`

+   `document.URL`

+   `document.write`

+   `document.open`

+   `eval`

在 Java 中，查看以下 API 的参数：

+   `Request.getParameter`

+   `innerHTML.innerText`

+   `getAttribute`

+   `getHeader`

+   `getServerName`

|

| CWE-306 关键功能缺少认证 | 特定 URL 或资源缺少认证是一个常见的安全问题，且难以通过任何工具检测。哪些 URL 可以在没有认证的情况下被访问，哪些 URL 需要认证，通常与业务逻辑紧密相关。此类安全问题也很难通过白盒源代码审查识别。基于历史项目记录，以下是 Java 源代码模式的一些提示：

+   使用部分 URL 匹配 API 来确定是否需要认证，如 `StartsWith` 和 `EndsWith`

+   验证之前没有进行路径规范化

+   验证之前没有进行数据标准化

|

# 安全编码模式和关键字

基于源代码关键字或特定模式的搜索技术的目标不是替代任何其他自动化代码扫描工具。它是通过搜索潜在的高风险字符串来支持白盒审查和自动化代码扫描工具。安全团队可以准备或定义一组可能导致安全问题的关键字或正则表达式字符串。一旦项目团队拥有一组搜索字符串，可以使用任何搜索工具（如 **GREP**）进行搜索，并分析搜索结果。这种搜索可以在部分源代码中进行，并且与编程语言无关。只要我们有明确定义的搜索字符串，搜索特定问题就变得非常简单。

下图展示了这种类型的白盒审查技术的一般过程：

![](img/00037.jpeg)

下面是如何根据特定模式或关键字搜索代码潜在安全风险的示例。你也可以参考 *OWASP 代码审查指南 2.0，附录——爬虫* *代码* 获取更多信息以及其他编程语言。

请查看以下表格：

| **安全问题类别** | **Java 代码模式/关键字示例** |
| --- | --- |
| 命令注入 | `Runtime.exec`, `ProcessBuilder` |
| 缓冲区溢出风险 | `strcpy`, `strcat`, `sprint`, `sscanf`, `vscanf`, `gets` |
| XML 注入 | `SAXParser`, `DocumentBuilderFactory`, `BeanReader`, `XmlReader`, `DOMParser`, `SAXReader`, `XMLInputFactory` |
| 敏感信息 |

+   后门、密码、管理员、root

+   加密、`getInstance`

+   `MessageDigest.getInstance`

+   编码、加密、共享密钥、令牌

+   URL、电子邮件、IP 地址

|

| HTTPS **中间人攻击** (**MITM**) |
| --- |

+   `ALLOW_ALL_HOSTNAME_VERIFIER`

+   `X509Certificate`, `X509TrustManager`

+   `getAcceptedIssuers`

|

| 不安全的加密技术 |
| --- |

+   RC4, SSL, AES, DEC, ECB, MD5, SHA1

+   `Java.util.Random`

+   `Cipher.newInstance("DES`

+   `Cipher.getInstance("ECB`

|

| XSS |
| --- |

+   `document.location`, `document.URL`

+   `document.referrer`, `document.write`, `document.print`

+   `document.body.innerHTML`

+   `window.location`, `window.execScript`

+   `window.setTimeout`, `window.open`

+   `request.getParameter`

|

| 反序列化问题 |
| --- |

+   `XMLDecoder`

+   `XStream`

+   `readObject`, `readResolve`, `readExternal`

|

| 用户数据输入 |
| --- |

+   `getParameter`, `getQueryString`, `getRequest`

+   `getCookies`, `getInputStream`, `getReader`

+   `getInputSteam`, `getMethod`, `getReader`

+   `getRemoteUser`, `getServerName`

|

以下是此类别中可以进行源代码扫描的安全代码扫描工具，基于正则表达式模式。通常，这些工具还会具有预定义的易受攻击的源代码模式和安全签名。建议审查这些安全签名，并根据您的项目环境定制这些正则表达式或字符串。请查看下表：

| **工具** | **参考资料** |
| --- | --- |
| drek |

+   **工具**: [`github.com/chrisallenlane/drek`](https://github.com/chrisallenlane/drek)

+   **签名**: [`github.com/chrisallenlane/drek-signatures/tree/master/signatures`](https://github.com/chrisallenlane/drek-signatures/tree/master/signatures)  （参考 `*.yml`）

|

| Graudit |
| --- |

+   **工具**: [`github.com/wireghoul/graudit`](https://github.com/wireghoul/graudit)

+   **签名**: [`github.com/wireghoul/graudit/tree/master/signatures`](https://github.com/wireghoul/graudit/tree/master/signatures)  （参考 `*.db`）

|

| **VisualCodeGrepper** (**VCG**) |
| --- |

+   **工具**: [`github.com/nccgroup/VCG`](https://github.com/nccgroup/VCG)

+   **签名**: [`github.com/nccgroup/VCG/tree/master/VisualCodeGrepper/bin/Release`](https://github.com/nccgroup/VCG/tree/master/VisualCodeGrepper/bin/Release)  （参考 `*.conf`）

|

| CRASS Grep IT  | 推荐使用此工具，因为它无需任何依赖，仅需要一个 shell 脚本即可执行。

+   **工具**: [`github.com/floyd-fuh/crass/blob/master/grep-it.sh`](https://github.com/floyd-fuh/crass/blob/master/grep-it.sh)

+   **签名**: [`github.com/floyd-fuh/crass/blob/master/grep-it.sh`](https://github.com/floyd-fuh/crass/blob/master/grep-it.sh)  （参考 `search "......"`)

|

这些都是静态代码分析工具，使用类似 GREP 的搜索方法来识别易受攻击的源代码。这种源代码审查方法对于禁止的 API、不安全的 API、弱加密算法或硬编码的秘密最为有效。它灵活性高，因此可以扫描部分源代码，无需完整构建项目，并且可以用于扫描多种编程语言，只要安全代码模式签名已正确定义。

# 案例研究 – Java Struts 安全审查

苏珊，作为一家软件公司的 CTO，寻求安全团队关于 Struts 的建议。苏珊理解，Struts 的安全审查不仅需要 Struts 的领域知识，还需要特定于 Struts 的威胁知识。为了识别 Struts 的安全性，需要自动化代码扫描、白盒审查、安全配置审查，以及带有恶意负载的黑盒测试，安全团队提出了以下结合行业实践资源的安全审查方法。此案例研究的目的是展示如何进行针对 Struts 安全性的白盒审查，而非提供全面的 Struts 安全审查指南。

苏珊和安全团队讨论了可能的审查方法，并为项目团队提供了一个 Struts 安全检查清单，作为代码审查的基准。

# Struts 安全审查方法

下表给出了 Java Struts 框架关键审查方法的示例：

| **Struts 安全审查方法** | **目标和参考资料** |
| --- | --- |
| Struts 安全检查 | 安全检查清单供开发人员用于执行 Struts 的安全实施和审查。Struts 官方网站提供了一个很好的参考。可以查看链接：[`struts.apache.org/security/`](https://struts.apache.org/security/)。 |
| Struts 潜在风险字符串 | 除了代码扫描外，我们还可以搜索可能导致 Struts 安全问题的特定字符串。对于 Struts 的安全性，我们更关注安全配置，`struts.xml`，而不是源代码。 |
| Struts 漏洞利用脚本 | 为了测试 Struts 的每个漏洞，建议参考已发布的漏洞利用脚本。参考：[`www.exploit-db.com/search/?action=search&q=struts`](https://www.exploit-db.com/search/?action=search&q=struts)。 |
| OWASP 依赖项 | 大多数已知的 Struts 漏洞在最新版本中已修复。OWASP 依赖项扫描工具可以帮助检测旧版本 Struts 的使用。查看链接：[`www.owasp.org/index.php/OWASP_Dependency_Check`](https://www.owasp.org/index.php/OWASP_Dependency_Check)。 |

# Struts 安全检查清单

安全检查清单将提醒团队在代码审查过程中应关注的重点。具体来说，对于 Struts 框架的安全性，Struts 安全实施检查清单总结了以下几点。Struts 安全参考资料的链接：[`struts.apache.org/security/`](https://struts.apache.org/security/)：

+   **配置浏览器插件**应仅在开发环境中使用

+   按安全级别将动作分组到一个命名空间中

+   将所有 JSP 文件放置在`WEB-INF`中，以避免 JSP 文件的直接访问

+   禁用开发模式`` `devMode` ``

+   在生产环境中减少日志记录级别

+   UTF-8 编码

+   验证`getText()`的输入数据参数

+   不要直接使用原始的`${} EL`表达式来处理输入参数

+   禁用静态方法访问

+   禁用动态方法调用

# 在`struts.xml`和 API 中搜索 Struts 安全字符串

这份直接与 Struts 安全问题相关的关键字列表将帮助我们使用搜索工具（如 drek 或 Graudit）定位并识别问题；请看以下表格：

| **Struts 安全性** | **粗体关键字搜索** |
| --- | --- |
| 开发模式 | `struts.devMode`。**审查提示**：建议值应在`struts.xml`中设置为 false。 |
| 动态方法调用 | `struts.enable.DynamicMethodInvocation`。**审查提示**：建议值应在`struts.xml`中设置为 false。 |
| OGNL 静态方法访问 | `struts.ognl.allowStaticMethodAccess`。**审查提示**：建议值应在`struts.xml`中设置为 false。 |
| 文件上传 | `Allowedtypes`.`maximumSize`.`allowedExtensions`.**审查提示**：这些参数应在`struts.xml`中定义，以限制文件上传的类型、大小和扩展名。请查看链接 [`struts.apache.org/core-developers/file-upload.html`](https://struts.apache.org/core-developers/file-upload.html)。 |
| 数据输入注入 | `findValue`，`getValue`，`setValue`。**审查提示**：审查这些 API 的外部输入参数，以避免`struts.xml`中的 OGNL 注入攻击。 |
| 验证 | `validate`。**审查提示**：验证的安全值应在`struts.xml`中设置为 true。 |
| 数据输入注入 | `request.getParameter`。**审查提示**：审查这些 API 的外部输入参数，以避免潜在的注入攻击。 |
| 类加载器操作 | `getClass`。**审查提示**：审查这些 API 的外部输入参数，以避免潜在的注入攻击。 |

# 总结

我们讨论了白盒审查的实践。为了进行有效的白盒审查，需要一些准备和输入，如源代码、威胁建模分析、架构和设计文档、自动化静态代码分析报告、配置文件和通信接口列表。

进行白盒源代码审查有几种方法。我们可以使用 doxygen 和 naturaldocs 从源代码生成文档和流程图。这将帮助我们全面了解源代码。然后，我们识别高风险模块进行手动代码检查。高风险模块是那些处理敏感信息、安全控制或管理功能的模块。

在白盒审查过程中，必须建立一个检查清单。这包括一些推荐的行业实践，如 OWASP 安全编码实践、OWASP 代码审查指南、CWE Top 25 和 OWASP Top 10。基于这些实践，建议组织可以构建自己最常见的安全问题及其缓解方法。

然后，最后但同样重要的是，我们讨论了安全编码模式和关键字。我们列出了针对安全问题的一些常见 Java 代码模式，并介绍了一些工具，如 drek、Graudit、VCG 和 CRASS Grep IT。

案例研究给出了一个特定于 Struts 框架的安全代码审查示例。在该案例中，团队应用了一些审查方法，并且定义了一个与 Struts 相关的安全检查清单。

在下一章节中，我们将探讨每个安全测试领域中的更多安全测试工具包。

# 问题

1.  以下哪项不是白盒审查的输入？

    1.  源代码

    1.  威胁建模文档

    1.  自动化静态代码分析结果

    1.  杀毒扫描结果

1.  doxygen 和 naturaldocs 工具的用途是什么？

    1.  直接从源代码生成文档

    1.  静态代码扫描

    1.  动态代码扫描

    1.  逆向工程

1.  以下哪项是高风险模块？

    1.  认证

    1.  授权

    1.  API 接口

    1.  以上所有

1.  以下哪个 API 与缓冲区溢出无关？

    1.  strcpy

    1.  strncat

    1.  memcpy

    1.  fwrite

1.  什么原因可能导致认证缺失？

    1.  使用部分 URL 匹配 API 来确定是否需要认证，如 StartsWith 和 EndsWith

    1.  在验证之前没有进行路径标准化

    1.  在验证之前没有进行数据规范化

    1.  以上所有

# 深入阅读

参考以下链接了解更多信息：

+   **US CERT 白盒测试**：[`www.us-cert.gov/bsi/articles/best-practices/white-box-testing/white-box-testing`](https://www.us-cert.gov/bsi/articles/best-practices/white-box-testing/white-box-testing)。

+   **Security Code Scan – .NET 静态代码分析器**：[`security-code-scan.github.io/`](https://security-code-scan.github.io/)

+   **SEI CERT 编码标准**：[`wiki.sei.cmu.edu/confluence/display/seccode/SEI+CERT+Coding+Standards`](https://wiki.sei.cmu.edu/confluence/display/seccode/SEI+CERT+Coding+Standards)。

+   **Find Security Bugs**：[`find-sec-bugs.github.io/`](http://find-sec-bugs.github.io/)。

+   **DevBug 是一个在线 PHP 安全代码分析 (SCA)**：[`www.devbug.co.uk/`](http://www.devbug.co.uk/)。

+   **MITRE 安全代码审查**：[`www.mitre.org/publications/systems-engineering-guide/enterprise-engineering/systems-engineering-for-mission-assurance/secure-code-review`](https://www.mitre.org/publications/systems-engineering-guide/enterprise-engineering/systems-engineering-for-mission-assurance/secure-code-review)。

+   **MITRE 网络威胁易感性评估**：[`www.mitre.org/publications/systems-engineering-guide/enterprise-engineering/systems-engineering-for-mission-assurance/cyber-threat-susceptibility-assessment`](https://www.mitre.org/publications/systems-engineering-guide/enterprise-engineering/systems-engineering-for-mission-assurance/cyber-threat-susceptibility-assessment)。

+   **PCI 优先级方法工具**：[`www.pcisecuritystandards.org/documents/Prioritized-Approach-v3_2.xlsx`](https://www.pcisecuritystandards.org/documents/Prioritized-Approach-v3_2.xlsx)。

+   **MSND 如何进行托管代码的安全代码审查**: [`cwiki.apache.org/confluence/display/WW/Security+Bulletins`](https://cwiki.apache.org/confluence/display/WW/Security+Bulletins)。

+   **Apache Struts CVE 列表**: [`www.cvedetails.com/vulnerability-list/vendor_id-45/product_id-6117/Apache-Struts.html`](https://www.cvedetails.com/vulnerability-list/vendor_id-45/product_id-6117/Apache-Struts.html)。

+   **Apache Struts 文件上传**: [`struts.apache.org/core-developers/file-upload.html`](https://struts.apache.org/core-developers/file-upload.html)。
