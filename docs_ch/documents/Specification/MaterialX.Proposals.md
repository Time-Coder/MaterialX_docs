<!-----
MaterialX 提案 v1.39
----->


# MaterialX:拟议的添加和更改

**版本 1.39 的提案**  
2024年9月15日


# 简介

[MaterialX 规范](./MaterialX.Specification.md)历来不仅包含当前功能的描述,还包含旨在最终实现的前瞻性提议功能。我们相信,明确哪些功能目前在库中得到支持,以及哪些部分记录了拟议的添加将是有益的。

因此,这些前瞻性提案已从正式规范文档移至本拟议添加和更改文档中进行讨论和辩论。一旦在代码库中实际实现,这些描述就可以迁移到适当的正式规范文档中。一旦社区达成普遍有利的共识,可以将新的 MaterialX 更改和添加提案添加到此文档中。



## 目录

**[简介](#introduction)**  

**[提案:通用](#propose-general)**  

**[提案:元素](#propose-elements)**  

**[提案:Stdlib 节点](#propose-stdlib-nodes)**  

**[提案:PBR 节点](#propose-pbr-nodes)**  

**[提案:NPR 节点](#propose-npr-nodes)**  


<p>&nbsp;<p><hr><p>

# Proposals: General<a id="propose-general"></a>


## 色彩空间

当 OCIO NanoColor 库(提供链接)可用时,MaterialX 应该支持该规范中的官方色彩空间名称,当前的 MaterialX 色彩空间名称作为别名支持。

MaterialX 还应该支持以下色彩空间:
* `lin_rec2020`
* `g22_rec2020`



<p>&nbsp;<p><hr><p>

# 提案:元素<a id="propose-elements"></a>


### AOV 输出元素

(README.md 的摘要:**新增着色器 AOV 支持**

以前,MaterialX 使用具有输出变量结构的自定义类型来定义着色器 AOV。但这种方法不太灵活,实际上尚未实现。在 v1.39 中,基于节点图的着色器实现可以包含新的 [&lt;aovoutput> 元素](./MaterialX.Specification.md#aov-output-elements)来定义 AOV,渲染器可以使用这些 AOV 输出除最终着色结果之外的额外信息通道,而基于文件的 &lt;implementation> 同样可以使用 [&lt;aov> 元素](./MaterialX.Specification.md#implementation-aov-elements)定义 AOV。
)

具有 "shader" 或 "material" 语义输出类型的功能节点图可能包含多个 &lt;aovoutput> 元素来声明任意输出变量("AOV"),渲染器可以看到并将其作为额外的信息流输出。AOVoutputs 必须是 float、color3 或 vector3 类型(用于预着色 "pattern" 值),或 BSDF 或 EDF(用于着色器节点输出值);期望渲染器从 BSDF 和 EDF 类型中提取适当的类颜色信息。在此功能节点图中实例化的着色器语义节点内定义的 AOV 可以通过在 &lt;aovoutput> 中提供 sourceaov 属性来 "传递" 并可能重命名(但不能以任何方式修改或操作)。

```xml
  <aovoutput name="name" type="type" aovname="aovname"
             nodename="node_to_connect_to" [sourceaov="aovname"]/>
```

&lt;aovoutput> 元素的属性为:

* name (string,必需):此 aov 输出定义元素的用户选择名称。
* type (string,必需):AOV 的类型,必须是上面列出的支持类型之一。
* aovname (string,必需):渲染器应用于 AOV 的名称。
* nodename (string,必需):其输出定义 AOV 值的节点的名称。
* sourceaov (string,可选):如果 nodename 是 surfaceshader 类型,则在 nodename 内定义的输出 AOV 的名称,以作为输出 AOV 传递。nodename 内定义的 sourceaov 的类型必须与 &lt;aovoutput> 类型匹配。

示例:

```xml
  <aovoutput name="Aalbedo" type="color3" aovname="albedo"
             nodename="coat_affected_diffuse_color"/>
  <aovoutput name="Adiffuse" type="BSDF" aovname="diffuse">
             nodename="diffuse_bsdf"/>
```

#### AovOutput 示例

使用 sourceaov 从着色器语义节点的实例中转发 AOV 的 &lt;aovoutput> 示例;这假设 &lt;standard_surface> 本身已为 "diffuse" 和 "specular" AOV 定义了 &lt;aovoutput>:

```xml
  <nodegraph name="NG_basic_surface_srfshader" nodedef="ND_basic_surface_srfshader">
    <image name="i_diff1" type="color3">
      <input name="file" type="filename"
                 value="txt/[diff_map_effect]/[diff_map_effect].<UDIM>.tif"/>
    </image>
    <mix name="diffmix" type="color3">
      <input name="bg" type="color3" interfacename="diff_albedo"/>
      <input name="fg" type="color3" nodename="i_diff1"/>
      <input name="mix" type="float" interfacename="diff_map_mix"/>
    </mix>
    <standard_surface name="stdsurf1" type="surfaceshader">
      <input name="base_color" type="color3" nodename="diffmix"/>
      <input name="diffuse_roughness" type="float" interfacename="roughness"/>
      <input name="specular_color" type="color3" interfacename="spec_color"/>
      <input name="specular_roughness" type="float" interfacename="roughness"/>
      <input name="specular_IOR" type="float" interfacename="spec_ior"/>
    </standard_surface>
    <output name="out" type="surfaceshader" nodename="stdsurf1"/>
    <aovoutput name="NGAalbedo" type="color3" aovname="albedo" nodename="diffmix"/>
    <aovoutput name="NGAdiffuse" type="BSDF" aovname="diffuse" nodename="stdsurf1"
                  sourceaov="diffuse"/>
    <aovoutput name="NGAspecular" type="BSDF" aovname="specular" nodename="stdsurf1"
                  sourceaov="specular"/>
  </nodegraph>
```

分层着色器或材质必须在将 AOV 类似值作为 AOV 输出之前内部处理来自源层的混合:目前没有用于混合在后着色混合 surfaceshaders 中定义的 AOV 的工具。

注意:虽然在语法上可以为在节点图中访问的几何图元值(如着色表面点和法线)创建 &lt;aovoutput>,但更希望渲染器直接从其内部着色状态或几何 primvars 派生此类信息。



#### Implementation AOV 元素

具有定义表面着色器的外部编译实现的 file 属性的 &lt;implementation> 元素可以包含一个或多个 &lt;aov> 元素来声明着色器可以输出到渲染器的任意输出变量("AOV")的名称和类型。AOV 必须是 float、color3、vector3、BSDF 或 EDF 类型。请注意,在 MaterialX 中,预着色 "pattern" 颜色的 AOV 通常是 color3 类型,而后着色的类颜色值通常是 BSDF 类型,发射类颜色值通常是 EDF 类型。具有 `nodegraph` 属性的 &lt;implementation> 不能包含 &lt;aov> 元素;相反,应使用节点图中的 &lt;aovoutput> 元素。

```xml
  <implementation name="IM_basicsurface_surface_rmanris"
                  nodedef="ND_basic_surface_surface" implname="basic_srf"
                  target="rmanris" file="basic_srf.C">
    ...<inputs>...
    <aov name="IMalbedo" type="color3" aovname="albedo"/"/>
    <aov name="IMdiffuse" type="BSDF" aovname="diffuse"/"/>
  </implementation>
```



### 材质继承

材质可以从其他材质继承,以添加或更改连接到不同输入的着色器;在此示例中,将位移着色器添加到上述 "Mgold" 材质以创建新的 "Mgolddsp" 材质:

```xml
  <noise2d name="noise1" type="float">
    <input name="amplitude" type="float" value="1.0"/>
    <input name="pivot" type="float" value="0.0"/>
  </noise2d>
  <displacement name="stddsp" type="displacementshader">
    <input name="displacement" type="float" nodename="noise1"/>
    <input name="scale" tpe="float" value="0.1"/>
  </displacement>
  <surfacematerial name="Mgolddsp" type="material" inherit="Mgold">
    <input name="displacementshader" type="displacementshader" nodename="stddsp"/>
  </surfacematerial>
```

还允许材质类型自定义节点的继承,以便可以在继承的材质中指定的输入值之上应用新的或更改的输入值。


<p>&nbsp;<p><hr><p>

# 提案:Stdlib 节点<a id="propose-stdlib-nodes"></a>


### 过程节点

<a id="node-tokenvalue"> </a>

### `tokenvalue`
常量 "interface token" 值,只能连接到节点中的 &lt;token>,不能连接到 &lt;input>。

|端口   |描述                         |类型            |默认值  |
|-------|----------------------------|----------------|---------|
|`value`|将发送到 `out` 的值|string, filename|__empty__|
|`out`  |输出:`value`                     |与 `value` 相同 |__empty__|


### 噪声节点

<a id="node-cellnoise1d"> </a>

### `cellnoise1d`
1D 细胞噪声,提议作为随机值生成的替代方法。

|端口    |描述                                      |类型            |默认值 |
|--------|-----------------------------------------|----------------|--------|
|`in`    |评估噪声的 1D 坐标|float           |        |
|`period`|噪声的周期                          |float, vector3  |__zero__|
|`out`   |输出:计算的噪声值                 |float, vector3  |__zero__|

<a id="node-worleynoise2d"> </a>

### `worleynoise2d`
此提案扩展现有的 `worleynoise2d` 节点以支持不同的距离度量和周期性。

|端口    |描述                              |类型                   |默认值 |可接受的值                          |
|--------|---------------------------------|-----------------------|--------|-----------------------------------------|
|`metric`|要返回的距离度量            |string                 |distance|distance, distance2, manhattan, chebyshev|
|`period`|噪声的周期                  |float, vector3         |__zero__|                                         |
|`out`   |输出:计算的噪声值         |float, vector2, vector3|__zero__|                                         |

<a id="node-worleynoise3d"> </a>

### `worleynoise3d`
此提案扩展现有的 `worleynoise3d` 节点以支持不同的距离度量和周期性。

|端口    |描述                              |类型                   |默认值 |可接受的值                          |
|--------|---------------------------------|-----------------------|--------|-----------------------------------------|
|`metric`|要返回的距离度量            |string                 |distance|distance, distance2, manhattan, chebyshev|
|`period`|噪声的周期                  |float, vector3         |__zero__|                                         |
|`out`   |输出:计算的噪声值         |float, vector2, vector3|__zero__|                                         |

`period` 输入是一个正整数距离,噪声函数在该步长重复的坐标处返回相同的值。值为 0 表示噪声不是周期性的。

#### 周期性噪声

在 #1201 中,决定将所有噪声的单独周期性版本优于将其添加到现有噪声中。

### 形状节点





### 几何节点



### 全局节点

<a id="node-ambientocclusion"> </a>

### `ambientocclusion`
计算当前表面点的环境光遮蔽,返回 0 到 1 之间的标量值。环境光遮蔽表示每个表面点对环境光照的可访问性,较大的值表示对光的可访问性更大。

`coneangle` 输入定义表面法线周围的半角,因此默认值 90 度表示完整的半球。

|端口         |描述                                      |类型 |默认值|
|-------------|-----------------------------------------|-----|-------|
|`coneangle`  |遮蔽锥的半角,以度为单位 |float|90.0   |
|`maxdistance`|考虑遮蔽的最大距离   |float|1e38   |
|`out`        |输出:计算的环境光遮蔽值     |float|       |



### 应用节点

<a id="node-updirection"> </a>

### `updirection`
当前场景 "up vector" 方向,由着色环境定义。

|端口   |描述                                         |类型   |默认值|可接受的值     |
|-------|--------------------------------------------|-------|-------|--------------------|
|`space`|返回 up vector 方向所在的空间|string |world  |model, object, world|
|`out`  |输出:`space` 中的 up 方向                 |vector3|       |                    |



### 数学节点

<a id="node-transformcolor"> </a>

### `transformcolor`
将传入的颜色从一个指定的色彩空间转换到另一个色彩空间,忽略上游可能提供的任何色彩空间声明。对于 color4 类型,alpha 通道值不受影响。`fromspace` 和 `tospace` 输入接受标准色彩空间的名称或应用程序理解的色彩空间;任何一个都可以为空以指定文档的工作色彩空间。

|端口       |描述                          |类型          |默认值  |
|-----------|-----------------------------|--------------|---------|
|`in`       |输入颜色                      |color3, color4|__zero__ |
|`fromspace`|从中转换 `in` 的色彩空间|string        |__empty__|
|`tospace`  |将 `in` 转换到的色彩空间  |string        |__empty__|
|`out`      |输出:转换后的颜色        |与 `in` 相同  |         |

<a id="node-triplanarblend"> </a>

### `triplanarblend`
从三个输入采样数据,并沿每个相应的坐标轴投影图像的平铺表示,使用几何法线计算三个采样的加权混合。

`iny` 输入沿从 +Y 轴回到原点的方向投影,+X 轴在右侧。

如果被采用,现有的 `triplanarprojection` 节点将重新实现为一个包装器,解析其三个图像输入并将它们传递给 `triplanarblend`。

|端口        |描述                                                                                    |类型           |默认值  |可接受的值       |
|------------|---------------------------------------------------------------------------------------|---------------|---------|----------------------|
|`inx`       |沿从 +X 轴回到原点的方向投影的图像             |float 或 colorN|__zero__ |                      |
|`iny`       |沿从 +Y 轴回到原点的方向投影的图像             |float 或 colorN|__zero__ |                      |
|`inz`       |沿从 +Z 轴回到原点的方向投影的图像             |float 或 colorN|__zero__ |                      |
|`position`  |评估投影的 3D 位置                                           |vector3        |_Pobject_|                      |
|`normal`    |用于混合的 3D 法线向量                                                         |vector3        |_Nobject_|                      |
|`blend`     |混合三个轴采样的加权因子,较高的值给出更柔和的混合|float          |1.0      |[0, 1]                |
|`filtertype`|要使用的纹理过滤类型                                                           |string         |linear   |closest, linear, cubic|
|`out`       |输出:混合值                                                                      |与 `inx` 相同  |__zero__ |                      |

<a id="node-maxcomponent"> </a>

### `maxcomponent`
将传入的 vectorN 或 colorN 流的所有组件的最大值作为 float 值输出。

|端口 |描述                          |类型            |默认值  |
|-----|-----------------------------|----------------|---------|
|`in` |输入值                      |vectorN, colorN |__zero__ |
|`out`|输出:`in` 的最大组件    |float           |0.0      |

<a id="node-mincomponent"> </a>

### `mincomponent`
将传入的 vectorN 或 colorN 流的所有组件的最小值作为 float 值输出。

|端口 |描述                          |类型            |默认值  |
|-----|-----------------------------|----------------|---------|
|`in` |输入值                      |vectorN, colorN |__zero__ |
|`out`|输出:`in` 的最小组件    |float           |0.0      |


### 调整节点

<a id="node-curveinversecubic"> </a>

### `curveinversecubic`
使用输入 `knots` 值上的反向 Catmull-Rom spline 查找重新映射 0-1 输入值。必须提供至少 2 个 knot 值,第一个和最后一个 knot 的重数为 2。

|端口   |描述                                    |类型      |默认值|
|-------|---------------------------------------|----------|-------|
|`in`   |输入值                                |float     |0.0    |
|`knots`|重新映射的输入 knot 值列表|floatarray|       |
|`out`  |输出:重新映射的值                     |float     |       |

<a id="node-curveuniformlinear"> </a>

### `curveuniformlinear`
输出一个 float、colorN 或 vectorN 值,在多个 `knotvalues` 值之间线性插值,使用 `in` 的值作为插值器。

|端口        |描述                                          |类型                                 |默认值|
|------------|---------------------------------------------|-------------------------------------|-------|
|`in`        |输入插值器值                          |float                                |0.0    |
|`knotvalues`|要插值的至少 2 个值的数组|floatarray, colorNarray, vectorNarray|       |
|`out`       |输出:插值值                       |float, colorN, vectorN               |       |

<a id="node-curveuniformcubic"> </a>

### `curveuniformcubic`
输出一个 float、colorN 或 vectorN 值,使用 Catmull-Rom spline 在多个 `knotvalues` 值之间平滑插值,使用 `in` 的值作为插值器。

|端口        |描述                                          |类型                                 |默认值|
|------------|---------------------------------------------|-------------------------------------|-------|
|`in`        |输入插值器值                          |float                                |0.0    |
|`knotvalues`|要插值的至少 2 个值的数组|floatarray, colorNarray, vectorNarray|       |
|`out`       |输出:插值值                       |float, colorN, vectorN               |       |

<a id="node-curveadjust"> </a>

### `curveadjust`
使用由指定 knot 值定义的向心 Catmull-Rom cubic spline 曲线输出输入值的平滑重新映射,使用输入 knot 值上的反向 spline 查找和输出 knot 值上的正向 spline。输入的所有通道将使用相同的曲线重新映射。

|端口        |描述                                                                                     |类型                  |默认值 |
|------------|----------------------------------------------------------------------------------------|----------------------|--------|
|`in`        |输入值                                                                                 |float, colorN, vectorN|__zero__|
|`numknots`  |knots 和 knotvalues 数组中的值数量                                         |integer               |2       |
|`knots`     |定义重新映射曲线的输入值列表;至少 2 个且最多 16 个值 |floatarray            |        |
|`knotvalues`|定义重新映射曲线的输出值列表;必须与 knots 长度相同|floatarray            |        |
|`out`       |输出:重新映射的值                                                                      |与 `in` 相同          |        |

<a id="node-curvelookup"> </a>

### `curvelookup`
输出一个 float、colorN 或 vectorN 值,在多个 knotvalue 值之间平滑插值,使用 `in` 在 `knots` 中的位置作为 knotvalues 插值器。

|端口        |描述                                                                              |类型                                 |默认值|
|------------|---------------------------------------------------------------------------------|-------------------------------------|-------|
|`in`        |输入插值器值                                                              |float                                |0.0    |
|`numknots`  |knots 和 knotvalues 数组中的值数量                                  |integer                              |2      |
|`knots`     |要在其中插值 `in` 的 knot 值列表;至少 2 个且最多 16 个值     |floatarray                           |       |
|`knotvalues`|每个 knot 位置要插值的值;必须与 knots 长度相同|floatarray, colorNarray, vectorNarray|       |
|`out`       |输出:插值值                                                           |float, colorN, vectorN               |       |



### 合成节点



### 条件节点

<a id="node-ifelse"> </a>

### `ifelse`
根据布尔选择器输入的值是 true 还是 false,输出两个输入流之一的值。

|端口     |描述                                         |类型                  |默认值 |
|---------|--------------------------------------------|----------------------|--------|
|`infalse`|如果 `which` 为 false 则输出的值             |float, colorN, vectorN|__zero__|
|`intrue` |如果 `which` 为 true 则输出的值              |与 `infalse` 相同     |__zero__|
|`which`  |选择从哪个输入获取值的选择器|boolean               |false   |
|`out`    |输出:选定的输入                          |与 `infalse` 相同     |        |



### 通道节点

<a id="node-separatecolor4"> </a>

### `separatecolor4`
将 color4 的 RGB 和 alpha 通道作为单独的输出输出。

|端口      |描述                      |类型  |默认值           |
|----------|-------------------------|------|------------------|
|`in`      |输入 color4 值           |color4|0.0, 0.0, 0.0, 0.0|
|`outcolor`|输出:`in` 的 RGB 通道 |color3|0.0, 0.0, 0.0     |
|`outa`    |输出:`in` 的 alpha 通道|float |0.0               |



<p>&nbsp;<p><hr><p>

# 提案:PBR 节点<a id="propose-pbr-nodes"></a>



<p>&nbsp;<p><hr><p>

# 提案:NPR 节点<a id="propose-npr-nodes"></a>

