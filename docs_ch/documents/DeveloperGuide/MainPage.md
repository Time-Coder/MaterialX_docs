# MaterialX 概述

- [English](../../../docs_en/documents/DeveloperGuide/MainPage.md)
- [简体中文](MainPage.md)

MaterialX 是一种开放标准，用于在计算机图形学中表示丰富的材质和外观开发内容，使其能够在应用程序和渲染器之间进行平台无关的描述和交换。MaterialX 于 2012 年在 [工业光魔](https://www.ilm.com/) 推出，自《星球大战：原力觉醒》和《千年隼号：走私者快跑》以来，一直是其故事片和实时体验中的关键技术。该项目于 2017 年作为开源项目发布，包括索尼影视图像工艺公司、皮克斯、Autodesk、Adobe 和 SideFX 等公司在内，为其持续开发做出了贡献。2021 年，MaterialX 成为 [学院软件基金会](https://www.aswf.io/) 的第七个托管项目。

## 开发者快速入门

- 下载最新版本的 [CMake](https://cmake.org/) 构建系统。
- 将 CMake 指向 MaterialX 库的根目录，并为您的平台和编译器生成 C++ 项目。
- 选择 `MATERIALX_BUILD_PYTHON` 选项以构建 Python 绑定。
- 选择 `MATERIALX_BUILD_VIEWER` 选项以构建 [MaterialX 查看器](https://github.com/AcademySoftwareFoundation/MaterialX/blob/main/documents/DeveloperGuide/Viewer.md)。
- 选择 `MATERIALX_BUILD_GRAPH_EDITOR` 选项以构建 [MaterialX 图形编辑器](https://github.com/AcademySoftwareFoundation/MaterialX/blob/main/documents/DeveloperGuide/GraphEditor.md)。 

## 支持的平台

MaterialX 代码库需要支持 C++17 的编译器，可以使用以下任何编译器进行构建：

- Microsoft Visual Studio 2017 或更高版本
- GCC 8 或更高版本
- Clang 5 或更高版本

MaterialX 的 Python 绑定基于 [PyBind11](https://github.com/pybind/pybind11)，支持 Python 3.9 及更高版本。

在 macOS 上，您需要 [安装 Xcode](https://developer.apple.com/xcode/resources/)，以便访问 Metal 工具以及编译器工具链。

## 构建 MaterialX

### 构建 MaterialX C++

通过 CMake 构建 MaterialX 时，会自动包含 MaterialX C++ 库。

要在 MaterialX 构建中启用 OpenImageIO 和 OpenColorIO 支持，可以使用以下附加选项：

- `MATERIALX_BUILD_OIIO`：请求使用 OpenImageIO（除了 stb_image）构建 MaterialXRender，扩展支持的图像格式集。OpenImageIO 的最低支持版本是 2.2。
- `MATERIALX_BUILD_OCIO`：请求构建支持自定义 OpenColorIO 色彩空间和变换的 MaterialXGenShader。OpenColorIO 的最低支持版本是 2.4。

请参阅 [MaterialX 单元测试](https://github.com/AcademySoftwareFoundation/MaterialX/tree/main/source/MaterialXTest) 页面，了解 GLSL、OSL 和 MDL 中着色器生成和渲染测试的文档。

### 构建 MaterialX Python

默认情况下，`MATERIALX_BUILD_PYTHON` 选项将使用开发人员路径中的活动 Python 版本。要选择特定版本的 Python，请使用以下一个或多个高级选项：

- `MATERIALX_PYTHON_VERSION`：用于构建 MaterialX Python 包的 Python 版本（例如 `3.9`）
- `MATERIALX_PYTHON_EXECUTABLE`：用于构建 MaterialX Python 包的 Python 可执行文件（例如 `C:/Python39/python.exe`）

生成 MaterialX Python 的其他选项包括以下内容：

- `MATERIALX_PYTHON_PYBIND11_DIR`：包含用于构建 MaterialX Python 的 PyBind11 源代码的文件夹路径。默认为包含的 PyBind11 源代码。

### 构建 MaterialX 查看器

选择 `MATERIALX_BUILD_VIEWER` 选项以构建 MaterialX 查看器。安装会将 `MaterialXView` 可执行文件复制到所选安装文件夹内的 `bin/` 目录中。

### 构建 API 文档

要为 MaterialX C++ API 生成 HTML 文档，请确保您的路径上有 [Doxygen](https://www.doxygen.org/) 版本，并在 CMake 中选择高级选项 `MATERIALX_BUILD_DOCS`。此选项将为您的项目添加一个名为 `MaterialXDocs` 的目标，可以作为开发环境中的独立步骤进行构建。

## 编辑器设置

MaterialX 应该适用于任何支持 CMake 或 CMake 可以为其生成项目的编辑器。
这里列出了一些常见的编辑器，以帮助开发人员开始使用。

### CLion

[CLion](https://www.jetbrains.com/clion/) 是一个跨平台的 IDE，可用于开发 MaterialX。
此外，它包含 CMake 并且对于非商业用途是免费的。

要开始使用 CLion，直接打开 MaterialX 存储库，它将为您加载 CMake 项目。
如果要启用 Python 等功能，请转到 `Settings -> Build, Execution and Deployment -> CMake` 并配置
CMake 选项，例如：

```
-DMATERIALX_BUILD_PYTHON=ON
-DMATERIALX_BUILD_VIEWER=ON
-DMATERIALX_BUILD_GRAPH_EDITOR=ON
```

要构建，请选择 `Build -> Build Project` 或选择要构建的特定配置。
要安装，请选择 `Build -> Install`

## 安装 MaterialX

构建项目的 `install` 目标会将 MaterialX C++ 和 Python 库安装到由 `CMAKE_INSTALL_PREFIX` 设置指定的文件夹中，并将 MaterialX Python 作为第三方库安装在您的 Python 环境中。可以通过将 `MATERIALX_INSTALL_PYTHON` 设置为 `OFF` 来禁用将 MaterialX Python 安装为第三方库。

## MaterialX 版本控制

MaterialX 代码库使用修改后的语义版本控制系统，其中 *主* 版本和 *次* 版本与相应的 [MaterialX 规范](https://materialx.org/Specification.html) 匹配，而 *构建* 版本代表该规范版本内的工程进展。MaterialX 文档同样标记有它们创作时的规范版本，并且它们可以加载到具有相等或更高规范版本的任何 MaterialX 代码库中。

从早期版本升级 MaterialX 文档是在导入时由 `Document::upgradeVersion()` 方法处理的，该方法应用以前规范修订中发生的语法和节点接口升级。这使得 MaterialX 的语法约定以及节点的名称和接口能够随着时间的推移而演变，而不会使早期版本的文档失效。

### MaterialX API 更改

以下规则描述了在版本升级中允许对 [MaterialX API](https://materialx.org/docs/api/classes.html) 进行的更改类别：

- 在 *构建* 版本升级中，只允许对 MaterialX API 进行非破坏性更改。对于在构建版本升级中修改的任何 API 调用，应使用原始 API 调用的已弃用 C++ 和 Python 包装器来保持向后兼容性。
- 在 *次* 版本和 *主* 版本升级中，允许对 MaterialX API 进行破坏性更改，但应仔细权衡其好处与成本。对 API 调用的任何破坏性更改都应在新版本的发行说明中突出显示。

### MaterialX 数据库更改

以下规则描述了在版本升级中允许对 [MaterialX 数据库](https://github.com/AcademySoftwareFoundation/MaterialX/tree/main/libraries) 进行的更改类别：

- 在 *构建* 版本升级中，只允许对 MaterialX 数据库进行附加更改和修复。允许进行附加更改以引入具有向后兼容默认值的新节点、节点版本和节点输入。允许进行数据库修复以更新节点实现以改善其与规范的一致性，而不对其名称或接口进行任何更改。
- 在 *次* 版本升级中，允许更改 MaterialX 节点的名称和接口，要求使用版本升级逻辑来维护早期版本文档的有效性和视觉解释。
- 在 *主* 版本升级中，允许更改 MaterialX 文档的语法规则，要求使用版本升级逻辑来维护早期版本文档的有效性和视觉解释。这些更改通常需要同时更新 MaterialX API 和数据库。

## 其他链接

- 主要的 [MaterialX 网站](http://www.materialx.org) 提供了项目历史、行业合作和最近演示的背景信息。
- [Python 脚本](https://github.com/materialx/MaterialX/tree/main/python/Scripts) 文件夹包含 MaterialX Python 代码的独立示例。
- [MaterialX 单元测试](https://github.com/materialx/MaterialX/tree/main/source/MaterialXTest) 文件夹包含 MaterialX C++ 有用模式的示例。
- [MaterialX 查看器](https://github.com/materialx/MaterialX/blob/main/documents/DeveloperGuide/Viewer.md) 是一个完整的跨平台 C++ 应用程序，基于 [MaterialX 着色器生成](https://github.com/materialx/MaterialX/blob/main/documents/DeveloperGuide/ShaderGeneration.md)
