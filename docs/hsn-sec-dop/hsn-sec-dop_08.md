# 安全编码最佳实践

安全架构设计和威胁建模后是安全编码阶段。在编码阶段，我们希望避免使用不安全的 API、缓冲区溢出、敏感信息泄露等问题。每个开发人员都很难熟悉所有的安全编码规则。因此，如何运用安全编码工具和技巧来发现主要的安全问题，将在本章中讨论。

本章将涵盖以下主题：

+   安全编码行业最佳实践

+   建立安全编码基准

+   安全编码意识培训

+   工具评估

+   工具优化

+   高风险模块审查

+   手动代码审查工具

+   安全代码扫描工具

+   安全编译

+   实践中的常见问题

# 安全编码行业最佳实践

安全编码是安全软件的基础。我们已经进行了威胁建模和安全架构设计。这些都需要通过安全编码来实现。安全编码对于开发团队来说可能是一个挑战，因为开发人员通常忙于开发新特性，且可能需要学习数百条安全编码规则。在我们更详细地讨论安全编码实践之前，我们将回顾一下可以参考的现有安全编码标准。

根据编程语言的不同，安全编码标准总结如下表所示：

| **参考标准** | **描述与参考** |
| --- | --- |
| CERT 安全编码 |

+   该标准为 C、C++、Java、Perl 和 Android 提供了安全编码标准。

|

| 查找安全漏洞 |
| --- |

+   它提供了 Java 的漏洞代码样本和解决方案的错误模式。

|

| CWE |
| --- |

+   它提供了从常见软件弱点的角度出发的漏洞源代码示例。编码示例涵盖 C、C++、Java 和 PHP。

|

| Android |
| --- |

+   Android 应用程序安全设计与安全编码指南

|

| OWASP SKF |
| --- |

+   OWASP 安全知识框架。

+   它可以作为一个内部安全知识库，包括 OWASP ASVS 和安全编码知识。

|

| PHP 安全 |
| --- |

+   OWASP PHP 安全备忘单

|

| OWASP 代码审查 |
| --- |

+   OWASP 代码审查项目

|

| Apple 安全编码指南 |
| --- |

+   Apple 安全编码指南

|

| Go |
| --- |

+   GO 语言的安全编码实践

|

| JavaScript |
| --- |

+   JavaScript 安全编码实践

|

| Python |
| --- |

+   OWASP Python 安全项目

|

我们理解了安全编码基准和标准。此外，关键挑战在于如何将这些安全编码规则应用到开发人员的日常编码活动中。以下是进行安全编码实践的推荐方法。

# 建立安全编码基准

安全编码基准是最基本的安全编码要求，也是项目团队向下一个阶段推进的检查清单。安全编码基准还是发布标准的一部分。建议始终使用基于行业最佳实践或标准的安全编码指南，例如前表中描述的 **CERT 安全编码**。

根据每种编程语言（如 PHP、Python、JavaScript、Android 和 iOS）定义安全编码基准。安全编码基准最好不仅包括安全编码规则，还包括安全风险、易受攻击的代码示例和建议的解决方案。以下是一个示例。

**安全代码问题 – 可预测的随机数**：

使用可预测的随机数可能会导致会话 ID、令牌或加密初始化向量的漏洞。建议使用`java.security.SecureRandom`代替`java.util.Random`：

**`// 易受攻击的代码`**

`Random rnd = New Random ();`

**`// 建议的代码`**

`SecureRandom rnd = SecureRandom();`

所有项目在发布前必须使用指定的代码扫描工具进行扫描。某些组织还可能会定义安全编码实践的发布标准。以下是一些示例：

+   扫描工具生成的扫描结果中的所有警告都必须检查

+   所有编译器警告（不仅仅是错误）都应检查并清除

+   扫描结果中每行代码的开放缺陷数不能超过某个百分比

此外，安全编码基准还需要相关的开发工具和实践中的培训；否则，这些安全编码规则将仅仅是文档。

# 安全编码意识培训

安全编码培训的目的是让开发团队了解我们即将执行的安全编码实践。在安全编码意识培训的初期阶段，重点将主要集中在以下内容：

+   什么是安全编码标准或基准？

+   常见的行业安全编码问题有哪些？

+   它们会如何影响开发者的日常任务？

+   安全代码扫描的发布标准

