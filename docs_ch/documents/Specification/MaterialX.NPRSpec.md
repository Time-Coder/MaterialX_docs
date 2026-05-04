<!-----
MaterialX NPR 着色节点 v1.39
----->


# MaterialX NPR 着色节点

**版本 1.39**  
Doug Smythe - Industrial Light & Magic  
Jonathan Stone - Lucasfilm Advanced Development Group  
2024年7月1日

# 简介

[MaterialX 标准节点](./MaterialX.StandardNodes.md)和 [MaterialX 基于物理的着色节点](./MaterialX.PBRSpec.md)文档描述了许多标准图案和着色节点,这些节点可用于在各种应用程序中构建用于基于物理渲染的节点图着色器。然而,在非真实感着色风格中,某些操作是理想的,但无法在某些渲染结构中实现。将主要用于真实感和非真实感着色风格的节点概念上分离到单独的库中也是有帮助的。

本文档描述了主要适用于非真实感(或称 NPR)渲染的一些 MaterialX 节点。架构不支持这些操作的渲染应用程序不需要支持这些节点。


## 目录

**[MaterialX NPR 库](#materialx-npr-library)**  
 [NPR 应用节点](#npr-application-nodes)  
 [NPR 实用节点](#npr-utility-nodes)  
 [NPR 着色节点](#npr-shading-nodes)  

**[参考文献](#references)**

<br>


# MaterialX NPR 库


## NPR 应用节点

<a id="node-viewdirection"> </a>

### `viewdirection`
当前场景视图方向,由着色环境定义。

视图方向是从观察者位置到当前着色位置的归一化向量。在 PBR 着色环境中,它表示主相机光线的入射方向,独立于任何次级或反射光线。

|端口   |描述                           |类型   |默认值|可接受的值     |
|-------|------------------------------|-------|------|--------------------|
|`space`|视图方向向量的空间|string |world  |model, object, world|
|`out`  |输出:视图方向                |vector3|       |                    |



## NPR 实用节点

<a id="node-facingratio"> </a>

### `facingratio`
视图方向和表面法线的几何朝向比率。

朝向比率计算为视图方向和表面法线之间的点积。

|端口           |描述                                                               |类型   |默认值 |
|---------------|------------------------------------------------------------------|-------|--------|
|`viewdirection`|输入视图方向向量                                           |vector3|_Vworld_|
|`normal`       |输入表面法线向量                                           |vector3|_Nworld_|
|`faceforward`  |使输出始终为正,朝向视图方向       |boolean|true    |
|`invert`       |通过将输出值乘以 -1 来反转它们                       |boolean|false   |
|`out`          |输出:表示视图方向和法线之间比率的浮点数|float  |        |




## NPR 着色节点

<a id="node-gooch-shade"> </a>

### `gooch_shade`
计算 Gooch[^Gooch1998] 光照模型的单次传递着色部分的颜色。

Gooch 着色通过根据表面法线和光线方向之间的角度混合颜色来提供说明性着色效果。它还在暖色和冷色之上提供了简单的 Phong 高光。

|端口                |描述                            |类型   |默认值      |
|--------------------|-------------------------------|-------|-------------|
|`warm_color`        |面向光线的颜色      |color3 |0.8, 0.8, 0.7|
|`cool_color`        |背向光线的颜色   |color3 |0.3, 0.3, 0.8|
|`specular_intensity`|高光的强度         |float  |1            |
|`shininess`         |高光的大小              |float  |64           |
|`light_direction`   |光线的世界空间方向 |vector3|1, -0.5, -0.5|
|`out`               |输出:Gooch 光照模型结果|color3 |             |

<br>


# 参考文献

[^Gooch1998]: Gooch 等人,**用于自动技术插图的非真实感光照模型**, <https://users.cs.northwestern.edu/~ago820/SIG98/gooch98.pdf>, 1998。
