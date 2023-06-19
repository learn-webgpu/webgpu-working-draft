## 3.1. 约定

### 3.1.1. 语法简写

在本规范中，使用以下语法简写：
\*\*点号（"."）语法，常见于编程语言中。 \*\*

短语"Foo.Bar"表示"value（或接口）Foo的成员Bar"。

短语"Foo.Bar is provided"表示"映射值Foo中存在成员Bar"。

**可选链（"?."）语法，源于JavaScript。**

短语"Foo?.Bar"表示"如果 Foo 为 null 或 undefined 或 Foo 中不存在 Bar，则为 undefined；否则为"Foo.Bar"。

例如，如果 buffer 是一个`GPUBuffer`，那么buffer?.\[\[device]].\[\[adapter]]表示"如果 buffer 为 null 或 undefined，则为 undefined；否则为 buffer 的\[\[device]]内部槽的\[\[adapter]]内部槽"。

**空值合并（"??"）语法，源于JavaScript。**

短语"x ?? y"表示"如果 x 不为 null/undefined，则为x；否则为y"。

**由同名内部槽支持的属性**

一个由 WebIDL 属性支持的内部槽。它可能是可变的，也可能是不可变的。

### 3.1.2. WebGPU 接口

WebGPU 接口定义了一个 WebGPU 对象，它可以在以下情况下使用：

*   在创建它的内容时间线上，它是一个 JavaScript 显式的 WebIDL 接口。
*   在所有其他时间线上，只能访问不可变属性。

WebGPU 接口可以定义以下特殊属性类型：

**不可变属性**

在对象初始化时设置的只读槽。它可以从任何时间线访问。

> 注意：由于该槽是不可变的，实现可能在多个时间线上有多个副本。采用这种方式定义不可变属性是为了避免在本规范中描述多个副本。 如果使用 \[\[带括号]] 命名，则它是一个内部槽。如果没有带括号，则它是一个只读的槽支持属性。

**内容时间线属性**

只能从创建对象的内容时间线访问的属性。

如果使用 \[\[带括号]] 命名，则它是一个内部槽。如果没有带括号，则它是一个槽支持属性。

包含 `GPUObjectBase` 的任何接口都是 WebGPU 接口。

```js
interface mixin GPUObjectBase {
    attribute USVString label;
};
```

> 创建新的 WebGPU 对象(`GPUObjectBase` parent, interface T, `GPUObjectDescriptorBase` descriptor)（其中 T 扩展自 `GPUObjectBase`）：
>
> 1.  设设备为 parent.\[\[device]]。
> 2.  让对象成为 T 的新实例。
> 3.  让内部成为 T.\[\[internals]] 的类型的新（未初始化）实例（可能会覆盖 GPUObjectBase.\[\[internals]]），只能从设备时间线访问。
> 4.  将 object.\[\[device]] 设为设备。
> 5.  将 object.\[\[internals]] 设为 internals。
> 6.  将 object.label 设为 descriptor.label。
> 7.  返回 \[object, internals]。

`GPUObjectBase` 具有以下不可变属性：

> \[\[internals]]，类型为内部对象，只读，可重写
>
> *   内部对象。
>
> *   对该对象的内容进行的操作断言它们正在设备时间线上运行，并且设备是有效的。
>
> *   对于每个子类型 `GPUObjectBase` 的接口，可以使用内部对象的子类型进行覆盖。此槽最初设置为该类型的未初始化对象。
>
> \[\[device]]，类型为设备，只读
>
> *   拥有内部对象的设备。
>
> *   对该对象的内容进行的操作断言它们正在设备时间线上运行，并且设备是有效的。

GPUObjectBase 具有以下内容时间线属性：

> label，类型为 `USVString`
>
> 开发人员提供的标签，以实现定义的方式使用。它可以被浏览器、操作系统或其他工具用于帮助开发人员识别底层内部对象。例如，在 GPUError 消息、控制台警告、浏览器开发工具和平台调试工具中显示标签。
>
> > 实现应该使用标签通过它们来识别 WebGPU 对象来增强错误消息。
> >
> > 但是，这不一定是唯一识别对象的方法：实现还应该使用其他可用信息，特别是当没有标签时。例如：
> >
> > *   在打印 `GPUTextureView` 时，使用父 `GPUTexture` 的标签。
> > *   在打印 `GPURenderPassEncoder` 或 `GPUComputePassEncoder` 时，使用父 `GPUCommandEncoder` 的标签。
> > *   在打印 `GPUCommandBuffer` 时，使用源 `GPUCommandEncoder` 的标签。
> > *   在打印 `GPURenderBundle` 时，使用源 `GPURenderBundleEncoder` 的标签。
>
> > 注意：标签是 `GPUObjectBase` 的一个属性。即使它们引用相同的底层对象（例如由 `getBindGroupLayout()` 返回的对象），两个 `GPUObjectBase` "包装"对象的标签状态完全分离。标签属性不会更改，除非从 JavaScript 设置。
> >
> > 这意味着一个底层对象可以关联多个标签。本规范未定义标签如何传播到设备时间线。标签使用方式完全由实现定义：错误消息可以显示最近设置的标签、所有已知的标签或根本不显示标签。
> >
> > 它被定义为 `USVString`，因为某些用户代理可能会将其提供给基础本机 API 的调试工具。

