# 为 MaterialX 做出贡献

感谢您对为 MaterialX 做出贡献的兴趣！本文档解释了我们的贡献流程和程序。

## 社区和讨论

有两种主要方式可以与 MaterialX 社区联系：

- [学院软件基金会 Slack](http://academysoftwarefdn.slack.com/) 的 MaterialX 频道。
此平台适用于一般问题、功能请求和对 MaterialX 项目整体的讨论。
您可以在 https://www.aswf.io/get-involved/ 请求加入学院软件基金会 Slack 的邀请。
- MaterialX GitHub 的 [Issues](https://github.com/AcademySoftwareFoundation/MaterialX/issues)
面板，用于跟踪错误以及讨论功能请求。

### 如何寻求帮助

如果您在安装、构建或使用库时遇到困难，但还没有理由怀疑您遇到了真正的错误，请先在
[学院软件基金会 Slack](http://academysoftwarefdn.slack.com/) 的 MaterialX 频道发布问题。
这是适合诸如“我如何...”之类问题的地方。

### 如何报告错误

MaterialX 使用
[GitHub Issues](https://github.com/AcademySoftwareFoundation/MaterialX/issues)
来报告和跟踪错误、增强功能和功能请求。

如果您提交错误报告，请务必注明您使用的是哪个版本的
MaterialX，在什么平台上（操作系统/版本、使用的编译器，
以及任何特殊的构建设置标志或其他异常环境问题）。请
具体说明以下内容，并提供足够的细节以便其他人可以
重现问题：

- 您尝试了什么。
- 发生了什么。
- 您期望发生什么。

### 如何报告安全漏洞

如果您认为在 MaterialX 中发现了潜在的安全漏洞，请参考
[SECURITY.md](SECURITY.md) 以负责任地披露它。

### 如何贡献错误修复或更改

要为项目贡献代码，您需要：

- 基本的 Git 知识。
- GitHub 上 MaterialX 存储库的 fork。
- 对项目开发工作流程的理解。
- 法律授权，即您需要签署贡献者
  许可协议。详见下文。

## 法律要求

MaterialX 是学院软件基金会的一个项目，遵循 Linux 基金会的开源软件最佳实践政策。

### 许可证

MaterialX 采用 [Apache 2.0](LICENSE.md) 许可证授权。
对项目的贡献应遵守该标准许可证。

### 贡献者许可协议

要为 MaterialX 做出贡献，您必须通过 *EasyCLA* 系统签署贡献者许可协议，该系统已与 GitHub 集成为拉取请求检查。

在提交拉取请求之前，您可以通过[此链接](https://organization.lfx.linuxfoundation.org/foundation/a09410000182dD2AAI/project/a092M00001KWrdoQAD/cla)签署表格。如果您在签署表格之前提交拉取请求，EasyCLA 检查将失败并显示红色 *NOT COVERED* 消息，您将有机会通过提供的链接再次签署表格。

- 如果您是个人，在自己的时间编写代码，并且确定您是所贡献的任何知识产权的唯一所有者，您可以作为个人贡献者签署 CLA。
- 如果您在工作中编写代码，或者您的雇主保留对您创建的知识产权的所有权，那么您公司的法律事务代表应签署公司贡献者许可协议。如果您的公司已经有签署的 CCLA 存档，请询问您的本地 CLA 管理员将您添加到公司的批准列表中。

MaterialX CLA 是 Linux Foundation 项目使用的标准表格，并[由 ASWF TAC 推荐](https://github.com/AcademySoftwareFoundation/tac/blob/main/process/contributing.md#contributor-license-agreement-cla)。

## 开发工作流程

### Git 基础

使用 MaterialX 需要基本了解 Git 和 GitHub 术语。如果您不熟悉这些概念，请查看 [GitHub 词汇表](https://help.github.com/articles/github-glossary/) 或浏览 [GitHub 帮助](https://help.github.com/)。

要做出贡献，您需要一个 GitHub 帐户。这是必需的，以便通过拉取请求将更改推送到上游存储库。

您还需要在本地开发机器上安装 [Git](https://git-scm.com/doc) 或 Git 客户端，如 [Git Fork](https://git-fork.com/) 或 [GitHub Desktop](https://desktop.github.com/download/)。

### 存储库结构和提交策略

MaterialX 项目中的开发工作通常直接在 `main` 分支上完成。此分支代表项目的最前沿，大部分新工作在此提出、测试、审查和合并。

在 `main` 分支上完成足够的工作后，MaterialX 领导层确定需要发布时，他们将用当前版本标签标记发布，并为未来的工作增加开发版本。这种基本的存储库结构保持维护成本低，同时易于理解。

`main` 分支可能包含未经测试的功能，通常不够稳定，无法发布。要检索稳定的源代码，请使用项目的任何[官方发布版本](https://github.com/AcademySoftwareFoundation/MaterialX/releases)。

### 使用 Fork，Luke。

在典型的工作流程中，您应该 *fork* MaterialX 存储库到您的帐户。这会在您的用户命名空间下创建存储库的副本，并作为您的开发分支的“基地”，您将从这里向上游存储库提交 *拉取请求* 以进行合并。

一旦您的 Git 环境准备就绪，下一步是在本地 *clone* 您 fork 的 MaterialX 存储库，并添加指向 upstream MaterialX 存储库的 *remote*。这些主题在 GitHub 文档 [克隆存储库](https://help.github.com/articles/cloning-a-repository/) 和 [为 fork 配置远程](https://help.github.com/articles/configuring-a-remote-for-a-fork/) 中有介绍。

### 拉取请求

贡献应作为 GitHub 拉取请求提交。如果您不熟悉这个概念，请参阅 [创建拉取请求](https://help.github.com/articles/creating-a-pull-request/)。

代码更改的开发周期应遵循以下协议：

1. 在本地存储库中创建一个主题分支，为该分支分配一个简短的名称，描述您正在处理的功能或修复。
2. 进行更改、编译和彻底测试。代码风格应与现有风格和约定匹配，更改应专注于拉取请求将要解决的主题。在不相关的主题分支中进行不相关的更改，并提交单独的拉取请求。
3. 将提交推送到您的 fork。
4. 从您的主题分支创建 GitHub 拉取请求。
5. 拉取请求将由项目维护者和贡献者审查，他们可能会讨论、提供建设性反馈、请求更改或批准工作。
6. 在获得所需的批准数量（如 [所需批准](#code-review-and-required-approvals) 中所述）后，维护者可以将更改合并到 `main` 分支中。

### 代码审查和所需批准

MaterialX 存储库内容的修改以协作方式进行。任何拥有 GitHub 帐户的人都可以通过拉取请求提出修改，项目维护者将予以考虑。

拉取请求在合并之前必须满足最低数量的维护者批准。与其为所有 PR 设定硬性规定，要求基于 proposed 更改的复杂性和风险，并考虑到 PR 开放讨论的时间长度。以下指南概述了项目既定的合并批准规则：

- 不修改当前行为或对现有功能进行直接修复的次要更改可以由存储库的单个维护者批准和合并。
- 修改当前行为或引入新功能的中等更改在合并前应获得 *两* 名维护者的批准。除非是紧急修复，否则作者应给社区至少 48 小时来审查 proposed 更改。
- 主要新功能和核心设计决策应在提交任何 PR 之前在 ASWF Slack 或 TSC 会议上进行充分讨论，以征求反馈、建立共识，并提醒所有利益相关者在 PR 出现时留意它。

### 开发者指南

以下指南代表了我们在 MaterialX 项目中努力遵循的编码标准。虽然并非所有现有代码都符合这些标准，但我们鼓励所有新贡献遵循这些实践，我们欢迎逐步改进以使现有代码与这些指南保持一致。

#### 命名约定

类名应使用 PascalCase，如 `NodeGraph` 或 `ShaderGenerator`。变量和函数名应使用以小写字母开头的 camelCase，如 `childName` 或 `getNode`。受保护和私有成员变量还需要下划线前缀，如 `_parent` 或 `_childMap`。常量应使用 UPPER_CASE 书写，单词之间用下划线分隔，如 `EMPTY_STRING` 或 `CATEGORY`。类型别名应附加适当的后缀以指示其用途，使用 `Ptr` 表示指针，`Vec` 表示向量，`Map` 表示映射，`Set` 表示集合。

#### 静态常量和类组织

类成员应按可见性递减的顺序组织：public、protected，然后是 private。静态常量应放在其各自可见性部分的末尾。字符串常量应在实现文件中定义，而不是在头文件中定义，以避免违反单一定义规则。`EMPTY_STRING` 常量应用于代替空字符串字面量（`""`），以提高清晰度和一致性。

#### 智能指针约定

公共 API 中的堆分配对象应始终使用 `shared_ptr` 进行内存管理。应为所有共享指针定义类型别名，遵循 `ElementPtr` 表示 `shared_ptr<Element>` 的模式。应提供这些类型别名的可变和 const 版本，例如 `ElementPtr` 和 `ConstElementPtr`。应避免使用原始指针，除非在实现细节中表示非拥有引用。

#### Const 正确性

不修改对象状态的方法应标记为 `const`。访问器方法应提供 const 版本，以便在 const 对象上使用。遵循 `ConstElementPtr` 模式的类型别名应用于指示通过共享指针的只读访问。不应在函数内修改的参数应声明为 const。

#### 参数传递和返回值

字符串和复杂对象应通过 `const&` 传递以避免不必要的复制。共享指针应按值传递，因为它们被设计为复制成本低廉。返回共享指针时，应按值返回而不是按引用返回。只要方法不修改对象的状态，就应将其标记为 `const`。

#### 线程安全

MaterialX 类支持多个并发读取器，但不支持并发读写，遵循标准 C++ 容器的模式。这种设计使得在读密集型工作负载（如着色器生成和场景遍历）中能够高效并行处理，同时保持实现简单并避免细粒度锁定的开销。

#### 异常处理

异常应用于异常情况，而不是用于正常控制流。应通过从 `Exception` 继承来定义自定义异常类型，以表示特定的错误类别。异常消息应具有描述性，并包含相关上下文以帮助调试。方法可能抛出的所有异常都应使用方法的文档中的 `@throws` 标签进行记录。捕获异常时，应尽可能捕获特定异常类型而不是通用异常。

#### 头文件包含

头文件包含应使用尖括号编写，路径相对于根源代码文件夹（例如 `#include <MaterialXCore/Element.h>`）。这确保了整个代码库中包含路径的一致性，无论引用文件的位置如何。

每个实现文件应首先包含其对应的头文件，因此 `Element.cpp` 中的第一个包含应该是 `Element.h`。这确保头文件是自包含的，不会意外依赖其他头文件的包含。

在对应的头文件之后，包含块应按层次结构排序，高级模块列在低级模块之前（例如 `MaterialXGenShader`，然后是 `MaterialXFormat`，然后是 `MaterialXCore`）。这最大限度地提高了捕获高级模块中缺失依赖项的机会，否则这些依赖项可能在构建时被隐藏。在包含块内，各个包含应按字母顺序排序，提供简单的规范顺序，开发人员可以轻松检查。

为了避免包含循环，开发人员可以自由利用在其他头文件中简单引用的类的前向声明。为了清晰和效率，开发人员可以自由利用传递性头文件包含，即已经被高级头文件包含的低级头文件不需要单独重新声明。

#### 编码风格

MaterialX 项目的编码风格由存储库中的 [clang-format](.clang-format) 文件定义，该文件支持 Clang 13 及更高版本。

在向存储库添加新的源文件时，使用提供的 clang-format 文件自动将代码对齐到 MaterialX 约定。修改现有代码时，请遵循周围的格式约定，使新的或修改的代码与当前代码融合。

#### 文档标准

公共 API 中的所有类和方法都应使用 Doxygen 注释进行文档化。类应使用 `@class` 标签进行文档化，结构体使用 `@struct` 标签，后跟简要描述和任何详细文档。方法文档应包括 `@param`、`@return` 和 `@throws` 标签（如果适用）。相关方法应使用 `/// @name GroupName` 部分分组在一起，以提高可读性。文件级文档应使用 `/// @file` 指令紧接在版权头之后放置。

#### 单元测试

每个 MaterialX 模块在 [MaterialXTest](source/MaterialXTest) 模块中都有一个配套文件夹，包含一组验证其功能的单元测试。为 MaterialX 贡献新代码时，请确保在 MaterialXTest 中包含适当的单元测试，以验证新代码的预期行为。

MaterialX 测试套件可以在提交拉取请求之前通过 MaterialXTest 在本地运行。收到拉取请求后，GitHub CI 流程将在所有平台上自动运行 MaterialXTest，合并更改之前需要成功的结果。
