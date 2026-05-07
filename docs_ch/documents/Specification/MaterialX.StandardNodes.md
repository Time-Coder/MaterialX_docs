<!-----
MaterialX 标准节点 v1.39
----->

- [English](../../../docs_en/documents/Specification/MaterialX.StandardNodes.md)
- [简体中文](MaterialX.StandardNodes.md)


# MaterialX 标准节点

**版本 1.39**  
Doug Smythe - Industrial Light & Magic  
Jonathan Stone - Lucasfilm Advanced Development Group  
2025年11月9日


# 简介

MaterialX 规范定义了一种内容架构，以平台和着色语言无关的方式描述材质、图像处理和着色网络，以及这些网络中的节点如何访问纹理和几何信息。

本文档描述了一组特定的**标准节点**，可用于读取和处理图像和几何属性数据，以及程序化创建新图像数据。这些 "stdlib" 节点是所有 MaterialX 实现的核心部分。其他节点在配套文档 [**MaterialX 基于物理的着色节点**](./MaterialX.PBRSpec.md) 和 [**MaterialX NPR 着色节点**](./MaterialX.NPRSpec.md) 中描述。

在下面的节点描述中，表格定义了节点输入和输出的名称、允许的类型、默认值，以及在适当情况下接受的值。对于输出，指定的默认值是节点禁用时输出（或从输入传递）的值。具有多个表格的节点描述接受任何单个表格中描述的输入/输出/类型的任意组合。

## 目录