一个基于案例研究或场景的易受攻击的源代码示例，比单纯的安全编码规则具有更好的培训效果。以下是该领域的良好参考，提供了大量易受攻击的代码和安全最佳实践代码示例：

+   **OWASP 安全知识框架**：[`www.securityknowledgeframework.org/`](https://www.securityknowledgeframework.org/)

+   **安卓应用安全设计与安全编码指南**：[`www.jssec.org/dl/android_securecoding_en.pdf`](http://www.jssec.org/dl/android_securecoding_en.pdf)

+   **查找 Java 安全漏洞模式**：[`find-sec-bugs.github.io/`](https://find-sec-bugs.github.io/)

+   **OWASP Teammentor**：[`owasp.teammentor.net/angular/user/index`](https://owasp.teammentor.net/angular/user/index)

# 工具评估

一旦团队意识到安全编码的重要性和挑战，它将寻找一些工具来简化安全编码。扫描工具的评估可能包括以下考虑因素：

| **考虑因素** | **描述** |
| --- | --- |
| 可用性 |

+   代码扫描工具的目标用户是开发者。可用性包括扫描源代码部分的能力、差异扫描、扫描报告、追溯到原始源代码等。

|

| 预算 |
| --- |

+   如果它是一个 IDE 插件商业工具，我们需要考虑它将需要多少并发用户许可证。

|

| 编程语言支持 |
| --- |

+   大多数工具支持 C/C++和 Java，但不支持脚本语言，如 Python、JavaScript 或 PHP。

+   对公司内部项目使用的编程语言进行调查，并优先支持即将使用的编程语言。

|

| 检测率和误报率 |
| --- |

+   任何扫描工具都有误报率，这取决于扫描引擎和规则。高误报率并不一定是坏事，它也可能意味着扫描器采取了更保守的策略。选择最适合项目的工具，而不是选择最知名的工具。

+   为了评估检测率，我们可以使用已知的漏洞项目。

|

| 扫描规则更新 |
| --- |

+   工具持续更新规则和扫描器非常重要。商业工具的一个关键优势是它们会有最新的扫描规则。

|

通常有两种代码扫描方式。第一种是使用 IDE 插件进行静态代码扫描，像拼写检查一样工作，对开发者来说更直观，方便学习和修正安全问题。第二种是使用每日构建进行代码扫描，生成每日扫描报告。开发者需要查看每日扫描报告，批量修复或评论安全问题。这种方式对于开发者来说可能不那么直观，但编译后的安全扫描可能提供更高的准确性。为了促进这两种扫描工具的采用，建议从小规模的试点团队开始。市场上有一些商业和开源工具，支持这两种扫描方式。

|  | **优点** | **缺点** |
| --- | --- | --- |
| **IDE 插件静态代码扫描** | 对开发者直观（像拼写检查一样工作）。 |

+   它可能会产生较高的误报。

+   它要求每个开发者都安装插件。

+   检测能力有限。

+   每个开发者的许可证费用。

+   需要强制要求每个开发者使用工具。

|

| **每日编译扫描** |
| --- |

+   基于项目集成和编译扫描的安全扫描准确性。

+   集中管理扫描规则和结果。

+   它容易建立安全指标并监控每个项目的结果。

|

+   需要完整可构建的源代码。

+   扫描结果需要进一步分配给开发者检查。当开发者被指派去检查报告时，他可能不熟悉其他模块。

|

代码扫描工具的评估包括检测率、假阳性率、潜在开销以及开发团队的可用性。用于静态代码扫描工具评估的脆弱代码项目列在下表中：

| **脆弱项目** | **描述** | **编程语言** |
| --- | --- | --- |
| **NIST 软件保障参考数据集项目** | 该项目提供了故意设计的不安全代码示例，可用于测试安全代码扫描工具的检测率。 | Java, C/C++, C#, PHP |
| **OWASP Node JS Goat** | 这是一个脆弱的网站，用于练习 OWASP 前 10 大安全测试，使用 NodeJS 构建。 | Node JS |
| **OWASP WebGoat .Net** | 这是一个脆弱的网站，用于练习 OWASP 前 10 大安全测试，使用 .NET 构建。 | .NET |
| **OWASP WebGoat PHP** | 这是一个脆弱的网站，用于练习 OWASP 前 10 大安全测试，使用 PHP 构建。 | PHP |
| **OWASP Rail****sGoat** | 这是一个脆弱的网站，用于练习 OWASP 前 10 大安全测试，使用 Ruby 构建。 | Ruby on Rails |

一旦安全团队在测试结果后选择了扫描工具，安全团队可能会与更多的开发团队进行沟通，讨论工具的采用。在工具采用之前，建议通过演示使用结果、处理扫描结果以及使用扫描工具来进行实践培训。

这个阶段的培训更多侧重于*如何做*而不是*做什么*。实践教程的示例包括如何使用扫描工具，如何审查安全问题，如何根据扫描结果进行修复，如何禁用某些扫描规则等。

# 工具优化

一旦团队使用了代码扫描工具一段时间，安全团队可能会根据用户反馈帮助优化工具、流程或规则。以下是大规模代码扫描采用中需要优化的一些关键因素：

| **关键因素** | **建议** |
| --- | --- |
| 扫描规则定制 | 规则定制的目的是帮助项目团队减少假阳性。安全团队可能会帮助禁用一些不适用于项目的规则，或者修改那些总是导致假阳性的规则。 |
| 推荐修复 | 理想情况下，IDE 插件不仅会显示安全警告，还会提供修复建议。然而，如果所使用的工具不支持团队，使用 OWASP 安全知识框架可以作为替代方案。 |
| 集成 | 将代码扫描工具集成到 Jenkins 和开发者的 IDE 插件中。自动化框架。与 Jenkins 的集成是 CI/CD 的基本组成部分。 |
| 报告 | 团队可能会请求进一步的质量指标报告，例如基于之前检查结果的增量扫描报告或跨项目的常见问题报告。 |

| 自动化平台 | 将安全编码自动化提升到下一个水平涉及将多个工具集成到自动化平台中。尝试以下开源工具来构建自己的安全编码自动化平台：

+   **SWAMP-in-a-Box**：

+   **JackHammer**：

|

# 高风险模块审查

自动化代码扫描工具可以帮助检测大多数源代码的安全问题。然而，对于高风险模块，仍然需要额外的关注。除了源代码扫描工具，我们还将应用黑盒测试或**动态应用安全测试**（**DAST**），这些将在后续章节中讨论。像黑客一样思考。黑客会对哪些模块感兴趣？哪些信息对黑客来说最有价值？整个应用程序中最薄弱的环节可能是什么？下表列出了需要进一步审查的高风险模块：

| **高风险模块** | **安全审查重点** |
| --- | --- |
| 认证 |

+   账户注册

+   登录和验证码

+   密码恢复或重置

+   密码更改

+   身份和密码存储及访问控制

+   多次失败后的账户锁定控制

|

| 授权 |
| --- |

+   敏感资源访问

+   管理员管理

|

| 配置 |
| --- |

+   配置中有两种审查类型：

    +   应用程序的安全配置，例如关闭调试模式和启用 TLS 通信。

    +   每个软件版本的配置影响。

|

| 财务 |
| --- |

+   支付功能

+   订单和购物车

|

| 文件处理 |
| --- |

+   文件上传

+   文件下载

|

| 数据库 |
| --- |

+   数据库查询

+   数据库的添加、更新和删除

|

| API 接口 |
| --- |

+   Restful API 接口

+   第三方集成接口

|

| 遗留 |
| --- |

+   不支持安全通信的模块

+   可能仍使用弱加密算法的模块

+   使用禁止或危险的 API

|

| 加密 |
| --- |

+   使用禁止的加密算法

+   在开发过程中源代码中硬编码的敏感信息或注释，例如 IP、电子邮件、密码或隐藏的快捷键

|

| 会话 |
| --- |

+   并发会话控制和检测

+   会话 ID 和过期时间的随机性

|

# 手动代码审查工具

手动代码审查可能需要一些时间。没有适当的工具和策略的手动代码审查就像是在大海捞针。如前所述，我们只对特定的高风险模块进行手动代码审查，而不是整个项目。除了选择目标范围，工具还可以帮助我们更高效地进行手动代码审查。以下是一些推荐的开源工具，虽然这些工具并不是专门为此目的设计的，但它们有助于提高源代码审查的效率：

| **工具** | **使用场景** |
| --- | --- |
| **AndroGuard** |

+   这包括许多 Python 分析模块，用于对 Android 应用进行逆向工程分析。

+   生成的图表可以通过 Gephi 查看。

|

| **Doxygen** |
| --- |

+   该工具支持多种编程语言，能够生成在线 HTML 或 PDF 文档。它还可以生成可由 Graphviz 查看的函数调用图。

+   它有助于我们概览程序并识别出我们应重点关注的高风险模块。

|

| **Kscope** |
| --- |

+   该工具可以分析 C 源代码，生成调用函数的树和调用图。

|

| **OpenGrok** |
| --- |

+   它提供类似 Google 的语法和正则表达式全文源代码搜索。它还可以基于搜索结果进行交叉引用。

|

| **WinMerge** |
| --- |

+   它可以比较两个文件和文件夹之间的差异。比较结果以视觉颜色呈现。在我们查找不同版本间的代码变更时非常有用。

+   对于非 Windows 平台，KDiff3 或 Meld 是替代的开源选项：

    +   [`kdiff3.sourceforge.net/`](http://kdiff3.sourceforge.net/)

    +   [`meldmerge.org/`](http://meldmerge.org/)

|

| **NCC Code Navi** |
| --- |

+   NCC Code Navi 工具的主要优势是能够在源代码文件中进行关键词搜索。右键点击启动 CERT 搜索代码搜索也是非常有用的。

|

# 安全代码扫描工具

在源代码扫描方面，没有一种适合所有的解决方案。也没有任何扫描工具可以做到零假阳性并具有 100%准确的检测率。因此，对于相同的编程语言，我们通常会应用至少两个扫描工具进行交叉验证。

这里列出的是一些 2018 年常用的开源安全编码分析工具。请注意，我们这里只列出了开源工具：

| **Tools** | **扫描工具的背景和关键特性** |
| --- | --- |
| **Retire.JS** |

+   检测易受攻击的 JavaScript 库，如 jQuery、AngularJS、Node 等。

+   它提供命令行、grunt 插件，并且还提供 OWASP ZAP 插件用于集成扫描。

|

| **Clang Static Analyzer** | 该工具提供 C、C++和 Objective C 的独立命令行分析。 |
| --- | --- |
| **Flawfinder** | 一个简单的 C/C++代码扫描工具。它是一个 Python 命令行扫描工具，可以根据需求轻松定制。 |
| **DREK** |

+   它像 GREP 一样通过正则表达式搜索特定的安全问题，但它可以生成 PDF 或 HTML 格式的扫描结果。

+   它可以通过正则表达式轻松扩展任何扫描规则，且可用于扫描任何编程语言。

|

| **Pylint** | Pylint 是 Python 编程语言的源代码检查工具。 |
| --- | --- |
| **PHPMD** |

+   PHP Mess Detector 是一个 PHP 源代码扫描工具。

|

| **DawnScanner** | Ruby Web 应用程序的安全扫描工具。 |
| --- | --- |
| **SpotBugs** |

+   这提供了一个独立的 GUI 和命令行界面。

+   SpotBugs 还可以作为 Eclipse 插件使用。它是 FindBugs 的继任者。

|

| **CPP Check** | 这是一个用于 C/C++的静态代码分析工具。 |
| --- | --- |
| **移动安全框架（MobSF）** | 移动安全框架是一个完全自动化的 Android 应用程序扫描工具。开发者只需上传 APK 到 MSF，MSF 将自动完成所有分析。 |
| **Clang 静态分析器** | 这是一个 C/C++和 Objective C 的代码分析工具。 |
| **ESLint** |

+   这是一个用于 JavaScript 的命令行代码扫描工具。

+   请参见此链接查看安全代码扫描规则：[`eslint.org/docs/rules/`](https://eslint.org/docs/rules/)。

|

| **JSHint** | 这是一个用于 JavaScript 代码扫描的工具，并且提供了 NodeJS 的命令行工具。 |
| --- | --- |
| **Infer** | 这是一个由 Facebook 提供的 Java、C/C++和 Objective C 的静态代码分析器。 |
| **Phan** | Phan 是一个 PHP 静态分析工具。 |
| **PHP 安全检查器** | 这个工具检查 PHP 项目依赖项中的已知安全问题。 |
| **OWASP Dependency check** | 该工具支持多种编程框架，并检查已公开的漏洞，使用更新的 NVD 数据源。该工具可以通过命令行运行或与 Jenkins 集成运行。 |
| **VisualCodeGrepper** (**VCG**) | VCG 是一个语言无关的扫描工具。扫描规则也可以通过正则表达式轻松定制。它还提供了针对常见禁止 API 的默认规则，并提供 GUI 和命令行工具来扫描任何源代码。 |
| **PMD** | 这是一个用于 Java 和 JavaScript 的源代码分析工具，主要用于检查常见编程缺陷。 |
| **Graudit** | 这是一个简单的脚本，利用 GREP 搜索特定的代码模式来查找潜在的安全问题。签名数据库模板提供了查找线索。 |
| **SonarQube** | 这个工具支持超过 20 种语言，并且可以与 CI 框架集成。它也提供 UI，便于查看质量代码扫描结果。 |
| **Brakeman** | Ruby on Rails 的静态分析安全扫描器。 |
| **bandit** | Python 源代码的安全分析工具。 |
| **Error Prone** | Error Prone 在编译时检测潜在的 Java 错误。 |
| **Dawn** | Dawn 是一个用于 Ruby Web 应用的静态分析安全扫描器。 |

下面是按语言的另一种分类：

| **编程语言** | **扫描工具** |
| --- | --- |
| C/C++ |

+   Infer

+   CPP Check

+   Flawfinder

+   Clang 静态分析器

|

| Java |
| --- |

+   Infer

+   SpotBugs

+   PMD

|

| Android |
| --- |

+   MobSF

|

| PHP |
| --- |

+   Phan

+   PHPMD

|

| Ruby |
| --- |

+   DawnScanner

|

| Python |
| --- |

+   Pylint

|

| JavaScript |
| --- |

+   ESLint

+   JSHint

+   Retire.JS

+   PMD

|

| 依赖漏洞 |
| --- |

+   OWASP Dependency check

+   PHP 安全检查器

+   Retire.JS

|

| 语言无关 |
| --- |

+   SonarQube

+   DREK

+   Graudit

+   VisualCodeGrepper

|

由于规则和检测引擎的能力，相同编程语言的扫描结果可能会有所不同。建议使用两个扫描工具对同一语言进行扫描。例如，使用一个商业工具进行日常编译扫描，另一个开源工具用于开发者的 IDE 插件。使用商业扫描工具有助于向客户说明服务是如何进行测试的，而开源扫描工具则为进一步定制和大规模部署提供了灵活性，无需预算限制。

# 安全编译

内存损坏和缓冲区溢出可能导致利用代码注入攻击。对于 C/C++编程语言，这些可以通过编译器选项进行保护。通过适当配置 C/C++编译器（GCC，MS Visual Studio），应用程序可以增加额外的运行时防御层，以防止利用代码注入攻击。这些通常被开发团队忽视。常见的安全选项总结如下表：

| **保护技术** | **安全选项** | **操作系统/编译器** |
| --- | --- | --- |
| 地址空间布局随机化（ASLR） | `echo 1 >/proc/sys/kernel/randomize_va_space` | Android, Linux 操作系统 |
| 基于栈的缓冲区溢出保护 | `-fstack-protector  –fstack-protector-all` | gcc |
| GOT 表保护 | `-Wl`, `-z`, `relro` | gcc |
| 动态链接路径 | `-Wl,--disable-new-dtags,--rpath [path]` | gcc |
| 不可执行栈 | `-Wl,-z,noexecstack` | gcc |
| 镜像随机化 | `–fpie –pie` | gcc |
| 不安全的 C 运行时函数检测 | `–D_FORTIFY_SOURCE=2  –Wformat-security` | gcc |
| 基于栈的缓冲区溢出防护（Canary） | `/GS` | MS（微软）Visual C++ |
| 地址空间布局随机化（ASLR） | `/DYNAMICBASE` | MS Visual C++ |
| CPU 级别的不可执行（NX）支持。数据执行防护（DEP） | `/NXCOMPAT` | MS Visual C++ |
| 安全结构化异常处理 | `/SAFESEH` | MS Visual C++ |
| 启用附加安全检查 | `/SDL` | MS Visual C++ |

进一步参考和每种保护技术的描述，请查看以下参考资料：

+   **SAFECode 开发实践**：[`www.safecode.org/publication/SAFECode_Dev_Practices0211.pdf`](https://www.safecode.org/publication/SAFECode_Dev_Practices0211.pdf)

+   **OWASP 基于 C 的工具链硬化**：[`www.owasp.org/index.php/C-Based_Toolchain_Hardening`](https://www.owasp.org/index.php/C-Based_Toolchain_Hardening)

+   **Linux 审核 ASLR**：[`linux-audit.com/linux-aslr-and-kernelrandomize_va_space-setting/`](https://linux-audit.com/linux-aslr-and-kernelrandomize_va_space-setting/)

+   **MS 安全最佳实践对于 C++**：[`msdn.microsoft.com/en-us/library/k3a3hzw7.aspx`](https://msdn.microsoft.com/en-us/library/k3a3hzw7.aspx)

+   **GCC 的安全编译器和链接器标志**：[`developers.redhat.com/blog/2018/03/21/compiler-and-linker-flags-gcc/`](https://developers.redhat.com/blog/2018/03/21/compiler-and-linker-flags-gcc/)

为了验证应用程序或环境是否已配置安全选项，以下工具非常有用：

+   **CheckSec**: [`www.trapkit.de/tools/checksec.html`](http://www.trapkit.de/tools/checksec.html)

+   **BinScope**: [`www.microsoft.com/en-us/download/details.aspx?id=44995`](https://www.microsoft.com/en-us/download/details.aspx?id=44995)

# 实践中的常见问题

市面上有许多商业和开源的安全编码工具。有哪种工具能够提供低假阳性率和高检测率吗？

回答：没有完美或卓越的工具能够提供高检测率并保持低假阳性率。每个工具提供的扫描结果不同。较高的阳性率也可能意味着更保守的扫描，识别更多潜在或可疑的代码问题。你会发现不同工具的检测率和扫描结果也有所不同。工具 A 可能能够检测到工具 B 无法识别的问题，反之亦然。在实际操作中，也建议使用至少两种工具进行代码扫描作为交叉参考审查。

扫描结果可能列出超过 1000 个问题。对于如何处理这些问题，有什么建议吗？

回答：对于大规模项目，出现此类问题是非常常见的。开发团队检查扫描工具识别出的所有问题可能会感到不知所措。这里有一些可以考虑的方法：

+   首先筛选并评估那些评分为高风险的问题。

+   根据项目自定义扫描规则，过滤掉与项目无关的规则。

+   对新添加或最近更改的源代码范围进行增量扫描。这可能取决于扫描工具是否提供增量扫描功能。

+   对相同根本原因/问题进行分类。也许 50%的问题是由相同的根本原因引起的，例如使用相同的模块。

# 总结

我们已经讨论了安全编码行业的最佳实践，例如 CERT、CWE、Android 安全编码、OWASP 代码审查和 Apple 安全编码指南。基于这些安全编码规则，我们建立了安全编码基准，作为安全政策和发布标准的一部分。为了让团队熟悉安全编码，我们准备了一个培训门户。建议安全编码知识门户不仅应提供编码规则，还应提供案例研究。

为了将安全编码应用到开发者的日常工作中，必须采用安全编码工具。我们评估了安全编码工具，考虑了可用性、预算、编程语言支持、检测率和扫描规则维护等因素。为了评估扫描工具的检测率，我们还引入了一些可作为测试项目的漏洞项目。

安全编码规则和最佳实践是指导方针。它们需要正确的安全编码工具来实现，同时还需要正确的方法以使其更加高效和有效。因此，我们讨论了代码审查方法以及高风险模块的示例。为了更高效地进行高风险模块的手动代码审查，我们还列出了一些有助于审查的工具。最后，我们列出了适用于不同编程语言的常见开源安全代码扫描工具。

在下一章中，我们将通过一个案例研究，介绍开发阶段的安全需求、威胁建模、安全架构、设计和实现。

# 问题

1.  以下哪个不包含在 CERT 安全编码标准中？

    1.  C/C++

    1.  Java

    1.  Android

    1.  PHP

1.  Find Security Bugs 主要用于以下哪种编程语言？

    1.  C/C++

    1.  Python

    1.  Java

    1.  Go

1.  以下哪个可以作为安全编码的发布标准？

    1.  所有源代码必须使用指定的代码扫描工具进行审查。

    1.  所有编译器警告应检查并清除。

    1.  扫描工具生成的所有警告必须进行检查

    1.  以上所有

1.  使用易受攻击的项目来评估代码扫描工具的主要目的是什么？

    1.  检测率和误报率

    1.  预算

    1.  许可证

    1.  性能

1.  以下哪个不防止缓冲区溢出漏洞代码注入？

    1.  地址空间布局随机化（ASLR）

    1.  CSRF 令牌

    1.  基于栈的缓冲区溢出保护

    1.  非执行栈

1.  以下哪个不用于扫描依赖性漏洞？

    1.  OWASP 依赖性检查

    1.  PHP 安全检查工具

    1.  Retire.JS

    1.  VisualCodeGrepper

1.  哪个是自动化的移动安全测试框架？

    1.  MobSF

    1.  OpenGrok

    1.  Retire.JS

    1.  SonarQube

1.  以下哪个工具不用于 Android 安全评估？

    1.  AndroGuard

    1.  MobSF（移动安全框架）

    1.  Flawfinder

    1.  SpotBugs

# 深入阅读

+   **NIST 500-297 静态分析工具报告**: [`nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.500-297.pdf`](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.500-297.pdf)

+   **Android 安全编码**: [`www.jssec.org/dl/android_securecoding_en.pdf`](https://www.jssec.org/dl/android_securecoding_en.pdf)

+   **OWASP PHP 安全备忘单**: [`www.owasp.org/index.php/PHP_Security_Cheat_Sheet`](https://www.owasp.org/index.php/PHP_Security_Cheat_Sheet)

+   **PHP 安全手册**: [`php.net/manual/en/security.php`](https://php.net/manual/en/security.php)

+   **OWASP 代码审查**: [`www.owasp.org/index.php/Category:OWASP_Code_Review_Project`](http://Https://www.owasp.org/index.php/Category:OWASP_Code_Review_Project)

+   **OWASP 安全编码实践**: [`www.owasp.org/index.php/OWASP_Secure_Coding_Practices_-_Quick_Reference_Guide`](https://www.owasp.org/index.php/OWASP_Secure_Coding_Practices_-_Quick_Reference_Guide)

+   **Apple 安全编码指南**: [`developer.apple.com/library/content/documentation/Security/Conceptual/SecureCodingGuide/Introduction.html`](https://developer.apple.com/library/content/documentation/Security/Conceptual/SecureCodingGuide/Introduction.html)

+   **Salesforce 安全性**: [`developer.salesforce.com/devcenter/security`](https://developer.salesforce.com/devcenter/security)

+   **OWASP Python 安全性**: [`www.pythonsecurity.org/`](http://www.pythonsecurity.org/)

+   **云应用安全开发的 SAFE 实践**: [`safecode.org/wp-content/uploads/2018/01/SAFECode_CSA_Cloud_Final1213.pdf`](https://safecode.org/wp-content/uploads/2018/01/SAFECode_CSA_Cloud_Final1213.pdf)

+   **C/C++ 禁用 API**: [`github.com/Microsoft/ChakraCore/blob/master/lib/Common/Banned.h`](https://github.com/Microsoft/ChakraCore/blob/master/lib/Common/Banned.h)

+   **超赞静态代码分析**: [`github.com/mre/awesome-static-analysis`](https://github.com/mre/awesome-static-analysis)

+   **Oracle Java 安全编码指南**: [`www.oracle.com/technetwork/java/seccodeguide-139067.html`](http://www.oracle.com/technetwork/java/seccodeguide-139067.html)

+   **FindSecBugs Java 漏洞模式**: [`Find-sec-bugs.github.io/bugs.htm`](https://Find-sec-bugs.github.io/bugs.htm)

+   **SEI CERT 安全编码标准**: [`wiki.sei.cmu.edu/confluence/display/seccode/SEI+CERT+Coding+Standards`](https://wiki.sei.cmu.edu/confluence/display/seccode/SEI+CERT+Coding+Standards)

+   **MITRE CWE 白皮书 V3.1**: [`cwe.mitre.org/data/published/cwe_v3.1.pdf`](https://cwe.mitre.org/data/published/cwe_v3.1.pdf)

+   **CheckMarx Go 安全编码**: [`checkmarx.gitbooks.io/go-scp/`](https://checkmarx.gitbooks.io/go-scp/)

+   **CheckMarx JavaScript 安全编码**: [`checkmarx.gitbooks.io/js-scp/`](https://checkmarx.gitbooks.io/js-scp/)
