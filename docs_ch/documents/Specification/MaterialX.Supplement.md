<!-----
MaterialX 补充说明 v1.39
----->


# MaterialX:补充说明

**版本 1.39**  
Doug Smythe - Industrial Light & Magic  
Jonathan Stone - Lucasfilm Advanced Development Group  
2023年3月25日  



# 简介

本文档详细介绍了有关 MaterialX 的附加信息以及如何将其整合到工作室管线中。文档描述了节点定义元素的推荐命名约定以及用于定义来自各种来源的节点定义和实现的包目录结构。

以前版本的 MaterialX 补充说明文档包含了其他节点类型的描述:这些节点描述现在已折叠回[主规范文档](./MaterialX.Specification.md#nodes)中,与所有其他标准节点描述放在一起。


## 目录

**[简介](#introduction)**  

**[推荐的元素命名约定](#recommended-element-naming-conventions)**  

**[材质和节点库文件结构](#material-and-node-library-file-structure)**  
 [示例](#examples)  

**[定义、资产和库](#definitions-assets-and-libraries)**  
 [使用节点图组织](#organization-using-node-graphs)  
 [发布定义](#publishing-definitions)  
 [依赖和组织](#dependencies-and-organization)  
 [部署、传输和转换](#deployment-transmission-and-translation)  

<br>


# 推荐的元素命名约定

虽然 MaterialX 元素可以被赋予主规范中 MaterialX 名称部分所述的任何有效名称,但遵守以下推荐的命名约定将使预测 nodedef 的名称(用于 implementation 和 nodegraph 元素)变得更容易,并有助于减少来自不同来源的元素具有相同名称的可能性。

**Nodedef**: "ND\__nodename_\__outputtype_[\__target_][\__version_]",或者对于给定输出类型有多个输入类型的节点(例如 &lt;convert>),"ND\__nodename_\__inputtype_\__outputtype_[\__target_][\__version_]"。

**Implementation**: "IM\__nodename_[\__inputtype_]\__outputtype_[\__target_][\__version_]"。

**Nodegraph**,作为节点的实现:"NG\__nodename_[\__inputtype_]\__outputtype_[\__target_][\__version_]"。

<br>


# 材质和节点库文件结构

随着工作室和供应商为各种目标开发共享的 MaterialX 材质和节点的定义和实现的库,为构成这些库的磁盘上的文件拥有一致、逻辑的组织结构变得有益。在本节中,我们为定义材质节点库、&lt;nodedef>、节点图实现和实际特定于目标的本地源代码的文件提出一种结构,以及应用程序和 MaterialX 内容查找和引用这些库中文件的机制。

文件夹层次结构中各个组成部分的图例:

| 术语 | 描述 |
| --- | --- |
| _libname_ | 库的名称;MaterialX 标准节点是 "stdlib" 库。库可以选择声明自己位于 <em>libname</em> 命名空间中,尽管这不是必需的。 |
| _target_ | 实现的目标,例如 "glsl"、"oslpattern"、"osl" 或 "mdl"。 |
| _sourcefiles_ | 目标的源文件(包括包含文件和 makefile),采用适用构建系统所需的任何格式和结构。 |


以下是构成 MaterialX 材质或节点定义库设置的各种文件的建议结构和命名。斜体术语应替换为适当的值,而粗体术语应按字面出现。文件名中的可选 "\_\*" 组件可以是文件内容的任何有用描述符,例如节点图的 "\_ng" 或材质的 "\_mtls"。


 _libname_/_libname_**\_defs.mtlx** (1)  
 _libname_/_libname_\_\***.mtlx** (2)  
 _libname_/_target_/_libname_\_target[\_\*]**\_impl.mtlx** (3)  
 _libname_/_target_/_sourcefiles_ (4)  



1. 库 _libname_ 中的 Nodedef 和其他定义。
2. 库 _libname_ 中的其他元素(例如节点的节点图实现、材质等)。
3. 特定于目标 _target_ 的 _libname_ 的实现元素。
4. 特定于目标 _target_ 的 _libname_ 实现的源代码文件。

请注意,nodedef 文件和节点图实现文件位于顶层 _libname_ 级别,而 &lt;implementation> 元素文件位于相应的 _libname_/_target_ 级别下,靠近其源代码文件。这样,工作室可以轻松安装仅与他们相关的实现,应用程序可以轻松找到特定所需目标的节点实现。库可以自由添加其他任意命名的文件夹以存放相关内容,例如材质库纹理的 "images" 子文件夹。

_libname_\_defs.mtlx 文件通常包含库的 nodedef,但也可能包含其他节点类型,如实现节点图、材质、looks 和任何其他元素类型。使用额外的 _libname_\_\*.mtlx 文件是可选的,但这些文件应由 _libname_\_defs.mtlx 文件 Xinclude。

MaterialX 文档或工具引用的文件(例如 XInclude 文件、&lt;image> 或其他 MaterialX 节点中的文件名,或 MaterialX 工具中的命令行参数)可以使用相对或完全合格的绝对文件系统路径来指定。相对路径被解释为相对于引用 MaterialX 文档本身的位置,或相对于当前 MaterialX 搜索路径中找到的位置:此路径可以通过应用程序设置(例如 MaterialXView 中的 `--path` 选项)或使用 MATERIALX_SEARCH_PATH 环境变量全局指定。这些搜索路径用于 XIncluded 定义和文件名输入值(例如节点的图像或 &lt;implementation> 的源代码),如果需要,应用程序可以为不同的上下文定义不同的搜索路径,例如文档处理与渲染。

标准库 `stdlib` 和 `pbrlib` 通常由 MaterialX 应用程序_自动_包含,而不是通过 .mtlx 文件中的显式 XInclude 指令。非标准库通过 XInclude 顶层 _libname_/_libname_\_defs.mtlx 文件包含到 MaterialX 文档中,该文件预计会依次 XInclude 库所需的任何其他 .mtlx 文件。


### 示例

在以下示例中,MXROOT 是当前 MaterialX 搜索路径中定义的根路径之一的占位符。

工作室自定义材质着色网络和示例库材质的库:

```
    MXROOT/mtllib/mtllib_defs.mtlx                (材质 nodedef 和节点图)
    MXROOT/mtllib/mtllib_mtls.mtlx                (使用 mtllib_defs 的材质库)
    MXROOT/mtllib/images/*.tif                    (mtllib_mtls <image> 节点使用的纹理文件)
```

文档可以使用以下方式包含上述库

```xml
   <xi:include href="mtllib/mtllib_defs.mtlx"/>
```

该文件将 XInclude `mtllib_mtls.mtlx`。`mtllib_mtls.mtlx` 中的 &lt;Image> 节点将使用诸如 "images/bronze_color.tif" 的 `file` 输入值,例如相对于 `mtllib_mtls.mtlx` 文件本身的路径。

标准节点定义和参考 OSL 实现:

```
    MXROOT/stdlib/stdlib_defs.mtlx                    (标准库节点定义)
    MXROOT/stdlib/stdlib_ng.mtlx                      (补充库节点节点图)
    MXROOT/stdlib/osl/stdlib_osl_impl.mtlx            (stdlib OSL 实现 elem 文件)
    MXROOT/stdlib/osl/*.{h,osl} (等等)                 (stdlib OSL 源文件)
```

MaterialX 的 shadergen 组件的 "genglsl" 和 "genosl" 实现的布局,引用上述标准 `stdlib_defs.mtlx` 文件:

```
    # 生成的 GLSL 实现
    MXROOT/stdlib/genglsl/stdlib_genglsl_impl.mtlx    (stdlib genGLSL 实现文件)
    MXROOT/stdlib/genglsl/stdlib_genglsl_cm_impl.mtlx (stdlib genGLSL 色彩管理实现文件)
    MXROOT/stdlib/genglsl/*.{inline,glsl}             (stdlib 通用 genGLSL 代码)

    # 生成的 OSL 实现
    MXROOT/stdlib/genosl/stdlib_genosl_impl.mtlx      (stdlib genOSL 实现文件)
    MXROOT/stdlib/genosl/stdlib_genosl_cm_impl.mtlx   (stdlib genOSL 色彩管理实现文件)
    MXROOT/stdlib/genosl/*.{inline,osl}               (stdlib 通用 genOSL 代码)
```

shadergen PBR 着色器库("pbrlib")的布局,具有 "genglsl" 和 "genosl"(分别为生成的 GLSL 和 OSL)目标的实现:

```
    MXROOT/pbrlib/pbrlib_defs.mtlx                    (PBR 库定义)
    MXROOT/pbrlib/pbrlib_ng.mtlx                      (PBR 库节点图)
    MXROOT/pbrlib/genglsl/pbrlib_genglsl_impl.mtlx    (引用 genGLSL 源的 pbr 实现文件)
    MXROOT/pbrlib/genglsl/*.{inline,glsl}             (pbr 通用 genGLSL 代码)
    MXROOT/pbrlib/genosl/pbrlib_genosl_impl.mtlx      (引用 genOSL 源的 pbr 实现文件)
    MXROOT/pbrlib/genosl/*.{inline,osl}               (pbr 通用 genOSL 代码)
```

<br>


# 定义、资产和库

在本节中,我们提出了一套管理唯一定义或资产并组织成库的指南,其中:

* 定义:直接对应于 &lt;nodedefs>,可以是源代码实现或基于现有节点定义。
* 资产:是一个术语,对应于定义加上定义的任何附加元数据和/或相关资源(如输入图像)。这些可以根据所需的语义组织成逻辑分组。
* 库:是资产的集合。


### 使用节点图组织

虽然可以在文档中只有一组连接的节点,但不可能有任何正式的唯一接口。这 invariably 会导致节点具有重复名称、无法控制暴露的接口以及无法随时间维护变体。

因此,定义的基本要求是将节点封装到 &lt;nodegraph> 中。这提供了:

1. 隐藏复杂性:所有节点都在图的范围内。从用户交互点来看,它使得能够根据需要"深入"到图中,但否则可以提供黑盒表示。
2. 标识符/路径唯一性:节点图名称减少了名称冲突的机会。例如,当放入两个节点图 "bar1" 和 "bar2" 时,两个都称为 "foo" 的顶级节点将具有唯一路径 "bar1/foo" 和 "bar2/foo"。  
3. 接口/节点签名控制,其中特定输入可以通过 "interfacename" 连接暴露,输出按需暴露。这与"隐藏"输入或输出不同,后者不改变签名。前者强制执行向用户暴露的内容,而后者只是接口提示。

对于单个输入,建议根据需要添加以下附加属性:

1. 现实世界单位:如果输入值取决于场景/几何体大小,则应始终添加单位属性。例如,如果图表示地板瓷砖,则要正确放置它需要瓷砖的大小。标准库提供了一组预设的 "distance" 单位。
2. 色彩空间:如果输入值以给定的色彩空间表示,则要支持正确转换为渲染色彩空间,应设置此属性。标准库提供了一组符合 AcesCg 1.2 配置的色彩空间名称。

虽然不是严格要求,但具有适当的默认值非常有用,因为没有这些默认值将为零值。因此,例如,纹理节点的 "scale" 属性不应默认为零。


### 发布定义

从 &lt;nodegraph> 可以创建(新的 &lt;nodedef>)或"发布"定义。发布允许以下重要功能:

1. 重用:能够重用唯一的节点定义,而不是复制图实现。
2. 变体:能够独立于实现创建或将变体应用于实例。
3. 互操作性:支持具有所有消费者都相互理解的共同属性的定义进行交换。

为了支持这些功能,建议始终指定以下属性:

1. 唯一名称标识符:这可以遵循所述的 ND\_ 和 NG\_ 约定。建议也对接口的签名进行编码以提供唯一性,特别是如果定义可能是多态的。
2. 命名空间标识符 (*):检查唯一性时始终使用命名空间,因此不需要将其作为名称标识符的一部分来提供唯一性。
    * 不应使用它,因为它会导致命名空间被多次 prepend。例如,具有命名空间 "myspace" 的 "foo" 节点的唯一标识符为 "myspace:node"。如果节点命名为 "myspace:node",则 resulting 标识符为 "myspace:myspace:node"。
    * 请注意,导入文档将根据需要 prepend 命名空间而不会重复命名空间。
3. 版本标识符:虽然这可以是通用字符串,但建议这是具有特定格式的模板,以允许已知的增量编号。例如,格式可以是 "v#.#" 以支持次要和主要版本控制。这需要将所有版本中的一个标记为默认版本。应注意确保这一点,因为找到的第一个将被用作默认值。
4. 节点组标识符:这可以是用于定义组织或用户界面呈现的一种机制。它也在某种程度上用于着色器生成,因为它提供了节点类型的提示。例如,&lt;image> 节点位于 "texture2d" 节点组中。
5. 文档字符串。虽然不是严格要求,但这提供了一些能力来说明节点的作用,并可用作用户界面帮助器。目前与此没有关联的格式,但可以嵌入格式。

请注意,作为核心分发的一部分提供了编纂发布逻辑的实用程序。

为了支持变体,建议使用 &lt;token> 和 &lt;variant>。



1. Token:这些允许使用相同的 "template" 作为文件名,而无需创建新定义。这可以减少所需定义的数量,也可以减少所需预设的数量。例如,token 可用于控制所需文件名标识符的格式、分辨率。
2. 变体和变体集:对于何时创建定义与使用带有变体的定义,没有硬性"规则",但一个可能的建议是在没有接口签名差异时使用变体(_讨论_?)。一些优点包括变体可以独立于定义打包和部署,和/或每个变体不需要部署新定义。请注意,目前只能进行值覆盖。


### 依赖和组织

添加的定义越多,包括基于其他定义的定义,给定具有一组节点实例的文档,发现需要哪些定义就越困难。

为了支持依赖项的可分离性,提出了以下逻辑高级语义分解:



1. 静态"核心"库定义。这些包括 stdlib、pbrlib 和 bxdf。建议始终加载这些并假设它们存在。为了可分离性,建议将这些都存储在单个运行时 Document 中。
2. 静态自定义库定义。这些基于核心库。建议不要使用 Xinclude 机制直接引用任何核心库。这可能导致加载重复的可能冲突的定义。"升级"机制将确保所有核心和自定义库都升级到适当的目标版本。为了可分离性,建议将这些都存储在单个运行时 Document 中。
3. 动态创建的定义。如果允许此功能,那么拥有这些定义可以更新的固定位置集可能会很有用。这可以是本地的用户或更新现有的自定义库。

可以添加额外的分组以提供语义组织(例如按 "nodegroup"),但建议它们位于公共库根或包内。

对于具有依赖资源的资产,有很多选项。其中两个是:



1. 将资源与定义 co-locate。这允许更轻松地"打包"单个资产,例如用于传输目的,但这可能需要额外的发现逻辑来查找与定义关联的资源,并可能导致资源重复。
2. 位于与定义平行的文件夹结构中。责任是维护这种平行结构,但资源的搜索逻辑复杂度与定义相同。

如果定义是源代码实现,则在生成期间需要额外的路径搜索逻辑以实现可发现性。

以下搜索路径可用:



* MATERIALX_SEARCH_PATH:此环境变量用作定义和资源搜索的一部分(例如相对文件名路径支持)。
* 源代码路径:这可以在代码生成时作为生成"上下文"的一部分注册。建议遵循源路径位置,这些位置将相对于任何自定义定义,使用代码生成器的 "language" 标识符来发现适当的源代码文件。

示例 Viewer 中可以找到路径使用的示例。逻辑如下:



* 模块/二进制路径设置为默认 "root" 定义路径。其他定义根通过 MATERIALX_SEARCH_PATH 包含。
* 根集被视为 "resources" 和 "libraries" 文件夹的父级,分别用于资源和定义。
* 资源的搜索路径根默认为 "`<rootpath>/resources`"。这允许处理作为资产定义一部分的资源。例如,位于 "`/myroot/shaders/brick.mtlx`" 的砖块着色器可能在位置 "`/myroot/textures/brick.exr`" 引用砖块纹理。将单个搜索路径设置为 "`/myroot`" 处理上述 "parallel" 文件夹组织,相对引用为 "`textures/brick.exr`"
* 对于给定路径的任何着色器,在解析依赖资源时可以添加到该着色器的路径。这可用于处理 co-located 文件夹组织。例如,着色器可能驻留在 "`/myroot/shader/brick.mtlx`",纹理在 "`/myroot/shader/textures/brick.exr`"。将根设置为 "`myroot/shader`",相对引用为 "`textures/brick.exr`" 将允许正确发现。

对于运行时,建议不要读取文档,而是"导入"它们。这允许诸如命名空间处理以及每个文档的源位置("sourceURI")标记等机制。在某个时候,所有依赖文档需要合并为一个,因为没有引用内存中文档的概念。标记是一种有用的机制,允许从主文档内容中过滤/排除定义。例如,可以"清除"主文档,同时保留库定义。

由于可能存在依赖于其他定义的定义,因此永远不建议卸载核心库,在卸载自定义或动态库时要小心。如果希望卸载任何库,重新加载所有定义可能会很有用。

请注意,代码生成是基于上下文的。如果上下文未清除,则将保留依赖的源代码实现。如果卸载定义,建议清除实现。


### 部署、传输和转换

给定一组定义,考虑它将如何部署是有用的。

对于一些文件访问可能受到限制或访问许多文件是性能问题的部署,可能需要预打包定义、源和相关资源。例如,这目前用于示例 Web viewer 部署。

一些部署可能不想处理非核心定义或可能无法处理(复杂)节点图。此外,定义可能作为着色器传输。因此,在创建新定义时,建议确定以下支持级别:



1. 扁平化:定义是否可以转换为具有源代码实现的一系列节点。
2. 烘焙:定义是否可以转换为图像。
3. 转换:实现是否可以转换映射/转换为另一个可以 consumed 的实现。
4. 着色器反射:是否可以将所需的元数据传递给着色器以进行 introspection。

不是规范正式部分的附加元数据可能导致消费者无法理解。