**[引言](#introduction)**  

**[标准源节点](#standard-source-nodes)**  
 [纹理节点](#texture-nodes)  
  [纹理节点说明](#texture-node-notes)  
 [程序化节点](#procedural-nodes)  
  [程序化节点说明](#procedural-node-notes)  
 [噪声节点](#noise-nodes)  
  [噪声节点说明](#noise-node-notes)  
 [形状节点](#shape-nodes)  
 [几何节点](#geometric-nodes)  
  [几何节点说明](#geometric-node-notes)  
 [应用程序节点](#application-nodes)  
  [应用程序节点说明](#application-node-notes)  

**[标准操作符节点](#standard-operator-nodes)**  
 [数学节点](#math-nodes)  
 [逻辑操作符节点](#logical-operator-nodes)  
 [调整节点](#adjustment-nodes)  
 [合成节点](#compositing-nodes)  
 [条件节点](#conditional-nodes)  
 [通道节点](#channel-nodes)  
 [卷积节点](#convolution-nodes)  

**[标准着色器节点](#standard-shader-nodes)**

<br>


# 标准源节点

源节点使用外部数据和/或程序化函数形成输出；它们没有任何必需的输入。每个源节点必须定义其输出类型。

本节定义了所有 MaterialX 实现都应支持的源节点。标准源节点分为以下类别:[纹理节点](#texture-nodes)、[程序化节点](#procedural-nodes)、[噪声节点](#noise-nodes)、[形状节点](#shape-nodes)、[几何节点](#geometric-nodes) 和 [应用程序节点](#application-nodes)。


## 纹理节点

纹理节点用于从图像或纹理贴图文件中读取过滤的图像数据，以便在节点图中进行处理。

```xml
  <image name="in1" type="color4">
    <input name="file" type="filename" value="layer1.tif"/>
    <input name="default" type="color4" value="0.5,0.5,0.5,1"/>
  </image>
  <image name="in2" type="color3">
    <input name="file" type="filename" value="<albedomap>"/>
    <input name="default" type="color3" value="0.18,0.18,0.18"/>
  </image>
```

标准纹理节点:

<a id="node-image"> </a>

### `image`

从单个图像或多层图像中的图层采样数据。在渲染几何体的上下文中使用时，图像基于几何体 UV 坐标映射到几何体上，图像的左下角映射到 (0, 0) UV 坐标（对于平铺图像，映射到分数 (0, 0) UV 坐标）。

&lt;image> 节点的类型决定输出的通道数，可能少于图像文件中的通道数，输出图像文件的前 N 个通道。因此，`float` &lt;image> 将返回 RGB 图像的红色通道，而 `color3` &lt;image> 将返回 RGBA 图像的 RGB 通道。如果 &lt;image> 节点的类型比引用的图像文件具有更多通道，则输出将在图像文件的 N 个通道之后的所有通道中包含零值。

`file` 输入值可以包括一个或多个替换以更改访问的文件名，如主规范文档中的 [文件名替换](./MaterialX.Specification.md#filename-substitutions) 部分所述。

如果未提供 `layer` 的值且输入文件有多个图层，则将使用 "default" 图层，如果没有 "default" 图层，则使用 "rgba"。注意:`image` 类型定义的通道数必须与命名图层中的通道数匹配。

`default` 输入是如果文件引用无法解析（例如，如果文件名中包含几何令牌、接口令牌或 hostattr，但未定义替换值或默认值，或者解析的文件 URI 无法读取），或者指定的图层在文件中不存在时使用的默认值。默认值必须与 &lt;image> 元素本身相同的类型。如果未定义 `default`，则默认颜色值在所有通道中将为 0.0。

|端口            |描述                                                                                                      |类型                  |默认值  |接受值                  |
|----------------|-----------------------------------------------------------------------------------------------------------------|----------------------|---------|---------------------------------|
|`file`          |图像文件的 URI                                                                                        |filename              |__empty__|                                 |
|`layer`         |要从多层输入文件中提取的图层名称                                                   |string                |__empty__|                                 |
|`default`       |如果无法解析文件引用，则使用的默认值                                                 |Same as `out`         |__zero__ |                                 |
|`texcoord`      |读取图像数据的 2D 纹理坐标                                                        |vector2               |_UV0_    |                                 |
|`uaddressmode`  |确定在采样图像之前如何处理 0-1 范围之外的 U 坐标                       |string                |periodic |constant, clamp, periodic, mirror|
|`vaddressmode`  |确定在采样图像之前如何处理 0-1 范围之外的 V 坐标                       |string                |periodic |constant, clamp, periodic, mirror|
|`filtertype`    |要使用的纹理过滤类型                                                                             |string                |linear   |closest, linear, cubic           |
|`framerange`    |字符串 "minframe-maxframe"，例如 "10-99"，用于指定图像文件允许的帧范围|string                |__empty__|                                 |
|`frameoffset`   |添加到当前帧号以获取图像文件帧号的数字                            |integer               |0        |                                 |
|`frameendaction`|当解析的图像帧号超出 `framerange` 范围时要执行的操作                                |string                |constant |constant, clamp, periodic, mirror|
|`out`           |输出:采样的纹理值                                                                                |float, colorN, vectorN|__zero__ |                                 |

<a id="node-tiledimage"> </a>

### `tiledimage`
从单个图像采样数据，并提供在 UV 空间中平铺和偏移图像的条款。

`file` 输入可以包括一个或多个替换以更改访问的文件名，如主规范文档中的 [文件名替换](./MaterialX.Specification.md#filename-substitutions) 部分所述。

|端口                |描述                                                                                                      |类型                  |默认值  |接受值                  |
|--------------------|-----------------------------------------------------------------------------------------------------------------|----------------------|---------|---------------------------------|
|`file`              |图像文件的 URI                                                                                        |filename              |__empty__|                                 |
|`default`           |如果无法解析文件引用，则使用的默认值                                                 |Same as `out`         |__zero__ |                                 |
|`texcoord`          |读取图像数据的 2D 纹理坐标                                                        |vector2               |_UV0_    |                                 |
|`uvtiling`          |给定图像沿 U 和 V 轴的平铺率                                                       |vector2               |1.0, 1.0 |                                 |
|`uvoffset`          |给定图像沿 U 和 V 轴的偏移                                                            |vector2               |0.0, 0.0 |                                 |
|`realworldimagesize`|文件图像表示的真实世界大小                                                                |vector2               |1.0, 1.0 |                                 |
|`realworldtilesize` |单个正方形 0-1 UV 瓦片的真实世界大小                                                               |vector2               |1.0, 1.0 |                                 |
|`filtertype`        |要使用的纹理过滤类型                                                                             |string                |linear   |closest, linear, cubic           |
|`framerange`        |字符串 "minframe-maxframe"，例如 "10-99"，用于指定图像文件允许的帧范围|string                |__empty__|                                 |
|`frameoffset`       |添加到当前帧号以获取图像文件帧号的数字                            |integer               |0        |                                 |
|`frameendaction`    |当解析的图像帧号超出 `framerange` 范围时要执行的操作                                |string                |constant |constant, clamp, periodic, mirror|
|`out`               |输出:采样的纹理值                                                                                |float, colorN, vectorN|__zero__ |                                 |

<a id="node-latlongimage"> </a>

### `latlongimage`
沿视图方向采样等角地图，具有可调整的纬度偏移。

`file` 输入可以包括一个或多个替换以更改访问的文件名，如主规范文档中的 [文件名替换](./MaterialX.Specification.md#filename-substitutions) 部分所述。

|端口      |描述                                                                        |类型    |默认值      |
|----------|-----------------------------------------------------------------------------------|--------|-------------|
|`file`    |图像文件的 URI                                                          |filename|__empty__    |
|`default` |如果无法解析文件引用，则使用的默认值                   |color3  |0.0, 0.0, 0.0|
|`viewdir` |决定从投影等角地图中采样值的视图方向|vector3 |0.0, 0.0, 1.0|
|`rotation`|纵向采样偏移，以度为单位                                       |float   |0.0          |
|`out`     |输出:采样的纹理值                                                  |color3  |0.0, 0.0, 0.0|

<a id="node-hextiledimage"> </a>

### `hextiledimage`
从单个图像采样数据，并提供在 UV 空间中六边形平铺和随机化图像的条款。

`file` 输入可以包括一个或多个替换以更改访问的文件名，如主规范文档中的 [文件名替换](./MaterialX.Specification.md#filename-substitutions) 部分所述。

`lumacoeffs` 输入表示当前工作色彩空间的亮度系数。如果无法确定特定的色彩空间，将使用 ACEScg (ap1) 亮度系数 [0.2722287, 0.6740818, 0.0536895]。支持色彩管理系统的应用程序可以选择从 CMS 检索工作色彩空间的亮度系数并直接传递给节点的实现，而不是将其暴露给用户。

|端口                |描述                                                                                        |类型             |默认值                        |
|--------------------|---------------------------------------------------------------------------------------------------|-----------------|-------------------------------|
|`file`              |图像文件的 URI                                                                          |filename         |__empty__                      |
|`default`           |如果无法解析文件引用，则使用的默认值                                   |Same as `out`    |__zero__                       |
|`texcoord`          |读取图像数据的 2D 纹理坐标                                          |vector2          |_UV0_                          |
|`tiling`            |给定图像沿 U 和 V 轴的平铺率                                         |vector2          |1.0, 1.0                       |
|`rotation`          |每瓦片旋转随机度，以度为单位                                                            |float            |0.0                            |
|`rotationrange`     |用于随机化每个瓦片旋转的范围，以度为单位                                          |vector2          |0.0, 360.0                     |
|`scale`             |应用于瓦片大小的每瓦片缩放随机乘数                                          |float            |1.0                            |
|`scalerange`        |用于随机化瓦片缩放的缩放乘数范围                                            |vector2          |0.5, 2.0                       |
|`offset`            |每瓦片平移随机度，以 UV 单位                                                        |float            |1.0                            |
|`offsetrange`       |用于随机化瓦片位置的 UV 单位偏移值范围                                |vector2          |0.0, 1.0                       |
|`falloff`           |用于在边缘处混合相邻瓦片的衰减宽度；较大的值产生更平滑的混合|float            |0.5                            |
|`falloffcontrast`   |应用于衰减混合的对比度，以锐化（值 >1）或软化（值 <1）过渡  |float            |0.5                            |
|`lumacoeffs`        |当前工作色彩空间的亮度系数                                           |color3           |0.2722287, 0.6740818, 0.0536895|
|`out`               |输出:采样的纹理值                                                                  |colorN           |__zero__                       |

<a id="node-triplanarprojection"> </a>

### `triplanarprojection`
从三个图像（或多层图像中的图层）采样数据，并沿三个 respective 坐标轴投影图像的平铺表示，使用几何法线计算三个样本的加权混合。

|端口            |描述                                                                                                      |类型                  |默认值  |接受值                  |
|----------------|-----------------------------------------------------------------------------------------------------------------|----------------------|---------|---------------------------------|
|`filex`         |要沿从 +X 轴回到原点方向投影的图像文件的 URI               |filename              |__empty__|                                 |
|`filey`         |要沿从 +Y 轴回到原点方向投影的图像文件的 URI               |filename              |__empty__|                                 |
|`filez`         |要沿从 +Z 轴回到原点方向投影的图像文件的 URI               |filename              |__empty__|                                 |
|`layerx`        |用于 X 轴投影的从多层输入文件中提取的图层名称                         |string                |__empty__|                                 |
|`layery`        |用于 Y 轴投影的从多层输入文件中提取的图层名称                         |string                |__empty__|                                 |
|`layerz`        |用于 Z 轴投影的从多层输入文件中提取的图层名称                         |string                |__empty__|                                 |
|`default`       |如果无法解析文件引用，则使用的默认值                                                 |Same as `out`         |__zero__ |                                 |
|`position`      |空间变化输入，指定评估投影的 3D 位置                        |vector3               |_Pobject_|                                 |
|`normal`        |空间变化输入，指定用于混合的 3D 法线向量                                      |vector3               |_Nobject_|                                 |
|`upaxis`        |哪个轴被视为'向上'，0 表示 X，1 表示 Y，或 2 表示 Z                                         |integer               |2        |0, 1, 2                          |
|`blend`         |使用几何法线混合样本的加权因子，值越高混合越柔和      |float                 |1.0      |                                 |
|`filtertype`    |要使用的纹理过滤类型                                                                             |string                |linear   |closest, linear, cubic           |
|`framerange`    |字符串 "minframe-maxframe"，例如 "10-99"，用于指定图像文件允许的帧范围|string                |__empty__|                                 |
|`frameoffset`   |添加到当前帧号以获取图像文件帧号的数字                            |integer               |0        |                                 |
|`frameendaction`|当解析的图像帧号超出 `framerange` 范围时要执行的操作                                |string                |constant |constant, clamp, periodic, mirror|
|`out`           |输出:混合的纹理值                                                                                |float, colorN, vectorN|__zero__ |                                 |


### 纹理节点说明

<a id="addressmode-values"> </a>

纹理节点的 `uaddressmode` 和 `vaddressmode` 输入支持以下值:

* "constant": 0-1 范围之外的纹理坐标返回节点的 `default` 输入的值。
* "clamp"： 在采样图像之前，纹理坐标被钳制到 0-1 范围。
* "periodic": 0-1 范围之外的纹理坐标 "环绕"，在采样图像之前有效地通过模 1 操作处理。
* "mirror": 0-1 范围之外的纹理坐标将被镜像回 0-1 范围，例如 u=-0.01 将返回 u=0.01 纹理坐标值，u=1.01 将返回 u=0.99 纹理坐标值。


<a id="filtertype-values"> </a>

纹理节点的 `filtertype` 输入支持选项 `closest`（最近邻单样本）、`linear` 和 `cubic`。


<a id="framerange-values"> </a>

使用 `file*` 输入的纹理节点还支持以下输入来处理所有 `file*` 输入的图像文件帧范围的边界条件:

* `framerange`（统一字符串）：字符串 "_minframe_-_maxframe_"，例如 "10-99"，用于指定图像文件允许具有的帧范围，通常是磁盘上图像文件的范围。默认为无界。
* `frameoffset`（整数）：添加到当前帧号以获取图像文件帧号的数字。例如，如果 `frameoffset` 为 25，则处理帧 100 将从图像文件序列中读取帧 125。默认为无帧偏移。
* `frameendaction`（统一字符串）：当解析的图像帧号超出 `framerange` 范围时要执行的操作:
    * "constant": 返回节点的 `default` 输入的值(默认操作)
    * "clamp": 对于 _minframe_ 之前的所有帧保持 minframe 图像,对于 _maxframe_ 之后的所有帧保持 maxframe 图像
    * "periodic": 帧号 "环绕",因此在 _maxframe_ 之后它将从 _minframe_ 重新开始(类似地在 _minframe_ 之前环绕回 _maxframe_)
    * "mirror": 帧号在 framerange 的端点处 "镜像" 或 "乒乓",因此读取 _maxframe_ 之后的帧将返回帧 _maxframe_-1 的图像,读取 _minframe_ 之前的帧将返回帧 _minframe_+1 的图像。

不支持任意帧号表达式和速度更改。



## 程序化节点

程序化节点用于以编程方式生成值数据。

```xml
  <constant name="n8" type="color3">
    <input name="value" type="color3" value="0.8,1.0,1.3"/>
  </constant>
  <ramptb name="n9" type="float">
    <input name="valuet" type="float" value="0.9"/>
    <input name="valueb" type="float" value="0.2"/>
  </ramptb>
```

标准程序化节点:

<a id="node-constant"> </a>

### `constant`
输出常量值。

|端口   |描述                         |类型                                    |默认值 |
|-------|------------------------------------|----------------------------------------|--------|
|`value`|将发送到 `out` 的值|float, colorN, vectorN, boolean, integer|__zero__|
|`out`  |输出:`value`                     |Same as `value`                         |__zero__|

|端口   |描述                         |类型           |默认值|
|-------|------------------------------------|---------------|-------|
|`value`|将发送到 `out` 的值|matrixNN       |__one__|
|`out`  |输出:`value`                     |Same as `value`|__one__|

|端口   |描述                         |类型            |默认值  |
|-------|------------------------------------|----------------|---------|
|`value`|将发送到 `out` 的值|string, filename|__empty__|
|`out`  |输出:`value`                     |Same as `value` |__empty__|


<a id="node-ramplr"> </a>

### `ramplr`
从左到右的线性值渐变。

|端口      |描述                                                        |类型                  |默认值 |
|----------|-------------------------------------------------------------------|----------------------|--------|
|`valuel`  |左侧 (U=0) 边缘的值                                       |float, colorN, vectorN|__zero__|
|`valuer`  |右侧 (U=1) 边缘的值                                      |Same as `valuel`      |__zero__|
|`texcoord`|2D texture coordinate at which the ramp interpolation is evaluated |vector2               |_UV0_   |
|`out`     |输出:插值的渐变值                                |Same as `valuel`      |__zero__|

<a id="node-ramptb"> </a>

### `ramptb`
从上到下的线性值渐变。

|端口      |描述                                                        |类型                  |默认值 |
|----------|-------------------------------------------------------------------|----------------------|--------|
|`valuet`  |顶部 (V=1) 边缘的值                                        |float, colorN, vectorN|__zero__|
|`valueb`  |底部 (V=0) 边缘的值                                     |Same as `valuet`      |__zero__|
|`texcoord`|2D texture coordinate at which the ramp interpolation is evaluated |vector2               |_UV0_   |
|`out`     |输出:插值的渐变值                                |Same as `valuet`      |__zero__|

<a id="node-ramp4"> </a>

### `ramp4`
四角双线性值渐变。

|端口      |描述                                                        |类型                  |默认值 |
|----------|-------------------------------------------------------------------|----------------------|--------|
|`valuetl` |左上角 (U=0, V=1) 的值                            |float, colorN, vectorN|__zero__|
|`valuetr` |右上角 (U=1, V=1) 的值                           |Same as `valuetl`     |__zero__|
|`valuebl` |左下角 (U=0, V=0) 的值                         |Same as `valuetl`     |__zero__|
|`valuebr` |右下角 (U=1, V=0) 的值                        |Same as `valuetl`     |__zero__|
|`texcoord`|2D texture coordinate at which the ramp interpolation is evaluated |vector2               |_UV0_   |
|`out`     |输出:插值的渐变值                                |Same as `valuetl`     |__zero__|

<a id="node-splitlr"> </a>

### `splitlr`
左右分割遮罩，在指定的 U 值处分割。

|端口      |描述                                                         |类型                  |默认值 |
|----------|--------------------------------------------------------------------|----------------------|--------|
|`valuel`  |左侧 (U=0) 边缘的值                                        |float, colorN, vectorN|__zero__|
|`valuer`  |右侧 (U=1) 边缘的值                                       |Same as `valuel`      |__zero__|
|`center`  |U-coordinate of the split; left of it is `valuel`, right is `valuer`|float                 |0.5     |
|`texcoord`|2D texture coordinate at which the ramp interpolation is evaluated  |vector2               |_UV0_   |
|`out`     |输出:插值的分割值                                |Same as `valuel`      |__zero__|

<a id="node-splittb"> </a>

### `splittb`
上下分割遮罩，在指定的 V 值处分割。

|端口      |描述                                                        |类型                  |默认值 |
|----------|-------------------------------------------------------------------|----------------------|--------|
|`valuet`  |顶部 (V=1) 边缘的值                                        |float, colorN, vectorN|__zero__|
|`valueb`  |底部 (V=0) 边缘的值                                     |Same as `valuet`      |__zero__|
|`center`  |V-coordinate of the split; below it is `valueb`, above is `valuet` |float                 |0.5     |
|`texcoord`|2D texture coordinate at which the ramp interpolation is evaluated |vector2               |_UV0_   |
|`out`     |输出:插值的分割值                               |Same as `valuet`      |__zero__|

<a id="node-randomfloat"> </a>

### `randomfloat`
基于输入信号和可选的 `seed` 值，生成 `min` 和 `max` 之间的稳定随机浮点值。使用 2d cellnoise 函数生成输出。

|端口      |描述                                                        |类型          |默认值 |
|----------|-------------------------------------------------------------------|--------------|--------|
|`in`      |初始随机化种子                                         |float, integer|__zero__|
|`min`     |最小输出值                                           |float         |__zero__|
|`max`     |最大输出值                                           |float         |__one__ |
|`seed`    |额外的随机化种子                                      |integer       |__zero__|
|`out`     |输出:随机化的值                                       |float         |__zero__|

<a id="node-randomcolor"> </a>

### `randomcolor`
基于输入信号和可选的 `seed` 值，生成随机色调、饱和度和亮度范围内的随机 RGB 颜色。输出类型 color3。

|端口            |描述                                                  |类型          |默认值 |
|----------------|-------------------------------------------------------------|--------------|--------|
|`in`            |初始随机化种子                                   |float, integer|__zero__|
|`huelow`        |最小色相值                                        |float         |__zero__|
|`huehigh`       |最大色相值                                        |float         |__one__ |
|`saturationlow` |最小饱和度值                                 |float         |__zero__|
|`saturationhigh`|The maximum saturation value                                 |float         |__one__ |
|`brightnesslow` |最小亮度值                                 |float         |__zero__|
|`brightnesshigh`|The maximum brightness value                                 |float         |__one__ |
|`seed`          |额外的随机化种子                                |integer       |__zero__|
|`out`           |输出:随机化的 RGB 颜色值                       |color3        |__zero__|


### 程序化节点说明

要缩放或偏移 `rampX` 或 `splitX` 或任何其他具有 `texcoord` 输入的节点的输入坐标，请使用 &lt;place2d> 节点，或由 vector2 &lt;multiply>、&lt;rotate> 和/或 &lt;add> 节点处理的 &lt;texcoord> 或类似几何节点，并连接到节点的 `texcoord` 输入。



## 噪声节点

噪声节点用于使用几种程序化噪声函数之一生成值数据。

```xml
  <noise2d name="n9" type="float">
    <input name="pivot" type="float" value="0.5"/>
    <input name="amplitude" type="float" value="0.05"/>
  </noise2d>
```

标准噪声节点:

<a id="node-noise2d"> </a>

### `noise2d`
1、2、3 或 4 通道的 2D Perlin 噪声。

|端口       |描述                                              |类型                  |默认值 |
|-----------|---------------------------------------------------------|----------------------|--------|
|`amplitude`|The center-to-peak amplitude of the noise                |Same as `out` or float|__one__ |
|`pivot`    |输出噪声的中心值                     |float                 |0.0     |
|`texcoord` |评估噪声的 2D 纹理坐标|vector2               |_UV0_   |
|`out`      |输出:计算的噪声值                         |float, vectorN        |__zero__|

<a id="node-noise3d"> </a>

### `noise3d`
1、2、3 或 4 通道的 3D Perlin 噪声。

|端口       |描述                                    |类型                  |默认值  |
|-----------|-----------------------------------------------|----------------------|---------|
|`amplitude`|The center-to-peak amplitude of the noise      |Same as `out` or float|__one__  |
|`pivot`    |输出噪声的中心值           |float                 |0.0      |
|`position` |评估噪声的 3D 位置|vector3               |_Pobject_|
|`out`      |输出:计算的噪声值               |float, vectorN        |__zero__ |

<a id="node-fractal2d"> </a>

### `fractal2d`
零中心 2D 分形噪声，1、2、3 或 4 通道，通过对多个八度的 2D Perlin 噪声求和创建，在每个八度增加频率并降低振幅。

|端口        |描述                                                    |类型                  |默认值  |
|------------|---------------------------------------------------------------|----------------------|---------|
|`amplitude` |噪声的中心到峰值振幅                      |Same as `out` or float|__one__  |
|`octaves`   |要相加的噪声八度数                    |integer               |3        |
|`lacunarity`|连续噪声八度之间的指数尺度      |float                 |2.0      |
|`diminish`  |每个八度噪声振幅衰减的速率|float                 |0.5      |
|`texcoord`  |评估噪声的 2D 纹理坐标      |vector2               |_UV0_    |
|`out`       |输出:计算的噪声值                               |float, vectorN        |__zero__ |

<a id="node-fractal3d"> </a>

### `fractal3d`
零中心 3D 分形噪声，1、2、3 或 4 通道，通过对多个八度的 3D Perlin 噪声求和创建，在每个八度增加频率并降低振幅。

|端口        |描述                                                    |类型                  |默认值  |
|------------|---------------------------------------------------------------|----------------------|---------|
|`amplitude` |噪声的中心到峰值振幅                      |Same as `out` or float|__one__  |
|`octaves`   |要相加的噪声八度数                    |integer               |3        |
|`lacunarity`|连续噪声八度之间的指数尺度      |float                 |2.0      |
|`diminish`  |每个八度噪声振幅衰减的速率|float                 |0.5      |
|`position`  |评估噪声的 3D 位置                |vector3               |_Pobject_|
|`out`       |输出:计算的噪声值                               |float, vectorN        |__zero__ |

<a id="node-cellnoise2d"> </a>

### `cellnoise2d`
2D 细胞噪声，1 或 3 通道(类型 float 或 vector3)。

|端口      |描述                                              |类型   |默认值|
|----------|---------------------------------------------------------|-------|-------|
|`texcoord`|The 2D texture coordinate at which the noise is evaluated|vector2|_UV0_  |
|`out`     |输出:计算的噪声值                         |float  |0.0    |

<a id="node-cellnoise3d"> </a>

### `cellnoise3d`
3D 细胞噪声，1 或 3 通道(类型 float 或 vector3)。

|端口      |描述                                    |类型   |默认值  |
|----------|-----------------------------------------------|-------|---------|
|`position`|The 3D position at which the noise is evaluated|vector3|_Pobject_|
|`out`     |输出:计算的噪声值               |float  |0.0      |

<a id="node-worleynoise2d"> </a>

### `worleynoise2d`
使用居中抖动的 2D Worley 噪声，输出 float（到最近特征的距离度量）、vector2（到最近 2 个特征的距离度量）或 vector3（到最近 3 个特征的距离度量）值。

|端口      |描述                                              |类型                   |默认值|接受值        |
|----------|---------------------------------------------------------|-----------------------|-------|-----------------------|
|`texcoord`|The 2D texture coordinate at which the noise is evaluated|vector2                |_UV0_  |                       |
|`jitter`  |抖动单元格中心位置的量            |float                  |1.0    |                       |
|`style`   |输出样式                                         |integer                |0      |0 (Distance), 1 (Solid)|
|`out`     |输出:计算的噪声值                         |float, vector2, vector3|0.0    |                       |

<a id="node-worleynoise3d"> </a>

### `worleynoise3d`
使用居中抖动的 3D Worley 噪声，输出 float（到最近特征的距离度量）、vector2（到最近 2 个特征的距离度量）或 vector3（到最近 3 个特征的距离度量）值。

|端口      |描述                                    |类型                   |默认值  |接受值        |
|----------|-----------------------------------------------|-----------------------|---------|-----------------------|
|`position`|The 3D position at which the noise is evaluated|vector3                |_Pobject_|                       |
|`jitter`  |抖动单元格中心位置的量  |float                  |1.0      |                       |
|`style`   |输出样式                               |integer                |0        |0 (Distance), 1 (Solid)|
|`out`     |输出:计算的噪声值               |float, vector2, vector3|__zero__ |                       |

<a id="node-unifiednoise2d"> </a>

### `unifiednoise2d`
在统一接口中支持 2D Perlin、Cell、Worley 或 Fractal 噪声的单个节点。

|端口         |描述                                                                                |类型   |默认值|接受值        |
|-------------|-------------------------------------------------------------------------------------------|-------|-------|-----------------------|
|`texcoord`   |评估噪声的 2D 纹理坐标                                  |vector2|_UV0_  |                       |
|`freq`       |噪声频率，值越高产生的噪声形状越小。                    |vector2|1, 1   |                       |
|`offset`     |偏移 2D 空间的量                                                              |vector2|0, 0   |                       |
|`jitter`     |抖动单元格中心位置的量                                              |float  |1      |                       |
|`outmin`     |最低输出值                                                                    |float  |0      |                       |
|`outmax`     |最高输出值                                                                   |float  |1      |                       |
|`clampoutput`|如果启用，输出将在最小和最大输出值之间钳位                     |boolean|true   |                       |
|`octaves`    |要相加的噪声八度数                                                |integer|3      |                       |
|`lacunarity` |连续噪声八度之间的指数尺度                                  |float  |2      |                       |
|`diminish`   |每个八度噪声振幅衰减的速率                            |float  |0.5    |                       |
|`type`       |要使用的噪声函数类型。0 (Perlin)、1 (Cell)、2 (Worley) 或 3 (Fractal) 之一|integer|0      |                       |
|`style`      |输出样式                                                                           |integer|0      |0 (Distance), 1 (Solid)|
|`out`        |输出:计算的噪声值                                                           |float  |0.0    |                       |

<a id="node-unifiednoise3d"> </a>

### `unifiednoise3d`
在统一接口中支持 3D Perlin、Cell、Worley 或 Fractal 噪声的单个节点。

|端口         |描述                                                                                |类型   |默认值  |接受值        |
|-------------|-------------------------------------------------------------------------------------------|-------|---------|-----------------------|
|`position`   |评估噪声的 3D 位置                                            |vector3|_Pobject_|                       |
|`freq`       |噪声频率，值越高产生的噪声形状越小。                    |vector3|1, 1, 1  |                       |
|`offset`     |偏移 3D 空间的量                                                              |vector3|0, 0, 0  |                       |
|`jitter`     |抖动单元格中心位置的量                                              |float  |1        |                       |
|`outmin`     |最低输出值                                                                    |float  |0        |                       |
|`outmax`     |最高输出值                                                                   |float  |1        |                       |
|`clampoutput`|如果启用，输出将在最小和最大输出值之间钳位                     |boolean|true     |                       |
|`octaves`    |要相加的噪声八度数                                                |integer|3        |                       |
|`lacunarity` |连续噪声八度之间的指数尺度                                  |float  |2        |                       |
|`diminish`   |每个八度噪声振幅衰减的速率                            |float  |0.5      |                       |
|`type`       |要使用的噪声函数类型。0 (Perlin)、1 (Cell)、2 (Worley) 或 3 (Fractal) 之一|integer|0        |                       |
|`style`      |输出样式                                                                           |integer|0        |0 (Distance), 1 (Solid)|
|`out`        |输出:计算的噪声值                                                           |float  |0.0      |                       |

<a id="node-flake2d"> </a>

### `flake2d`
在 2D 空间中生成程序化薄片图案，适用于模拟汽车油漆等材料中的金属薄片。

|端口         |描述                                                                                        |类型   |默认值 |
|-------------|---------------------------------------------------------------------------------------------------|-------|--------|
|`texcoord`   |评估雪花图案的 2D 纹理坐标                                  |vector2|_UV0_   |
|`size`       |单个雪花的尺寸，值越小产生越大的雪花                         |float  |0.01    |
|`roughness`  |单个雪花的表面粗糙度，控制法线的变化                    |float  |0.1     |
|`coverage`   |图案中雪花的密度，范围从 0.0 （无雪花） 到 1.0 （最大）                |float  |0.5     |
|`normal`     |用作雪花法线扰动基础的表面法线向量                          |vector3|_Nworld_|
|`tangent`    |用于构建切空间的表面切向量                                    |vector3|_Tworld_|
|`bitangent`  |用于构建切空间的表面副切向量                                  |vector3|_Bworld_|
|`id`         |输出:每个雪花的唯一标识符。无雪花时为 0                                           |integer|0       |
|`rand`       |输出:每个雪花的随机值以提供额外变化。无雪花时为 0.0                          |float  |0.0     |
|`presence`   |输出:每个雪花的存在度；类似深度的值（越高越接近表面）。无雪花时为 0.0.|float  |0.0     |
|`flakenormal`|Output: the computed flake normal. Base normal if no flake present                                 |vector3|_Nworld_|

<a id="node-flake3d"> </a>

### `flake3d`
在 3D 空间中生成程序化薄片图案，适用于模拟汽车油漆等材料中的金属薄片。

|端口         |描述                                                                                        |类型   |默认值  |
|-------------|---------------------------------------------------------------------------------------------------|-------|---------|
|`position`   |评估雪花图案的 3D 位置                                            |vector3|_Pobject_|
|`size`       |单个雪花的尺寸，值越小产生越大的雪花                         |float  |0.01     |
|`roughness`  |单个雪花的表面粗糙度，控制法线的变化                    |float  |0.1      |
|`coverage`   |图案中雪花的密度，范围从 0.0 （无雪花） 到 1.0 （最大）                |float  |0.5      |
|`normal`     |用作雪花法线扰动基础的表面法线向量                          |vector3|_Nworld_ |
|`tangent`    |用于构建切空间的表面切向量                                    |vector3|_Tworld_ |
|`bitangent`  |用于构建切空间的表面副切向量                                  |vector3|_Bworld_ |
|`id`         |输出:每个雪花的唯一标识符。无雪花时为 0                                           |integer|0        |
|`rand`       |输出:每个雪花的随机值以提供额外变化。无雪花时为 0.0                          |float  |0.0      |
|`presence`   |输出:每个雪花的存在度；类似深度的值（越高越接近表面）。无雪花时为 0.0.|float  |0.0      |
|`flakenormal`|Output: the computed flake normal. Base normal if no flake present                                 |vector3|_Nworld_ |

### 噪声节点说明

要缩放或偏移由 3D 噪声节点（如 `noise3d`、`fractal3d` 或 `cellnoise3d`）生成的噪声图案，请使用 &lt;position> 或其他 [几何节点](#geometric-nodes)（见下文）连接到 vector3 &lt;multiply> 和/或 &lt;add> 节点，再连接到噪声节点的 `position` 输入。

要缩放或偏移由 2D 噪声节点（如 `noise2d` 或 `cellnoise2d`）生成的噪声图案，请使用 &lt;place2d> 节点，或由 vector2 &lt;multiply>、&lt;rotate> 和/或 &lt;add> 节点处理的 &lt;texcoord> 或类似几何节点，并连接到节点的 `texcoord` 输入。



## 形状节点

形状节点用于在 UV 空间中生成形状或图案。

```xml
  <checkerboard name="n10" type="color3">
    <input name="color1" type="color3" value="1.0,0.0,0.0"/>
    <input name="color2" type="color3" value="0.0,0.0,1.0"/>
    <input name="uvtiling" type="vector2" value="8, 8"/>
  </checkerboard>
```

标准形状节点:

<a id="node-checkerboard"> </a>

### `checkerboard`
2D 棋盘格图案。

|端口      |描述                                                                                          |类型   |默认值      |
|----------|-----------------------------------------------------------------------------------------------------|-------|-------------|
|`color1`  |The first color used in the checkerboard pattern.                                                    |color3 |1.0, 1.0, 1.0|
|`color2`  |The second color used in the checkerboard pattern.                                                   |color3 |0.0, 0.0, 0.0|
|`uvtiling`|The tiling of the checkerboard pattern along each axis, with higher values producing smaller squares.|vector2|8, 8         |
|`uvoffset`|The offset of the checkerboard pattern along each axis                                               |vector2|0, 0         |
|`texcoord`|输入 2D 空间。                                                                                  |vector2|_UV0_        |
|`out`     |输出:棋盘格图案                                                                     |color3 |             |

<a id="node-line"> </a>

### `line`
2D 线条图案:如果 texcoord 距离由 `point1` 和 `point2` 定义的线段小于 `radius` 距离，则输出 1，否则输出 0。

|端口      |描述                                                    |类型   |默认值   |
|----------|---------------------------------------------------------------|-------|----------|
|`texcoord`|输入 2D 空间                                             |vector2|_UV0_     |
|`center`  |添加到 point1 和 point2 坐标的偏移值|vector2|0, 0      |
|`radius`  |线条的半径或'半厚度'                     |float  |0.1       |
|`point1`  |第一个端点的 UV 坐标                        |vector2|0.25, 0.25|
|`point2`  |第二个端点的 UV 坐标                       |vector2|0.75, 0.75|
|`out`     |输出:线条图案                                       |float  |          |

<a id="node-circle"> </a>

### `circle`
2D 圆形（圆盘）图案:如果 texcoord 在由 `center` 和 `radius` 定义的圆内，则输出 1，否则输出 0。

|端口      |描述                        |类型   |默认值|
|----------|-----------------------------------|-------|-------|
|`texcoord`|输入 2D 空间                 |vector2|_UV0_  |
|`center`  |圆的中心坐标|vector2|0, 0   |
|`radius`  |圆的半径           |float  |0.5    |
|`out`     |输出:圆形图案         |float  |       |

<a id="node-cloverleaf"> </a>

### `cloverleaf`
2D 四叶草图案:正方形边缘上的四个半圆，由 `center` 和 `radius` 定义。如果 texcoord 在图案内，则输出 1，否则输出 0。

|端口      |描述                            |类型   |默认值|
|----------|---------------------------------------|-------|-------|
|`texcoord`|输入 2D 空间                     |vector2|_UV0_  |
|`center`  |三叶草的中心坐标|vector2|0, 0   |
|`radius`  |三叶草的半径           |float  |0.5    |
|`out`     |输出:三叶草图案         |float  |       |

<a id="node-hexagon"> </a>

### `hexagon`
2D 六边形图案:如果 texcoord 在由 `center` 和 `radius` 定义的圆内接的六边形形状内，则输出 1；否则输出 0。

|端口      |描述                         |类型   |默认值|
|----------|------------------------------------|-------|-------|
|`texcoord`|输入 2D 空间                  |vector2|_UV0_  |
|`center`  |六边形的中心坐标|vector2|0, 0   |
|`radius`  |六边形的半径           |float  |0.5    |
|`out`     |输出:六边形图案         |float  |       |

<a id="node-grid"> </a>

### `grid`
创建给定平铺、偏移和线宽的网格图案，(1, 1, 1) 白色线条在 (0, 0, 0) 黑色背景上。图案可以是规则的或交错的。

|端口       |描述                                                                  |类型   |默认值 |
|-----------|-----------------------------------------------------------------------------|-------|--------|
|`texcoord` |输入 2D 空间                                                           |vector2|_UV0_   |
|`uvtiling` |平铺因子，值越高产生越密集的网格。                   |vector2|1.0, 1.0|
|`uvoffset` |UV 偏移                                                                    |vector2|0.0, 0.0|
|`thickness`|网格线的厚度                                              |float  |0.05    |
|`staggered`|如果为 true，每隔一行将偏移 50% 以产生'砖墙'图案|boolean|false   |
|`out`      |输出:网格图案                                                     |color3 |        |

<a id="node-crosshatch"> </a>

### `crosshatch`
创建具有给定平铺、偏移和线宽的十字交叉图案。图案可以是规则的或交错的。

|端口       |描述                                                                            |类型   |默认值 |
|-----------|---------------------------------------------------------------------------------------|-------|--------|
|`texcoord` |输入 2D 空间                                                                     |vector2|_UV0_   |
|`uvtiling` |平铺因子，值越高产生越密集的网格。                             |vector2|1.0, 1.0|
|`uvoffset` |UV 偏移                                                                              |vector2|0.0, 0.0|
|`thickness`|网格线的厚度                                                        |float  |0.05    |
|`staggered`|如果为 true，每隔一行将偏移 50% 以产生'交替菱形'图案|boolean|false   |
|`out`      |输出:十字交叉图案                                                         |color3 |        |

<a id="node-tiledcircles"> </a>

### `tiledcircles`
创建具有定义平铺和大小（直径）的圆的黑白图案。图案可以是规则的或交错的。

|端口       |描述                                                                                                                                                                         |类型   |默认值 |
|-----------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-------|--------|
|`texcoord` |输入 2D 空间                                                                                                                                                                  |vector2|_UV0_   |
|`uvtiling` |平铺因子，值越高产生越密集的网格。                                                                                                                          |vector2|1.0, 1.0|
|`uvoffset` |UV 偏移                                                                                                                                                                           |vector2|0.0, 0.0|
|`size`     |平铺图案中圆形的直径。如果 `size` 为 1.0，平铺中相邻圆形的边缘将恰好接触。                                                 |float  |0.5     |
|`staggered`|如果为 true，每隔一行将偏移 50%，并且平铺的间距将在 V 方向上调整以将圆形居中于等边三角形网格的顶点上|boolean|false   |
|`out`      |输出:平铺圆形图案                                                                                                                                                    |color3 |        |

<a id="node-tiledcloverleafs"> </a>

### `tiledcloverleafs`
创建具有定义平铺和大小（外切形状的圆的直径）的四叶草的黑白图案。图案可以是规则的或交错的。

|端口       |描述                                                                                                                                    |类型   |默认值 |
|-----------|-----------------------------------------------------------------------------------------------------------------------------------------------|-------|--------|
|`texcoord` |输入 2D 空间                                                                                                                             |vector2|_UV0_   |
|`uvtiling` |平铺因子，值越高产生越密集的网格。                                                                                     |vector2|1.0, 1.0|
|`uvoffset` |UV 偏移                                                                                                                                      |vector2|0.0, 0.0|
|`size`     |平铺图案中三叶草的外直径。如果 size 为 1.0，平铺中相邻三叶草的边缘将恰好接触。|float  |0.5     |
|`staggered`|如果为 true，将在原始图案之间生成额外的三叶草图案，在 U 和 V 方向上偏移 50%                         |boolean|false   |
|`out`      |输出:平铺三叶草图案                                                                                                           |color3 |        |

<a id="node-tiledhexagons"> </a>

### `tiledhexagons`
创建具有定义平铺和大小（外切形状的圆的直径）的六边形的黑白图案。图案可以是规则的或交错的。

|端口       |描述                                                                                                                                                                           |类型   |默认值 |
|-----------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-------|--------|
|`texcoord` |输入 2D 空间                                                                                                                                                                    |vector2|_UV0_   |
|`uvtiling` |平铺因子，值越高产生越密集的网格。                                                                                                                            |vector2|1.0, 1.0|
|`uvoffset` |UV 偏移                                                                                                                                                                             |vector2|0.0, 0.0|
|`size`     |平铺图案中六边形的内直径。如果 size 为 1.0，U 方向平铺中相邻六边形的边缘将恰好接触                                 |float  |0.5     |
|`staggered`|如果为 true，每隔一行将偏移 50%，并且平铺的间距将在 V 方向上调整以将六边形居中于等边三角形网格的顶点上。|boolean|false   |
|`out`      |输出:平铺六边形图案                                                                                                                                                     |color3 |        |



## 几何节点

几何节点用于从节点图中引用局部几何属性:

```xml
  <position name="wp1" type="vector3">
    <input name="space" type="string" value="world"/>
  </position>
  <texcoord name="c1" type="vector2">
    <input name="index" type="integer" value="1"/>
  </texcoord>
```

标准几何节点:

<a id="node-position"> </a>

### `position`
与当前处理的数据相关的坐标，在特定坐标系中定义。

|端口   |描述                                |类型   |默认值      |接受值     |
|-------|-------------------------------------------|-------|-------------|--------------------|
|`space`|The coordinate space of the output position|string |object       |model, object, world|
|`out`  |输出:`space` 中的位置            |vector3|0.0, 0.0, 0.0|                    |

<a id="node-normal"> </a>

### `normal`
与当前处理的数据相关的归一化几何法线，在特定坐标系中定义。

|端口   |描述                                |类型   |默认值      |接受值     |
|-------|-------------------------------------------|-------|-------------|--------------------|
|`space`|The coordinate space of the output position|string |object       |model, object, world|
|`out`  |输出:`space` 中的法线              |vector3|0.0, 1.0, 0.0|                    |

<a id="node-tangent"> </a>

### `tangent`
与当前处理的数据相关的几何切线向量，在特定坐标系中定义。

|端口   |描述                                                                          |类型   |默认值      |接受值     |
|-------|-------------------------------------------------------------------------------------|-------|-------------|--------------------|
|`space`|The coordinate space of the output position                                          |string |object       |model, object, world|
|`index`|The index of the texcoord space to use to compute the tangent vector                 |integer|0            |                    |
|`out`  |输出:与第 `index` 个坐标空间相关的切向量，在 `space` 中|vector3|1.0, 0.0, 0.0|                    |

<a id="node-bitangent"> </a>

### `bitangent`
与当前处理的数据相关的几何副切线向量，在特定坐标系中定义。

|端口   |描述                                                                            |类型   |默认值      |接受值     |
|-------|---------------------------------------------------------------------------------------|-------|-------------|--------------------|
|`space`|The coordinate space of the output position                                            |string |object       |model, object, world|
|`index`|The index of the texcoord space to use to compute the bitangent vector                 |integer|0            |                    |
|`out`  |输出:与第 `index` 个坐标空间相关的副切向量，在 `space` 中|vector3|0.0, 0.0, 1.0|                    |

<a id="node-bump"> </a>

### `bump`
通过沿其世界空间法线偏移表面世界空间位置计算的归一化法线。

|端口       |描述                         |类型   |默认值 |
|-----------|------------------------------------|-------|--------|
|`height`   |偏移表面法线的量。|float  |0       |
|`scale`    |调整高度量的标量。 |float  |1       |
|`normal`   |表面法线                      |vector3|_Nworld_|
|`tangent`  |表面切向量              |vector3|_Tworld_|
|`bitangent`|Surface bitangent vector            |vector3|_Bworld_|
|`out`      |输出:偏移后的表面法线       |vector3|        |

<a id="node-texcoord"> </a>

### `texcoord`
与当前处理的数据相关的 2D 或 3D 纹理坐标

|端口   |描述                |类型            |默认值 |
|-------|---------------------------|----------------|--------|
|`index`|Texcoord index             |integer         |0       |
|`out`  |输出:纹理坐标|vector2, vector3|__zero__|

<a id="node-geomcolor"> </a>

### `geomcolor`
与当前位置的当前几何体相关的颜色，通常通过每顶点颜色值绑定。类型必须与绑定到几何体的 "color" 的类型匹配。

|端口   |描述                 |类型         |默认值 |
|-------|----------------------------|-------------|--------|
|`index`|index of the geometric color|integer      |0       |
|`out`  |输出:几何颜色     |float, colorN|__zero__|

<a id="node-geompropvalue"> </a>

### `geompropvalue`
当前绑定几何体的指定变化几何属性（由 &lt;geompropdef> 定义）的值。此节点的类型必须与引用的 geomprop 的类型匹配。

|端口      |描述                                                                         |类型                                    |默认值  |
|----------|------------------------------------------------------------------------------------|----------------------------------------|---------|
|`geomprop`|The geometric property to be referenced                                             |string                                  |__empty__|
|`default` |A value to return if the specified `geomprop` is not defined on the current geometry|float, colorN, vectorN, boolean, integer|__zero__ |
|`out`     |输出:`geomprop` 值                                                        |Same as `default`                       |__zero__ |

<a id="node-geompropvalueuniform"> </a>

### `geompropvalueuniform`
当前绑定几何体的指定统一几何属性（由 &lt;geompropdef> 定义）的值。此节点的类型必须与引用的 geomprop 的类型匹配。

|端口      |描述                                                                                 |类型             |默认值  |
|----------|--------------------------------------------------------------------------------------------|-----------------|---------|
|`geomprop`|The geometric property to be referenced                                                     |string           |__empty__|
|`default` |A value to return if the specified `geomprop` is not defined on the current geometry        |string, filename |__zero__ |
|`out`     |输出:如果当前几何体上未定义指定的 `geomprop`，则返回的值|Same as `default`|__zero__ |


### 几何节点说明

可以为 &lt;geomcolor> 和 &lt;geompropvalue> 节点的 color3/color4 类型属性指定 `colorspace` 属性，以声明颜色属性值所在的色彩空间；默认为 "none" 表示无色彩空间声明（因此无色彩空间转换）。



## 应用程序节点

应用程序节点用于在节点图中引用应用程序定义的属性，没有输入:

```xml
  <frame name="f1" type="float"/>
  <time name="t1" type="float"/>
```

标准应用程序节点:

<a id="node-frame"> </a>

### `frame`
主机环境定义的当前帧号。

|端口 |描述                                            |类型 |默认值|
|-----|-------------------------------------------------------|-----|-------|
|`out`|Output: frame number as defined by the host environment|float|1.0    |

<a id="node-time"> </a>

### `time`
主机环境定义的当前时间（秒）。

|端口 |描述                                                       |类型 |默认值|
|-----|------------------------------------------------------------------|-----|-------|
|`out`|Output: current time in seconds as defined by the host environment|float|0.0    |


### 应用程序节点说明

应用程序可以使用任何适当的方法将当前帧号或时间通信给 &lt;frame> 或 &lt;time> 节点的实现，无论是通过内部状态变量、自定义输入、将当前帧号除以本地 "每秒帧数" 值（仅 &lt;time> 节点）或其他方法。实时应用程序可能返回某种形式的挂钟时间。


<br>

# 标准操作符节点

操作符节点处理一个或多个必需的输入流以形成输出。与其他节点一样，每个操作符必须定义其输出类型，在大多数情况下，这也决定了所需输入流的类型。

```xml
  <multiply name="n7" type="color3">
    <input name="in1" type="color3" nodename="n5"/>
    <input name="in2" type="float" value="2.0"/>
  </multiply>
  <over name="n11" type="color4">
    <input name="fg" type="color4" nodename="n8"/>
    <input name="bg" type="color4" nodename="inbg"/>
  </over>
  <add name="n2" type="color3">
    <input name="in1" type="color3" nodename="n12"/>
    <input name="in2" type="color3" nodename="img4"/>
  </add>
```

合成操作符的输入称为 "fg" 和 "bg"（加上 float 和 color3 变体的 "alpha"，以及 `mix` 操作符所有变体的 "mix"），而大多数其他操作符的输入如果只有一个输入则称为 "in"，如果有多个输入则称为 "in1"、"in2" 等。如果实现不支持特定操作符，它通常应不加修改地传递 "bg"、"in" 或 "in1" 输入。

本节定义了所有 MaterialX 实现都应支持的操作符节点。标准操作符节点分为以下类别:[数学节点](#math-nodes)、[调整节点](#adjustment-nodes)、[合成节点](#compositing-nodes)、[条件节点](#conditional-nodes)、[通道节点](#channel-nodes) 和 [卷积节点](#convolution-nodes)。



## 数学节点

数学节点有一个或两个空间变化输入，用于对一个空间变化输入流中的值执行数学运算，或使用指定的数学运算组合两个空间变化输入流。对输入流的每个通道执行给定的数学运算，并且每个输入的数据类型必须与输入流的数据类型匹配，或者是将分别应用于每个通道的 float 值。


<a id="node-add"> </a>

### `add`
将值添加到传入的 float/color/vector/matrix

|端口 |描述                   |类型                           |默认值 |
|-----|------------------------------|-------------------------------|--------|
|`in1`|The primary input stream      |float, colorN, vectorN, integer|__zero__|
|`in2`|要添加到 `in1` 的流    |Same as `in1`                  |__zero__|
|`out`|Output: sum of `in1` and `in2`|Same as `in1`                  |`in1`   |

|端口 |描述                   |类型           |默认值 |
|-----|------------------------------|---------------|--------|
|`in1`|The primary input stream      |colorN, vectorN|__zero__|
|`in2`|要添加到 `in1` 的流    |float          |__zero__|
|`out`|Output: sum of `in1` and `in2`|Same as `in1`  |`in1`   |

|端口 |描述                   |类型                  |默认值 |
|-----|------------------------------|----------------------|--------|
|`in1`|The primary input stream      |matrixNN              |__one__ |
|`in2`|要添加到 `in1` 的流    |Same as `in1` or float|__zero__|
|`out`|Output: sum of `in1` and `in2`|Same as `in1`         |`in1`   |

<a id="node-subtract"> </a>

### `subtract`
从传入的 float/color/vector/matrix 中减去一个值

|端口 |描述                      |类型                           |默认值 |
|-----|---------------------------------|-------------------------------|--------|
|`in1`|The primary input stream         |float, colorN, vectorN, integer|__zero__|
|`in2`|要从 `in1` 中减去的流|Same as `in1`                  |__zero__|
|`out`|Output: `in1` minus `in2`        |Same as `in1`                  |`in1`   |

|端口 |描述                      |类型           |默认值 |
|-----|---------------------------------|---------------|--------|
|`in1`|The primary input stream         |colorN, vectorN|__zero__|
|`in2`|要从 `in1` 中减去的流|float          |__zero__|
|`out`|Output: `in1` minus `in2`        |Same as `in1`  |`in1`   |

|端口 |描述                      |类型                  |默认值 |
|-----|---------------------------------|----------------------|--------|
|`in1`|The primary input stream         |matrixNN              |__one__ |
|`in2`|要从 `in1` 中减去的流|Same as `in1` or float|__zero__|
|`out`|Output: `in1` minus `in2`        |Same as `in1`         |`in1`   |

<a id="node-multiply"> </a>

### `multiply`
将两个值相乘。标量和向量类型按分量相乘，而矩阵使用标准矩阵乘法相乘。

|端口 |描述                       |类型                  |默认值 |
|-----|----------------------------------|----------------------|--------|
|`in1`|The primary input stream          |float, colorN, vectorN|__zero__|
|`in2`|要与 `in1` 相乘的流   |Same as `in1` or float|__one__ |
|`out`|Output: product of `in1` and `in2`|Same as `in1`         |`in1`   |

|端口 |描述                       |类型         |默认值|
|-----|----------------------------------|-------------|-------|
|`in1`|The primary input stream          |matrixNN     |__one__|
|`in2`|要与 `in1` 相乘的流   |Same as `in1`|__one__|
|`out`|Output: product of `in1` and `in2`|Same as `in1`|`in1`  |

<a id="node-divide"> </a>

### `divide`
将一个值除以另一个值。标量和向量类型按分量相除，而对于矩阵，`in1` 乘以 `in2` 的逆矩阵。

|端口 |描述                   |类型                  |默认值 |
|-----|------------------------------|----------------------|--------|
|`in1`|The primary input stream      |float, colorN, vectorN|__zero__|
|`in2`|The stream to divide `in1` by |Same as `in1` or float|__one__ |
|`out`|Output: `in1` divided by `in2`|Same as `in1`         |`in1`   |

|端口 |描述                   |类型         |默认值|
|-----|------------------------------|-------------|-------|
|`in1`|The primary input stream      |matrixNN     |__one__|
|`in2`|The stream to divide `in1` by |Same as `in1`|__one__|
|`out`|Output: `in1` divided by `in2`|Same as `in1`|`in1`  |

<a id="node-modulo"> </a>

### `modulo`
将传入的 float/color/vector 除以一个值并减去整数部分后的剩余分数。Modulo 始终返回非负结果。

|端口 |描述                  |类型                  |默认值 |接受值|
|-----|-----------------------------|----------------------|--------|---------------|
|`in1`|The primary input stream     |float, colorN, vectorN|__zero__|               |
|`in2`|The stream to modulo `in1` by|Same as `in1` or float|__one__ |`in2` != 0     |
|`out`|Output: `in1` modulo `in2`   |Same as `in1`         |`in1`   |               |

<a id="node-fract"> </a>

### `fract`
返回浮点输入的小数部分。

|端口 |描述                        |类型                  |默认值 |
|-----|-----------------------------------|----------------------|--------|
|`in` |主要输入流           |float, colorN, vectorN|__zero__|
|`out`|Output: The fractional part of `in`|Same as `in`          |`in`    |

<a id="node-invert"> </a>

### `invert`
从 `amount` 中减去所有通道的传入 float、color 或 vector，输出:`amount - in`。

|端口    |描述                    |类型                  |默认值 |
|--------|-------------------------------|----------------------|--------|
|`in`    |主要输入流       |float, colorN, vectorN|__zero__|
|`amount`|The value to subtract `in` from|Same as `in` or float |__one__ |
|`out`   |输出:`amount` 减去 `in`    |Same as `in`          |`in`    |

<a id="node-absval"> </a>

### `absval`
传入的 float/color/vector 的每通道绝对值。

|端口 |描述                   |类型                  |默认值 |
|-----|------------------------------|----------------------|--------|
|`in` |主要输入流      |float, colorN, vectorN|__zero__|
|`out`|Output: absolute value of `in`|Same as `in`          |`in`    |

<a id="node-sign"> </a>

### `sign`
传入的 float/color/vector 值的每通道符号:负数为 -1，正数为 +1，零为 0。

|端口 |描述             |类型                  |默认值 |
|-----|------------------------|----------------------|--------|
|`in` |主要输入流|float, colorN, vectorN|__zero__|
|`out`|Output: sign of `in`    |Same as `in`          |`in`    |

<a id="node-floor"> </a>

### `floor`
小于或等于传入的 float/color/vector 的每通道最近整数值。输出保持每通道浮点，即与输入相同的类型，除了 float 输入的 &lt;floor> 还有一个输出整数类型的变体。

|端口 |描述                                       |类型                  |默认值 |
|-----|--------------------------------------------------|----------------------|--------|
|`in` |主要输入流                          |float, colorN, vectorN|__zero__|
|`out`|Output: nearest integer less than or equal to `in`|Same as `in`          |`in`    |

|端口 |描述                                       |类型   |默认值|
|-----|--------------------------------------------------|-------|-------|
|`in` |主要输入流                          |float  |0.0    |
|`out`|Output: nearest integer less than or equal to `in`|integer|`in`   |

<a id="node-ceil"> </a>

### `ceil`
大于或等于传入的 float/color/vector 的每通道最近整数值。输出保持每通道浮点，即与输入相同的类型，除了 float 输入的 &lt;ceil> 还有一个输出整数类型的变体。

|端口 |描述                                          |类型                  |默认值 |
|-----|-----------------------------------------------------|----------------------|--------|
|`in` |主要输入流                             |float, colorN, vectorN|__zero__|
|`out`|Output: nearest integer greater than or equal to `in`|Same as `in`          |`in`    |

|端口 |描述                                          |类型   |默认值|
|-----|-----------------------------------------------------|-------|-------|
|`in` |主要输入流                             |float  |0.0    |
|`out`|Output: nearest integer greater than or equal to `in`|integer|`in`   |

<a id="node-round"> </a>

### `round`
将传入的 float/color/vector 值的每个通道四舍五入到最近的整数值。

|端口 |描述                                |类型                  |默认值 |
|-----|-------------------------------------------|----------------------|--------|
|`in` |主要输入流                   |float, colorN, vectorN|__zero__|
|`out`|Output: `in` rounded to the nearest integer|Same as `in`          |`in`    |

|端口 |描述                                |类型   |默认值|
|-----|-------------------------------------------|-------|-------|
|`in` |主要输入流                   |float  |0.0    |
|`out`|Output: `in` rounded to the nearest integer|integer|`in`   |

<a id="node-power"> </a>

### `power`
将传入的 float/color 值提升到指定的指数，通常用于 "gamma" 调整。

|端口 |描述                   |类型                  |默认值 |
|-----|------------------------------|----------------------|--------|
|`in1`|The primary input stream      |float, colorN, vectorN|__zero__|
|`in2`|The exponent to raise `in1` to|Same as `in1` or float|__one__ |
|`out`|Output: `in1` raised to `in2` |Same as `in1`         |`in1`   |

<a id="node-safepower"> </a>

### `safepower`
将传入的 float/color 值提升到指定的指数。负的 "in1" 值将导致负的输出值。

|端口 |描述                   |类型                  |默认值 |
|-----|------------------------------|----------------------|--------|
|`in1`|The primary input stream      |float, colorN, vectorN|__zero__|
|`in2`|The exponent to raise `in1` to|Same as `in1` or float|__one__ |
|`out`|Output: `in1` raised to `in2` |Same as `in1`         |`in1`   |

<a id="node-sin"> </a>

### `sin`
传入值的正弦，期望以弧度表示。

|端口 |描述             |类型          |默认值 |
|-----|------------------------|--------------|--------|
|`in` |主要输入流|float, vectorN|__zero__|
|`out`|Output: sin of `in`     |Same as `in`  |`in`    |

<a id="node-cos"> </a>

### `cos`
传入值的余弦，期望以弧度表示。

|端口 |描述             |类型          |默认值 |
|-----|------------------------|--------------|--------|
|`in` |主要输入流|float, vectorN|__zero__|
|`out`|Output: cos of `in`     |Same as `in`  |`in`    |

<a id="node-tan"> </a>

### `tan`
传入值的正切，期望以弧度表示。

|端口 |描述             |类型          |默认值 |
|-----|------------------------|--------------|--------|
|`in` |主要输入流|float, vectorN|__zero__|
|`out`|Output: tan of `in`     |Same as `in`  |`in`    |

<a id="node-asin"> </a>

### `asin`
传入值的反正弦。输出将以弧度表示。

|端口 |描述             |类型          |默认值 |接受值    |
|-----|------------------------|--------------|--------|-------------------|
|`in` |主要输入流|float, vectorN|__zero__|[-__one__, __one__]|
|`out`|Output: asin of `in`    |Same as `in`  |`in`    |                   |

<a id="node-acos"> </a>

### `acos`
传入值的反余弦。输出将以弧度表示。

|端口 |描述             |类型          |默认值 |接受值    |
|-----|------------------------|--------------|--------|-------------------|
|`in` |主要输入流|float, vectorN|__zero__|[-__one__, __one__]|
|`out`|Output: acos of `in`    |Same as `in`  |`in`    |                   |

<a id="node-atan2"> </a>

### `atan2`
表达式 (`iny`/`inx`) 的反正切。输出将以弧度表示。

|端口 |描述                                                                                     |类型          |默认值 |
|-----|------------------------------------------------------------------------------------------------|--------------|--------|
|`iny`|Vertical component of the point to which the the angle is to be found                           |float, vectorN|__zero__|
|`inx`|Horizontal component of the point to which the angle is to be found                             |Same as `iny` |__one__ |
|`out`|Output: angle relative to the X-axis of the line joining the origin and the point (`inx`, `iny`)|Same as `iny` |`iny`   |

<a id="node-sqrt"> </a>

### `sqrt`
传入值的平方根。 

|端口 |描述                |类型          |默认值 |接受值     |
|-----|---------------------------|--------------|--------|--------------------|
|`in` |主要输入流   |float, vectorN|__zero__|[__zero__, __+inf__)|
|`out`|Output: square root of `in`|Same as `in`  |`in`    |                    |

<a id="node-ln"> </a>

### `ln`
传入值的自然对数。 

|端口 |描述                      |类型          |默认值|接受值       |
|-----|---------------------------------|--------------|-------|----------------------|
|`in` |主要输入流         |float, vectorN|__one__| (__zero__, __+inf__) |
|`out`|Output: natural logarithm of `in`|Same as `in`  |`in`   |                      |

<a id="node-exp"> </a>

### `exp`
$e$ 的传入值次幂。

|端口 |描述                |类型          |默认值 |
|-----|---------------------------|--------------|--------|
|`in` |主要输入流   |float, vectorN|__zero__|
|`out`|Output: exponential of `in`|Same as `in`  |`in`    |

<a id="node-clamp"> </a>

### `clamp`
将传入的值每通道钳制到指定的 float/color/vector 值范围。

|端口  |描述                                                       |类型                  |默认值 |
|------|------------------------------------------------------------------|----------------------|--------|
|`in`  |要钳制的输入流                                    |float, colorN, vectorN|__zero__|
|`low` |`in` 中低于此值的任何值将被设置为此值 |Same as `in` or float |__zero__|
|`high`|`in` 中高于此值的任何值将被设置为此值|Same as `low`         |__one__ |
|`out` |输出:钳位后的 `in`                                              |Same as `in`          |`in`    |

<a id="node-trianglewave"> </a>

### `trianglewave`
从给定的标量输入生成三角波。生成的波从零到一，在整数边界上重复。

|端口 |描述                  |类型 |默认值|
|-----|-----------------------------|-----|-------|
|`in` |主要输入流     |float|0      |
|`out`|Output: generated wave signal|float|       |

<a id="node-min"> </a>

### `min`
选择两个传入值中的最小值。

|端口 |描述                       |类型                  |默认值 |
|-----|----------------------------------|----------------------|--------|
|`in1`|The first input stream            |float, colorN, vectorN|__zero__|
|`in2`|The second input stream           |Same as `in1` or float|__zero__|
|`out`|Output: minimum of `in1` and `in2`|Same as `in1`         |`in1`   |

<a id="node-max"> </a>

### `max`
选择两个传入值中的最大值。

|端口 |描述                       |类型                  |默认值 |
|-----|----------------------------------|----------------------|--------|
|`in1`|The first input stream            |float, colorN, vectorN|__zero__|
|`in2`|The second input stream           |Same as `in1` or float|__zero__|
|`out`|Output: maximum of `in1` and `in2`|Same as `in1`         |`in1`   |

<a id="node-normalize"> </a>

### `normalize`
输出归一化的传入 vectorN 流。 

|端口 |描述            |类型        |默认值 |
|-----|-----------------------|------------|--------|
|`in` |要归一化的向量|vectorN     |__zero__|
|`out`|Output: normalized `in`|Same as `in`|`in`    |

<a id="node-magnitude"> </a>

### `magnitude`
输出传入 vectorN 流的浮点幅度（向量长度）；不能用于 float 或 colorN 流。注意:vector4 流中的第四个通道没有特殊处理，例如不作为齐次 "w" 值。

|端口 |描述              |类型   |默认值 |
|-----|-------------------------|-------|--------|
|`in` |输入向量流      |vectorN|__zero__|
|`out`|Output: magnitude of `in`|float  |0.0     |

<a id="node-distance"> </a>

### `distance`
测量 2D、3D 或 4D 中两点之间的距离。

|端口 |描述                             |类型         |默认值 |
|-----|----------------------------------------|-------------|--------|
|`in1`|Point to calculate distance from        |vectorN      |__zero__|
|`in2`|Point to calculate distance to          |Same as `in1`|__zero__|
|`out`|Output: distance between `in1` and `in2`|float        |        |

<a id="node-dotproduct"> </a>

### `dotproduct`
输出两个传入 vectorN 流的（浮点）点积；不能用于 float 或 colorN 流。

|端口 |描述                           |类型         |默认值 |
|-----|--------------------------------------|-------------|--------|
|`in1`|The first input vector stream         |vectorN      |__zero__|
|`in2`|The second input vector stream        |Same as `in1`|__zero__|
|`out`|Output: dot product of `in1` and `in2`|float        |0.0     |

<a id="node-crossproduct"> </a>

### `crossproduct`
输出两个传入 vector3 流的（矢量3）叉积；不能用于任何其他流类型。禁用的 crossproduct 节点将 `in1` 的值不变地传递。

|端口 |描述                             |类型   |默认值      |
|-----|----------------------------------------|-------|-------------|
|`in1`|The first input vector stream           |vector3|0.0, 0.0, 0.0|
|`in2`|The second input vector stream          |vector3|0.0, 0.0, 0.0|
|`out`|Output: cross product of `in1` and `in2`|vector3|`in1`        |

<a id="node-transformpoint"> </a>

### `transformpoint`
将传入的 vector3 坐标从一个指定空间变换到另一个空间；不能用于任何其他流类型。

|端口       |描述                                                                                                                                          |类型   |默认值      |
|-----------|-----------------------------------------------------------------------------------------------------------------------------------------------------|-------|-------------|
|`in`       |要变换的点                                                                                                                          |vector3|0.0, 0.0, 0.0|
|`fromspace`|渲染目标理解的向量空间的名称，用于将 `in` 点从该空间变换；可以为空以指定渲染器的工作空间。|string |__empty__    |
|`tospace`  |渲染目标理解的向量空间的名称，用于将 `in` 点变换到该空间。                                          |string |__empty__    |
|`out`      |输出:从 `fromspace` 变换到 `tospace` 的点                                                                                              |vector3|`in`         |

<a id="node-transformvector"> </a>

### `transformvector`
将传入的 vector3 坐标从一个指定空间变换到另一个空间；不能用于任何其他流类型。

|端口       |描述                                                                                                                                           |类型   |默认值      |
|-----------|------------------------------------------------------------------------------------------------------------------------------------------------------|-------|-------------|
|`in`       |要变换的向量                                                                                                                          |vector3|0.0, 0.0, 0.0|
|`fromspace`|渲染目标理解的向量空间的名称，用于将 `in` 向量从该空间变换；可以为空以指定渲染器的工作空间。|string |__empty__    |
|`tospace`  |渲染目标理解的向量空间的名称，用于将 `in` 向量变换到该空间。                                          |string |__empty__    |
|`out`      |输出:从 `fromspace` 变换到 `tospace` 的点                                                                                               |vector3|`in`         |

<a id="node-transformnormal"> </a>

### `transformnormal`
将传入的 vector3 法线从一个指定空间变换到另一个空间；不能用于任何其他流类型。

|端口       |描述                                                                                                                                           |类型   |默认值      |
|-----------|------------------------------------------------------------------------------------------------------------------------------------------------------|-------|-------------|
|`in`       |要变换的法线                                                                                                                          |vector3|0.0, 0.0, 1.0|
|`fromspace`|渲染目标理解的向量空间的名称，用于将 `in` 法线从该空间变换；可以为空以指定渲染器的工作空间。|string |__empty__    |
|`tospace`  |渲染目标理解的向量空间的名称，用于将 `in` 法线变换到该空间。                                          |string |__empty__    |
|`out`      |输出:从 `fromspace` 变换到 `tospace` 的点                                                                                               |vector3|`in`         |

<a id="node-transformmatrix"> </a>

### `transformmatrix`
用指定的矩阵变换传入的 vectorN。

|端口 |描述                      |类型    |默认值 |
|-----|---------------------------------|--------|--------|
|`in` |要变换的向量         |vector2 |__zero__|
|`mat`|用于变换 `in` 的矩阵      |matrix33|__one__ |
|`out`|输出:`in` 被 `mat` 变换后的结果|vector2 |`in`    |

|端口 |描述                      |类型    |默认值 |
|-----|---------------------------------|--------|--------|
|`in` |要变换的向量         |vector3 |__zero__|
|`mat`|用于变换 `in` 的矩阵      |matrixNN|__one__ |
|`out`|输出:`in` 被 `mat` 变换后的结果|vector3 |`in`    |

|端口 |描述                      |类型    |默认值 |
|-----|---------------------------------|--------|--------|
|`in` |要变换的向量         |vector4 |__zero__|
|`mat`|用于变换 `in` 的矩阵      |matrix44|__one__ |
|`out`|输出:`in` 被 `mat` 变换后的结果|vector4 |`in`    |

<a id="node-normalmap"> </a>

### `normalmap`
将法线向量从编码的切线空间变换到世界空间。输入法线向量假定所有通道都在 [0-1] 范围内，这通常是法线贴图输出的方式。

|端口       |描述                                                                     |类型          |默认值      |
|-----------|--------------------------------------------------------------------------------|--------------|-------------|
|`in`       |由 `normal`、`tangent`、`bitangent` 端口定义的帧空间中的输入法线|vector3       |0.5, 0.5, 1.0|
|`scale`    |应用于法线的缩放因子                                           |float, vector2|__one__      |
|`normal`   |从中变换 `in` 的局部帧的法线                          |vector3       |_Nworld_     |
|`tangent`  |从中变换 `in` 的局部帧的切线                         |vector3       |_Tworld_     |
|`bitangent`|从中变换 `in` 的局部帧的副切线                       |vector3       |_Bworld_     |
|`out`      |输出:世界空间中的输出法线                                            |vector3       |`normal`     |

<a id="node-hextilednormalmap"> </a>

### `hextilednormalmap`
从单个法线贴图中采样数据，并提供在 UV 空间中进行六边形平铺和随机化法线贴图的条款。

`file` 输入可以包括一个或多个替换以更改访问的文件名，如主规范文档中的 [文件名替换](./MaterialX.Specification.md#filename-substitutions) 部分所述。

`strength` 输入控制采样的法线贴图对最终法线的影响强度。值 0.0 保持表面法线不变，1.0 以全强度应用采样的法线，值 >1.0 放大的法线扰动。

|端口                |描述                                                                                         |类型    |默认值      |
|--------------------|----------------------------------------------------------------------------------------------------|--------|-------------|
|`file`              |图像文件的 URI                                                                           |filename|__empty__    |
|`default`           |如果无法解析文件引用，则使用的默认值                                    |vector3 |0.5, 0.5, 1.0|
|`texcoord`          |读取图像数据的 2D 纹理坐标                                           |vector2 |_UV0_        |
|`tiling`            |给定图像沿 U 和 V 轴的平铺率                                          |vector2 |1.0, 1.0     |
|`rotation`          |每瓦片旋转随机度，以度为单位                                                             |float   |0.0          |
|`rotationrange`     |用于随机化每个瓦片旋转的范围，以度为单位                                           |vector2 |0.0, 360.0   |
|`scale`             |应用于瓦片大小的每瓦片缩放随机乘数                                           |float   |1.0          |
|`scalerange`        |用于随机化瓦片缩放的缩放乘数范围                                             |vector2 |0.5, 2.0     |
|`offset`            |每瓦片平移随机度，以 UV 单位                                                         |float   |1.0          |
|`offsetrange`       |用于随机化瓦片位置的 UV 单位偏移值范围                                 |vector2 |0.0, 1.0     |
|`falloff`           |用于在边缘处混合相邻瓦片的衰减宽度；较大的值产生更平滑的混合 |float   |0.5          |
|`strength`          |控制采样的法线贴图对最终法线的影响强度。                              |float   |1.0          |
|`flip_g`            |如果为 true，取反采样法线贴图的绿色通道以适应切空间约定|boolean |false        |
|`normal`            |表面法线                                                                                      |vector3 |_Nworld_     |
|`tangent`           |表面切向量                                                                              |vector3 |_Tworld_     |
|`bitangent`         |表面副切向量                                                                            |vector3 |_Bworld_     |
|`out`               |输出:采样的法线贴图值                                                                |vector3 |0.5, 0.5, 1.0|

<a id="node-creatematrix"> </a>

### `creatematrix`
从三个 vector3 或四个 vector3 或 vector4 输入构建 3x3 或 4x4 矩阵。也可以从 vector3 输入值创建 matrix44，在这种情况下，创建 matrix44 时，`in1`-`in3` 的第四个值将设置为 0.0，`in4` 的第四个值将设置为 1.0。

|端口 |描述                   |类型    |默认值      |
|-----|------------------------------|--------|-------------|
|`in1`|`out` 的第一行        |vector3 |1.0, 0.0, 0.0|
|`in2`|`out` 的第二行       |vector3 |0.0, 1.0, 0.0|
|`in3`|`out` 的第三行        |vector3 |0.0, 0.0, 1.0|
|`out`|输出:构造的矩阵|matrix33|__one__      |

|端口 |描述                             |类型    |默认值      |
|-----|----------------------------------------|--------|-------------|
|`in1`|`out` 的第一行，后跟 0 |vector3 |1.0, 0.0, 0.0|
|`in2`|`out` 的第二行，后跟 0|vector3 |0.0, 1.0, 0.0|
|`in3`|`out` 的第三行，后跟 0 |vector3 |0.0, 0.0, 1.0|
|`in4`|`out` 的第四行，后跟 1|vector3 |0.0, 0.0, 0.0|
|`out`|输出:构造的矩阵          |matrix44|__one__      |

|端口 |描述                   |类型    |默认值           |
|-----|------------------------------|--------|------------------|
|`in1`|`out` 的第一行        |vector4 |1.0, 0.0, 0.0, 0.0|
|`in2`|`out` 的第二行       |vector4 |0.0, 1.0, 0.0, 0.0|
|`in3`|`out` 的第三行        |vector4 |0.0, 0.0, 1.0, 0.0|
|`in4`|`out` 的第四行       |vector4 |0.0, 0.0, 0.0, 1.0|
|`out`|输出:构造的矩阵|matrix44|__one__           |

<a id="node-transpose"> </a>

### `transpose`
转置传入的矩阵。

|端口 |描述              |类型        |默认值|
|-----|-------------------------|------------|-------|
|`in` |输入矩阵         |matrixNN    |__one__|
|`out`|输出:`in` 的转置|Same as `in`|`in`   |

<a id="node-determinant"> </a>

### `determinant`
输出传入矩阵的行列式。

|端口 |描述                |类型    |默认值|
|-----|---------------------------|--------|-------|
|`in` |输入矩阵           |matrixNN|__one__|
|`out`|输出:`in` 的行列式|float   |1.0    |

<a id="node-invertmatrix"> </a>

### `invertmatrix`
反转传入的矩阵。

|端口 |描述            |类型        |默认值|
|-----|-----------------------|------------|-------|
|`in` |输入矩阵       |matrixNN    |__one__|
|`out`|输出:`in` 的逆矩阵|Same as `in`|`in`   |

<a id="node-rotate2d"> </a>

### `rotate2d`
绕原点旋转传入的 2D 向量。

|端口    |描述                                                                        |类型   |默认值 |
|--------|-----------------------------------------------------------------------------------|-------|--------|
|`in`    |要旋转的输入向量                                                         |vector2|0.0, 0.0|
|`amount`|要旋转的角度，以度为单位指定。正值逆时针旋转|float  |0.0     |
|`out`   |输出:旋转后的向量                                                             |vector2|`in`    |

<a id="node-rotate3d"> </a>

### `rotate3d`
绕指定的单位轴向量旋转传入的 3D 向量。

|端口    |描述                                                                        |类型   |默认值      |
|--------|-----------------------------------------------------------------------------------|-------|-------------|
|`in`    |要旋转的输入向量                                                         |vector3|0.0, 0.0, 0.0|
|`amount`|要旋转的角度，以度为单位指定。正值逆时针旋转|float  |0.0          |
|`axis`  |`in` 绕其旋转的单位轴向量                                         |vector3|0.0, 1.0, 0.0|
|`out`   |输出:旋转后的向量                                                             |vector3|`in`         |

<a id="node-reflect"> </a>

### `reflect`
绕表面法线向量反射传入的 3D 向量。

|端口    |描述                                             |类型   |默认值      |
|--------|--------------------------------------------------------|-------|-------------|
|`in`    |要反射的输入向量                                 |vector3|1.0, 0.0, 0.0|
|`normal`|垂直于反射 `in` 的表面的法线向量|vector3|_Nworld_     |
|`out`   |输出:`in` 关于 `normal` 的反射               |vector3|             |

<a id="node-refract"> </a>

### `refract`
通过具有给定表面法线和相对折射率的表面折射传入的 3D 向量。

|端口    |描述                                                                    |类型   |默认值      |
|--------|-------------------------------------------------------------------------------|-------|-------------|
|`in`    |要折射的输入向量                                                        |vector3|1.0, 0.0, 0.0|
|`normal`|垂直于折射 `in` 的表面的法线向量                     |vector3|_Nworld_     |
|`ior`   |表面内部相对于外部的相对折射率|float  |1.0          |
|`out`   |输出:`in` 通过 `normal` 的折射                                    |vector3|             |

<a id="node-place2d"> </a>

### `place2d`
将传入的 2D 纹理坐标从一个参考帧变换到另一个参考帧。

`operationorder` 输入控制变换操作的执行顺序。`SRT` 选项执行 -pivot、scale、rotate、translate、+pivot。`TRS` 选项执行 -pivot、translate、rotate、scale、+pivot。

|端口            |描述                                            |类型   |默认值   |接受值 |
|----------------|-------------------------------------------------------|-------|----------|----------------|
|`texcoord`      |要变换的输入纹理坐标                 |vector2|0.0, 0.0  |                |
|`pivot`         |旋转和缩放 `texcoord` 的枢轴点|vector2|0.0,0.0   |                |
|`scale`         |应用于 `in` 的缩放因子                        |vector2|1.0,1.0   |                |
|`rotate`        |旋转 `in` 的角度，以度为单位                      |float  |0.0       |                |
|`offset`        |平移 `in` 的量                               |vector2|0.0,0.0   |                |
|`operationorder`|执行变换操作的顺序  |integer|0         |0 (SRT), 1 (TRS)|
|`out`           |输出:变换后的纹理坐标                |vector2|`texcoord`|                |

<a id="node-dot"> </a>

### `dot`
无操作，将其输入不变地传递到输出。

用户可以使用 dot 节点在节点图布局 UI 中塑造边缘连接路径或提供文档检查点。Dot 节点还可以将来自 `constant` 或其他具有 uniform="true" 输出的节点的统一值传递给统一输入和令牌。

|端口 |描述                       |类型                                                                |默认值 |
|-----|----------------------------------|--------------------------------------------------------------------|--------|
|`in` |输入数据流             |float, colorN, vectorN, matrixNN, boolean, integer, string, filename|__zero__|
|`out`|输出:不变的输入流|Same as `in`                                                        |__zero__|


## 逻辑操作符节点

逻辑操作符节点有一个或两个布尔类型的输入，用于通过节点图构建更高级别的逻辑流。

<a id="node-and"> </a>

### `and`
对两个输入布尔值进行逻辑 AND 运算。

|端口 |描述            |类型   |默认值|
|-----|-----------------------|-------|-------|
|`in1`|第一个输入流 |boolean|false  |
|`in2`|第二个输入流|boolean|false  |
|`out`|输出:`in1` AND `in2`|boolean|`in1`  |

<a id="node-or"> </a>

### `or`
对两个输入布尔值进行逻辑包含 OR 运算。

|端口 |描述            |类型   |默认值|
|-----|-----------------------|-------|-------|
|`in1`|第一个输入流 |boolean|false  |
|`in2`|第二个输入流|boolean|false  |
|`out`|输出:`in1` OR `in2` |boolean|`in1`  |

<a id="node-xor"> </a>

### `xor`
对两个输入布尔值进行逻辑异或 OR 运算。

|端口 |描述            |类型   |默认值|
|-----|-----------------------|-------|-------|
|`in1`|第一个输入流 |boolean|false  |
|`in2`|第二个输入流|boolean|false  |
|`out`|输出:`in1` XOR `in2`|boolean|`in1`  |

<a id="node-not"> </a>

### `not`
对输入布尔值进行逻辑 NOT 运算。

|端口 |描述     |类型   |默认值|
|-----|----------------|-------|-------|
|`in` |输入流|boolean|false  |
|`out`|输出:NOT `in`|boolean|true   |


## 调整节点

调整节点有一个名为 "in" 的输入，并对传入流中的值应用指定的函数。

<a id="node-contrast"> </a>

### `contrast`
使用 `amount` 作为线性斜率乘数增加或减少传入 `in` 值的对比度。

|端口    |描述                                                                                                                     |类型                  |默认值 |接受值     |
|--------|--------------------------------------------------------------------------------------------------------------------------------|----------------------|--------|--------------------|
|`in`    |要调整的输入颜色流                                                                                           |float, colorN, vectorN|__zero__|                    |
|`amount`|对比度调整的斜率乘数。大于 1.0 的值增加对比度，0.0 到 1.0 之间的值降低对比度。|Same as `in` or float |__one__ |[__zero__, __+inf__)|
|`pivot` |对比度调整的中心枢轴值；这是对比度调整时不会改变的值。                      |Same as `amount`      |__half__|                    |
|`out`   |输出:调整后的颜色值                                                                                                |Same as `in`          |`in`    |                    |

<a id="node-remap"> </a>

### `remap`
将传入的值从一个值范围 [`inlow`, `inhigh`] 线性重映射到另一个 [`outlow`, `outhigh`]。

|端口     |描述                    |类型                  |默认值 |
|---------|-------------------------------|----------------------|--------|
|`in`     |要调整的输入流|float, colorN, vectorN|__zero__|
|`inlow`  |输入范围的低值  |Same as `in` or float |__zero__|
|`inhigh` |输入范围的高值 |Same as `inlow`       |__one__ |
|`outlow` |输出范围的低值 |Same as `inlow`       |__zero__|
|`outhigh`|输出范围的高值|Same as `inlow`       |__one__ |
|`out`    |输出:调整后的值     |Same as `in`          |`in`    |

<a id="node-range"> </a>

### `range`
将传入的值从一个值范围重映射到另一个，可选地应用伽马校正 "在中间"。 

|端口     |描述                                             |类型                  |默认值 |
|---------|--------------------------------------------------------|----------------------|--------|
|`in`     |要调整的输入流                         |float, colorN, vectorN|__zero__|
|`inlow`  |输入范围的低值                           |Same as `in` or float |__zero__|
|`inhigh` |输入范围的高值                          |Same as `inlow`       |__one__ |
|`gamma`  |应用于重映射输入的指数的倒数|Same as `inlow`       |__one__ |
|`outlow` |输出范围的低值                          |Same as `inlow`       |__zero__|
|`outhigh`|输出范围的高值                         |Same as `inlow`       |__one__ |
|`doclamp`|如果为 true，输出将被钳位到 [`outlow`, `outhigh`] |boolean               |false   |
|`out`    |输出:调整后的值                              |Same as `in`          |`in`    |

<a id="node-smoothstep"> </a>

### `smoothstep`
输出输入值从 [`low`, `high`] 到 [0,1] 的平滑 Hermite 插值重映射。

|端口  |描述                               |类型                  |默认值 |
|------|------------------------------------------|----------------------|--------|
|`in`  |smoothstep 函数的输入值|float, colorN, vectorN|__zero__|
|`low` |输入范围的低值             |Same as `in` or float |__zero__|
|`high`|输出范围的低值            |Same as `low`         |__one__ |
|`out` |输出:调整后的值                |Same as `in`          |`in`    |

<a id="node-luminance"> </a>

### `luminance`
输出一个灰度值，在所有颜色通道中包含传入 RGB 颜色的亮度。

`lumacoeffs` 输入表示当前工作色彩空间的亮度系数。如果无法确定特定的色彩空间，将使用 ACEScg (ap1) 亮度系数 [0.2722287, 0.6740818, 0.0536895]。支持色彩管理系统的应用程序可以选择从 CMS 检索工作色彩空间的亮度系数并直接传递给 <luminance> 节点的实现，而不是向用户公开它。

|端口        |描述                                             |类型        |默认值                        |
|------------|--------------------------------------------------------|------------|-------------------------------|
|`in`        |要转换的输入颜色流                  |colorN      |__zero__                       |
|`lumacoeffs`|当前工作色彩空间的亮度系数|color3      |0.2722287, 0.6740818, 0.0536895|
|`out`       |输出:`in` 的亮度                           |Same as `in`|`in`                           |

<a id="node-rgbtohsv"> </a>

### `rgbtohsv`
将传入的颜色从 RGB 转换为 HSV 空间（H 和 S 范围为 0 到 1）；如果存在 alpha 通道，则保持不变。此转换不受当前色彩空间的影响。

|端口 |描述                           |类型        |默认值 |
|-----|--------------------------------------|------------|--------|
|`in` |要转换的输入颜色流|colorN      |__zero__|
|`out`|Output: `in` converted from RGB to HSV|Same as `in`|`in`    |

<a id="node-hsvtorgb"> </a>

### `hsvtorgb`
将传入的颜色从 HSV 转换为 RGB 空间；如果存在 alpha 通道，则保持不变。此转换不受当前色彩空间的影响。

|端口 |描述                           |类型        |默认值 |
|-----|--------------------------------------|------------|--------|
|`in` |要转换的输入颜色流|colorN      |__zero__|
|`out`|Output: `in` converted from HSV to RGB|Same as `in`|`in`    |

<a id="node-hsvadjust"> </a>

### `hsvadjust`
通过将输入颜色转换为 HSV，将 `amount.x` 添加到色相，将饱和度乘以 `amount.y`，将值乘以 `amount.z`，然后转换回 RGB 来调整 RGB 颜色的色相、饱和度和值。

正 `amount.x` 值沿 "红到绿到蓝" 方向旋转色相，`amount.x` 为 1.0 相当于 360 度（即无操作）旋转。允许负值或大于 1.0 的色相调整值，在 0-1 边界处环绕。RGB 和 HSV 空间之间的内部转换不受当前色彩空间的影响，对于 color4 输入，alpha 值保持不变。

|端口    |描述                                                                 |类型        |默认值      |
|--------|----------------------------------------------------------------------------|------------|-------------|
|`in`    |要调整的输入颜色流                                       |colorN      |__zero__     |
|`amount`|色相偏移、饱和度缩放和亮度缩放，分别位于 (x, y, z)|vector3     |0.0, 1.0, 1.0|
|`out`   |输出:调整后的值                                                  |Same as `in`|`in`         |

<a id="node-saturate"> </a>

### `saturate`
调整颜色的饱和度，如果存在 alpha 通道，则保持不变。

`lumacoeffs` 输入表示当前工作色彩空间的亮度系数。如果无法确定特定的色彩空间，将使用 ACEScg (ap1) 亮度系数 [0.2722287, 0.6740818, 0.0536895]。支持色彩管理系统的应用程序可以选择从 CMS 检索工作色彩空间的亮度系数并直接传递给节点的实现，而不是向用户公开它。

|端口        |描述                                                    |类型        |默认值                        |
|------------|---------------------------------------------------------------|------------|-------------------------------|
|`in`        |要调整的输入颜色流                          |colorN      |__zero__                       |
|`amount`    |饱和度 `in` 的乘数                              |float       |1.0                            |
|`lumacoeffs`|用于计算去饱和值的亮度系数|color3      |0.2722287, 0.6740818, 0.0536895|
|`out`       |输出:调整后的值                                     |Same as `in`|`in`                           |

<a id="node-colorcorrect"> </a>

### `colorcorrect`
将各种调整节点组合成一个艺术家友好的颜色校正节点。对于 color4 输入，alpha 值不变。

|端口           |描述                                                              |类型        |默认值|
|---------------|-------------------------------------------------------------------------|------------|-------|
|`in`           |输入颜色流                                                   |colorN      |__one__|
|`hue`          |旋转颜色色相                                                    |float       |0      |
|`saturation`   |乘以输入颜色饱和度级别                              |float       |1      |
|`gamma`        |对颜色应用伽马校正                                  |float       |1      |
|`lift`         |提升暗色值，保持白色值不变         |float       |0      |
|`gain`         |乘数增加浅色值，保持黑色值不变|float       |1      |
|`contrast`     |线性增加或减少颜色对比度                         |float       |1      |
|`contrastpivot`|应用对比度的枢轴值                                |float       |0.5    |
|`exposure`     |对数亮度乘数，为 2^`exposure`                        |float       |0      |
|`out`          |输出:颜色校正值                                        |Same as `in`|       |


## 合成节点

合成节点有两个（必需的）输入，名为 `fg` 和 `bg`，并应用函数将它们组合。合成节点分为五个子类:[预乘节点](#premult-nodes)、[混合节点](#blend-nodes)、[合并节点](#merge-nodes)、[遮罩节点](#masking-nodes)和[混合节点](#mix-node)。


### 预乘节点

预乘节点操作 4 通道(color4)输入/输出，有一个名为 `in` 的输入，并应用或取消应用 alpha 到 float 或 RGB 颜色。

<a id="node-premult"> </a>

### `premult`
将输入的 RGB 通道乘以输入的 Alpha 通道。

|端口 |描述                         |类型  |默认值           |
|-----|------------------------------------|------|------------------|
|`in` |要预乘的输入流|color4|0.0, 0.0, 0.0, 1.0|
|`out`|输出:预乘后的 `in`          |color4|`in`              |

<a id="node-unpremult"> </a>

### `unpremult`
将输入的 RGB 通道除以输入的 Alpha 通道。如果 Alpha 值为零，则 `in` 值不变地传递。

|端口 |描述                           |类型  |默认值           |
|-----|--------------------------------------|------|------------------|
|`in` |要取消预乘的输入流|color4|0.0, 0.0, 0.0, 1.0|
|`out`|输出:取消预乘后的 `in`          |color4|`in`              |


### 混合节点

混合节点接受两个 1-4 通道输入并对所有通道应用相同的运算符（alpha 的数学运算与 R 或 RGB 相同）；下面，"F" 和 "B" 分别指 `fg` 和 `bg` 输入的任何单个通道。


<a id="node-plus"> </a>

### `plus`
添加两个 1-4 通道输入，可选在 bg 输入和结果之间混合。

|端口 |描述                                                                       |类型         |默认值 |接受值|
|-----|----------------------------------------------------------------------------------|-------------|--------|---------------|
|`fg` |前景输入流                                                       |float, colorN|__zero__|               |
|`bg` |背景输入流                                                       |Same as `fg` |__zero__|               |
|`mix`|`bg` (mix=0) 和 'plus' 操作结果(mix=1)之间的混合值|float        |1.0     |[0, 1]         |
|`out`|输出:`fg` 加 `bg`                                                            |Same as `fg` |`bg`    |               |

<a id="node-minus"> </a>

### `minus`
减去两个 1-4 通道输入，可选在 bg 输入和结果之间混合。

|端口 |描述                                                                        |类型         |默认值 |接受值|
|-----|-----------------------------------------------------------------------------------|-------------|--------|---------------|
|`fg` |前景输入流                                                        |float, colorN|__zero__|               |
|`bg` |背景输入流                                                        |Same as `fg` |__zero__|               |
|`mix`|`bg` (mix=0) 和 'minus' 操作结果(mix=1)之间的混合值|float        |1.0     |[0, 1]         |
|`out`|输出                                                                             |Same as `fg` |`bg`    |               |

<a id="node-difference"> </a>

### `difference`
两个 1-4 通道输入的绝对值差，可选在 bg 输入和结果之间混合。

|端口 |描述                                                                             |类型         |默认值 |接受值|
|-----|----------------------------------------------------------------------------------------|-------------|--------|---------------|
|`fg` |前景输入流                                                             |float, colorN|__zero__|               |
|`bg` |背景输入流                                                             |Same as `fg` |__zero__|               |
|`mix`|`bg` (mix=0) 和 'difference' 操作结果(mix=1)之间的混合值|float        |1.0     |[0, 1]         |
|`out`|输出                                                                                  |Same as `fg` |`bg`    |               |

<a id="node-burn"> </a>

### `burn`
接受两个 1-4 通道输入并对所有通道应用相同的运算符:
```
1-(1-B)/F
```

|端口 |描述                                                                       |类型         |默认值 |接受值|
|-----|----------------------------------------------------------------------------------|-------------|--------|---------------|
|`fg` |前景输入流                                                       |float, colorN|__zero__|               |
|`bg` |背景输入流                                                       |Same as `fg` |__zero__|               |
|`mix`|`bg` (mix=0) 和 'burn' 操作结果(mix=1)之间的混合值|float        |1.0     |[0, 1]         |
|`out`|输出                                                                            |Same as `fg` |`bg`    |               |

<a id="node-dodge"> </a>

### `dodge`
接受两个 1-4 通道输入并对所有通道应用相同的运算符:
```
B/(1-F)
```

|端口 |描述                                                                        |类型         |默认值 |接受值|
|-----|-----------------------------------------------------------------------------------|-------------|--------|---------------|
|`fg` |前景输入流                                                        |float, colorN|__zero__|               |
|`bg` |背景输入流                                                        |Same as `fg` |__zero__|               |
|`mix`|`bg` (mix=0) 和 'dodge' 操作结果(mix=1)之间的混合值|float        |1.0     |[0, 1]         |
|`out`|输出                                                                             |Same as `fg` |`bg`    |               |

<a id="node-screen"> </a>

### `screen`
接受两个 1-4 通道输入并对所有通道应用相同的运算符:
```
1-(1-F)*(1-B)
```

|端口 |描述                                                                         |类型         |默认值 |接受值|
|-----|------------------------------------------------------------------------------------|-------------|--------|---------------|
|`fg` |前景输入流                                                         |float, colorN|__zero__|               |
|`bg` |背景输入流                                                         |Same as `fg` |__zero__|               |
|`mix`|`bg` (mix=0) 和 'screen' 操作结果(mix=1)之间的混合值|float        |1.0     |[0, 1]         |
|`out`|输出                                                                              |Same as `fg` |`bg`    |               |

<a id="node-overlay"> </a>

### `overlay`
接受两个 1-4 通道输入并对所有通道应用相同的运算符:
```
2FB if B<0.5;
1-2(1-F)(1-B) if B>=0.5
```

|端口 |描述                                                                          |类型         |默认值 |接受值|
|-----|-------------------------------------------------------------------------------------|-------------|--------|---------------|
|`fg` |前景输入流                                                          |float, colorN|__zero__|               |
|`bg` |背景输入流                                                          |Same as `fg` |__zero__|               |
|`mix`|`bg` (mix=0) 和 'overlay' 操作结果(mix=1)之间的混合值|float        |1.0     |[0, 1]         |
|`out`|输出                                                                               |Same as `fg` |`bg`    |               |


### 合并节点

合并节点接受两个 4 通道(color4)输入并使用内置的 alpha 通道来控制 `fg` 和 `bg` 输入的合成；"F" 和 "B" 分别指 `fg` 和 `bg` 输入的各个非 alpha 通道，而 "f" 和 "b" 指 `fg` 和 `bg` 输入的 alpha 通道。合并节点未为 1 通道或 3 通道输入定义，不能用于 vector<em>N</em> 流。


<a id="node-disjointover"> </a>

### `disjointover`
接受两个 color4 输入并使用内置的 alpha 通道来控制 fg 和 bg 输入的合成:
```
F+B         if f+b<=1
F+B(1-f)/b  if f+b>1
alpha: min(f+b,1)
```

|端口 |描述                                                                               |类型  |默认值           |接受值|
|-----|------------------------------------------------------------------------------------------|------|------------------|---------------|
|`fg` |前景输入流                                                               |color4|0.0, 0.0, 0.0, 0.0|               |
|`bg` |背景输入流                                                               |color4|0.0, 0.0, 0.0, 0.0|               |
|`mix`|`bg` (mix=0) 和 'disjointover' 操作结果(mix=1)之间的混合值|float |1.0               |[0, 1]         |
|`out`|输出                                                                                    |color4|`bg`              |               |

<a id="node-in"> </a>

### `in`
接受两个 color4 输入并使用内置的 alpha 通道来控制 fg 和 bg 输入的合成:
```
RGB = Fb
Alpha = fb
```

|端口 |描述                                                                     |类型  |默认值           |接受值|
|-----|--------------------------------------------------------------------------------|------|------------------|---------------|
|`fg` |前景输入流                                                     |color4|0.0, 0.0, 0.0, 0.0|               |
|`bg` |背景输入流                                                     |color4|0.0, 0.0, 0.0, 0.0|               |
|`mix`|`bg` (mix=0) 和 'in' 操作结果(mix=1)之间的混合值|float |1.0               |[0, 1]         |
|`out`|输出                                                                          |color4|`bg`              |               |

<a id="node-mask"> </a>

### `mask`
接受两个 color4 输入并使用内置的 alpha 通道来控制 fg 和 bg 输入的合成:
```
RGB = Bf
Alpha = bf
```

|端口 |描述                                                                       |类型  |默认值           |接受值|
|-----|----------------------------------------------------------------------------------|------|------------------|---------------|
|`fg` |前景输入流                                                       |color4|0.0, 0.0, 0.0, 0.0|               |
|`bg` |背景输入流                                                       |color4|0.0, 0.0, 0.0, 0.0|               |
|`mix`|`bg` (mix=0) 和 'mask' 操作结果(mix=1)之间的混合值|float |1.0               |[0, 1]         |
|`out`|输出                                                                            |color4|`bg`              |               |

<a id="node-matte"> </a>

### `matte`
接受两个 color4 输入并使用内置的 alpha 通道来控制 fg 和 bg 输入的合成:
```
RGB = Ff+B(1-f)
Alpha = f+b(1-f)
```

|端口 |描述                                                                        |类型  |默认值           |接受值|
|-----|-----------------------------------------------------------------------------------|------|------------------|---------------|
|`fg` |前景输入流                                                        |color4|0.0, 0.0, 0.0, 0.0|               |
|`bg` |背景输入流                                                        |color4|0.0, 0.0, 0.0, 0.0|               |
|`mix`|`bg` (mix=0) 和 'matte' 操作结果(mix=1)之间的混合值|float |1.0               |[0, 1]         |
|`out`|输出                                                                             |color4|`bg`              |               |

<a id="node-out"> </a>

### `out`
接受两个 color4 输入并使用内置的 alpha 通道来控制 fg 和 bg 输入的合成:
```
RGB = F(1-b)
Alpha = f(1-b)
```

|端口 |描述                                                                      |类型  |默认值           |接受值|
|-----|---------------------------------------------------------------------------------|------|------------------|---------------|
|`fg` |前景输入流                                                      |color4|0.0, 0.0, 0.0, 0.0|               |
|`bg` |背景输入流                                                      |color4|0.0, 0.0, 0.0, 0.0|               |
|`mix`|`bg` (mix=0) 和 'out' 操作结果(mix=1)之间的混合值|float |1.0               |[0, 1]         |
|`out`|输出                                                                           |color4|`bg`              |               |

<a id="node-over"> </a>

### `over`
接受两个 color4 输入并使用内置的 alpha 通道来控制 fg 和 bg 输入的合成:
```
RGB = F+B(1-f)
Alpha = f+b(1-f)
```

|端口 |描述                                                                       |类型  |默认值           |接受值|
|-----|----------------------------------------------------------------------------------|------|------------------|---------------|
|`fg` |前景输入流                                                       |color4|0.0, 0.0, 0.0, 0.0|               |
|`bg` |背景输入流                                                       |color4|0.0, 0.0, 0.0, 0.0|               |
|`mix`|`bg` (mix=0) 和 'over' 操作结果(mix=1)之间的混合值|float |1.0               |[0, 1]         |
|`out`|输出                                                                            |color4|`bg`              |               |


### 遮罩节点

遮罩节点接受一个 1-4 通道输入 `in` 加上一个单独的 float `mask` 输入，并对所有 "in" 通道应用相同的运算符；"F" 指 `in` 输入的任何单个通道，而 "m" 指 mask 输入。


<a id="node-inside"> </a>

### `inside`
"inside" 遮罩操作，返回 Fm

|端口  |描述                      |类型         |默认值 |接受值|
|------|---------------------------------|-------------|--------|---------------|
|`in`  |要遮罩的输入流    |float, colorN|__zero__|               |
|`mask`|遮罩输入信号         |float        |1.0     |[0, 1]         |
|`out` |输出:`in` 乘以 `mask`|Same as `in` |`in`    |               |

<a id="node-outside"> </a>

### `outside`
"outside" 遮罩操作，返回 F(1-m)

|端口  |描述                        |类型         |默认值 |接受值|
|------|-----------------------------------|-------------|--------|---------------|
|`in`  |要遮罩的输入流      |float, colorN|__zero__|               |
|`mask`|遮罩输入信号           |float        |1.0     |[0, 1]         |
|`out` |输出:`in` 乘以 1-`mask`|Same as `in` |`in`    |               |


### Mix 节点

Mix 节点接受两个 1-4 通道输入 `fg` 和 `bg` 加上一个单独的 1 通道(float)或 N 通道（与 `fg` 和 `bg` 相同的类型和通道数）`mix` 输入，并根据 mix 值混合 `fg` 和 `bg`，对于 "float" `mix` 类型统一混合，对于非 float `mix` 类型按通道混合；"F" 指 `in` 输入的任何单个通道，而 "m" 指 mix 输入的相应通道。

<a id="node-mix"> </a>

### `mix`
"mix" 操作，根据混合量从 "bg" 混合到 "fg"，返回 Fm+B(1-m)

|端口 |描述                            |类型                  |默认值 |接受值|
|-----|---------------------------------------|----------------------|--------|---------------|
|`fg` |前景输入流            |float, colorN, vectorN|__zero__|               |
|`bg` |背景输入流            |Same as `fg`          |__zero__|               |
|`mix`|将 `bg` 混合到 `fg` 的量         |float                 |0.0     |[0, 1]         |
|`out`|输出:mix 操作的结果|Same as `fg`          |`bg`    |               |

|端口 |描述                    |类型           |默认值 |
|-----|-------------------------------|---------------|--------|
|`fg` |前景输入流    |colorN, vectorN|__zero__|
|`bg` |背景输入流    |Same as `fg`   |__zero__|
|`mix`|将 `bg` 混合到 `fg` 的量 |Same as `fg`   |__zero__|
|`out`|输出                         |Same as `fg`   |`bg`    |

另请参阅下面的 [标准着色器节点](#standard-shader-nodes) 部分，了解 [`mix` 节点](#node-mix-shader) 的其他着色器语义变体。



## 条件节点

条件节点用于比较两个流的值，或从多个流中选择一个值。


<a id="node-ifgreater"> </a>

### `ifgreater`

根据 `value1` 输入是否大于 `value2` 输入，输出 `in1` 或 `in2` 流的值。

|端口    |描述                                       |类型                                     |默认值 |
|--------|--------------------------------------------------|-----------------------------------------|--------|
|`value1`|要比较的第一个值                    |float, integer                           |__one__ |
|`value2`|要比较的第二个值                   |Same as `value1`                         |__zero__|
|`in1`   |如果 `value1` > `value2` 则输出的值流 |float, colorN, vectorN, matrixNN, integer|__zero__|
|`in2`   |如果 `value1` <= `value2` 则输出的值流|Same as `in1`                            |__zero__|
|`out`   |输出:比较的结果              |Same as `in1`                            |`in1`   |

|端口    |描述                        |类型            |默认值 |
|--------|-----------------------------------|----------------|--------|
|`value1`|要比较的第一个值     |float, integer  |__one__ |
|`value2`|要比较的第二个值    |Same as `value1`|__zero__|
|`out`   |输出:如果 `value1` > `value2` 则为 true|boolean         |false   |

<a id="node-ifgreatereq"> </a>

### `ifgreatereq`

根据 `value1` 输入是否大于或等于 `value2` 输入，输出 `in1` 或 `in2` 流的值。

|端口    |描述                                       |类型                                     |默认值 |
|--------|--------------------------------------------------|-----------------------------------------|--------|
|`value1`|要比较的第一个值                    |float, integer                           |__one__ |
|`value2`|要比较的第二个值                   |Same as `value1`                         |__zero__|
|`in1`   |如果 `value1` >= `value2` 则输出的值流|float, colorN, vectorN, matrixNN, integer|__zero__|
|`in2`   |如果 `value1` < `value2` 则输出的值流 |Same as `in1`                            |__zero__|
|`out`   |输出:比较的结果              |Same as `in1`                            |`in1`   |

|端口    |描述                         |类型            |默认值 |
|--------|------------------------------------|----------------|--------|
|`value1`|要比较的第一个值      |float, integer  |__one__ |
|`value2`|要比较的第二个值     |Same as `value1`|__zero__|
|`out`   |输出:如果 `value1` >= `value2` 则为 true|boolean         |false   |

<a id="node-ifequal"> </a>

### `ifequal`

根据 `value1` 输入是否等于 `value2` 输入，输出 `in1` 或 `in2` 流的值。

|端口    |描述                                       |类型                                     |默认值 |
|--------|--------------------------------------------------|-----------------------------------------|--------|
|`value1`|The first value to be compared                    |float, integer                           |__one__ |
|`value2`|The second value to be compared                   |Same as `value1`                         |__zero__|
|`in1`   |The value stream to output if `value1` = `value2` |float, colorN, vectorN, matrixNN, integer|__zero__|
|`in2`   |The value stream to output if `value1` != `value2`|Same as `in1`                            |__zero__|
|`out`   |输出:比较的结果              |Same as `in1`                            |`in1`   |

|端口    |描述                                       |类型                                     |默认值 |
|--------|--------------------------------------------------|-----------------------------------------|--------|
|`value1`|The first value to be compared                    |boolean                                  |false   |
|`value2`|The second value to be compared                   |boolean                                  |false   |
|`in1`   |The value stream to output if `value1` = `value2` |float, colorN, vectorN, matrixNN, integer|__zero__|
|`in2`   |The value stream to output if `value1` != `value2`|Same as `in1`                            |__zero__|
|`out`   |输出:比较的结果              |Same as `in1`                            |`in1`   |

|端口    |描述                        |类型            |默认值 |
|--------|-----------------------------------|----------------|--------|
|`value1`|The first value to be compared     |float, integer  |__one__ |
|`value2`|The second value to be compared    |Same as `value1`|__zero__|
|`out`   |输出:如果 `value1` = `value2` 则为 true|boolean         |false   |

|端口    |描述                        |类型   |默认值|
|--------|-----------------------------------|-------|-------|
|`value1`|The first value to be compared     |boolean|false  |
|`value2`|The second value to be compared    |boolean|false  |
|`out`   |输出:如果 `value1` = `value2` 则为 true|boolean|false  |

<a id="node-switch"> </a>

### `switch`

根据选择器输入 `which` 的值，输出多达十个输入流中的一个的值。注意，并非所有输入都需要连接。输出的类型与 `in1` 相同，默认值为 __zero__。

|端口   |描述                                                                                                                |类型                            |默认值 |
|-------|---------------------------------------------------------------------------------------------------------------------------|--------------------------------|--------|
|`in1`  |使用 `which` 选择的输入流                                                                                  |float, colorN, vectorN, matrixNN|__zero__|
|`in2`  |使用 `which` 选择的输入流                                                                                  |Same as `in1`                   |__zero__|
|`in3`  |使用 `which` 选择的输入流                                                                                  |Same as `in1`                   |__zero__|
|`in4`  |使用 `which` 选择的输入流                                                                                  |Same as `in1`                   |__zero__|
|`in5`  |使用 `which` 选择的输入流                                                                                  |Same as `in1`                   |__zero__|
|`in6`  |使用 `which` 选择的输入流                                                                                  |Same as `in1`                   |__zero__|
|`in7`  |使用 `which` 选择的输入流                                                                                  |Same as `in1`                   |__zero__|
|`in8`  |使用 `which` 选择的输入流                                                                                  |Same as `in1`                   |__zero__|
|`in9`  |使用 `which` 选择的输入流                                                                                  |Same as `in1`                   |__zero__|
|`in10` |使用 `which` 选择的输入流                                                                                  |Same as `in1`                   |__zero__|
|`which`|选择器，用于选择从哪个输入获取值；输出来自输入 floor(`which`)+1，钳制到 1-10 范围|float, integer                  |__zero__|
|`out`  |输出:选定的输入                                                                                                 |Same as `in1`                   |`in1`   |



## 通道节点

通道节点用于对流执行通道操作和数据类型转换。


<a id="node-extract"> </a>

### `extract`

从 __colorN__、__vectorN__ 或 __matrixNN__ 流中隔离单个通道。

当输入是 __colorN__ 或 __vectorN__ 时，节点按索引提取单个 float 分量。当输入是 __matrix33__ 或 __matrix44__ 时，节点按索引提取行向量。

|端口   |描述                                 |类型           |默认值 |
|-------|--------------------------------------------|---------------|--------|
|`in`   |从中提取 `out` 的输入流|colorN, vectorN|__zero__|
|`index`|要提取的 `in` 中通道的索引 |integer        |0       |
|`out`  |输出:`in` 的第 `index` 个通道       |float          |0.0     |

|端口   |描述                                  |类型              |默认值 |
|-------|---------------------------------------------|------------------|--------|
|`in`   |从中提取行的输入矩阵 |matrix33, matrix44|__zero__|
|`index`|要提取的 `in` 中行的索引      |integer           |0       |
|`out`  |输出:作为向量的 `in` 的第 `index` 行|vector3, vector4  |__zero__|

`index` 的有效范围应在用户界面中钳制到 $[0，N）$，其中 __N__ 是分量的数量（对于 vector/color 输入）或行数（对于矩阵输入）。`index` 是一个统一的非变化值。任何超出有效范围的 `index` 值都应导致错误。

<a id="node-convert"> </a>

### `convert`
将流从一种数据类型转换为另一种。

|端口 |描述                |类型           |默认值 |
|-----|---------------------------|---------------|--------|
|`in` |要转换的输入流|boolean        |false   |
|`out`|输出:转换后的值|float, integer |__zero__|

|端口 |描述                              |类型   |默认值|
|-----|-----------------------------------------|-------|-------|
|`in` |要转换的输入流              |integer|0      |
|`out`|输出:对于任何非零输入值为 true|boolean|false  |

|端口 |描述                |类型   |默认值|
|-----|---------------------------|-------|-------|
|`in` |要转换的输入流|integer|0      |
|`out`|输出:转换后的值|float  |0.0    |

|端口 |描述                             |类型                   |默认值 |
|-----|----------------------------------------|-----------------------|--------|
|`in` |要转换的输入流             |boolean, float, integer|__zero__|
|`out`|Output: copy input value to all channels|colorN, vectorN        |__zero__|

|端口 |描述                |类型           |默认值 |
|-----|---------------------------|---------------|--------|
|`in` |要转换的输入流|colorN, vectorN|__zero__|
|`out`|Output: the converted value|colorM, vectorM|__zero__|

|端口 |描述                                                      |类型                                    |默认值 |
|-----|-----------------------------------------------------------------|----------------------------------------|--------|
|`in` |要转换的输入流                                      |boolean, integer, float, colorN, vectorN|__zero__|
|`out`|Output: an unlit surface shader emitting the input value as color|surfaceshader                           |        |

For boolean input values, all numeric output values will be either __zero__ or __one__.

For colorN/vectorN to colorM/vectorM:

  * if _N_ is the same as _M_, then channels are directly copied.
  * if _N_ is larger than _M_, then the first _M_ channels are used and the excess channels ignored.
  * if _N_ is smaller than _M_, then the _N_ channels are directly copied and additional channels are populated with 0, aside from the fourth channel which is populated with 1.


<a id="node-combine2"> </a>

### `combine2`

将两个流中的通道组合到单个兼容类型输出流的相同数量的通道中。

|端口 |描述                                                      |类型   |默认值 |
|-----|-----------------------------------------------------------------|-------|--------|
|`in1`|将发送到 `out` 第一个通道的输入流 |float  |0.0     |
|`in2`|将发送到 `out` 第二个通道的输入流|float  |0.0     |
|`out`|输出:组合后的值                                       |vector2|0.0, 0.0|

|端口 |描述                                                      |类型  |默认值           |
|-----|-----------------------------------------------------------------|------|------------------|
|`in1`|The input stream that will be sent to the first channel of `out` |color3|0.0, 0.0, 0.0     |
|`in2`|The input stream that will be sent to the second channel of `out`|float |0.0               |
|`out`|Output: the combined value                                       |color4|0.0, 0.0, 0.0, 0.0|

|端口 |描述                                                      |类型   |默认值           |
|-----|-----------------------------------------------------------------|-------|------------------|
|`in1`|The input stream that will be sent to the first channel of `out` |vector3|0.0, 0.0, 0.0     |
|`in2`|The input stream that will be sent to the second channel of `out`|float  |0.0               |
|`out`|Output: the combined value                                       |vector4|0.0, 0.0, 0.0, 0.0|

|端口 |描述                                                      |类型   |默认值           |
|-----|-----------------------------------------------------------------|-------|------------------|
|`in1`|The input stream that will be sent to the first channel of `out` |vector2|0.0, 0.0          |
|`in2`|The input stream that will be sent to the second channel of `out`|vector2|0.0, 0.0          |
|`out`|Output: the combined value                                       |vector4|0.0, 0.0, 0.0, 0.0|

<a id="node-combine3"> </a>

### `combine3`

将三个流中的通道组合到单个兼容类型输出流的相同数量的通道中。

|端口 |描述                                                      |类型           |默认值 |
|-----|-----------------------------------------------------------------|---------------|--------|
|`in1`|The input stream that will be sent to the first channel of `out` |float          |0.0     |
|`in2`|The input stream that will be sent to the second channel of `out`|float          |0.0     |
|`in3`|The input stream that will be sent to the third channel of `out` |float          |0.0     |
|`out`|Output: the combined value                                       |color3, vector3|__zero__|

<a id="node-combine4"> </a>

### `combine4`

将四个流中的通道组合到单个兼容类型输出流的相同数量的通道中。

|端口 |描述                                                      |类型           |默认值 |
|-----|-----------------------------------------------------------------|---------------|--------|
|`in1`|The input stream that will be sent to the first channel of `out` |float          |0.0     |
|`in2`|The input stream that will be sent to the second channel of `out`|float          |0.0     |
|`in3`|The input stream that will be sent to the third channel of `out` |float          |0.0     |
|`in4`|The input stream that will be sent to the fourth channel of `out`|float          |0.0     |
|`out`|Output: the combined value                                       |color4, vector4|__zero__|

<a id="node-separate2"> </a>

### `separate2`

将 2 通道流的通道拆分为单独的 float 输出。

|端口  |描述                     |类型   |默认值 |
|------|--------------------------------|-------|--------|
|`in`  |要分离的输入流|vector2|0.0, 0.0|
|`outx`|输出:`in` 的 x 通道   |float  |0.0     |
|`outy`|输出:`in` 的 y 通道   |float  |0.0     |

对于 vector2 输入 `in`,`outx` 和 `outy` 分别对应 `in` 的 x 和 y 分量。

<a id="node-separate3"> </a>

### `separate3`

将 3 通道流的通道拆分为单独的 float 输出。

|端口  |描述                     |类型  |默认值      |
|------|--------------------------------|------|-------------|
|`in`  |要分离的输入流|color3|0.0, 0.0, 0.0|
|`outr`|Output: the r channel of `in`   |float |0.0          |
|`outg`|Output: the g channel of `in`   |float |0.0          |
|`outb`|Output: the b channel of `in`   |float |0.0          |

|端口  |描述                     |类型   |默认值      |
|------|--------------------------------|-------|-------------|
|`in`  |要分离的输入流|vector3|0.0, 0.0, 0.0|
|`outx`|Output: the x channel of `in`   |float  |0.0          |
|`outy`|Output: the y channel of `in`   |float  |0.0          |
|`outz`|Output: the z channel of `in`   |float  |0.0          |

当输入 `in` 是 color3 时，`outr`、`outg` 和 `outb` 分别对应 `in` 的 r、g 和 b 分量。

当输入 `in` 是 vector3 时，`outx`、`outy` 和 `outz` 分别对应 `in` 的 x、y 和 z 分量。

<a id="node-separate4"> </a>

### `separate4`

将 4 通道流的通道拆分为单独的 float 输出。

|端口  |描述                     |类型  |默认值           |
|------|--------------------------------|------|------------------|
|`in`  |要分离的输入流|color4|0.0, 0.0, 0.0, 0.0|
|`outr`|Output: the r channel of `in`   |float |0.0               |
|`outg`|Output: the g channel of `in`   |float |0.0               |
|`outb`|Output: the b channel of `in`   |float |0.0               |
|`outa`|Output: the a channel of `in`   |float |0.0               |

|端口  |描述                     |类型   |默认值           |
|------|--------------------------------|-------|------------------|
|`in`  |要分离的输入流|vector4|0.0, 0.0, 0.0, 0.0|
|`outx`|Output: the x channel of `in`   |float  |0.0               |
|`outy`|Output: the y channel of `in`   |float  |0.0               |
|`outz`|Output: the z channel of `in`   |float  |0.0               |
|`outw`|Output: the w channel of `in`   |float  |0.0               |

When the input `in` is a color4, `outr`, `outg`, `outb`, and `outa` correspond to the r-, g-, b-, and alpha components of `in`, respectively.

When the input `in` is a vector4, `outx`, `outy`, `outz`, and `outw` correspond to the x-, y-, z-, and w-components of `in`, respectively.


## 卷积节点

卷积节点有一个名为 "in" 的输入，并在输入流上应用定义的卷积函数。其中一些节点可能在光线追踪应用程序中无法实现；它们是为了纯 2D 图像处理应用程序的利益而提供的。


<a id="node-blur"> </a>

### `blur`

对输入流应用卷积模糊。

|端口        |描述                                |类型                  |默认值 |接受值|
|------------|-------------------------------------------|----------------------|--------|---------------|
|`in`        |要模糊的输入流             |float, colorN, vectorN|__zero__|               |
|`size`      |0-1 UV 空间中模糊核的大小|float                 |0.0     |               |
|`filtertype`|模糊中使用的空间滤波器        |string                |box     |box, gaussian  |
|`out`       |输出:模糊后的 `in`                   |Same as `in`          |`in`    |               |

<a id="node-heighttonormal"> </a>

### `heighttonormal`

将标量高度图转换为类型为 `vector3` 的切线空间法线图。输出法线图的所有通道都编码在 [0-1] 范围内，使其能够存储在无符号图像格式中。

|端口      |描述                                                                      |类型   |默认值      |
|----------|---------------------------------------------------------------------------------|-------|-------------|
|`in`      |输入标量高度图                                                      |float  |0.0          |
|`scale`   |应用于 `in` 信号的乘数                                            |float  |1.0          |
|`texcoord`|计算高度场梯度所依据的纹理坐标|vector2|_UV0_        |
|`out`     |输出:从 `in` 计算的切线空间法线                                  |vector3|0.5, 0.5, 1.0|


<br>


# 标准着色器节点

标准 MaterialX 库定义了以下操作 "shader" 语义类型的节点和节点变体。标准库着色器不响应外部照明；请参考 [**MaterialX 基于物理的着色节点**](./MaterialX.PBRSpec.md#materialx-pbs-library) 文档以获取响应照明的其他节点和着色器构造函数的定义，以及 [**MaterialX NPR 着色节点**](./MaterialX.NPRSpec.md) 以获取适用于非真实感渲染的着色器和节点的定义。

<a id="node-surface-unlit"> </a>

### `surface_unlit`
无光表面着色器节点，表示可以发射和透射光线但不接收来自光源或其他表面的照明的表面。

|端口                |描述                          |类型         |默认值      |
|--------------------|-------------------------------------|-------------|-------------|
|`emission`          |表面发射量          |float        |1.0          |
|`emission_color`    |表面发射颜色               |color3       |1.0, 1.0, 1.0|
|`transmission`      |表面透射量      |float        |0.0          |
|`transmission_color`|表面透射颜色           |color3       |1.0, 1.0, 1.0|
|`opacity`           |表面镂空不透明度               |float        |1.0          |
|`out`               |输出:无光表面着色器闭包 |surfaceshader|             |

<a id="node-displacement"> </a>

### `displacement`
构建描述表面几何修改的位移着色器。

标量签名沿表面法线方向位移，而矢量签名允许使用 (dPdu, dPdv, N) 坐标在切线/法线空间中位移。

|端口          |描述                             |类型              |默认值 |
|--------------|----------------------------------------|------------------|--------|
|`displacement`|位移量或方向        |float, vector3    |__zero__|
|`scale`       |位移的缩放因子       |float             |1.0     |
|`out`         |输出:计算出的位移着色器|displacementshader|        |

<a id="node-mix-shader"> </a>

### `mix`
两个表面/位移/体积着色器闭包之间的线性混合。

|端口  |描述                                           |类型         |默认值|
|------|------------------------------------------------------|-------------|-------|
|`bg`  |背景表面着色器                        |surfaceshader|       |
|`fg`  |前景表面着色器                        |surfaceshader|       |
|`mix` |用于混合两个输入着色器的混合因子|float        |0.0    |
|`out` |输出:表面着色器闭合                        |surfaceshader|       |

|端口  |描述                                           |类型              |默认值|
|------|------------------------------------------------------|------------------|-------|
|`bg`  |背景位移着色器                   |displacementshader|       |
|`fg`  |前景位移着色器                   |displacementshader|       |
|`mix` |用于混合两个输入着色器的混合因子|float             |0.0    |
|`out` |输出:位移着色器闭合                   |displacementshader|       |

|端口  |描述                                           |类型        |默认值|
|------|------------------------------------------------------|------------|-------|
|`bg`  |背景体积着色器                         |volumeshader|       |
|`fg`  |前景体积着色器                         |volumeshader|       |
|`mix` |用于混合两个输入着色器的混合因子|float       |0.0    |
|`out` |输出:体积着色器闭合                         |volumeshader|       |