> 注意：理想情况下，WebGPU 接口不应该阻止其父对象，例如拥有它们的 \[\[device]）被垃圾回收。然而，不能保证这一点，因为在某些实现中，持有对父对象的强引用可能是必需的。
>
> 因此，开发人员应该假设 WebGPU 接口可能在所有子对象被垃圾回收之前都不能被垃圾回收。这可能会导致一些资源的分配时间比预期的长。
>
> 调用 WebGPU 接口（例如 `GPUDevice.destroy()` 或 `GPUBuffer.destroy()`）上的 destroy 方法应该比依赖垃圾回收更可取，如果需要可预测的分配资源的释放。

### 3.1.3. 内部对象

内部对象跟踪只能在设备时间线上使用的 WebGPU 对象的状态，在设备时间线上的槽中，这些槽可能是可变的。

> **设备时间线槽**
>
> 只能从设备时间线访问的内部槽。

对内部对象的可变状态的所有读/写都是从执行在单个有序设备时间线上的步骤进行的。这些步骤可能来自任何多个代理的内容时间线算法。

注意：一个“代理”指的是一个 JavaScript“线程”（即主线程或 Web Worker）。

### 3.1.4. 对象描述符

对象描述符保存创建对象所需的信息，通常是通过 GPUDevice 的 create\* 方法之一完成的。

```js
dictionary GPUObjectDescriptorBase {  
    USVString label;  
};  
```

GPUObjectDescriptorBase 包含以下成员：

> label，类型为`USVString`，`GPUObjectBase.label` 的初始值。

## 3.2. 异步性

### 3.2.1. 无效的内部对象和传染性无效

WebGPU中的对象创建操作不返回 Promise，但仍然在内部是异步的。返回的对象引用在设备时间轴上操作的内部对象。大多数在设备时间轴上发生的错误不会通过异常或拒绝而是通过在相关设备上生成的GPUErrors进行通信。

内部对象可以是有效的或无效的。无效的对象将永远不会在以后变得有效，但某些有效的对象可能会变得无效。

如果无法创建对象，则从创建开始对象无效。例如，如果对象描述符未描述有效对象，或者没有足够的内存来分配资源，则会发生这种情况。

大多数类型的内部对象在创建后不能变为无效状态，但仍可能变得无法使用，例如如果所有者设备丢失或销毁，或者对象具有特殊的内部状态，如缓冲区状态“已销毁”。

某些类型的内部对象在创建后可能会变为无效状态，具体来说是设备、适配器、GPUCommandBuffers和命令/通道/束编码器。

只有当满足以下要求时，给定的 `GPUObjectBase` 对象才可以与 targetObject 一起使用：

*   对象必须有效
*   对象.\[\[device]]必须有效。
*   对象.\[\[device]]必须等于targetObject.\[\[device]]。

### 3.2.2. Promise顺序

WebGPU中的几个操作返回Promise。

*   [`GPU`](https://www.w3.org/TR/webgpu/#gpu).[`requestAdapter()`](https://www.w3.org/TR/webgpu/#dom-gpu-requestadapter)
*   [`GPUAdapter`](https://www.w3.org/TR/webgpu/#gpuadapter).[`requestDevice()`](https://www.w3.org/TR/webgpu/#dom-gpuadapter-requestdevice)
*   [`GPUAdapter`](https://www.w3.org/TR/webgpu/#gpuadapter).[`requestAdapterInfo()`](https://www.w3.org/TR/webgpu/#dom-gpuadapter-requestadapterinfo)
*   [`GPUDevice`](https://www.w3.org/TR/webgpu/#gpudevice).[`createComputePipelineAsync()`](https://www.w3.org/TR/webgpu/#dom-gpudevice-createcomputepipelineasync)
*   [`GPUDevice`](https://www.w3.org/TR/webgpu/#gpudevice).[`createRenderPipelineAsync()`](https://www.w3.org/TR/webgpu/#dom-gpudevice-createrenderpipelineasync)
*   [`GPUBuffer`](https://www.w3.org/TR/webgpu/#gpubuffer).[`mapAsync()`](https://www.w3.org/TR/webgpu/#dom-gpubuffer-mapasync)
*   [`GPUShaderModule`](https://www.w3.org/TR/webgpu/#gpushadermodule).[`getCompilationInfo()`](https://www.w3.org/TR/webgpu/#dom-gpushadermodule-getcompilationinfo)
*   [`GPUQueue`](https://www.w3.org/TR/webgpu/#gpuqueue).[`onSubmittedWorkDone()`](https://www.w3.org/TR/webgpu/#dom-gpuqueue-onsubmittedworkdone)
*   [`GPUDevice`](https://www.w3.org/TR/webgpu/#gpudevice).[`lost`](https://www.w3.org/TR/webgpu/#dom-gpudevice-lost)
*   [`GPUDevice`](https://www.w3.org/TR/webgpu/#gpudevice).[`popErrorScope()`](https://www.w3.org/TR/webgpu/#dom-gpudevice-poperrorscope)

WebGPU不对这些Promise解决（解决或拒绝）的顺序做出任何保证，除非满足以下条件：

> *   如果在p2 = q.`onSubmittedWorkDone()`之前调用p1 = b.`mapAsync()`，并且b最后仅在q上使用，则p2不能在p1解决之前解决。

应用程序不得依赖任何其他Promise解决顺序。

### 3.3. 坐标系统

*   在标准化设备坐标系（NDC）中，Y轴向上：NDC中的点（-1.0，-1.0）位于NDC的左下角。此外，NDC 中的 x 和 y 应在 -1.0 和 1.0 之间（包括两端），而NDC中的 z 应在 0.0 和 1.0 之间（包括两端）。在 NDC 范围之外的顶点不会引入任何错误，但它们将被剪切。

*   在帧缓冲区坐标、视口坐标和片元/像素坐标中，Y轴向下：原点（0,0）位于这些坐标系的左上角。

*   窗口/呈现坐标与帧缓冲区坐标匹配。

*   纹理坐标中原点（0,0）的UV表示纹理内存中的第一个texel（最低字节）。

> 注意：WebGPU 的坐标系统与 DirectX 的坐标系统匹配。

## 3.4. 编程模型

### 3.4.1. 时间轴

本节非规范性。

具有用户代理前端和GPU后端的计算机系统具有在不同时间轴上并行工作的组件：

> **内容时间轴**\
> 与Web脚本执行相关联。它包括调用本规范中描述的所有方法。
>
> 要从`GPUDevice`设备上的操作中向内容时间轴发出步骤，请为`GPUDevice`设备排队一个全局任务，其中包含这些步骤。
>
> **设备时间轴**
>
> 与由用户代理发出的GPU设备操作相关联。它包括适配器、设备和GPU资源和状态对象的创建，这些操作从控制GPU的用户代理部分的角度来看通常是同步的，但可以存在于单独的操作系统进程中。
>
> **队列时间轴**
>
> 与GPU的计算单元上的操作执行相关联。它包括实际在GPU上运行的绘制、复制和计算作业。

> 例子 1
>
> 以下显示了与每个时间轴相关的步骤和值的样式。这种样式是非规范性的；规范文本始终描述关联。
>
> **不可变值示例定义**：可用于任何时间轴。
>
> **内容时间轴示例定义**：只能在内容时间轴上使用。
>
> **设备时间轴示例定义**：只能在设备时间轴上使用。
>
> **队列时间轴示例定义**：只能在队列时间轴上使用。
>
> 在内容时间轴上执行的步骤如下所示：\
> 不可变值示例定义。内容时间轴示例定义。
>
> 在设备时间轴上执行的步骤如下所示：
> 不可变值示例定义。设备时间轴示例定义。
>
> 在队列时间轴上执行的步骤如下所示：
> 不可变值示例定义。队列时间轴示例定义。

在本规范中，当结果值取决于在内容时间轴以外的任何时间轴上发生的工作时，使用异步操作。它们在JavaScript中表示为回调和Promise。

> 例子 2
>
> `GPUComputePassEncoder.dispatchWorkgroups()`:
>
> 1.  用户通过调用`GPUComputePassEncoder`的方法对dispatchWorkgroups命令进行编码，该方法在内容时间轴上执行。
>
> 2.  用户通过调用`GPUQueue.submit()`发出`GPUCommandBuffer`，并将其交给用户代理，在设备时间轴上处理它，通过调用OS驱动程序进行低级提交。
>
> 3.  提交由GPU调用调度程序分派到实际的计算单元以执行，在队列时间轴上执行。

> 例子 3
>
> `GPUDevice.createBuffer()`:
>
> 1.  用户填写`GPUBufferDescriptor`并使用它创建`GPUBuffer`，该操作在内容时间轴上执行。
>
> 2.  用户代理在设备时间轴上创建低级缓冲区。

> 例子 4
>
> `GPUBuffer.mapAsync()`:
>
> 1.  用户在内容时间轴上请求映射`GPUBuffer`，并返回一个Promise。
>
> 2.  用户代理检查缓冲区当前是否被GPU使用，并在此使用完成后提醒自己回来检查。
>
> 3.  在GPU在队列时间轴上操作缓冲区后，用户代理将其映射到内存并解析Promise。

### 3.4.2. 内存模型

本节非规范性。

一旦在应用程序初始化过程中获得了`GPUDevice`，我们可以将WebGPU平台描述为包含以下层：

1.  实现规范的用户代理。

2.  具有该设备的低级本地API驱动程序的操作系统。

3.  实际的CPU和GPU硬件。

WebGPU平台的每个层可能具有不同的内存类型，用户代理在实现规范时需要考虑这一点：

*   由脚本创建的脚本拥有的内存，例如`ArrayBuffer`，通常不可被GPU驱动程序访问。

*   用户代理可能有不同的进程负责运行内容和与GPU驱动程序通信。在这种情况下，它使用进程间共享内存来传输数据。

*   专用GPU具有高带宽的自己的内存，而集成GPU通常与系统共享内存。

大多数物理资源都分配在内存类型中，对于GPU计算或渲染是有效的。当用户需要向GPU提供新数据时，该数据可能需要首先跨越进程边界，以达到与GPU驱动程序通信的用户代理部分。然后它可能需要对驱动程序可见，这有时需要将其复制到由驱动程序分配的暂存内存中。最后，它可能需要传输到专用GPU内存中，可能会将内部布局更改为最适合GPU操作的布局。

所有这些转换都由用户代理的WebGPU实现完成。

> 注意：此示例描述了最坏情况，而在实践中，实现可能不需要跨越进程边界，或者可能能够直接向用户公开由驱动程序管理的内存（在ArrayBuffer后面），从而避免任何数据复制。

### 3.4.3. 资源用法

物理资源可以在GPU上使用，具有内部用法：

> **input**\
> 用于绘制或调度调用的输入数据缓冲区。保留内容。可由缓冲区`INDEX`、缓冲区`VERTEX`或缓冲区`INDIRECT`使用。
>
> **constant**\
> 从着色器的角度看是恒定的资源绑定。保留内容。可由缓冲区`UNIFORM`或纹理`TEXTURE_BINDING`使用。
>
> **storage**\
> 可写存储资源绑定。可由缓冲区`STORAGE`或纹理`STORAGE_BINDING`使用。
>
> **storage-read**\
> 只读存储资源绑定。保留内容。可由缓冲区`STORAGE`使用。
>
> **attachment**\
> 在渲染通道中用作输出附件的纹理。可由纹理`RENDER_ATTACHMENT`使用。
>
> **attachment-read**\
> 在渲染通道中用作只读附件的纹理。保留内容。可由纹理`RENDER_ATTACHMENT`使用。

我们将子资源定义为整个缓冲区或纹理子资源之一。

某些内部用法与其他用法兼容。子资源可以处于将多个用法组合在一起的状态。我们认为列表U是兼容的用法列表，如果（且仅如果）它满足以下任一规则：

*   U中的每个用法都是`input`、`constant`、`storage-read`或`attachment-read`。

*   U中的每个用法都是`storage`。

*   U包含一个元素：`attachment`。

强制使用只能组合成兼容的用法列表，可以使API限制在处理内存时何时可能发生数据竞争。这种属性使针对WebGPU编写的应用程序更有可能在不同的平台上运行而无需修改。

通常，当实现处理使用不同方式使用子资源的操作时，它会安排将资源转换为新状态。在某些情况下，例如在打开的GPURenderPassEncoder内部，由于硬件限制，这样的转换是不可能的。我们将这些地方定义为用法范围。

主要的使用规则是，对于任何一个子资源，在一个用法范围内其内部用法列表必须是兼容的用法列表。

例如，在同一`GPURenderPassEncoder`中将相同的缓冲区绑定为存储和输入会将编码器以及拥有的`GPUCommandEncoder`置为错误状态。这些用法的组合不构成兼容的用法列表。

> 注意：允许在单个用法范围内存在多个可写存储缓冲区/纹理用法的竞争条件。

包含在`GPURenderPassColorAttachment.view`和`GPURenderPassColorAttachment.resolveTarget`提供的视图中的纹理的子资源被视为用于此渲染通道的用法范围的附件。

### 3.4.4. 同步

对于物理资源的每个子资源，在Queue时间轴上跟踪其内部使用标志集。

在Queue时间轴上，有一个使用范围的有序序列。在每个范围的持续时间内，任何给定子资源的内部使用标志集都是恒定的。子资源可以在使用范围之间的边界处转换到新的用法。

本规范定义以下使用范围：

*   在通道之外（在GPUCommandEncoder中），每个（非状态设置）命令都是一个使用范围（例如，copyBufferToTexture()）。

*   在计算通道中，每个调度命令（`dispatchWorkgroups()`或`dispatchWorkgroupsIndirect()`）是一个使用范围。如果可以通过该命令访问，则子资源在使用范围中“使用”。在调度内，对于由当前`GPUComputePipeline`的\[\[layout]]使用的每个绑定组槽，每个由该绑定组引用的子资源在使用范围内均“使用”。状态设置计算通道命令（例如`setBindGroup()`）不直接contributing到使用范围中；它们改变了在调度命令中检查的状态。

*   一个渲染通道是一个使用范围。如果被任何（状态设置或非状态设置）命令引用，则子资源在使用范围内“使用”。例如，在`setBindGroup()`中，每个绑定组中的每个子资源在渲染通道的使用范围内“使用”。

> 问题 1 上述内容可能应该讨论GPU命令。但我们目前还没有一种引用特定GPU命令（如dispatch）的方法。

> 注意：上述规则意味着以下示例资源用法包含在使用范围验证中：
>
> *   在渲染通道中，用于任何`setBindGroup()`调用的子资源，无论当前绑定的管道的着色器或布局实际上是否依赖于这些绑定，或者绑定组是否被另一个“set”调用所覆盖。
>
> *   用于任何`setVertexBuffer()`调用中的缓冲区，无论任何绘制调用是否依赖于该缓冲区，或者该缓冲区是否被另一个“set”调用所覆盖。
>
> *   用于任何`setIndexBuffer()`调用中的缓冲区，无论任何绘制调用是否依赖于该缓冲区，或者该缓冲区是否被另一个“set”调用所覆盖。
>
> *   由`beginRenderPass()`在`GPURenderPassDescriptor`中用作颜色附件、解析附件或深度/模板附件的纹理子资源，无论着色器实际上是否依赖于这些附件。
>
> *   在可见性为0或仅针对计算阶段可见但在渲染通道中使用（或反之亦然）的绑定组条目中使用的资源。

在命令编码期间，每个子资源的每个使用都记录在命令缓冲区中的一个使用范围中。对于每个使用范围，实现通过组合在使用范围中使用的每个子资源的所有内部使用标志的列表来执行使用范围验证。如果其中任何一个列表不是兼容的使用列表，则`GPUCommandEncoder.finish()`将生成验证错误。

## 3.5. 核心内部对象

### 3.5.1. Adapters

Adapter 标识了系统上 WebGPU 的实现：即基于浏览器的平台上的计算/渲染功能实例，以及在该功能之上的浏览器 WebGPU 实现的实例。

Adapters 并不唯一地表示底层实现：多次调用 `requestAdapter()` 会返回不同的 adapter 对象。

每个 adapter 对象只能用于创建一个 device，在成功的 `requestDevice()` 调用后，adapter 就变为无效。此外，adapter 对象也可能随时过期。

> 注意：这确保应用程序在创建 device 时使用最新的系统状态进行 adapter 选择。这也通过使各种场景看起来相似来鼓励更多的鲁棒性：首次初始化、由于拔掉 adapter 而重新初始化、由于测试 `GPUDevice.destroy()` 调用而重新初始化等。

如果一个 adapter 具有显著的性能缺陷，以换取更广泛的兼容性、更可预测的行为或更好的隐私，则可将其视为后备 adapter。并不要求每个系统都有后备 adapter 可用。

Adapter 具有以下内部 slot：

> \[\[features]]，类型为 ordered set<GPUFeatureName>，只读
>
> *   可用于在此 adapter 上创建设备的特性。
>
> \[\[limits]]，类型为 supported limits，只读
>
> *   可用于在此 adapter 上创建设备的最佳限制。
>
> *   每个 adapter 限制必须与其 supported limits 中的默认值相同或更好。
>
> \[\[fallback]]，类型为 boolean
>
> *   如果设置为 true，表示该 adapter 是后备 adapter。
>
> \[\[unmaskedIdentifiers]]，类型为 ordered set<DOMString>
>
> *   用户代理选择为此 adapter 报告的 GPUAdapterInfo 字段名称列表。最初填充了用户代理选择未经用户同意报告的任何 GPUAdapterInfo 字段的名称。

Adapter 通过 `GPUAdapter` 暴露。

### 3.5.2. Devices

Device 是 adapter 的逻辑实例化，通过它可以创建内部对象。它可以在多个代理之间共享（例如，专用 worker）。

Device 拥有从它创建的所有内部对象的独占所有权：当 device 变为无效（丢失或销毁）时，直接（例如 `createTexture()`）或间接（例如 `createView()`）在其上创建的所有对象都会变得隐式不可用。

Device 具有以下内部 slot：

> \[\[adapter]]，类型为 adapter，只读
>
> *   创建此 device 的 adapter。
>
> \[\[features]]，类型为 ordered set<GPUFeatureName>，只读
>
> *   可在此 device 上使用的特性。即使底层 adapter 可以支持它们，也不能使用其他特性。
>
> \[\[limits]]，类型为 supported limits，只读
>
> *   可在此 device 上使用的限制。即使底层 adapter 可以支持更好的限制，也不能使用更好的限制。

> 当使用 GPUDeviceDescriptor 描述符从 adapter adapter 创建新的设备 device 时：
>
> *   将 device.\[\[adapter]] 设为 adapter。
>
> *   将 device.\[\[features]] 设为 descriptor.requiredFeatures 中的值集合。
>
> *   让 device.\[\[limits]] 成为一个 supported limits 对象，其中包含默认值。对于 descriptor.requiredLimits 中的每个 (key, value) 对，将 device.\[\[limits]] 中对应于 key 的成员设置为 value 的更好值或 supported limits 中的默认值。

任何时候，用户代理需要撤销对设备的访问，就会在设备的 device timeline 上调用 lose the device(device, "unknown")，可能超前于当前在该 timeline 上排队的其他操作。

如果一个操作失败并具有可观察地更改设备上对象状态或潜在地损坏内部实现/驱动程序状态的副作用，就应将该设备失效以防止这些更改被观察到。

> 注意：对于所有非应用程序发起的设备失效（通过 destroy()），用户代理应无条件发出开发者可见的警告，即使已处理了失效的 promise。这些情况应该很少见，并且该信号对开发人员非常重要，因为 WebGPU API 的大多数部分都试图像没有问题一样运行，以避免中断应用程序的运行时流程：不会引发验证错误，大多数 promise 正常解析等等。

> 要失去设备(device, reason)：
>
> 1.  使设备无效。
>
> 2.  让 gpuDevice 成为与 device 对应的 content timeline GPUDevice。
>
> 3.  在 gpuDevice 的内容时间轴上执行以下步骤：
>
> > 1.  使用 reason 设定为 reason 和 message 设定为一个实现定义的值的新 GPUDeviceLostInfo 解析 device.lost。
> >
> > > 注意：message 不应透露不必要的用户/系统信息，也不应该由应用程序解析。
>
> 4.  完成任何未完成的 mapAsync() 步骤。
>
> 5.  完成任何未完成的 onSubmittedWorkDone() 步骤。
>
> > 注意：设备失效后不会生成任何错误。参见 § 22 错误和调试。

Device 通过 GPUDevice 暴露。

## 3.6. 可选功能

WebGPU 适配器和设备具有能力，描述了在不同实现之间不同的 WebGPU 功能，通常由于硬件或系统软件限制。能力要么是特性，要么是限制。

用户代理不能透露超过 32 个可区分的配置或桶。

适配器的能力必须符合 § 4.2.1 适配器能力保证。

只有支持的能力可以在 `requestDevice()` 中请求；请求不支持的能力会导致失败。

设备的能力恰好是在 `requestDevice()` 中请求的能力。无论适配器的能力如何，这些能力都会被强制执行。

有关隐私方面的考虑，请参见 § 2.2.1 机器特定的功能和限制。

### 3.6.1. 特性

特性是一组可选的 WebGPU 功能，不是所有实现都支持，通常由于硬件或系统软件限制。

如果在设备创建时请求了特性（在 requiredFeatures 中），则可以使用特性中包含的功能。否则，通常会导致验证错误，使用现有的 API 表面的新方式，结果如下所示：

*   使用新的方法或枚举值始终会引发 TypeError。

*   使用具有（正确类型的）非默认值的新字典成员通常会导致验证错误。

*   使用新的 WGSL 启用指令始终会导致 `createShaderModule()` 验证错误。

如果 `GPUObjectBase` 对象的 \[\[device]].\[\[features]] 包含特性，则为 `GPUFeatureName` 特性启用 `GPUObjectBase` 对象。有关每个特性启用的功能的说明，请参见特性索引。

### 3.6.2. 限制

每个限制是设备上 WebGPU 使用的数字限制。

每个限制都有一个默认值。每个适配器都保证支持默认值或更好的值。如果在 requiredLimits 中未显式指定值，则使用默认值。

一个限制值可能比另一个更好。一个更好的限制值总是放宽验证，使得更多的程序是有效的。对于每个限制类别，“更好”都有定义。

不同的限制具有不同的**限制类别**：

> ***最大值***
>
> *   该限制对 API 中传递的某些值强制执行最大值。
>
> *   更高的值更好。
>
> *   只能设置为≥默认值的值。较低的值将被夹紧到默认值。
>
> ***对齐***
>
> *   该限制对 API 中传递的某些值强制执行最小对齐方式；即，该值必须是该限制的倍数。
>
> *   较低的值更好。
>
> *   只能设置为默认值以下的 2 的幂次方。不是 2 的幂次方的值是无效的。较高的 2 的幂次方将被夹紧到默认值。

> 注意：设置“更好”的限制可能不一定是理想的，因为它们可能会对性能产生影响。因此，为了提高跨设备和实现的可移植性，应用程序通常应请求适合其内容的“最差”限制（理想情况下，是默认值）。

受支持的限制对象具有 WebGPU 定义的每个限制的值：

| 限制名称                                                                                                                                                | 类型          | [限制类](https://www.w3.org/TR/webgpu/#limit-class)                 | [默认值](https://www.w3.org/TR/webgpu/#limit-default) |
| --------------------------------------------------------------------------------------------------------------------------------------------------- | ----------- | ---------------------------------------------------------------- | -------------------------------------------------- |
| `maxTextureDimension1D`[](https://www.w3.org/TR/webgpu/#dom-supported-limits-maxtexturedimension1d)                                                 | `GPUSize32` | [maximum](https://www.w3.org/TR/webgpu/#limit-class-maximum)     | 8192                                               |
| 使用 "1d" 维度创建的纹理的 size.width 允许的最大值                                                                                                                  |             |                                                                  |                                                    |
| `maxTextureDimension2D`                                                                                                                             | `GPUSize32` | [maximum](https://www.w3.org/TR/webgpu/#limit-class-maximum)     | 8192                                               |
| 使用尺寸“2d”创建的纹理的size. wide和size.高的最大允许值                                                                                                               |             |                                                                  |                                                    |
| `maxTextureDimension3D`[](https://www.w3.org/TR/webgpu/#dom-supported-limits-maxtexturedimension3d)                                                 | `GPUSize32` | [maximum](https://www.w3.org/TR/webgpu/#limit-class-maximum)     | 2048                                               |
| 使用尺寸“3d”创建的纹理的size. wide、size.高和size.DepthOrArrayLayers的最大允许值                                                                                       |             |                                                                  |                                                    |
| `maxTextureArrayLayers`[](https://www.w3.org/TR/webgpu/#dom-supported-limits-maxtexturearraylayers)                                                 | `GPUSize32` | [maximum](https://www.w3.org/TR/webgpu/#limit-class-maximum)     | 256                                                |
| 使用维度“2d”创建的纹理的size. DepthOrArrayLayers的最大允许值                                                                                                        |             |                                                                  |                                                    |
| `maxBindGroups`                                                                                                                                     | `GPUSize32` | [maximum](https://www.w3.org/TR/webgpu/#limit-class-maximum)     | 4                                                  |
| 创建GPUPipelineLayout时bindGroupLayout中允许的最大GPUBindGroupLayout数量                                                                                       |             |                                                                  |                                                    |
| `maxBindGroupsPlusVertexBuffers`                                                                                                                    | `GPUSize32` | [maximum](https://www.w3.org/TR/webgpu/#limit-class-maximum)     | 24                                                 |
| 同时使用的绑定组和顶点缓冲区插槽的最大数量，计算最高索引下方的任何空插槽。在createRenderPipeline（）和绘制调用中验证                                                                                |             |                                                                  |                                                    |
| `maxBindingsPerBindGroup`                                                                                                                           | `GPUSize32` | [maximum](https://www.w3.org/TR/webgpu/#limit-class-maximum)     | 1000                                               |
| 创建GPUBindGroupLayout时可用的绑定索引数。注意：此限制是规范的，但任意的。使用默认绑定槽限制，不可能在一个绑定组中使用1000个绑定，但这允许GPUBindGroupLayoutEnter.绑定值高达999。此限制允许实现将绑定空间视为合理内存空间内的数组，而不是稀疏映射结构 |             |                                                                  |                                                    |
| `maxDynamicUniformBuffersPerPipelineLayout`                                                                                                         | `GPUSize32` | [maximum](https://www.w3.org/TR/webgpu/#limit-class-maximum)     | 8                                                  |
| 跨GPUPipelineLayout的GPUBindGroupLayoutEntry条目的最大数量，它们是具有动态偏移的统一缓冲区。请参阅超出绑定槽限制。                                                                       |             |                                                                  |                                                    |
| `maxDynamicStorageBuffersPerPipelineLayout`                                                                                                         | `GPUSize32` | [maximum](https://www.w3.org/TR/webgpu/#limit-class-maximum)     | 4                                                  |
| 跨GPUPipelineLayout的GPUBindGroupLayoutEntry条目的最大数量，它们是具有动态偏移量的存储缓冲区。请参阅超出绑定槽限制。                                                                      |             |                                                                  |                                                    |
| `maxSampledTexturesPerShaderStage`                                                                                                                  | `GPUSize32` | [maximum](https://www.w3.org/TR/webgpu/#limit-class-maximum)     | 16                                                 |
| 对于每个可能的GPUShaderStage阶段，GPUPipelineLayout中作为采样纹理的GPUBindGroupLayoutEntry条目的最大数量。请参阅超出绑定槽限制。                                                         |             |                                                                  |                                                    |
| `maxSamplersPerShaderStage`                                                                                                                         | `GPUSize32` | [maximum](https://www.w3.org/TR/webgpu/#limit-class-maximum)     | 16                                                 |
| 对于每个可能的GPUShaderStage阶段，GPUPipelineLayout中作为采样器的GPUBindGroupLayoutEntry条目的最大数量。请参阅超出绑定槽限制                                                           |             |                                                                  |                                                    |
| `maxStorageBuffersPerShaderStage`                                                                                                                   | `GPUSize32` | [maximum](https://www.w3.org/TR/webgpu/#limit-class-maximum)     | 8                                                  |
| 对于每个可能的GPUShaderStage阶段，GPUPipelineLayout中作为存储缓冲区的GPUBindGroupLayoutEntry条目的最大数量。请参阅超出绑定槽限制。                                                        |             |                                                                  |                                                    |
| `maxStorageTexturesPerShaderStage`                                                                                                                  | `GPUSize32` | [maximum](https://www.w3.org/TR/webgpu/#limit-class-maximum)     | 4                                                  |
| 对于每个可能的GPUShaderStage阶段，GPUPipelineLayout中作为存储纹理的GPUBindGroupLayoutEntry条目的最大数量。请参阅超出绑定槽限制。                                                         |             |                                                                  |                                                    |
| `maxUniformBuffersPerShaderStage`                                                                                                                   | `GPUSize32` | [maximum](https://www.w3.org/TR/webgpu/#limit-class-maximum)     | 12                                                 |
| 对于每个可能的GPUShaderStage阶段，GPUPipelineLayout中作为统一缓冲区的GPUBindGroupLayoutEntry条目的最大数量。请参阅超出绑定槽限制。                                                        |             |                                                                  |                                                    |
| `maxUniformBufferBindingSize`                                                                                                                       | `GPUSize64` | [maximum](https://www.w3.org/TR/webgpu/#limit-class-maximum)     | 65536 bytes                                        |
| 具有GPUBindGroupLayoutEntry条目的绑定的最大GPUBufferBinding. size，其中entry.缓冲区？.type是`"uniform"`                                                               |             |                                                                  |                                                    |
| `maxStorageBufferBindingSize`                                                                                                                       | `GPUSize64` | [maximum](https://www.w3.org/TR/webgpu/#limit-class-maximum)     | 134217728 bytes (128 MiB)                          |
| 具有GPUBindGroupLayoutEntry条目的绑定的最大GPUBufferBinding. size，其中entry.缓冲区？.type是“存储”或“只读存储”。                                                              |             |                                                                  |                                                    |
| `minUniformBufferOffsetAlignment`                                                                                                                   | `GPUSize32` | [alignment](https://www.w3.org/TR/webgpu/#limit-class-alignment) | 256 bytes                                          |
| GPUBufferBinding.偏移所需的对齐方式和setBindGroup（）中提供的动态偏移量，用于与条目的GPUBindGroupLayoutEntry条目进行绑定。缓冲区？. type是`"uniform"`                                       |             |                                                                  |                                                    |
| `minStorageBufferOffsetAlignment`                                                                                                                   | `GPUSize32` | [alignment](https://www.w3.org/TR/webgpu/#limit-class-alignment) | 256 bytes                                          |
| 对于与GPUBindGroupLayoutEntry条目的绑定，GPUBufferBinding.偏移所需的对齐方式和setBindGroup（）中提供的动态偏移量。缓冲区？. type是“存储”或“只读存储”。                                          |             |                                                                  |                                                    |
| `maxVertexBuffers`                                                                                                                                  | `GPUSize32` | [maximum](https://www.w3.org/TR/webgpu/#limit-class-maximum)     | 8                                                  |
| 创建GPURenderPipeline时的最大缓冲区数                                                                                                                         |             |                                                                  |                                                    |
| `maxBufferSize`                                                                                                                                     | `GPUSize64` | [maximum](https://www.w3.org/TR/webgpu/#limit-class-maximum)     | 268435456 bytes (256 MiB)                          |
| 创建GPUBuffer时的最大大小                                                                                                                                   |             |                                                                  |                                                    |
| `maxVertexAttributes`                                                                                                                               | `GPUSize32` | [maximum](https://www.w3.org/TR/webgpu/#limit-class-maximum)     | 16                                                 |
| 创建GPURenderPipeline时跨缓冲区的最大属性总数。                                                                                                                    |             |                                                                  |                                                    |
| `maxVertexBufferArrayStride`                                                                                                                        | `GPUSize32` | [maximum](https://www.w3.org/TR/webgpu/#limit-class-maximum)     | 2048 bytes                                         |
| 创建GPURenderPipeline时允许的最大数组步长                                                                                                                       |             |                                                                  |                                                    |
| `maxInterStageShaderComponents`                                                                                                                     | `GPUSize32` | [maximum](https://www.w3.org/TR/webgpu/#limit-class-maximum)     | 60                                                 |
| 用于阶段间通信（如顶点输出或片段输入）的输入或输出变量的最大允许组件数。                                                                                                                |             |                                                                  |                                                    |
| `maxInterStageShaderVariables`                                                                                                                      | `GPUSize32` | [maximum](https://www.w3.org/TR/webgpu/#limit-class-maximum)     | 16                                                 |
| 阶段间通信（如顶点输出或片段输入）允许的最大进审量或输出变量。                                                                                                                     |             |                                                                  |                                                    |
| `maxColorAttachments`                                                                                                                               | `GPUSize32` | [maximum](https://www.w3.org/TR/webgpu/#limit-class-maximum)     | 8                                                  |
| GPURenderPipelineDescriptor.片段.目标、GPURenderPassDescriptor. colattachments和GPURenderPassLayout.color Format中允许的最大颜色附件数。                              |             |                                                                  |                                                    |
| `maxColorAttachmentBytesPerSample`                                                                                                                  | `GPUSize32` | [maximum](https://www.w3.org/TR/webgpu/#limit-class-maximum)     | 32                                                 |
| 跨所有颜色附件保存一个渲染管道输出数据样本（像素或子像素）所需的最大字节数。                                                                                                              |             |                                                                  |                                                    |
| `maxComputeWorkgroupStorageSize`                                                                                                                    | `GPUSize32` | [maximum](https://www.w3.org/TR/webgpu/#limit-class-maximum)     | 16384 bytes                                        |
| 用于计算阶段GPUShaderModule入口点的工作组存储的最大字节数。                                                                                                               |             |                                                                  |                                                    |
| `maxComputeInvocationsPerWorkgroup`                                                                                                                 | `GPUSize32` | [maximum](https://www.w3.org/TR/webgpu/#limit-class-maximum)     | 256                                                |
| 计算阶段GPUShaderModule入口点的workgroup\_size维度乘积的最大值。                                                                                                     |             |                                                                  |                                                    |
| `maxComputeWorkgroupSizeX`                                                                                                                          | `GPUSize32` | [maximum](https://www.w3.org/TR/webgpu/#limit-class-maximum)     | 256                                                |
| 计算阶段GPUShaderModule入口点的workgroup\_sizeX维的最大值                                                                                                        |             |                                                                  |                                                    |
| `maxComputeWorkgroupSizeY`                                                                                                                          | `GPUSize32` | [maximum](https://www.w3.org/TR/webgpu/#limit-class-maximum)     | 256                                                |
| 计算阶段GPUShaderModule入口点的workgroup\_sizeY维度的最大值。                                                                                                      |             |                                                                  |                                                    |
| `maxComputeWorkgroupSizeZ`                                                                                                                          | `GPUSize32` | [maximum](https://www.w3.org/TR/webgpu/#limit-class-maximum)     | 64                                                 |
| 计算阶段GPUShaderModule入口点的workgroup\_sizeZ维度的最大值。                                                                                                      |             |                                                                  |                                                    |
| `maxComputeWorkgroupsPerDimension`                                                                                                                  | `GPUSize32` | [maximum](https://www.w3.org/TR/webgpu/#limit-class-maximum)     | 65535                                              |
| 调度工作组参数的最大值（workgroupCountX、workgroupCountY、workgroupCountZ）。                                   |             |                                                                  |                                                    |

#### 3.6.2.1. GPUSupportedLimits

`GPUSupportedLimits`接口公开了适配器或设备支持的限制。参见`GPUAdapter.limits`和`GPUDevice.limits`。

```js
[Exposed=(Window, DedicatedWorker), SecureContext]  
    interface GPUSupportedLimits {  
        readonly attribute unsigned long maxTextureDimension1D;  
        readonly attribute unsigned long maxTextureDimension2D;  
        readonly attribute unsigned long maxTextureDimension3D;  
        readonly attribute unsigned long maxTextureArrayLayers;  
        readonly attribute unsigned long maxBindGroups;  
        readonly attribute unsigned long maxBindGroupsPlusVertexBuffers;  
        readonly attribute unsigned long maxBindingsPerBindGroup;  
        readonly attribute unsigned long maxDynamicUniformBuffersPerPipelineLayout;  
        readonly attribute unsigned long maxDynamicStorageBuffersPerPipelineLayout;  
        readonly attribute unsigned long maxSampledTexturesPerShaderStage;  
        readonly attribute unsigned long maxSamplersPerShaderStage;  
        readonly attribute unsigned long maxStorageBuffersPerShaderStage;  
        readonly attribute unsigned long maxStorageTexturesPerShaderStage;  
        readonly attribute unsigned long maxUniformBuffersPerShaderStage;  
        readonly attribute unsigned long long maxUniformBufferBindingSize;  
        readonly attribute unsigned long long maxStorageBufferBindingSize;  
        readonly attribute unsigned long minUniformBufferOffsetAlignment;  
        readonly attribute unsigned long minStorageBufferOffsetAlignment;  
        readonly attribute unsigned long maxVertexBuffers;  
        readonly attribute unsigned long long maxBufferSize;  
        readonly attribute unsigned long maxVertexAttributes;  
        readonly attribute unsigned long maxVertexBufferArrayStride;  
        readonly attribute unsigned long maxInterStageShaderComponents;  
        readonly attribute unsigned long maxInterStageShaderVariables;  
        readonly attribute unsigned long maxColorAttachments;  
        readonly attribute unsigned long maxColorAttachmentBytesPerSample;  
        readonly attribute unsigned long maxComputeWorkgroupStorageSize;  
        readonly attribute unsigned long maxComputeInvocationsPerWorkgroup;  
        readonly attribute unsigned long maxComputeWorkgroupSizeX;  
        readonly attribute unsigned long maxComputeWorkgroupSizeY;  
        readonly attribute unsigned long maxComputeWorkgroupSizeZ;  
        readonly attribute unsigned long maxComputeWorkgroupsPerDimension;  
};
```

#### 3.6.2.2. GPUSupportedFeatures

`GPUSupportedFeatures`是一个类似集合的接口。其集合条目是适配器或设备支持的`GPUFeatureName`值。它只能包含`GPUFeatureName`枚举中的字符串。

```js
[Exposed=(Window, DedicatedWorker), SecureContext]  
interface GPUSupportedFeatures {  
    readonly setlike<DOMString>;  
};
```

> 注：`GPUSupportedFeatures`集合条目的类型为`DOMString`，以允许用户代理优雅地处理在规范的后续修订中添加的有效`GPUFeatureNames`，但用户代理尚未更新以识别这些名称。如果集合条目类型为`GPUFeatureName`，则以下代码会抛出TypeError而不是报告false：
>
> > 检查是否支持未识别的特性：
> >
> > ```js
> > if (adapter.features.has('unknown-feature')) {  
> >     // 使用unknown-feature  
> > } else {  
> >     console.warn('unknown-feature在该适配器中不受支持。');  
> > }
> > ```

#### 3.6.2.3. WGSLLanguageFeatures

`WGSLLanguageFeatures`是一个类似集合的接口。其集合条目是所有适配器支持的WGSL语言扩展的字符串名称。

```js
[Exposed=(Window, DedicatedWorker), SecureContext]  
interface WGSLLanguageFeatures {  
    readonly setlike<DOMString>;  
};
```

#### 3.6.2.4. GPUAdapterInfo

`GPUAdapterInfo`暴露了关于显卡适配器的各种标识信息。

`GPUAdapterInfo`的所有成员都不能保证被填充。在用户代理器的决定下，哪些值被公开是不确定的，在某些设备上可能没有任何值被填充。因此，应用程序必须能够处理任何可能的`GPUAdapterInfo`值，包括缺少这些值的情况。

有关隐私方面的考虑，请参见§2.2.6适配器标识符。

```js
[Exposed=(Window, DedicatedWorker), SecureContext]  
interface GPUAdapterInfo {  
    readonly attribute DOMString vendor;  
    readonly attribute DOMString architecture;  
    readonly attribute DOMString device;  
    readonly attribute DOMString description;  
};
```

`GPUAdapterInfo`具有以下属性：

> **vendor**，类型为DOMString，只读\
> 如果可用，则为适配器的供应商名称。否则为空字符串。
>
> **architecture**，类型为DOMString，只读\
> 如果可用，则为适配器所属的GPU家族或类的名称。否则为空字符串。
>
> **device**，类型为DOMString，只读\
> 如果可用，则为适配器的供应商特定标识符。否则为空字符串。
>
> 注意：这是表示适配器类型的值。例如，它可能是PCI设备ID。它不能像序列号一样唯一地标识给定的硬件。
>
> **description**，类型为DOMString，只读\
> 如果可用，则为驱动程序报告的适配器的人类可读字符串描述。否则为空字符串。
>
> 注意：因为没有对描述应用任何格式，所以不建议尝试解析此值。根据GPUAdapterInfo更改其行为的应用程序（例如，为已知的驱动程序问题应用临时解决方案）应尽可能依赖其他字段。

> 要为给定的适配器创建新的适配器信息，请执行以下步骤：
>
> 1.  创建一个新的`GPUAdapterInfo`，命名为`adapterInfo`。
>
> 2.  让unmaskedValues成为adapter.\[\[unmaskedIdentifiers]]。
>
> 3.  如果unmaskedValues包含“vendor”并且供应商已知：
>
> > 1.  将adapterInfo.vendor设置为适配器供应商的名称，作为规范化的标识符字符串。
>
> 否则：
>
> > 1.  将adapterInfo.vendor设置为空字符串或作为供应商的合理近似值作为规范化的标识符字符串。
>
> 4.  如果unmaskedValues包含“architecture”并且架构已知：
>
> > 1.  将adapterInfo.architecture设置为表示适配器所属适配器系列或类的规范化标识符字符串。
>
> 否则：
>
> > 1.  将adapterInfo.architecture设置为空字符串或作为架构的合理近似值作为规范化的标识符字符串。
>
> 5.  如果unmaskedValues包含“device”并且设备已知：
>
> > 1.  将adapterInfo.device设置为表示适配器供应商特定标识符的规范化标识符字符串。
>
> 否则：
>
> > 1.  将adapterInfo.device设置为空字符串或作为供应商特定标识符的合理近似值作为规范化的标识符字符串。
>
> 6.  如果unmaskedValues包含“description”并且描述已知：
>
> > 1.  将adapterInfo.description设置为驱动程序报告的适配器描述。
>
> 否则：
>
> > 1.  将adapterInfo.description设置为空字符串或作为描述的合理近似值。
>
> 7.  返回adapterInfo。

> 规范化的标识符字符串遵循以下模式：\
> \[a-z0-9]+(-\[a-z0-9]+)\*
>
> > 例子 6
> >
> > 有效的规范化标识符字符串示例包括：
> >
> > *   gpu
> >
> > *   3d
> >
> > *   0x3b2f
> >
> > *   next-gen
> >
> > *   series-x20-ultra

## 3.7. 扩展文档

“扩展文档”是描述新功能的附加文档，这些功能不是WebGPU / WGSL规范的一部分，是构建在这些规范之上的功能，通常包括一个或多个新API特性标志和/或WGSL启用指令，或与其他草案Web规范的交互。

WebGPU实现不能公开扩展功能；这样做是规范违规行为。新功能只有在被集成到WebGPU规范（本文档）和/或WGSL规范中后才成为WebGPU标准的一部分。

## 3.8. 来源限制

WebGPU允许访问存储在图像、视频和画布中的图像数据。由于着色器可用于间接推断已上传到GPU的纹理的内容，因此对跨域媒体的使用施加了限制。

如果图像源不是来源清洁的，则WebGPU不允许上传图像源。

这也意味着使用WebGPU渲染的画布的来源清洁标志永远不会设置为false。

有关针对图像和视频元素发出CORS请求的更多信息，请参见：

*   HTML § 2.5.4 CORS设置属性

*   HTML § 4.8.3 img元素img

*   HTML § 4.8.11 媒体元素HTMLMediaElement

## 3.9. 任务来源

### 3.9.1. WebGPU任务来源

WebGPU定义了一个名为WebGPU任务来源的新任务来源。它用于uncapturederror事件和GPUDevice.lost。

> 要在GPUDevice设备上排队全局任务，执行以下步骤：
>
> 1.  在WebGPU任务源上排队全局任务，使用用于创建设备的全局对象和步骤。

### 3.9.2. 自动到期任务来源

WebGPU定义了一个名为自动到期任务来源的新任务来源。它用于特定对象的自动定时到期（销毁）：

*   从getCurrentTexture（）返回的GPUTextures

*   从HTMLVideoElements创建的GPUExternalTextures

> 要使用GPUDevice设备和一系列步骤steps排队自动到期任务，请执行以下步骤：
>
> 1.  在自动到期任务源上排队全局任务，使用用于创建设备的全局对象和步骤。

自动到期任务源的任务应该具有高优先级；特别是，一旦排队，它们应该在用户定义的（JavaScript）任务之前运行。

> 注意：此行为更可预测，严格性有助于开发人员通过及早检测可能难以检测到的隐式生命周期的错误假设来编写更可移植的应用程序。仍强烈建议开发人员在多个实现中进行测试。\
> 实现说明：可以通过在事件循环处理模型内的固定点插入额外的步骤而不是运行实际任务来实现高优先级到期“任务”。

## 3.10. 颜色空间与编码

WebGPU 不提供颜色管理。WebGPU 中的所有值（例如纹理元素）都是原始数值，而不是经过颜色管理的颜色值。

WebGPU 与颜色管理输出（通过 GPUCanvasConfiguration）和输入（通过 copyExternalImageToTexture() 和 importExternalTexture()）进行接口。因此，必须在 WebGPU 数值和外部颜色值之间进行颜色转换。每个这样的接口点都在本地定义了一种编码（颜色空间、传输函数和 alpha 预乘），其中 WebGPU 数值应该被解释。

WebGPU 允许在 PredefinedColorSpace 枚举中的所有颜色空间。注意，每个颜色空间都在引用的 CSS 定义中定义了一个扩展范围，以表示其空间外的颜色值（无论在色度还是亮度上）。

超出色域的预乘 RGBA 值是指任何 R/G/B 通道值超过 alpha 通道值的值。例如，预乘 sRGB RGBA 值 \[1.0，0，0，0.5] 表示（未预乘）颜色 \[2，0，0]，带有 50% 的 alpha，在 CSS 中写为 rgb(srgb 2 0 0 / 50%)。就像任何 sRGB 色域外的颜色值一样，这是扩展颜色空间中的一个定义明确的点（除非 alpha 为 0，在这种情况下没有颜色）。但是，当这些值输出到可见画布时，结果是未定义的（请参见 GPUCanvasAlphaMode "premultiplied"）。

### 3.10.1. 颜色空间转换

通过将一个颜色的表示从一个空间转换为另一个空间，根据上述定义进行翻译。

如果源值的 RGBA 通道少于 4 个，则在进行颜色空间/编码和 alpha 预乘之前，缺少的绿/蓝/alpha 通道将分别设置为 0、0、1。转换后，如果目标需要少于 4 个通道，则忽略额外的通道。

> 注意：灰度图像通常用 RGB 值（V，V，V）或 RGBA 值（V，V，V，A）表示其颜色空间。

在转换过程中，颜色不会被压缩丢失：如果源颜色值超出目标颜色空间的色域范围，则从一个颜色空间转换到另一个颜色空间将导致值超出范围\[0，1]。例如，如果源是 rgba16float，在像 Display-P3 这样的更广泛的颜色空间中或者是预乘的并包含超出色域的值，则可能会发生这种情况。

同样，如果源值具有高位深度（例如每个分量具有 16 位的 PNG）或扩展范围（例如具有 float16 存储的画布），则这些颜色将通过颜色空间转换保留，中间计算具有至少源的精度。

### 3.10.2. 颜色空间转换简化

如果颜色空间/编码转换的源和目标相同，则不需要进行转换。通常情况下，如果转换的任何给定步骤是一个恒等函数（no-op），实现应该省略它，以提高性能。

为了获得最佳性能，应用程序应设置其颜色空间和编码选项，以使整个过程中所需的转换数量最小化。对于 GPUImageCopyExternalImage 的各种图像源：

*   ImageBitmap：

    *   通过 premultiplyAlpha 控制。

    *   通过 colorSpaceConversion 控制。

*   2d canvas：

    *   始终预乘。

    *   颜色空间通过 colorSpace 上下文创建属性控制。

*   WebGL canvas：

    *   预乘度通过 WebGLContextAttributes 中的 premultipliedAlpha 选项控制。

    *   颜色空间通过 WebGLRenderingContext 的 drawingBufferColorSpace 状态控制。

> 注意：在依赖于这些功能之前，请检查浏览器实现支持情况。

### 3.11. 从 JavaScript 到 WGSL 的数值转换

WebGPU API的几个部分（可覆盖的管道常量和渲染通道清除值）接受来自WebIDL的数值（double或float），并将它们转换为WGSL值（bool、i32、u32、f32、f16）。

将类型为 double 或 float 的IDL值 idlValue 转换为WGSL类型 T，可能会引发 TypeError：

> 注意：这个 TypeError 是在设备时间轴上生成的，不会显露给JavaScript。

1.  断言 idlValue 是一个有限的值，因为它不是无限制的 double 或无限制的 float 。

2.  让 v 成为从 ! 将 idlValue 转换为ECMAScript值得到的 ECMAScript 数字。

3.  **如果 T 是 bool**

    返回与将 v 转换为类型为 boolean 的IDL值的结果的 WGSL bool 值相对应的值。

    > 注意：此算法在将ECMAScript值转换为IDL double或float值之后调用。如果原始的 ECMAScript 值是一个非数值、非布尔值，例如 \[] 或 {}，那么 WGSL bool 的结果可能与将 ECMAScript 值直接转换为IDL boolean 时的结果不同。

    **如果 T 是 i32**

    返回与将 v 转换为类型为 \[EnforceRange] long 的IDL值的结果相对应的 WGSL i32 值。

    **如果 T 是 u32**

    返回与将 v 转换为类型为 \[EnforceRange] unsigned long 的IDL值的结果相对应的 WGSL u32 值。

    **如果 T 是 f32**

    返回与将 v 转换为类型为 float 的IDL值的结果相对应的 WGSL f32 值。

    **如果 T 是 f16**

    1.  让 wgslF32 成为与将 v 转换为类型为 float 的IDL值的结果相对应的 WGSL f32 值。

    2.  返回 f16(wgslF32)，即将 WGSL f32 值转换为 f16 的结果，其定义在 WGSL 浮点数转换中。

    > 注意：只要值在 f32 的范围内，即使该值超出了 f16 的范围，也不会引发错误。

将 GPUColor 颜色转换为纹理格式 format 的纹素值，可能会引发 TypeError：

> 注意：这个 TypeError 是在设备时间轴上生成的，不会显露给JavaScript。

1.  如果 format 的组件（断言它们都具有相同的类型）是：

    **浮点类型或规范化类型**

    让 T 成为 f32。

    **有符号整数类型**

    让 T 成为 i32。

    **无符号整数类型**

    让 T 成为 u32。

2.  让 wgslColor 成为类型为 vec4<T> 的 WGSL 值，其中的 4 个分量是颜色的 RGBA 通道，每个通道都将被转换为 WGSL 类型 T。

3.  使用与 23.3.7 节输出合并步骤相同的转换规则将 wgslColor 转换为 format，并返回结果。

    > 注意：对于非整数类型，具体选择的值是实现定义的。对于规范化类型，该值将被夹紧到类型的范围内。

> 注意：换句话说，写入的值将与一个输出表示为 f32、i32 或 u32 的 vec4 的 WGSL 着色器写入的值相同。
