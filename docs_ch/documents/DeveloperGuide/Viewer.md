# MaterialX 查看器

MaterialX 查看器利用着色器生成从 MaterialX 图构建 GLSL 着色器，使用 NanoGUI 框架渲染结果。支持标准的模式和基于物理的着色节点集，并且可以将自定义节点库作为附加库路径包含。

## 示例图片

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

## 构建 MaterialX 查看器
在 CMake 中选择 `MATERIALX_BUILD_VIEWER` 选项来构建 MaterialX 查看器。安装会将 **MaterialXView** 可执行文件复制到所选安装文件夹内的 `/bin` 目录中。

## 查看器选项摘要

1. **`加载网格`**：加载 OBJ 或 glTF 格式的新几何体。
2. **`加载材质`**：加载 MTLX 格式的材质文档。
3. **`加载环境`**：加载 HDR 格式的经纬度环境光。
4. **`属性编辑器`**：查看或编辑当前材质的属性。
5. **`高级设置`** ：资产和渲染选项。

## 几何体

MaterialX 查看器的默认显示几何体是 Arnold Shader Ball，由 Autodesk 的 Solid Angle 团队贡献给 MaterialX 项目。要更改显示几何体，请点击 `加载网格` 并导航到 [Geometry](https://github.com/AcademySoftwareFoundation/MaterialX/tree/main/resources/Geometry) 文件夹以获取 OBJ 格式的其他模型。

如果加载的几何体包含多个几何组，则会出现 `选择几何体` 下拉框，允许用户选择哪个组处于活动状态。活动的几何组将用于后续操作，如材质分配和渲染属性更改。

## 材质

要更改显示的材质，请点击 `加载材质` 并导航到 [Materials/Examples/StandardSurface](https://github.com/AcademySoftwareFoundation/MaterialX/tree/main/resources/Materials/Examples/StandardSurface) 或 [Materials/Examples/UsdPreviewSurface](https://github.com/AcademySoftwareFoundation/MaterialX/tree/main/resources/Materials/Examples/UsdPreviewSurface) 文件夹，其中包含一组 MTLX 格式的示例材质。

将材质加载到查看器后，可以通过点击 `属性编辑器` 并滚动参数列表来检查和调整其参数。可以通过点击 `保存材质` 将编辑后的材质保存到文件系统。

可以在单个会话中组合多个材质文档，方法是导航到 `高级设置` 并启用 `合并材质`。启用此设置后加载新材质会将它们添加到当前材质列表中，可以通过 `分配材质` 下拉框将它们分配给几何体。或者，可以使用 `左` 和 `右` 箭头循环浏览可用材质列表。

如果将包含 `look` 元素的材质文档加载到查看器中，则 look 中的任何材质分配都将应用于与指定几何字符串匹配的几何组。有关包含 look 元素的材质文档示例，请参见 [standard_surface_look_brass_tiled.mtlx](https://github.com/AcademySoftwareFoundation/MaterialX/tree/main/resources/Materials/Examples/StandardSurface/standard_surface_look_brass_tiled.mtlx)。

## 光照

查看器的默认光照环境是来自 HDRI Haven 的 San Giuseppe Bridge 环境。要将另一个环境加载到查看器中，请点击 `加载环境` 并导航到 [Lights](https://github.com/AcademySoftwareFoundation/MaterialX/tree/main/resources/Lights) 文件夹，或加载纬度-经度格式的任何 HDR 环境。如果磁盘上的 HDR 文件具有匹配名称的配套 MaterialX 文档，则此文档将作为环境的直接光照 rig 加载；否则仅渲染间接光照。如果磁盘上的 HDR 文件在 `irradiance` 子文件夹中有配套图像，则此图像将作为环境的漫反射卷积加载；否则，将在加载时使用球谐函数生成漫反射卷积。

可以通过 `高级设置` 下的 `阴影贴图` 选项启用主方向光的阴影贴图。如果给定几何体可用，可以通过 `环境光遮蔽` 选项启用环境光遮蔽。可以通过增加 `环境采样` 的值来提高环境光照的保真度，但这需要额外的 GPU 资源并可能影响查看器的交互性。

## 图像

默认情况下，MaterialX 查看器使用 `stb_image` 加载和保存图像文件，它支持常见的 8 位格式，如 JPEG、PNG、TGA 和 BMP，以及用于高动态范围图像的 HDR 格式。如果您需要访问 EXR 和 TIFF 等其他图像格式，则可以构建支持 `OpenImageIO` 的 MaterialX 查看器。要使用 OpenImageIO 构建 MaterialX，请在 CMake 中选中 `MATERIALX_BUILD_OIIO` 选项，并使用 `MATERIALX_OIIO_DIR` 选项指定 OpenImageIO 安装的位置。

## 键盘快捷键

- `R`：从文件重新加载当前材质。按住 `SHIFT` 也可重新加载所有标准库。
- `G`：将当前 GLSL 着色器源代码保存到文件。
- `O`：将当前 OSL 着色器源代码保存到文件。
- `M`：将当前 MDL 着色器源代码保存到文件。
- `L`：从文件加载 GLSL 着色器源代码。在加载之前编辑源文件提供了一种调试和实验着色器源代码的方法。
- `D`：将当前材质中的每个节点图保存为 DOT 文件。有关此格式的更多详细信息，请参见 www.graphviz.org。
- `F`：捕获当前帧并保存到文件。
- `W`：创建楔形渲染并保存到文件。有关其他控件，请参见 `高级设置`。
- `T`：将当前材质转换为不同的着色模型。有关其他控件，请参见 `高级设置`。
- `B`：将当前材质烘焙到纹理。有关其他控件，请参见 `高级设置`。
- `上` ：选择上一个几何体。
- `下` ：选择下一个几何体。
- `右` ：切换到下一个材质。
- `左` ：切换到上一个材质。
- `+` ：用相机放大。
- `-` ：用相机缩小。

## 命令行选项

以下是 MaterialXView 的常见命令行选项，完整列表可以通过 `--help` 选项显示。
- `--material [FILENAME]` ：指定要在查看器中显示的 MTLX 文档的文件名
- `--mesh [FILENAME]` ：指定要在查看器中显示的 OBJ 或 glTF 网格的文件名
- `--meshRotation [VECTOR3]` ：指定显示网格的旋转，作为三个逗号分隔的浮点数，表示绕 X、Y 和 Z 轴的旋转（以度为单位）（默认为 0,0,0）
- `--meshScale [FLOAT]` ：指定显示网格的统一缩放
- `--cameraPosition [VECTOR3]` ：指定相机位置，作为三个逗号分隔的浮点数（默认为 0,0,5）
- `--cameraTarget [VECTOR3]` ：指定相机目标位置，作为三个逗号分隔的浮点数（默认为 0,0,0）
- `--cameraViewAngle [FLOAT]` ：指定相机的视角，或为零表示正交投影（默认为 45）
- `--cameraZoom [FLOAT]` ：指定相机的缩放因子，实现为网格缩放乘数（默认为 1）
- `--envRad [FILENAME]` ：指定要显示的环境光文件的文件名，存储为纬度-经度格式的 HDR 环境辐射
- `--envMethod [INTEGER]` ：指定环境光照方法（0 = 过滤重要性采样，1 = 预过滤环境贴图，默认为 0）
- `--envSampleCount [INTEGER]` ： 指定环境采样数量（默认为 16）
- `--lightRotation [FLOAT]` ：指定光照环境绕 Y 轴的旋转（以度为单位）（默认为 0）
- `--path [FILEPATH]` ：指定额外的数据搜索路径位置（例如 '/projects/MaterialX'）。在定位数据库、XInclude 引用和引用图像时将查询此绝对路径。
- `--library [FILEPATH]` ：指定额外的数据库文件夹（例如 'vendorlib', 'studiolib'）。在加载数据库时，此相对路径将附加到数据搜索路径中的每个位置。
- `--screenWidth [INTEGER]` ：指定屏幕图像的宽度（以像素为单位）（默认为 1280）
- `--screenHeight [INTEGER]` ：指定屏幕图像的高度（以像素为单位）（默认为 960）
- `--screenColor [VECTOR3]` ：指定查看器的背景颜色，作为三个逗号分隔的浮点数（默认为 0.3,0.3,0.32）
- `--captureFilename [FILENAME]` ：指定第一个渲染帧应写入的文件名
- `--refresh [FLOAT]` ：指定查看器的刷新周期（以毫秒为单位）（默认为 50，设置为 -1 以禁用）
- `--remap [TOKEN1:TOKEN2]` ：指定加载 MaterialX 文档时从一个令牌到另一个令牌的重新映射
- `--skip [NAME]` ：指定跳过与给定名称属性匹配的元素
- `--terminator [STRING]` ：指定对文件前缀强制执行给定的终止符字符串
- `--help` ：显示完整的命令行选项列表
