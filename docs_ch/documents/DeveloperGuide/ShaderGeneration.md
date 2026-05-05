# 着色器生成

- [English](../../../docs_en/documents/DeveloperGuide/ShaderGeneration.md)
- [简体中文](ShaderGeneration.md)

## 1.1 范围
MaterialX 中实现了一个着色器生成框架。这可以帮助应用程序将与设备无关的 MaterialX 数据描述转换为特定渲染器的可执行着色器代码。名为 MaterialXGenShader 的库模块包含核心着色器生成功能，而对特定语言的支持位于单独的库中，例如 [MaterialXGenGlsl](https://github.com/AcademySoftwareFoundation/MaterialX/tree/main/source/MaterialXGenGlsl)、[MaterialXGenOsl](https://github.com/AcademySoftwareFoundation/MaterialX/tree/main/source/MaterialXGenOsl)。

请注意，此系统没有运行时，生成的输出是源代码，而不是二进制可执行代码。生成的源代码需要在由渲染器执行之前由着色器语言编译器编译。参见图 1 了解系统的高级概述。

![使用多个着色器生成器进行着色器生成](https://raw.githubusercontent.com/AcademySoftwareFoundation/MaterialX/main/documents/Images/shadergen.png)

**图 1：** 使用多个着色器生成器进行着色器生成。

## 1.2 语言和着色器生成器
MaterialX 描述不包含设备特定的细节，所有实现细节都需要由着色器生成器处理。每种支持的着色语言都有一个着色器生成器。然而，对于每种语言，也可能需要针对不同渲染器的变体。例如；支持 GLSL 的 OpenGL 渲染器可以使用前向渲染或延迟渲染，每种渲染方式对着色器的构建方式都有非常不同的要求。另一个例子是不同的渲染器支持 OSL，但具有不同的闭包集或闭包参数。因此，可以为每种语言/目标组合定义单独的着色器生成器。

使用类继承和特化来创建对新语言的支持或为新的目标自定义现有语言支持。要为目标添加新的着色器生成器，您需要添加一个从基类 `ShaderGenerator` 或现有派生着色器生成器类（`GlslShaderGenerator`、`OslShaderGenerator` 等）派生的新 C++ 类，并重写您需要自定义的方法。您可能还需要派生一个新的 `Syntax` 类，用于处理不同着色语言之间的语法差异。然后，您需要确保为您想要支持的所有节点（标准库节点和其他库中的节点）定义了实现，要么在适用的情况下重用现有实现，要么添加新的实现。请参阅 **1.3 节点实现** 了解如何完成此操作。

请注意，在添加节点定义时不需要定义着色器生成器。可以稍后添加新的着色器生成器，并且可以为现有节点添加新目标的节点实现。

## 1.3 节点实现
有四种不同的方法来定义节点的实现：
1. 使用内联表达式。
2. 使用用目标语言编写的函数。
3. 使用定义节点执行操作的节点图。
4. 使用在着色器生成期间动态发出代码的 C++ 类。

在接下来的子章节中将解释每种方法。对于所有方法，实现都与具有明确定义的输入和输出接口的特定 `nodedef` 相关联。

### 1.3.1 内联表达式
提供的代码生成器支持一种非常简单的表达式语言用于内联代码。这对于可以用单行代码表示操作的简单节点很有用。内联将减少函数调用的数量并产生更紧凑的代码。使用的语法与目标着色语言相同，另外使用双花括号包裹的节点输入端口作为变量：`{{input}}`。代码生成器将这些变量替换为分配或连接到相应输入的值。图 2 给出了一个示例。

使用 `<implementation>` 元素将表达式连接到 `nodedef`，如图 2 所示。第一种选择是将内联代码保存在文件中。文件扩展名用于区分内联表达式和源代码函数，使用 `filename.inline`。第二种选择是使用 `sourcecode` 直接嵌入内联代码。如果逻辑可以适合一行代码，这是内联的推荐方法。

```xml
<!-- 节点 <add> 的节点定义元素 -->
<nodedef name="ND_add_float" node="add">
  <input name="in1" type="float" />
  <input name="in2" type="float" />
  <output name="out" type="float" defaultinput="in1" />
</nodedef>
<nodedef name="ND_add_color3" node="add" type="color3">
  <input name="in1" type="color3" />
  <input name="in2" type="color3" />
  <output name="out" type="color3" defaultinput="in1" />
</nodedef>
<!-- ... 更多类型 ... -->

<!-- 节点 <add> 的实现元素 -->
<implementation name="IM_add_float" nodedef="ND_add_float" file="mx_add.inline" />
<implementation name="IM_add_color3" nodedef="ND_add_color3" file="mx_add.inline" />
<!-- ... 更多类型 ... -->

<!-- 节点 <mix> 的节点定义元素 -->
<nodedef name="ND_mix_float" node="mix">
  <input name="fg" type="float" />
  <input name="bg" type="float" />
  <input name="mix" type="float" />
  <output name="out" type="float" defaultinput="bg" />
</nodedef>
<nodedef name="ND_mix_color3" node="mix">
  <input name="fg" type="color3" />
  <input name="bg" type="color3" />
  <input name="mix" type="color3" />
  <output name="out" type="color3" defaultinput="bg" />
</nodedef>
<!-- ... 更多类型 ... -->

<!-- 节点 <mix> 的实现元素 -->
<implementation name="IM_mix_float" nodedef="ND_mix_float" sourcecode="mix({{bg}}, {{fg}}, {{mix}})" />
<implementation name="IM_mix_color3" nodedef="ND_mix_color3" sourcecode="mix({{bg}}, {{fg}}, {{mix}})" />
<!-- ... 更多类型 ... -->
```
```c++
// 文件 'mx_add.inline' 包含：
{{in1}} + {{in2}}
```

**图 2：** 实现节点 `<add>` 和 `<mix>` 的内联表达式。`<add>` 的代码存储在附加文件中，而 `<mix>` 的代码作为 `<implementation>` 声明的一部分指定。 

### 1.3.2 着色语言函数
对于无法通过内联表达式实现的节点，可以使用函数定义代替。函数签名应与 nodedef 的输入和输出接口匹配。参见图 3 了解示例。使用 `<implementation>` 元素将源代码连接到 nodedef，有关更多信息，请参阅 [MaterialX 规范](https://materialx.org/Specification.html)。

```xml
<!-- 节点定义元素 -->
<nodedef name="ND_image_color3" node="image">
  <input name="file" type="filename" value="" uniform="true" />
  <input name="layer" type="string" value="" uniform="true" />
  <input name="default" type="color3" value="0.0, 0.0, 0.0" />
  <input name="texcoord" type="vector2" defaultgeomprop="UV0" />
  <input name="uaddressmode" type="string" value="periodic" uniform="true" />
  <input name="vaddressmode" type="string" value="periodic" uniform="true" />
  <input name="filtertype" type="string" value="linear" uniform="true" />
  <input name="framerange" type="string" value="" uniform="true" />
  <input name="frameoffset" type="integer" value="0" uniform="true" />
  <input name="frameendaction" type="string" value="constant" uniform="true" />
  <output name="out" type="color3" default="0.0, 0.0, 0.0" />
</nodedef>

<!-- 实现元素 -->
<implementation name="IM_image_color3_osl" nodedef="ND_image_color3" file="mx_image_color3.osl" target="genosl" />
```
```c++
// 文件 'mx_image_color3.osl' 包含：
void mx_image_color3(string file, string layer, color defaultvalue,
                     vector2 texcoord, string uaddressmode, string vaddressmode, string filtertype,
                     string framerange, int frameoffset, string frameendaction,
                     output color out)
{
    // 采样纹理
    out = texture(file, texcoord.x, texcoord.y,
                  "interp", filtertype,
                  "subimage", layer,
                  "missingcolor", defaultvalue,
                  "wrap", uaddressmode);
}
```
**图 3：** OSL 中节点 `<image>` 的着色语言函数实现。

### 1.3.3 节点图实现
作为定义源代码的替代方案，还可以选择引用节点图作为 nodedef 的实现。唯一的要求是节点图和 nodedef 具有匹配的输入和输出。

这对于为执行某些常见操作的一组节点创建复合节点很有用。然后可以在其他节点图中将其作为节点引用。它对于为未知节点创建兼容性图也很有用。如果某个节点是由第三方创建的，并且其实现未知或专有，则可以使用已知节点创建兼容性图，并将其作为替代实现进行引用。通过将 nodedef 属性设置在节点图定义上，将节点图链接到 nodedef。参见图 4 了解示例。

```xml
<nodedef name="ND_checker_float" node="checker">
  <input name="texcoord" type="vector2" defaultgeomprop="UV0" />
  <input name="uvtiling" type="vector2" value="8.0, 8.0" />
  <output name="out" type="float" />
</nodedef>
<nodegraph name="IM_checker_float" nodedef="ND_checker_float">
  <multiply name="mult1" type="vector2">
    <input name="in1" type="vector2" interfacename="texcoord" />
    <input name="in2" type="vector2" interfacename="uvtiling" />
  </multiply>
  <floor name="floor1" type="vector2">
    <input name="in" type="vector2" nodename="mult1" />
  </floor>
  <dotproduct name="dotproduct1" type="float">
    <input name="in1" type="vector2" nodename="floor1" />
    <input name="in2" type="vector2" value="1, 1" />
  </dotproduct>
  <modulo name="modulo1" type="float">
    <input name="in1" type="float" nodename="dotproduct1" />
    <input name="in2" type="float" value="2" />
  </modulo>
  <output name="out" type="float" nodename="modulo1" />
</nodegraph>
```
**图 4：** 使用节点图的 Checker 节点实现。

### 1.3.4 动态代码生成
在某些情况下，静态源代码不足以实现节点。代码可能需要根据节点上设置的参数进行自定义。或者对于硬件渲染目标，可能需要创建顶点流或统一输入以提供节点实现所需的数据。

在这种情况下，可以添加一个 C++ 类来处理节点的实现。该类应从基类 `ShaderNodeImpl` 派生。它应通过重写 `getTarget()` 来指定其适用的目标。然后需要通过调用 `ShaderGenerator::registerImplementation()` 为 `ShaderGenerator` 注册它。参见图 5 了解示例。

当为 nodedef 使用 `ShaderNodeImpl` 类时，相应的 `<implementation>` 元素不需要 file 属性，因为不使用静态源代码。此时 `<implementation>` 元素仅作为声明，表明存在针对特定目标的 nodedef 实现。

请注意，通过使用 `ShaderNodeImpl` 类来实现节点，它不再是数据驱动的，如上述其他三种方法所示。因此，建议仅在内联表达式或静态源代码函数不足以处理节点实现时使用此方法。

```c++
/// OSL 中 'foo' 节点的实现
class FooOsl : public ShaderNodeImpl
{
  public:
    static ShaderNodeImplPtr create() { return std::make_shared<FooOsl>(); }

    const string& getTarget() const override { return OslShaderGenerator::TARGET; }

    void emitFunctionDefinition(const ShaderNode& node, GenContext& context,
                                ShaderStage& stage) const override
    {
        // 如果需要，为节点发出函数定义
    }

    void emitFunctionCall(const ShaderNode& node, GenContext& context,
                          ShaderStage& stage) const override
    {
        // 为节点发出函数调用或内联着色器代码
    }
};
```
```c++
OslShaderGenerator::OslShaderGenerator() :
    ShaderGenerator(std::make_shared<OslSyntax>())
{
    ...
    // 为应该使用的 nodedef 注册 foo 实现
    registerImplementation("IM_foo_color2_osl", FooOsl::create);
    registerImplementation("IM_foo_color3_osl", FooOsl::create);
    registerImplementation("IM_foo_color4_osl", FooOsl::create);
    ...
}
```
**图 5：** 用于动态代码生成的 C++ 类。

## 1.4 着色器生成步骤
本节概述了从 MaterialX 描述生成着色器的一般步骤。`ShaderGenerator` 基类及其支持类将为您处理这些步骤，但如果需要自定义更改以支持新目标，了解涉及的步骤是很有好处的。

着色器生成支持从材质中的 `output` 元素或 `shaderref` 元素开始生成着色器。`output` 可以是节点图上的输出端口或插入到节点网络中任何位置的输出元素。通过调用您的着色器生成器类并传入这些元素类型之一来生成着色器。给定的元素及其所有上游依赖项将被翻译成目标着色语言中的单个整体着色器。

```c++
// 从给定元素开始生成着色器，将该元素及其所有上游依赖项翻译成着色器代码。
ShaderPtr ShaderGenerator::generate(const string& name,
                                    ElementPtr element,
                                    GenContext& context)
```

着色器生成过程可以分为初始化和代码生成。初始化包括以下步骤：
1. 创建图的优化版本作为树，以给定的输入元素为根，并且仅连接使用的上游依赖项。这涉及删除图中未使用的路径，将常量节点转换为常量值，并为具有指定默认连接但未连接的端口添加任何默认节点。删除未使用的路径通常涉及常量折叠和修剪永远不会采用的条件分支。由于最终的着色器将由着色器语言编译器编译，并接受大量额外的优化，因此我们不需要在此优化步骤中做太多工作。然而，一些图级别的优化可以使生成的着色器更小，并在着色器编译期间节省时间和内存。它还将产生更易读的源代码，这对调试目的很有好处。此优化步骤也是进行特定目标所需的其他自定义优化的好地方。例如图的简化，可能涉及用近似节点替换昂贵的节点，识别可以合并的公共子图等。
2. 节点按拓扑顺序排序。由于一个节点可以被图中的许多其他节点引用，我们需要对节点进行排序，以便依赖于其他节点的节点在所有依赖节点之后。此步骤还确保图中没有循环依赖。
3. 创建着色器的阶段。对于硬件着色器，这通常是顶点阶段和像素阶段，但可以根据需要添加其他阶段。至少需要一个像素阶段，因此即使是像 OSL 这样没有多阶段概念的着色器，也需要创建一个像素阶段。
4. 建立着色器阶段的统一变量和变化变量接口。这包括正在使用的图接口端口，以及已发布到接口的内部端口（后者的一个示例是硬件着色器生成器，其中图像纹理文件名被转换为纹理采样器，需要发布以便目标应用程序可以绑定）。图中的每个节点也被调用以有机会创建其实现所需的任何统一变量或变化变量。
5. 跟踪每个节点的范围信息。此信息需要处理条件节点的分枝。例如，如果节点仅在可变条件的特定分支中使用，我们希望仅在该范围内（即采用相应分支时）计算此节点。节点可以在全局范围、单个条件范围或多个条件范围中使用。

初始化步骤的输出是使用 `ShaderNode`、`ShaderInput`、`ShaderOutput`、`ShaderGraph` 等类构建的新图表示。这是一个针对着色器生成优化的图表示，可以快速访问和遍历节点和端口，以及缓存着色器生成所需的额外信息。

初始化之后，代码生成步骤由 `ShaderGenerator` 类和派生类处理。这部分特定于所使用的特定生成器，但通常包括以下步骤：
1. 按照 Syntax 类指定的方式发出类型定义。
2. 为所有具有着色语言函数实现的原子节点发出函数定义。对于使用动态代码生成的节点，调用其 `ShaderNodeImpl` 实例来生成函数。对于通过图实现的节点，发出代表图计算的函数定义。
3. 发出将所有统一变量设置为默认值的着色器签名。随后可以从返回的 `Shader` 实例访问着色器统一变量，以便应用程序能够将值绑定到它们。
4. 以正确的依赖顺序发出所有节点的函数调用，将上游节点的输出结果作为下游节点的输入传播。对于使用内联表达式的节点，发出内联表达式而不是函数调用。
5. 生成最终着色器输出并分配给着色器输出变量。

请注意，如果您的系统不适合整个图的单个整体着色器，可以在图中的任何点调用生成器来处理 `output` 元素，并为子部分生成代码。然后由应用程序决定在哪里分割图，并在所有子部分生成后组装它们的着色器代码。

## 1.5 着色器阶段

支持创建多个着色器阶段。这对于在硬件渲染目标上为多个阶段生成单独的代码是必需的。所有目标都必须始终创建 `pixel` 阶段，即使是像 OSL 这样原生没有阶段概念的着色语言也是如此。阶段是存储生成的着色器代码以及所有统一变量、输入和输出的地方。这由 `ShaderStage` 类处理，生成完成后可以从中检索数据。

一个或多个 `ShaderStage` 实例被创建并存储在 `Shader` 类上。除了 `pixel` 阶段外，硬件生成器总是指定一个 `vertex` 阶段。如果需要其他阶段，也可以添加它们。创建着色器输入变量时，您需要指定变量应该在哪个阶段中使用，有关着色器变量创建的更多信息，请参见 1.7。

使用静态源代码（函数或内联表达式）的节点实现始终发出到 `pixel` 阶段。使用静态源代码不支持控制 `vertex` 阶段或其他阶段。为了做到这一点，您必须为您的节点使用自定义 `ShaderNodeImpl` 子类的动态代码生成。然后您可以分别控制它如何影响所有阶段。在 `emitFunctionDefinition` 和 `emitFunctionCall` 内部，您可以使用开始/结束着色器阶段宏为每个阶段添加单独的部分。图 6 展示了 GLSL 的 texcoord 节点如何在 `vertex` 和 `pixel` 阶段中发出不同的代码。

## 1.6 着色器变量
当从节点图或 shaderref 生成着色器时，这些元素上的输入和参数将作为统一变量发布在生成的着色器上。可以从生成的 `Shader` 和 `ShaderStage` 实例中读取创建的统一变量列表。然后可以向用户显示着色器统一变量，并由应用程序设置它们的值。

### 1.6.1 变量创建
向着色器阶段添加新的统一变量、输入和输出是通过首先创建一个 `VariableBlock` 来存储它们来完成的。有一些预定义的标识符用于常用的变量块。对于统一变量，例如有一个名为 `HW::PUBLIC_UNIFORMS` 和一个名为 `HW::PRIVATE_UNIFORMS`。公共用于如上所述要发布给用户的统一变量，私有用于节点实现所需但由应用程序设置且不发布的统一变量。对于硬件目标，还有称为 `connector blocks` 的特定变量块，用于将数据从一个阶段发送到另一个阶段，连接各个阶段。名为 `HW::VERTEX_DATA` 的连接器块用于将数据从 `vertex` 阶段发送到 `pixel` 阶段。可以根据每个着色器生成器目标的需要自定义变量块的创建和处理。

生成后，应用程序可以从 `ShaderStage` 实例查询和访问所有变量块。

图 6 展示了需要此功能的节点实现如何创建着色器输入和连接器变量。

```c++
// GLSL 中 'texcoord' 节点的实现
class TexCoordGlsl : public ShaderNodeImpl
{
  public:
    static ShaderNodeImplPtr create()
    {
        return std::make_shared<TexCoordGlsl>();
    }

    void TexCoordNodeGlsl::createVariables(const ShaderNode& node, GenContext&,
                                           Shader& shader) const
    {
        const ShaderOutput* output = node.getOutput();
        const ShaderInput* indexInput = node.getInput(INDEX);
        const string index = indexInput ? indexInput->getValue()->getValueString() : "0";

        ShaderStage& vs = shader.getStage(Stage::VERTEX);
        ShaderStage& ps = shader.getStage(Stage::PIXEL);

        addStageInput(HW::VERTEX_INPUTS, output->getType(), "i_texcoord_" + index, vs);
        addStageConnector(HW::VERTEX_DATA, output->getType(), "texcoord_" + index, vs, ps);
    }

    void TexCoordNodeGlsl::emitFunctionCall(const ShaderNode& node,
                                            GenContext& context,
                                            ShaderStage& stage) const
    {
        const ShaderGenerator& shadergen = context.getShaderGenerator();

        const ShaderInput* indexInput = node.getInput(INDEX);
        const string index = indexInput ? indexInput->getValue()->getValueString() : "0";
        const string variable = "texcoord_" + index;

        DEFINE_SHADER_STAGE(stage, Stage::VERTEX)
        {
            VariableBlock& vertexData = stage.getOutputBlock(HW::VERTEX_DATA);
            const string prefix = vertexData.getInstance() + ".";
            ShaderPort* texcoord = vertexData[variable];
            if (!texcoord->isEmitted())
            {
                shadergen.emitLine(prefix + texcoord->getVariable() + " = i_" + variable, stage);
                texcoord->setEmitted();
            }
        }

        DEFINE_SHADER_STAGE(stage, Stage::PIXEL)
        {
            VariableBlock& vertexData = stage.getInputBlock(HW::VERTEX_DATA);
            const string prefix = vertexData.getInstance() + ".";
            ShaderPort* texcoord = vertexData[variable];
                shadergen.emitLineBegin(stage);
            shadergen.emitOutput(node.getOutput(), true, false, context, stage);
            shadergen.emitString(" = " + prefix + texcoord->getVariable(), stage);
            shadergen.emitLineEnd(stage);
        }
    }
};
```
**图 6：** GLSL 中节点 `texcoord` 的实现。使用 `ShaderNodeImpl` 子类来控制着色器变量创建和代码生成到单独的着色器阶段。

### 1.6.2 变量命名约定

创建着色器变量并将值绑定到它们需要在着色器生成器端和应用程序端达成一致。应用程序必须知道变量的用途才能向其绑定有意义的数据。处理此问题的一种方法是使用语义。所有创建的着色器变量都可以分配一个语义（如果目标应用程序使用的话）。着色器生成不强制使用特定的语义集，因此对于使用此功能的语言和应用程庁，可以使用任何语义。对于不使用语义的语言，需要使用变量命名约定。

内置着色器生成器和随附的节点实现对着色器变量有命名约定。从内置功能派生并利用它们的自定义着色器生成器最好使用相同的约定。统一变量以 `u_` 为前缀，顶点输入以 `i_` 为前缀。对于不使用语义的语言，图 7 显示了具有预定义绑定规则的变量（输入和统一变量）使用的命名：

应用数据输入变量

| 名称                                | 类型    | 绑定 |
| :---                                | :--:    | :--- |
| i_position                          | vec3    | 对象空间中的顶点位置。 |
| i_normal                            | vec3    | 对象空间中的顶点法线。 |
| i_tangent                           | vec3    | 对象空间中的顶点切线。 |
| i_bitangent                         | vec3    | 对象空间中的顶点副切线。 |
| i_texcoord_N                        | vec2    | 第 N 个 uv 集的顶点纹理坐标。 |
| i_color_N                           | vec4    | 第 N 个颜色集的顶点颜色。 |


统一变量

| 名称                                | 类型    | 绑定 |
| :---                                | :--:    | :--- |
| u_worldMatrix                       | mat4    | 世界变换。 |
| u_worldInverseMatrix                | mat4    | 世界变换，取反。 |
| u_worldTransposeMatrix              | mat4    | 世界变换，转置。 |
| u_worldInverseTransposeMatrix       | mat4    | 世界变换，取反，转置。 |
| u_viewMatrix                        | mat4    | 视图变换。 |
| u_viewInverseMatrix                 | mat4    | 视图变换，取反。 |
| u_viewTransposeMatrix               | mat4    | 视图变换，转置。 |
| u_viewInverseTransposeMatrix        | mat4    | 视图变换，取反，转置。 |
| u_projectionMatrix                  | mat4    | 投影变换。 |
| u_projectionInverseMatrix           | mat4    | 投影变换，取反。 |
| u_projectionTransposeMatrix         | mat4    | 投影变换，转置。 |
| u_projectionInverseTransposeMatrix  | mat4    | 投影变换，取反，转置。 |
| u_worldViewMatrix                   | mat4    | 世界-视图变换。 |
| u_viewProjectionMatrix              | mat4    | 视图-投影变换。 |
| u_worldViewProjectionMatrix         | mat4    | 世界-视图-投影变换。 |
| u_viewPosition                      | vec3    | 观察者的世界空间位置。 |
| u_viewDirection                     | vec3    | 观察者的世界空间方向。 |
| u_frame                             | float   | 主机应用程序定义的当前帧号。 |
| u_time                              | float   | 当前时间（秒）。 |
| u_geomprop_\<name>                  | \<type> | 给定 \<type> 类型的命名属性，其中 \<name> 是几何体上变量的名称。 |
| u_numActiveLightSources             | int     | 当前活动光源的数量。请注意，在着色器中，这被限制为允许的最大光源数量。 |
| u_lightData[]                       | struct  | 包含活动光源参数的 LightData 结构数组。`LightData` 结构根据绑定的光照着色器的要求动态构建。 |
| u_\<unitType>UnitTarget[]           | integer  | 指示给定单元类型定义 (\<unitType>) 的目标单元的属性。 |

**图 7：** 预定义变量及其绑定规则的列表。
