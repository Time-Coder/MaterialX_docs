<!-----
MaterialX 规范 v1.39 的 README
----->

**MaterialX** 是一个开放标准,用于表示计算机图形学中丰富的材质和外观开发内容,使其能够在不同应用程序和渲染器之间进行平台无关的描述和交换。MaterialX 满足了对通用开放标准的需求,该标准用于表示描述计算机图形模型外观所需的数据值和关系,包括着色网络、图案和纹理、复杂的嵌套材质以及几何体分配。为了进一步促进可互换的 CG 外观设置,MaterialX 还定义了大量的标准着色和处理节点,并提供了精确的功能扩展机制。

本文件夹中的文档构成了完整的 MaterialX 规范,版本 1.39。

* [**MaterialX 规范**](./MaterialX.Specification.md) - 主要规范,描述定义和核心功能
* [**MaterialX 标准节点**](./MaterialX.StandardNodes.md) - 描述标准节点库
* [**MaterialX 基于物理的着色节点**](./MaterialX.PBRSpec.md) - 描述 BSDF(双向散射分布函数)和其他着色函数节点,这些节点可用于通过节点图构建复杂的分层渲染着色器
* [**MaterialX NPR 着色节点**](./MaterialX.NPRSpec.md) - 指定专为非真实感和风格化渲染设计的着色节点
* [**MaterialX 几何扩展**](./MaterialX.GeomExts.md) - 额外的 MaterialX 元素,用于定义与几何体相关的信息,如集合、属性和材质分配
* [**MaterialX 补充说明**](./MaterialX.Supplement.md) - 描述自定义节点定义库的推荐命名和结构化约定
* [**MaterialX:拟议的添加和更改**](./MaterialX.Proposals.md) - 描述对规范各个组件的未来更新建议

<p>

---


**MaterialX v1.39** 相较于 v1.38 提供以下增强功能:


**MaterialX 几何扩展**

主 MaterialX 规范文档中处理各种几何相关功能的部分现已拆分为独立的 [**MaterialX 几何扩展**](./MaterialX.GeomExts.md) 文档,描述了集合、几何名称表达式、几何相关数据类型、Geometry Info 元素以及其中使用的 GeomProp 和 Token 元素,以及 Look、Property、Visibility 和分配元素。

通过这种拆分,如果应用程序支持主规范中描述的所有内容(例如节点图着色网络和材质的元素以及标准节点集),同时使用应用程序的本机机制或类似 USD 的方式来描述这些材质到几何体的分配,则可以声称与 MaterialX 兼容。应用程序还可以额外支持 MaterialX 几何扩展,从而使用单一统一表示来描述完整的 CG 对象外观。


**数组类型现在为统一且固定长度**

许多着色语言不支持具有可变长度的动态数组类型,因此 MaterialX 现在仅支持具有固定最大长度的数组,并且所有数组类型的节点输入必须是统一的;节点不再允许输出数组类型。数组类型的输入可以伴随一个统一整数输入,声明数组中实际使用的数组元素数量(&lt;curveadjust> 节点已以这种方式更新)。由于此更改,未实现的 &lt;arrayappend> 节点已被移除。


**可连接的统一输入和新的 Tokenvalue 节点**

现在明确允许将统一节点输入连接到 &lt;constant> 节点的输出。这使得定义统一值并在节点图中的多个位置使用它成为可能。

类似地,材质和其他节点实例中的 &lt;token> 现在可以连接到新的 &lt;tokenvalue> 节点的输出:这本质上是一个 &lt;constant> 节点,但它连接到 &lt;token> 而不是 &lt;input>。


**标准化的色彩空间名称**

