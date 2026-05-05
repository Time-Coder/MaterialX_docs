<!-----
MaterialX 几何扩展 v1.39
----->

- [English](../../../docs_en/documents/Specification/MaterialX.GeomExts.md)
- [简体中文](MaterialX.GeomExts.md)


# MaterialX 几何扩展

**版本 1.39**  
Doug Smythe - Industrial Light & Magic  
Jonathan Stone - Lucasfilm Advanced Development Group  
2022年10月21日  


# 简介

核心 [**MaterialX 规范**](./MaterialX.Specification.md) 定义了许多元素类型和特定的功能节点定义，可用于描述着色网络和材质的结构，包括自定义着色操作的定义和功能。

有许多格式可用于描述着色材质与可渲染几何体之间的关联以及与几何体相关的各种数据和元数据。出于性能和其他原因，通常希望使用本机应用程序机制或类似 Pixar 的 USD[^1] 来描述这些关联。然而，能够在单个应用程序无关的文件格式中存储 CG 对象 "look" 的完整描述具有重大价值。本文档描述了核心 MaterialX 规范的扩展，可用于定义几何体集合、每个几何体的几何属性及其值，以及在命名 looks 中将材质、变体、可见性和渲染属性分配给特定几何体，可以直接分配或通过几何名称表达式或命名集合分配。


## 目录

