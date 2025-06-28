# 评估

# 第一章

1.  **基础设施即代码**（**IaC**）是 IT 基础设施的一种类型，通过代码自动管理和配置，而不是使用手动过程。

1.  IaC 的好处是最小化构建和部署基础设施的风险，并实现基础设施的自动化管理。

1.  AWS CloudFormation 的主要目标是通过 AWS 资源优化你的 IaC 设计，并提供一个完整的解决方案来基于 Amazon AWS 技术堆栈构建现代基础设施。

1.  AWS CloudFormation 旨在简化 IaC 开发。从使用文件（JSON 或 YAML）设计 IaC 或使用设计器生成 CloudFormation 模板文件开始。然后，上传该模板文件到 AWS CloudFormation 服务器。接着，CloudFormation 将生成我们在模板文件中已定义的所有资源，AWS 将在特定的容器中部署它们。这个过程是自动运行的。

1.  我们可以通过在文件中编写脚本或使用 CloudFormation 设计器来构建 CloudFormation 模板。我们可以使用 JSON 和 YAML 格式开发 CloudFormation 模板文件。

1.  在某些情况下，我们希望在基础设施环境中部署某个特定资源。这个资源由多个不同的资源组成，这些资源也被其他资源使用。这个场景可以通过在 CloudFormation 中实现嵌套堆栈来完成。资源模块化是轻松管理基础设施的关键。

1.  实现 CloudFormation StackSets 的目的是在不同区域部署同一资源单元。如果我们想要在不同区域部署相同的资源，可以使用 CloudFormation StackSets。你还可以在不同区域配置某些设置。

# 第二章

1.  CloudFormation 堆栈是 CloudFormation 的一个实例，它由一组 AWS 资源组成，基于 Amazon AWS 技术堆栈构建基础设施。

1.  使用 Web 管理控制台构建 CloudFormation 的主要好处是更易于使用，因为我们可以通过图形化操作进行。我们不需要记住 AWS CLI 的所有命令。

1.  通过 AWS CLI 构建 CloudFormation 可以带来简化的好处。因为 AWS CLI 在终端上运行，如果与使用管理控制台相比，它不需要更多的带宽。

# 第三章

1.  CloudFormation 模板是一个脚本文件，通常以 JSON 或 YAML 格式编写，用于基于 Amazon AWS 技术构建基础设施。

1.  要开发一个 CloudFormation 模板，你需要掌握编写 JSON 或 YAML 格式的知识。一个 CloudFormation 模板的骨架 JSON 格式如下：

```
{
 "AWSTemplateFormatVersion" : "version date",
 "Description" : "JSON string",
 "Metadata" : {
   template metadata
 },
 "Parameters" : {
   set of parameters
 },
 "Mappings" : {
   set of mappings
 },
 "Conditions" : {
   set of conditions
 },
 "Transform" : {
   set of transforms
 },
 "Resources" : {
   set of resources
 },
 "Outputs" : {
   set of outputs
 }
}
```

你也可以用 YAML 编写它：

```
AWSTemplateFormatVersion: "version date"
Description:
 String
Metadata:
 template metadata
Parameters:
 set of parameters
Mappings:
 set of mappings
Conditions:
 set of conditions
Transform:
 set of transforms
Resources:
 set of resources
Outputs:
 set of outputs
```

1.  要实现 CloudFormation 模板，你可以手动在文件中编写 CloudFormation 的 JSON 或 YAML。你也可以通过 AWS 的 CloudFormation 设计工具来构建模板。然后，你应该了解 AWS 资源类型。你可以在 [`docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-product-property-reference.html`](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-product-property-reference.html) 上阅读相关内容。最后，你可以做更多的实践。你可以使用 AWS 上的模板示例，链接在 [`docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-sample-templates.html`](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-sample-templates.html)。

# 第四章

1.  CloudFormation StackSet 是一组跨账户和区域部署的 Stack 集合。你可以在一些区域执行多个区域的预配。

1.  根据 AWS 文档，我们最多可以在管理员账户中创建 20 个 StackSet，每个 StackSet 最多包含 500 个 Stack 实例。参考链接：[`docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/stacksets-limitations.html`](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/stacksets-limitations.html)

# 第五章

1.  一般来说，我们可以使用管理控制台和 AWS CLI 部署 AWS Lambda 函数，步骤如下：

+   +   基于这个模板创建 CloudFormation 模板，链接：[`docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-lambda-function.html`](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-lambda-function.html)

    +   使用 Web 管理控制台或 AWS CLI 部署模板

    +   配置 IAM 权限，以便能够调用 Lambda 函数的用户

1.  要为 Lambda 函数开发 CloudFormation 模板，你应该按照 [`docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-lambda-function.html`](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-lambda-function.html) 上的模板说明进行操作。

以下是 JSON 模板：

```
{
  "Type" : "AWS::Lambda::Function",
  "Properties" : {
    "Code" : Code,
    "DeadLetterConfig" : DeadLetterConfig,
    "Description" : String,
    "Environment" : Environment,
    "FunctionName" : String,
    "Handler" : String,
    "KmsKeyArn" : String,
    "MemorySize" : Integer,
    "ReservedConcurrentExecutions" : Integer,
    "Role" : String,
    "Runtime" : String,
    "Timeout" : Integer,
    "TracingConfig" : TracingConfig,
    "VpcConfig" : VPCConfig,
    "Tags" : [ Resource Tag, ... ]
  }
}
```

以下是 YAML 模板：

```
Type: "AWS::Lambda::Function"
Properties: 
  Code:
    Code
  DeadLetterConfig:
    DeadLetterConfig
  Description: String
  Environment:
    Environment
  FunctionName: String
  Handler: String
  KmsKeyArn: String
  MemorySize: Integer
  ReservedConcurrentExecutions: Integer
  Role: String
  Runtime: String
  Timeout: Integer
  TracingConfig:
    TracingConfig
  VpcConfig:
    VPCConfig
  Tags: 
    Resource Tag
```

1.  首先，我们需要在 CloudFormation 上配置权限。你应该部署以下 Stack 文件：`AWSCloudFormationStackSetAdministrationRole.yml` 和 `AWSCloudFormationStackSetExecutionRole.yml` 文件。阅读 第四章，*AWS CloudFormation StackSets* 以了解更多步骤。然后，你可以像平常一样部署 StackSet。