MaterialX 中的[标准色彩空间名称](./MaterialX.Specification.md#color-spaces-and-color-management-systems)现在已在规范中明确定义,并与 ACES 1.2 OCIO 配置文件中的定义保持一致。通过此更改,MaterialX 文档中不再需要定义 "cms" 或 "cmsconfig",因此这两个属性已被弃用。此外,两个新的色彩空间 "srgb_displayp3" 和 "lin_displayp3" 已作为标准色彩空间添加。


**消除歧义的 Nodedef 和 Nodegraph 引用**

通常,提供给节点的输入集及其类型与节点本身的输出类型相结合,足以消除应该应用哪个 nodedef 签名的歧义。在极少数情况下,如果这还不够,现在允许任何节点实例指定 nodedef 的名称,以完全消除预期节点签名的歧义。

此外,&lt;nodegraph> 以前可以通过提供 "nodedef" 属性声明自己是特定 &lt;nodedef> 的实现,这仍然是建立此关联的首选方法。现在,也允许 [&lt;implementation> 元素](39/MaterialX.Specification.md#custom-node-definition-using-implementation-elements) 提供 "nodegraph" 属性,声明该 nodegraph 是 &lt;implementation> 中指定的 nodedef 的实现。这允许单个 nodegraph 成为多个 nodedef 的实现,例如两个具有相同基础实现的不同节点名称,或者如果两个版本的 nodedef 之间的唯一区别是默认值。


**移除通用 Swizzle 运算符**

使用通道名称字符串并允许任意通道重新排序的标准 &lt;swizzle> 节点按照之前的规范实现效率非常低(在某些着色语言中几乎不可能),因此已被移除。节点图应改用 &lt;extract>(现在是标准节点)、&lt;separateN> 和 &lt;combineN> 节点的组合来执行任意通道重新排序。此外,允许任意通道重新排序并使用字符串 "swizzle" 通道命名的输入的先前 "channels" 属性已被移除。


**新的无光照表面着色器和标准材质**

标准库中添加了一个用于无光照表面的新 &lt;surface_unlit> 节点。

此外,标准 &lt;surfacematerial> 材质现在通过添加单独的 `backsurface` 输入来支持单面或双面表面。


**Typedef 的继承和提示**

Typedef 现在可以从其他类型(包括内置类型)继承,并可以提供有关其值的提示,如浮点精度。这些新的 "inherit" 和 "hint" 属性本身只是关于类型的元数据提示;应用程序和代码生成器仍然需要为所有自定义类型提供自己的精确定义。


**新的和更新的标准库节点**

在 1.39 版本中,我们取消了 "标准节点" 和 "补充节点" 之间的区别,现在两者的描述都可以在主规范文档中找到。使用节点图在标准发行版中实现的节点在规范中标注为 "(NG)",以区别于在每个渲染目标的本机着色语言中实现的节点。

此外,以下新的运算符节点已添加到标准库中:

* [过程节点](./MaterialX.Specification.md#procedural-nodes):**tokenvalue**、**checkerboard**、**fractal2d**、**cellnoise1d**、**unifiednoise2d**、**unifiednoise3d**
* [几何节点](./MaterialX.Specification.md#geometric-nodes):**bump**、**geompropvalueuniform**
* [数学节点](./MaterialX.Specification.md#math-nodes):布尔 **and**、**or**、**not**;**distance**、**transformcolor**、**creatematrix** 和 **triplanarblend**,以及 **floor** 和 **ceil** 的整数输出变体
* [调整节点](./MaterialX.Specification.md#adjustment-nodes):**curveinversecubic**、**curveuniformlinear**、**curveuniformcubic** 和 **colorcorrect**
* [条件节点](./MaterialX.Specification.md#conditional-nodes):布尔输出变体的 **ifgreater**、**ifgreatereq** 和 **ifequal**;新的 **ifelse** 节点
* [通道节点](./MaterialX.Specification.md#channel-nodes):**extractrowvector** 和 **separatecolor4**


**新的基于物理的着色节点**

已添加以下新的标准基于物理的着色节点:

* [EDF 节点](./MaterialX.PBRSpec.md#edf-nodes):**generalized_schlick_edf**
* [着色器节点](./MaterialX.PBRSpec.md#shader-nodes):**environment**(经纬环境光源)


**其他更改**

* 删除了描述表达式和函数曲线的 "valuerange" 和 "valuecurve" 属性,转而使用新的 &lt;curveinversecubic> / &lt;curveuniformcubic> 等节点。
* 用于 color3/4 类型值的 &lt;geomcolor>、&lt;geompropvalue> 和 &lt;geompropvalueuniform> 节点现在可以接受 "colorspace" 属性来声明属性值的色彩空间。
* &lt;cellnoise2d> 和 &lt;cellnoise3d> 节点现在除了 float 输出外,还支持 vector<em>N</em> 输出类型。
* &lt;noise2d/3d>、&lt;fractal2d/3d>、&lt;cellnoise2d/3d> 和 &lt;worleynoise2d/3d> 节点现在支持 "period" 输入。
* &lt;worleynoise2d> 和 &lt;worleynoise3d> 节点现在支持多种不同的距离度量。
* &lt;time> 节点不再有 "每秒帧数" 输入:现在始终期望应用程序使用适当的方法生成 "当前时间(秒)"。删除 "fps" 输入是因为可变帧率的实时应用程序没有静态的 "fps",而且将 fps 这样依赖于情况的值硬编码到着色网络中通常不好。
* 现在除了 "model"、"object" 和 "world" 空间外,还定义了标准的 "tangent" 空间,并且 &lt;heighttonormal> 节点现在接受统一的 "space" 输入来定义输出法线向量的空间。
* &lt;switch> 节点现在支持 10 个输入,而不仅仅是 5 个。
* &lt;surface> 和 &lt;displacement> 节点现在是主规范的一部分,而不是基于物理的着色节点。
* &lt;Token> 元素现在明确允许作为复合节点图的子元素,并且 token 值现在可以定义 enum/enumvalues。
* &lt;nodedef> 中的输入现在可以向代码生成器提供有关其预期解释的 "hints",例如 "transparency" 或 "opacity"。
* &lt;Attributedef> 元素现在可以定义 enum/enumvalues 来列出属性的可接受值或标签/映射值。
* 如果字符串输入指定了 "enum" 列表,则该列表现在被视为允许值的 "严格" 列表;不允许该列表之外的任何值。要使输入非严格,必须从输入中省略 "enum" 属性。


v1.39 的建议:

* 为各种几何属性节点添加布尔 "bound" 输出,以便在给定属性不存在时材质可以灵活处理。特别是像 &lt;texcoord> 这样不允许用户指定名称的节点。