**[简介](#introduction)**  

**[几何表示](#geometry-representation)**  
 [灯光](#lights)  
 [几何名称表达式](#geometry-name-expressions)  
 [集合](#collections)  
 [几何前缀](#geometry-prefixes)  

**[其他 MaterialX 数据类型](#additional-materialx-data-types)**  

**[其他文件名替换](#additional-filename-substitutions)**  

**[Geometry Info 元素](#geometry-info-elements)**  
 [GeomInfo 定义](#geominfo-definition)  
 [GeomProp 元素](#geomprop-elements)  
 [几何 Token 元素](#geometry-token-elements)  
 [TokenDefault 元素](#tokendefault-elements)  
 [保留的 GeomProp 名称](#reserved-geomprop-names)  

**[Look 和 Property 元素](#look-and-property-elements)**  
 [Property 定义](#property-definition)  
 [Look 定义](#look-definition)  
 [Assignment 元素](#assignment-elements)  
 [MaterialAssign 元素](#materialassign-elements)  
 [VariantAssign 元素](#variantassign-elements)  
 [Visibility 元素](#visibility-elements)  
 [PropertyAssign 元素](#propertyassign-elements)  
 [Look 示例](#look-examples)  

**[参考文献](#references)**

<br>


# 几何表示

几何体在 MaterialX 内容中被引用但未具体定义。可以在任何元素中使用 `geomfile` 属性可选地声明定义几何体的文件；该 `geomfile` 声明将适用于该元素范围内引用的任何几何名称，例如任何 `geom` 属性，包括定义集合内容的属性（但通过 `collection` 属性引用集合内容时除外）。如果未为任何特定 `geom` 属性的范围定义 geomfile，则假定宿主应用程序可以解析几何体定义的位置。

MaterialX 规范中使用的几何命名约定旨在与 Alembic ([http://www.alembic.io/](http://www.alembic.io/)) 和 USD ([http://graphics.pixar.com/usd](http://graphics.pixar.com/usd)) 中使用的约定兼容。"几何体" 可以是宿主应用程序可能支持的任何特定几何对象，包括但不限于多边形、网格、细分曲面、NURBS、隐式曲面、粒子集、体积、灯光、过程定义的对象等。MaterialX 的唯一要求是使用下面指定的约定命名几何体，可以分配给材质并可以渲染。

几何体的命名应遵循类似于 UNIX 完整路径的语法:

```
   /string1/string2/string3/...
```

例如，初始 "/" 后跟一个或多个由 "/" 分隔的层次级别字符串，以最终字符串结尾且没有 "/"。构成层次级别的 path 组件的字符串不能包含空格或 "/" 或为几何名称表达式保留的任何字符（见下文）。各个实现可能对层次级别名称可以使用的字符有进一步限制，因此为了 ultimate 兼容性，建议仅使用由大写或小写字母、数字 0-9 和下划线（"_"）组成的名称。

几何名称（例如完整路径名称）在设置中引用的整个几何体集中必须是唯一的。请注意，_指定的几何路径中没有隐含的变换层次结构_:路径只是几何体的名称。然而，几何名称的路径性质可以用于几何名称表达式模式匹配和分配中的优势。

注意:如果几何网格被划分为分区，则父网格的语法为:

```
   /path/to/geom/meshname
```

对于子分区，语法为:

```
   /path/to/geom/meshname/partitionname
```

对非叶子位置的分配层次化地应用于指定位置下方的所有几何体，除非它们是另一个分配的目标。由此延伸，对 "/" 的分配应用于 MaterialX 设置中的_所有_几何体，除非它们是另一个分配的目标。


## 灯光

计算机图形资产通常包括灯光作为资产的一部分，例如汽车的前灯。MaterialX 本身不定义 "light" 对象，而是允许以与几何体相同的方式通过类似 UNIX 的路径引用外部定义的灯光对象。MaterialX 不描述灯光对象的位置、视图或形状:MaterialX 假定这些属性存储在外部表示中。

可以通过使灯光几何体不可见来在 looks 中关闭（静音）灯光对象几何体，可以使用 &lt;look> 内的 &lt;materialassign> 完成 "light" 上下文的着色器材质分配，并且可以使用灯光几何体的 &lt;visibility> 声明处理照明和阴影分配。详见下面的 [**Look 定义**](#look-definition) 部分。



## 几何名称表达式

MaterialX 文件中的某些元素支持通过表达式进行几何体规范。MaterialX 中几何名称表达式的语法在很大程度上遵循 Unix 环境中文件名的 "glob" 模式，并针对几何体引用的特定需求进行了一些扩展。

在单个层次级别内（例如在 "/" 之间）：

* `*`  匹配 0 个或多个字符
* `?`  匹配恰好一个字符
* `[]` 用于匹配方括号内的任何单个字符，其中 "-" 表示匹配前一个字符和后一个字符之间的任何字符
* `{}` 用于匹配大括号内的逗号分隔的字符串或表达式

此外，`/` 将仅匹配几何名称中的单个 `/`，例如作为层次级别的边界，而 `//` 将匹配单个 `/`，或任何数量层次级别之间的两个 `/`；`//` 可用于指定任何层次深度的匹配。如果几何名称以 `//*` 结尾，则最终的 `*` 将仅匹配层次结构中的叶子几何体。几何名称 `//*` 本身将匹配整个场景中的所有叶子几何体，而名称 `//*//` 将匹配任何级别的所有几何体，包括嵌套几何体，而名称 `/a/b/c//*//` 将匹配 `/a/b/c` 以下任何级别的所有几何体。需要注意的是，对于具有分区的网格，MaterialX 几何名称使用 `//*` 时，分区而不是网格被视为叶子几何体。



## 集合

集合是构建几何体列表（可以是几何体层次结构中的任何路径）的食谱，可以作为一次分配给大量几何体的简写。集合可以从特定几何体列表、匹配定义的几何名称表达式的几何体、其他集合或这些的任何组合构建。

一个 **&lt;collection>** 元素包含要包含的几何表达式和/或集合列表，以及可选的要排除的几何表达式列表:

```xml
  <collection name="collectionname" [includegeom="geomexpr1[,geomexpr2]..."]
             [includecollection="collectionname1[,collectionname2]..."]
             [excludegeom="geomexpr3[,geomexpr4]..."]/>
```

必须指定 `includegeom` 和/或 `includecollection`。`includegeom` 和 `includecollection` 列表首先应用，然后应用 `excludegeom` 列表。这可以用于分段构建集合的内容，或添加表达式匹配的几何体然后移除特定的不希望的匹配几何体。集合的内容可以用于定义另一个集合的一部分。每个 `includecollection` 集合的内容在被添加到正在构建的集合之前会作为一个整体进行评估。

如果包含的文件能够定义 MaterialX 兼容的集合(例如 Alembic 或 USD 文件),则可以在任何允许 <code>collection="<em>name</em>"</code> 引用的情况下引用其集合。



## 几何前缀

作为简写便利，MaterialX 允许指定一个 `geomprefix` 属性，该属性将被附加到 `geomname` 或 `geomnamearray` 类型的数据值（例如 `<geominfo>`, `<collection>`, `<materialassign>`， 和 `<visibility>` 元素中的 `geom` 属性）指定的范围内元素定义的 `geomprefix` 中，类似于 MaterialX 允许指定一个 `fileprefix` 属性，该属性将被附加到类型为 "filename" 的输入值。对于类型为 "geomnamearray" 的数据值，`geomprefix` 将被附加到每个逗号分隔的几何名称。由于前缀和几何体的值是字符串连接的，`geomprefix` 的值通常应以 "/" 结尾。Geomprefix 常用于拆分几何体路径中所有几何体名称的前导部分，例如定义 "资产根" 路径。

因此，以下 MTLX 文件片段是等效的:


```xml
  <materialx>
    <collection name="c_plastic" includegeom="/a/b/g1, /a/b/g2, /a/b/g5, /a/b/c/d/g6"/>
  </materialx>

  <materialx geomprefix="/a/b/">
    <collection name="c_plastic" includegeom="g1, g2, g5, c/d/g6"/>
  </materialx>
```

<br>


# 其他 MaterialX 数据类型

支持 MaterialX 几何扩展的系统支持以下其他标准数据类型:

**GeomName** 和 **GeomNameArray**： 类型为 "geomname" 的属性只是用引号括起来的字符串，但具体表示使用 [**几何表示**](#geometry-representation) 和 [**几何名称表达式**](#geometry-name-expressions) 部分中描述的约定的单个几何体的名称。geomname 允许使用几何名称表达式，只要它解析为单个几何体。类型为 "geomnamearray" 的属性是用引号括起来的字符串，包含一个或多个 geomname 值的逗号分隔列表，并且可以解析为任意数量的几何体。

<br>


# 其他文件名替换

各种节点的文件名输入值可以包含一个或多个特殊字符串，这些字符串将由应用程序替换为从当前几何体、MaterialX 状态或宿主应用程序环境派生的值。支持 MaterialX 几何扩展的应用程序还支持以下文件名替换:


| Token | Description |
| ---- | ---- |
| &lt;<em>geometry token</em>> | 为当前几何体声明的 &lt;geominfo> 元素或作为统一 primvar 值（通常是字符串或整数）的指定 token 的值。 |


只有完全支持几何扩展的应用程序才允许在更大的文件名字符串中使用 &lt;_geometry token_>。所有应用程序都应允许使用 "&lt;_geometry token_>" 作为完整的文件名字符串，在这种情况下，存储在几何体中的字符串 primvar 值将作为文件名使用不变；字符串 primvar 值本身可能允许包含另一个 token，例如 &lt;UDIM>， 渲染器可能能够解析并替换它。

<br>


# Geometry Info 元素

Geometry Info ("geominfo") 元素用于定义一组具有常量值的命名几何属性，并将它们与特定外部几何体关联。

geominfo 元素最常见的用途是定义映射到几何体的纹理贴图图像的文件名（或文件名的一部分）。通常，每个几何体都有几种类型的纹理，例如颜色、粗糙度、凹凸、不透明度等:每个纹理名称字符串将是 &lt;geominfo> 中的一个单独的 &lt;token>。这些图像可以包含多个几何体的纹理数据，这些几何体要么列在 &lt;geominfo> 元素的 `geom` 属性中，要么被组装成一个集合，并指定该集合的名称作为 `collection` 属性的值。


## GeomInfo 定义

一个 **&lt;geominfo>** 元素包含一个或多个几何属性和/或 token 定义，并将它们及其值与 &lt;geominfo> 元素的 `geom` 或 `collection` 属性中列出的所有几何体关联:

```xml
  <geominfo name="name" [geom="geomexpr1,geomexpr2,geomexpr3"] [collection="coll"]>
    ...geometry property and token value definitions...
  </geominfo>
```

请注意，没有两个 &lt;geominfo> 可以为相同的几何体定义相同的几何属性或 token，无论该几何体是直接指定的，还是通过几何名称表达式匹配的，还是包含在指定的集合中。

GeomInfo 元素的属性:

* `name` (string, required)： GeomInfo 元素的唯一名称
* `geom` (geomnamearray, optional)： GeomInfo 应用到的几何体和/或几何名称表达式的列表
* `collection` (string, optional)： 几何集合的名称

可以指定 `geom` 或 `collection`，但不能同时指定两者。



### GeomProp 元素

核心 MaterialX 规范定义了一个几何属性，或 "geomprop"，作为在特定空间和/或索引中引用的几何体的内在或用户定义的表面坐标属性，并提供了一些节点以检索这些属性在着色网络节点图中的值，以及一个 &lt;geompropdef> 元素用于定义自定义几何属性的名称和输出类型，除了标准的属性:`position`, `normal`, `tangent`, `bitangent`, `texcoord` 和 `geomcolor`。

MaterialX 几何扩展在此基础上允许使用 &lt;geomprop> 元素定义特定几何体的几何属性的特定统一值，而不是依赖于这些值在外部定义。这可能包括应用程序特定的元数据、从照明包传递给渲染器的属性或其他几何体特定数据。geomprop 可能还会指定 `unittype` 和 `unit` （如果适用）以指示几何属性的值的单位；参见主 MaterialX 规范中的 [**单位** 部分](./MaterialX.Specification.md#units)，尽管通常 &lt;geompropdef> 会定义 `unittype` 和 `unit`，而 geomprop 只会提供覆盖默认单位的 `unit`。

```xml
    <geomprop name="propname" type="proptype" value="value"/>
```

GeomProp 元素具有以下属性:

* `name` (string, required)： 要定义的几何属性的名称
* `type` (string, required)： 给定属性的数据类型
* `value` (any MaterialX type, required)： 要分配给给定属性的值。
* `unittype` (attribute, string, optional)： 该属性的单位类型，例如 "distance"，必须由 &lt;unittypedef> 定义。默认不指定 unittype。
* `unit` (attribute, string, optional)： 该属性的具体单位。默认不指定 unit。

只有 float 和 vector<em>N</em> 几何属性可以指定 `unittype` 和 `unit`。

例如，可以为几何体指定一个唯一的表面 ID 值:

```xml
  <geompropdef name="surfid" type="integer"/>
  <geominfo name="gi1" geom="/a/g1">
    <geomprop name="surfid" type="integer" value="15"/>
  </geominfo>
```

可以使用 `<geompropvalue>` 节点从节点图中访问 geomprop 值:

```xml
  <geompropvalue name="srfidval1" type="integer" geomprop="surfid" default="0">
```

一个 &lt;geomprop> 也可以用于定义几何体指定的 &lt;geominfo> 的内在变化几何属性（例如 "geomcolor"）的默认值，如果当前几何体本身没有为该属性定义值，则相应的几何节点（例如 &lt;geomcolor>）将返回该默认值。

```xml
  <geominfo name="gi2" geom="/a/g2">
    <geomprop name="geomcolor" type="color3" value="0.5, 0, 0"/>
  </geominfo>
```



### 几何 Token 元素

Token 元素可以在 &lt;geominfo> 元素中使用，以定义与特定几何体关联的常量（通常是字符串或整数）命名值。这些几何体 token 值可以被替换到图像节点中的文件名中；参见上面的 [**其他文件名替换**](#additional-filename-substitutions) 部分以获取详细信息:

```xml
  <token name="tokenname" type="tokentype" value="value"/>
```

"value" 可以是任何 MaterialX 类型，但由于 token 用于文件名替换，因此建议使用字符串和整数值。

Token 元素具有以下属性:

* `name` (string, required)： 要定义的几何体 token 的名称
* `type` (string, required)： 几何体 token 的类型
* `value` (any MaterialX type, optional)： 为该 token 名称分配给此几何体的值。

例如，可以为几何体指定一个纹理标识符值:

```xml
  <geominfo name="gi1" geom="/a/g1">
    <token name="txtid" type="string" value="Lengine"/>
  </geominfo>
```

然后可以在文件名中引用该 token 的值:

```xml
  <image name="cc1" type="color3">
    <input name="file" type="filename"
        value="txt/color/asset.color.<txtid>.tif"/>
  </image>
```

文件名中的 &lt;txtid> 将被替换为每个几何体的 txtid token 的值。


### TokenDefault 元素

TokenDefault 元素定义指定几何体 token 名称的默认值；如果当前几何体没有定义显式的 token 值，则此默认值将在文件名字符串替换中使用。由于 TokenDefault 不适用于任何特定的几何体，因此必须在 &lt;geominfo> 元素之外使用。

```xml
  <tokendefault name="diffmap" type="string" value="color1"/>
```


### 保留的 GeomProp 名称

涉及基于 u，v 坐标隐式计算的纹理文件名（例如 &lt;UDIM> 和 &lt;UVTILE>）的工作流程可以通过显式列出任何给定几何体的它们解析的值来变得更高效。MaterialX 规范为这一目的保留了两个 geomprop 名称，`udimset` 和 `uvtileset`，每个都是包含逗号分隔的 UDIM 或 UVTILE 值的 stringarray:

```xml
  <geominfo name="gi4" geom="/a/g1,/a/g2">
    <geomprop name="udimset" type="stringarray" value="1002,1003,1012,1013"/>
  </geominfo>

  <geominfo name="gi5" geom="/a/g4">
    <geomprop name="uvtileset" type="stringarray" value="u2_v1,u2_v2"/>
  </geominfo>
```

<br>


# Look 和 Property 元素

**Look** 元素定义了将材质、可见性和其他属性分配给几何体和几何体集合。在 MaterialX 中，每个声明的材质、可见性类型或属性都与一组几何体相关联，而不是为每个几何体定义特定的材质或属性。

**Property** 元素定义可以分配给几何体或集合的非材质属性。有许多标准的 MaterialX 属性类型可以应用于任何渲染目标，以及一种机制可以为几何体或集合定义目标特定的属性。

一个 MaterialX 文档可以包含多个属性和/或 look 元素。


## Property 定义

一个 **&lt;property>** 元素定义了几何体的特定 look 的非材质属性的名称、类型和值；&lt;**propertyset**> 元素用于将一组 &lt;property>s 组合成一个单独的命名对象。属性或 propertysets 与特定几何体或集合之间的连接是在 &lt;look> 元素中完成的，因此这些属性可以在不同的几何体之间重用，并且可以在某些 look 中启用而在其他 look 中禁用。&lt;Property> 元素只能在 &lt;propertyset>s 中使用；它们不能独立使用，尽管可以在 &lt;look> 中使用专用的 &lt;propertyassign> 元素一次声明属性名称、类型、值和分配。

```xml
  <propertyset name="set1">
    <property name="twosided" type="boolean" value="true"/>
    <property name="trace_maxdiffusedepth" target="rmanris" type="float" value="3"/>
  </propertyset>
```

以下属性在 MaterialX 中被认为是标准的，并且应该在所有支持这些概念的平台上得到尊重:


| Property | Type | Default Value |
| --- | --- | --- |
| **`twosided`** | boolean | false |
| **`matte`** | boolean | false |

其中 `twosided` 表示即使表面法线朝向相机，几何体也应该被渲染，而 `matte` 表示几何体应该挡住，或 "matte" 出它后面的内容（包括在 alpha 通道中）。

在上面的例子中，"trace_maxdiffusedepth" 属性是目标特定的，通过将其 `target` 属性设置为“rmanris”来限制其上下文。



## Look 定义

一个 **&lt;look>** 元素包含一个或多个材质、变体、可见性和/或 propertyset 分配声明:

```xml
  <look name="lookname" [inherit="looktoinheritfrom"]>
    ...materialassign, variantassign, visibilityassign, property/propertysetassign declarations...
  </look>
```

Looks 可以通过包含 `inherit` 属性继承另一个 look 的分配。然后，look 可以指定其他分配，这些分配将应用于从源 look 继承的内容之上或代替它。这对于定义一个基础 look 以及一个或多个 "variation" look 很有用。一个继承的 look 本身可以继承自另一个 look，但一个 look 只能继承自一个父 look。

可以将多个 looks 分组到一个 **LookGroup**，例如以指示哪些 looks 定义了特定资产:

```xml
  <lookgroup name="lookgroupname" looks="look1[,look2[,look3...]]" [default="lookname"]/>
```

其中 `lookgroupname` 是要定义的 lookgroup 的名称，`look1`/`look2`/等是包含在 lookgroup 中的 &lt;look> 或 &lt;lookgroup> 元素的名称（lookgroup 名称将解析为该 lookgroup 递归包含的 looks),`default`（如果指定）指定 `looks` 中定义的 looks 中的一个作为默认 look 使用。一个 look 可以包含在任何数量的 lookgroups 中。

&lt;Look> 和 &lt;lookgroup> 元素还支持其他属性，例如 `xpos`, `ypos` 和 `uicolor` 如上面的 Standard UI Attributes 部分所述。


## Assignment 元素

各种类型的分配元素在 looks 中用于将材质、分类的可见性和属性分配给特定几何体，或将变体分配给材质。

对于将分配给几何体的元素，`geom` 属性中的路径名称或存储在集合中的名称不需要严格解析为 "叶子" 路径位置或实际可渲染几何体名称:分配也可以应用于中间的 "分支" 几何体路径位置，这将应用于路径层次结构中更深层次的任何几何体，这些几何体没有另一个"更接近叶子"级别的分配。例如，对 "/a/b/c" 的分配将有效地应用于 "/a/b/c/d" 和 "/a/b/c/foo/bar"（以及任何其他以 "/a/b/c/" 开头的完整路径名称），如果对 "/a/b/c/d"、"/a/b/c/foo" 或 "/a/b/c/foo/bar" 没有其他分配。如果一个 look 继承自另一个 look，子 look 可以替换对任何特定路径位置的分配（例如，对 "/a/b/c" 的子分配将优先于父 look 对 "/a/b/c" 的分配），但父 look 对更"叶子"级别的路径位置的分配将优先于子 look 对更高"分支"级别的位置的分配。


### MaterialAssign 元素

MaterialAssign 元素在 &lt;look> 中使用，将指定的材质连接到一个或多个几何体或集合（可以指定 `geom` 或 `collection`，但不能同时指定两者）。

```xml
  <materialassign name="maname" material="materialname"
                 [geom="geomexpr1[,geomexpr2...]"] [collection="collectionname"]
                 [exclusive=true|false]>
    ...optional variantassign elements...
  </materialassign>
```

材料分配通常假定是互斥的，即任何单个几何体只分配给一个材质。因此，分配声明应按文件中出现的顺序进行处理，如果任何几何体出现在多个 &lt;materialassign>s 中，则最后一个 &lt;materialassign> 赢得。然而，某些应用程序允许将多个材质分配给同一个几何体，只要着色器节点类型不重叠。如果 `exclusive` 属性设置为 false（默认为 true），则较早的材料分配仍然对所有未在较晚分配的材料中定义的着色器节点类型生效:对于每个着色器节点类型，引用匹配着色器节点类型的最后一个分配的材料中的着色器获胜。如果特定应用程序不支持将多个材料分配给同一个几何体，则忽略 `exclusive` 的值，并且只将最后一个完整材料及其着色器分配给几何体，并且解析器应发出警告。


### VariantAssign 元素

VariantAssign 元素在 &lt;materialassign> 或 &lt;look> 中使用，将一个变体集中定义的值应用于一个分配的材质，或应用于 look 中的所有适用材质。

```xml
  <look name="look1">
    <variantassign name="va1" variantset="varset1" variant="var1"/>
    <materialassign name="ma1" material="material1" geom="...">
      <variantassign name="va2" variantset="varset2" variant="var2"/>
    </materialassign>
    <materialassign name="ma2" material="material2" geom="..."/>
    ...
  </look>
```

VariantAssign 元素具有以下属性:

* `name` (string, required)： VariantAssign 元素的唯一名称
* `variantset` (string, required)： 要从中应用变体的变体集的名称
* `variant` (string, required)： 要使用的 `variantset` 中的变体名称

在上面的例子中，在变体 "var1" 中定义的输入/token 值将应用于 "material1" 或 "material2" 中找到的任何同名输入/token，除非被 &lt;variantset> 中定义的 `node` 或 `nodedef` 属性限制，而变体 "var2" 中定义的值仅应用于 "material1" 中的匹配名称绑定。VariantAssigns 按照在作用域内的指定顺序应用，其中 &lt;materialassign> 内的优先于直接子元素的 &lt;look>。


### Visibility 元素

Visibility 元素在 &lt;look> 中使用，定义一个 "查看器" 对象与其他几何体之间的各种类型的泛化可见性。"查看器对象" 简单地是一个能够在某些渲染上下文中"看到"其他几何体的几何体，因此可能需要指定它在不同上下文中"看到"的几何体列表；最常见的例子是光源和主要渲染相机。

```xml
  <visibility name="vname" [viewergeom="objectname"]
             [geom="geomexpr1[,geomexpr2...]"] [collection="collectionname"]
             [vistype="visibilitytype"] [visible="false"]/>
```

Visibility 元素具有以下属性:

* `name` (string, required): Visibility 元素的唯一名称
* `viewergeom` (geomnamearray, optional)： 受 &lt;visibility> 分配影响的查看器几何体对象列表
* `viewercollection` (string, optional)： 包含受 &lt;visibility> 分配影响的查看器几何体对象的集合名称
* `geom` (geomnamearray, optional): `viewergeom` 对象应该(或不应该)"看到"的几何体和/或几何名称表达式
* `collection` (string, optional): `viewergeom` 对象应该(或不应该)"看到"的定义几何体集合的名称
* `vistype` (string, optional)： 正在定义的可见性类型；见下表
* `visible` (boolean, optional)： 如果为 false，则 geom/collection 对象对这种特定类型的可见性不可见；默认为 "true"。

`viewergeom` 属性(和/或 `viewercollection` 属性引用的集合内容)通常指的是一个光源(或光源列表)或其他"几何体查看"对象的名称。如果省略 `viewergeom`/`viewercollection`，则该可见性适用于给定渲染上下文中的所有适用查看器(相机、光源、几何体)；`viewergeom`/`viewercollection` 通常不为 `vistype` "camera" 指定。必须定义 `geom` 或 `collection` 但不能同时定义两者；同样，不能同时定义 `viewergeom` 和 `viewercollection`。

`vistype` 属性指的是特定类型的可见性。如果特定的 `vistype` 未在 &lt;look> 中分配,则所有几何体对 `viewergeom` 的所有 `vistype` 默认可见;这意味着要使某些几何体的子集可见(整体或对特定 `vistype`),必须首先为所有几何体分配 `visible="false"` 的 &lt;visibility>。在 &lt;look> 中对同一 `vistype` 的附加 &lt;visibility> 分配将应用于当前可见性状态。以下 `vistype` 由 MaterialX 预定义;应用程序可以定义其他 `vistype`:


| Vistype | Description |
| --- | --- |
| **`camera`** | 相机或"主"射线可见性 |
| **`illumination`** | geom 或集合被 viewergeom 光源照亮 |
| **`shadow`** | geom 或集合从 viewergeom 光源投射阴影 |
| **`secondary`** | geom 或集合对 viewergeom 几何体的间接/反弹射线可见性 |


如果未指定 `vistype`,则该可见性分配适用于 _所有_ 可见性类型,并且实际上将优先于同一几何体的任何特定 `vistype` 设置:几何体分配一个没有 `vistype` 且 `visible="false"` 的 &lt;`visibility>` 将对相机、阴影、次级射线或任何其他射线或渲染类型不可见。此机制可以用于干净地隐藏资产中不需要的某些变体,例如不同的服装部件或替代损坏形状。

如果 &lt;visibility> `geom` 或 `collection` 引用灯光几何体,则分配 `vistype="camera"` 确定灯光对象本身是否对相机/查看器可见(例如"你是否看到灯泡"),而分配 `visible="false"` 且没有 `vistype` 将静音灯光,使其既不可见也不发出任何光线。

对于 "secondary" vistype,`viewergeom` 应该是可渲染几何体而不是灯光，以声明某些其他几何体是否对间接反弹照明或 `viewergeom` 的反射可见。在这个例子中，"/b" 不会在反射中显示也不会对 "/a" 贡献间接反弹照明，而几何体 "/c" 不会对 _任何_ 次级射线可见:

```xml
  <visibility name="v2" viewergeom="/a" geom="/b" vistype="secondary" visible="false"/>
  <visibility name="v3" geom="/c" vistype="secondary" visible="false"/>
```


### PropertyAssign 元素

PropertyAssign 和 PropertySetAssign 元素在 &lt;look> 中使用，将指定的属性值或属性集连接到一个或多个几何体或集合。

```xml
  <propertyassign name="paname" property="propertyname" type="type" value="value"
                 [target="target"]
                 [geom="geomexpr1[,geomexpr2...]"] [collection="collectionname"]/>
  <propertysetassign name="psaname" propertyset="propertysetname"
                 [geom="geomexpr1[,geomexpr2...]"] [collection="collectionname"]/>
```

可以指定 `geom` 或 `collection`，但不能同时指定两者。可以对同一个几何体或集合进行多个属性/属性集分配，只要没有冲突的分配。如果有任何冲突的分配，则由宿主应用程序决定如何解决此类冲突，但宿主应用程序应按 look 中列出的顺序应用属性分配，因此通常可以安全地假设如果两个属性/属性集分配为同一个几何体设置不同的属性值，则后一个分配将获胜。


## Look 示例

此示例定义了四个集合、一个灯光着色器和材质，以及一个属性集，然后由两个 looks 使用:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<materialx>
  <!-- assume <nodedef> and <surfacematerial> elements to define Mplastic1,2 and Mmetal1,2 are placed or included here -->
  <collection name="c_plastic" includegeom="/a/g1,/a/g2,/a/g5"/>
  <collection name="c_metal" includegeom="/a/g3,/a/g4"/>
  <collection name="c_lamphouse" includegeom="/a/lamp1/housing*Mesh"/>
  <collection name="c_setgeom" includegeom="/b"/>
  <nodedef name="ND_disklgt_lgt" node="disk_lgt">
    <input name="emissionmap" type="filename" value=""/>
    <input name="gain" type="float" value="1.0"/>
    <output name="out" type="lightshader"/>
  </nodedef>
  <disk_lgt name="LSheadlight">
    <input name="gain" type="float" value="500.0"/>
  </disk_lgt>
  <lightmaterial name="Mheadlight">
    <input name="lightshader" type="lightshader" nodename="LSheadlight"/>
  </lightmaterial>
  <propertyset name="standard">
    <property name="displacementbound_sphere" target="rmanris" type="float"
           value="0.05"/>
    <property name="trace_maxdiffusedepth" target="rmanris" type="float" value="5"/>
  </propertyset>
  <look name="lookA">
    <materialassign name="ma1" material="Mplastic1" collection="c_plastic"/>
    <materialassign name="ma2" material="Mmetal1" collection="c_metal"/>
    <materialassign name="ma3" material="Mheadlight" geom="/a/b/headlight"/>
    <visibility name="v1" viewergeom="/a/b/headlight" vistype="shadow" geom="/" visible="false"/>
    <visibility name="v2" viewergeom="/a/b/headlight" vistype="shadow" collection="c_lamphouse"/>
    <propertysetassign name="psa1" propertysetname="standard" geom="/"/>
  </look>
  <look name="lookB">
    <materialassign name="ma4" material="Mplastic2" collection="c_plastic"/>
    <materialassign name="ma5" material="Mmetal2" collection="c_metal"/>
    <propertysetassign name="psa2" propertysetname="standard" geom="/"/>
    <!-- make the setgeom invisible to camera but still visible to shadows and reflections -->
    <visibility name="v3" vistype="camera" collection="c_setgeom" visible="false"/>
  </look>
  <lookgroup name="assetlooks" looks="lookA,lookB" default="lookA"/>
</materialx>
```

<br>


# 参考文献

[^1]: <https://graphics.pixar.com/usd/release/index.html>

