<p align="center">
  <img src="https://raw.githubusercontent.com/AcademySoftwareFoundation/MaterialX/main/documents/Images/MaterialXLogo.png" height="170" />
</p>

- [English](../docs_en/README.md)
- [简体中文](README.md)

[![许可证](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://github.com/AcademySoftwareFoundation/MaterialX/blob/main/LICENSE)
[![版本](https://img.shields.io/github/v/release/AcademySoftwareFoundation/MaterialX)](https://github.com/AcademySoftwareFoundation/MaterialX/releases/latest)
[![构建状态](https://github.com/AcademySoftwareFoundation/MaterialX/workflows/main/badge.svg)](https://github.com/AcademySoftwareFoundation/MaterialX/actions)
[![CII 最佳实践](https://bestpractices.coreinfrastructure.org/projects/6025/badge)](https://bestpractices.coreinfrastructure.org/projects/6025)

## 简介

MaterialX 是一种开放标准，用于在计算机图形学中表示丰富的材质和外观开发内容，使其能够在应用程序和渲染器之间进行平台无关的描述和交换。MaterialX 于 2012 年在 [工业光魔](https://www.ilm.com/) 推出，自《星球大战：原力觉醒》和《千年隼号：走私者快跑》以来，一直是其故事片和实时体验中的关键技术。该项目于 2017 年作为开源项目发布，包括索尼影视图像工艺公司、皮克斯、Autodesk、Adobe 和 SideFX 等公司在内，为其持续开发做出了贡献。2021 年，MaterialX 成为 [学院软件基金会](https://www.aswf.io/) 的第七个托管项目。

## 开发者快速入门

- 下载最新版本的 [CMake](https://cmake.org/) 构建系统。
- 将 CMake 指向 MaterialX 库的根目录，并为您的平台和编译器生成 C++ 项目。
- 选择 `MATERIALX_BUILD_PYTHON` 选项以构建 Python 绑定。
- 选择 `MATERIALX_BUILD_VIEWER` 选项以构建 [MaterialX 查看器](documents/DeveloperGuide/Viewer.md)。
- 选择 `MATERIALX_BUILD_GRAPH_EDITOR` 选项以构建 [MaterialX 图形编辑器](documents/DeveloperGuide/GraphEditor.md)。

## 支持的平台

MaterialX 代码库需要支持 C++17 的编译器，可以使用以下任何编译器进行构建：

- Microsoft Visual Studio 2017 或更高版本
- GCC 8 或更高版本
- Clang 5 或更高版本

MaterialX 的 Python 绑定基于 [PyBind11](https://github.com/pybind/pybind11)，支持 Python 3.9 及更高版本。

## MaterialX 查看器

[MaterialX 查看器](documents/DeveloperGuide/Viewer.md) 利用着色器生成从 MaterialX 图构建 GLSL 着色器，使用 NanoGUI 框架渲染结果。

**图 1：** MaterialX 查看器中的程序化和统一材质
<p float="left">
  <img title="标准表面大理石材质"
       src="https://raw.githubusercontent.com/AcademySoftwareFoundation/MaterialX/main/documents/Images/MaterialXView_Marble.png"
       width="24%" />
  <img title="标准表面铜材质"
       src="https://raw.githubusercontent.com/AcademySoftwareFoundation/MaterialX/main/documents/Images/MaterialXView_Copper.png"
       width="24%" />
  <img title="标准表面塑料材质"
       src="https://raw.githubusercontent.com/AcademySoftwareFoundation/MaterialX/main/documents/Images/MaterialXView_Plastic.png"
       width="24%" />
  <img title="标准表面汽车漆材质"
       src="https://raw.githubusercontent.com/AcademySoftwareFoundation/MaterialX/main/documents/Images/MaterialXView_Carpaint.png"
       width="24%" />
</p>

**图 2：** MaterialX 查看器中经过纹理和色彩空间管理的材质
<p float="left">
  <img title="标准表面瓷砖黄铜材质"
       src="https://raw.githubusercontent.com/AcademySoftwareFoundation/MaterialX/main/documents/Images/MaterialXView_TiledBrass.png"
       width="49%" />
  <img title="标准表面瓷砖木材材质"
       src="https://raw.githubusercontent.com/AcademySoftwareFoundation/MaterialX/main/documents/Images/MaterialXView_TiledWood.png"
       width="49%" />
</p>

## 开放国际象棋套装

开放国际象棋套装是一个开放参考资产，由 Standard Surface 着色模型中的 [MaterialX 文件](resources/Materials/Examples/StandardSurface/standard_surface_chess_set.mtlx) 和 glTF 格式的 [几何文件](resources/Geometry) 组成。它由 Moeen Sayed 和 Mujtaba Sayed 创作，并由 Side Effects 贡献给 MaterialX 项目。

**图 3：** 在 Arnold for Maya 中渲染的开放国际象棋套装
![在 Arnold for Maya 中渲染的开放国际象棋套装](https://raw.githubusercontent.com/AcademySoftwareFoundation/MaterialX/main/documents/Images/OpenChessSet_Arnold_01.png)

**图 4：** 在 Karma XPU for Houdini 中渲染的开放国际象棋套装
![在 Karma XPU for Houdini 中渲染的开放国际象棋套装](https://raw.githubusercontent.com/AcademySoftwareFoundation/MaterialX/main/documents/Images/OpenChessSet_Karma_01.png)

## 预构建二进制文件

以下包包含最新版本的预构建二进制文件，包括 MaterialX 查看器、Python 库和示例资产：

- [Microsoft Windows (Visual Studio 2022, Python 3.13)](https://github.com/AcademySoftwareFoundation/MaterialX/releases/latest/download/MaterialX_Windows_VS2022_x64_Python313.zip)
- [MacOS (Xcode 16, Python 3.13)](https://github.com/AcademySoftwareFoundation/MaterialX/releases/latest/download/MaterialX_MacOS_Xcode_16_Python313.zip)
- [Linux (GCC 14, Python 3.13)](https://github.com/AcademySoftwareFoundation/MaterialX/releases/latest/download/MaterialX_Linux_GCC_14_Python313.zip)

## 其他资源

- [开发者指南](http://www.materialx.org/docs/api/index.html) 包含面向开发者的 MaterialX 概述以及构建和 API 文档。
- [Python 脚本](python/Scripts) 文件夹包含 MaterialX Python 代码的独立示例。
- [JavaScript](javascript) 文件夹包含为 MaterialX 构建 JavaScript 绑定的详细信息。
- 在 [ASWF 开源日](https://materialx.org/assets/ASWF_OSD2025_MaterialX_Final.pdf) 和 [SIGGRAPH 基于物理的着色课程](https://blog.selfshadow.com/publications/s2020-shading-course/#materialx) 上的演示提供了 MaterialX 开发路线图的详细信息。
