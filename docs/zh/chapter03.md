## 3.1. 约定
### 3.1.1. 语法简写
在本规范中，使用以下语法简写： 
**点号（"."）语法，常见于编程语言中。 **

短语"Foo.Bar"表示"value（或接口）Foo的成员Bar"。

短语"Foo.Bar is provided"表示"映射值Foo中存在成员Bar"。

**可选链（"?."）语法，源于JavaScript。** 

短语"Foo?.Bar"表示"如果Foo为null或undefined或Foo中不存在Bar，则为undefined；否则为Foo.Bar"。

例如，如果buffer是一个GPUBuffer，那么buffer?.[[device]].[[adapter]]表示"如果buffer为null或undefined，则为undefined；否则为buffer的[[device]]内部槽的[[adapter]]内部槽"。 

**空值合并（"??"）语法，源于JavaScript。**

短语"x ?? y"表示"如果x不为null/undefined，则为x；否则为y"。 

**由同名内部槽支持的属性** 

一个由WebIDL属性支持的内部槽。它可能是可变的，也可能是不可变的。

### 3.1.2. WebGPU 接口
WebGPU 接口定义了一个 WebGPU 对象，它可以在以下情况下使用：

-   在创建它的内容时间线上，它是一个 JavaScript 显式的 WebIDL 接口。
-   在所有其他时间线上，只能访问不可变属性。 

WebGPU 接口可以定义以下特殊属性类型：

**不可变属性** 

在对象初始化时设置的只读槽。它可以从任何时间线访问。

> 注意：由于该槽是不可变的，实现可能在多个时间线上有多个副本。采用这种方式定义不可变属性是为了避免在本规范中描述多个副本。 如果使用 [[带括号]] 命名，则它是一个内部槽。如果没有带括号，则它是一个只读的槽支持属性。 

**内容时间线属性** 

只能从创建对象的内容时间线访问的属性。 

如果使用 [[带括号]] 命名，则它是一个内部槽。如果没有带括号，则它是一个槽支持属性。 

包含 GPUObjectBase 的任何接口都是 WebGPU 接口。

```js
interface mixin GPUObjectBase {
    attribute USVString label;
};
```

> 创建新的 WebGPU 对象(GPUObjectBase parent, interface T, GPUObjectDescriptorBase descriptor)（其中 T 扩展自 GPUObjectBase）：
> 
> 1.  设设备为 parent.[[device]]。
> 2.  让对象成为 T 的新实例。
> 3.  让内部成为 T.[[internals]] 的类型的新（未初始化）实例（可能会覆盖 GPUObjectBase.[[internals]]），只能从设备时间线访问。
> 4.  将 object.[[device]] 设为设备。
> 5.  将 object.[[internals]] 设为 internals。
> 6.  将 object.label 设为 descriptor.label。
> 7.  返回 [object, internals]。 

GPUObjectBase 具有以下不可变属性： 

[[internals]]，类型为内部对象，只读，可重写 
内部对象。 

对该对象的内容进行的操作断言它们正在设备时间线上运行，并且设备是有效的。 

对于每个子类型 GPUObjectBase 的接口，可以使用内部对象的子类型进行覆盖。此槽最初设置为该类型的未初始化对象。 

[[device]]，类型为设备，只读 

拥有内部对象的设备。

对该对象的内容进行的操作断言它们正在设备时间线上运行，并且设备是有效的。 


GPUObjectBase 具有以下内容时间线属性： 

label，类型为 USVString 

开发人员提供的标签，以实现定义的方式使用。它可以被浏览器、操作系统或其他工具用于帮助开发人员识别底层内部对象。例如，在 GPUError 消息、控制台警告、浏览器开发工具和平台调试工具中显示标签。 

实现应该使用标签通过它们来识别 WebGPU 对象来增强错误消息。

但是，这不一定是唯一识别对象的方法：实现还应该使用其他可用信息，特别是当没有标签时。例如：

-   在打印 GPUTextureView 时，使用父 GPUTexture 的标签。
-   在打印 GPURenderPassEncoder 或 GPUComputePassEncoder 时，使用父 GPUCommandEncoder 的标签。
-   在打印 GPUCommandBuffer 时，使用源 GPUCommandEncoder 的标签。
-   在打印 GPURenderBundle 时，使用源 GPURenderBundleEncoder 的标签。

注意：标签是 GPUObjectBase 的一个属性。即使它们引用相同的底层对象（例如由 getBindGroupLayout() 返回的对象），两个 GPUObjectBase "包装"对象的标签状态完全分离。标签属性不会更改，除非从 JavaScript 设置。 

这意味着一个底层对象可以关联多个标签。本规范未定义标签如何传播到设备时间线。标签使用方式完全由实现定义：错误消息可以显示最近设置的标签、所有已知的标签或根本不显示标签。 

它被定义为 USVString，因为某些用户代理可能会将其提供给基础本机 API 的调试工具。 

注意：理想情况下，WebGPU 接口不应该阻止其父对象，例如拥有它们的 [[device]）被垃圾回收。然而，不能保证这一点，因为在某些实现中，持有对父对象的强引用可能是必需的。

因此，开发人员应该假设 WebGPU 接口可能在所有子对象被垃圾回收之前都不能被垃圾回收。这可能会导致一些资源的分配时间比预期的长。 

调用 WebGPU 接口（例如 GPUDevice.destroy() 或 GPUBuffer.destroy()）上的 destroy 方法应该比依赖垃圾回收更可取，如果需要可预测的分配资源的释放。 

### 3.1.3. 内部对象
内部对象跟踪只能在设备时间线上使用的 WebGPU 对象的状态，在设备时间线上的槽中，这些槽可能是可变的。 

> **设备时间线槽** 
> 
> 只能从设备时间线访问的内部槽。 
    
对内部对象的可变状态的所有读/写都是从执行在单个有序设备时间线上的步骤进行的。这些步骤可能来自任何多个代理的内容时间线算法。 
 
注意：一个“代理”指的是一个 JavaScript“线程”（即主线程或 Web Worker）。

### 3.1.4. 对象描述符
对象描述符保存创建对象所需的信息，通常是通过 GPUDevice 的 create* 方法之一完成的。  
```js
dictionary GPUObjectDescriptorBase {  
    USVString label;  
};  
```
GPUObjectDescriptorBase 包含以下成员：  
> label，类型为USVString，GPUObjectBase.label 的初始值。

## 3.2. 异步性
### 3.2.1. 无效的内部对象和传染性无效
WebGPU中的对象创建操作不返回Promise，但仍然在内部是异步的。返回的对象引用在设备时间轴上操作的内部对象。大多数在设备时间轴上发生的错误不会通过异常或拒绝而是通过在相关设备上生成的GPUErrors进行通信。

内部对象可以是有效的或无效的。无效的对象将永远不会在以后变得有效，但某些有效的对象可能会变得无效。

如果无法创建对象，则从创建开始对象无效。例如，如果对象描述符未描述有效对象，或者没有足够的内存来分配资源，则会发生这种情况。

大多数类型的内部对象在创建后不能变为无效状态，但仍可能变得无法使用，例如如果所有者设备丢失或销毁，或者对象具有特殊的内部状态，如缓冲区状态“已销毁”。

某些类型的内部对象在创建后可能会变为无效状态，具体来说是设备、适配器、GPUCommandBuffers和命令/通道/束编码器。

只有当满足以下要求时，给定的 GPUObjectBase 对象才可以与 targetObject 一起使用：  
* 对象必须有效
* 对象.[[device]]必须有效。
* 对象.[[device]]必须等于targetObject.[[device]]。

### 3.2.2. Promise顺序
WebGPU中的几个操作返回Promise。

* GPU.requestAdapter()

* GPUAdapter.requestDevice()

* GPUAdapter.requestAdapterInfo()

* GPUDevice.createComputePipelineAsync()

* GPUDevice.createRenderPipelineAsync()

* GPUBuffer.mapAsync()

* GPUShaderModule.getCompilationInfo()

* GPUQueue.onSubmittedWorkDone()

* GPUDevice.lost

* GPUDevice.popErrorScope()

WebGPU不对这些Promise解决（解决或拒绝）的顺序做出任何保证，除非满足以下条件：

* 如果在p2 = q.onSubmittedWorkDone()之前调用p1 = b.mapAsync()，并且b最后仅在q上使用，则p2不能在p1解决之前解决。  

应用程序不得依赖任何其他Promise解决顺序。

### 3.3. 坐标系统

* 在标准化设备坐标系（NDC）中，Y轴向上：NDC中的点（-1.0，-1.0）位于NDC的左下角。此外，NDC中的x和y应在-1.0和1.0之间（包括两端），而NDC中的z应在0.0和1.0之间（包括两端）。在NDC范围之外的顶点不会引入任何错误，但它们将被剪切。

* 在帧缓冲区坐标、视口坐标和片元/像素坐标中，Y轴向下：原点（0,0）位于这些坐标系的左上角。

* 窗口/呈现坐标与帧缓冲区坐标匹配。

* 纹理坐标中原点（0,0）的UV表示纹理内存中的第一个texel（最低字节）。

注意：WebGPU的坐标系统与DirectX的坐标系统匹配。

## 3.4. 编程模型
### 3.4.1. 时间轴
本节非规范性。

具有用户代理前端和GPU后端的计算机系统具有在不同时间轴上并行工作的组件：

**内容时间轴**  
与Web脚本执行相关联。它包括调用本规范中描述的所有方法。

要从GPUDevice设备上的操作中向内容时间轴发出步骤，请为GPUDevice设备排队一个全局任务，其中包含这些步骤。

**设备时间轴**

与由用户代理发出的GPU设备操作相关联。它包括适配器、设备和GPU资源和状态对象的创建，这些操作从控制GPU的用户代理部分的角度来看通常是同步的，但可以存在于单独的操作系统进程中。

**队列时间轴**

与GPU的计算单元上的操作执行相关联。它包括实际在GPU上运行的绘制、复制和计算作业。

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
> 在内容时间轴上执行的步骤如下所示：  
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
> GPUComputePassEncoder.dispatchWorkgroups():  
> 1. 用户通过调用GPUComputePassEncoder的方法对dispatchWorkgroups命令进行编码，该方法在内容时间轴上执行。
> 
> 2. 用户通过调用GPUQueue.submit()发出GPUCommandBuffer，并将其交给用户代理，在设备时间轴上处理它，通过调用OS驱动程序进行低级提交。
> 
> 3. 提交由GPU调用调度程序分派到实际的计算单元以执行，在队列时间轴上执行。

> 例子 3
> 
> GPUDevice.createBuffer():  
> 1. 用户填写GPUBufferDescriptor并使用它创建GPUBuffer，该操作在内容时间轴上执行。
> 
> 2. 用户代理在设备时间轴上创建低级缓冲区。

> 例子 4
> 
> GPUBuffer.mapAsync():  
> 1. 用户在内容时间轴上请求映射GPUBuffer，并返回一个Promise。
> 
> 2. 用户代理检查缓冲区当前是否被GPU使用，并在此使用完成后提醒自己回来检查。
> 
> 3. 在GPU在队列时间轴上操作缓冲区后，用户代理将其映射到内存并解析Promise。

### 3.4.2. 内存模型
本节非规范性。

一旦在应用程序初始化过程中获得了GPUDevice，我们可以将WebGPU平台描述为包含以下层：

1. 实现规范的用户代理。

2. 具有该设备的低级本地API驱动程序的操作系统。

3. 实际的CPU和GPU硬件。

WebGPU平台的每个层可能具有不同的内存类型，用户代理在实现规范时需要考虑这一点：

* 由脚本创建的脚本拥有的内存，例如ArrayBuffer，通常不可被GPU驱动程序访问。

* 用户代理可能有不同的进程负责运行内容和与GPU驱动程序通信。在这种情况下，它使用进程间共享内存来传输数据。

* 专用GPU具有高带宽的自己的内存，而集成GPU通常与系统共享内存。

大多数物理资源都分配在内存类型中，对于GPU计算或渲染是有效的。当用户需要向GPU提供新数据时，该数据可能需要首先跨越进程边界，以达到与GPU驱动程序通信的用户代理部分。然后它可能需要对驱动程序可见，这有时需要将其复制到由驱动程序分配的暂存内存中。最后，它可能需要传输到专用GPU内存中，可能会将内部布局更改为最适合GPU操作的布局。

所有这些转换都由用户代理的WebGPU实现完成。

注意：此示例描述了最坏情况，而在实践中，实现可能不需要跨越进程边界，或者可能能够直接向用户公开由驱动程序管理的内存（在ArrayBuffer后面），从而避免任何数据复制。

### 3.4.3. 资源用法
物理资源可以在GPU上使用，具有内部用法：

> **input**  
> 用于绘制或调度调用的输入数据缓冲区。保留内容。可由缓冲区INDEX、缓冲区VERTEX或缓冲区INDIRECT使用。
> 
> **constant**  
> 从着色器的角度看是恒定的资源绑定。保留内容。可由缓冲区UNIFORM或纹理TEXTURE_BINDING使用。
> 
> **storage**  
> 可写存储资源绑定。可由缓冲区STORAGE或纹理STORAGE_BINDING使用。
> 
> **storage-read**  
> 只读存储资源绑定。保留内容。可由缓冲区STORAGE使用。
> 
> **attachment**  
> 在渲染通道中用作输出附件的纹理。可由纹理RENDER_ATTACHMENT使用。
> 
> **attachment-read**  
> 在渲染通道中用作只读附件的纹理。保留内容。可由纹理RENDER_ATTACHMENT使用。

我们将子资源定义为整个缓冲区或纹理子资源之一。

某些内部用法与其他用法兼容。子资源可以处于将多个用法组合在一起的状态。我们认为列表U是兼容的用法列表，如果（且仅如果）它满足以下任一规则：  
* U中的每个用法都是input、constant、storage-read或attachment-read。

* U中的每个用法都是storage。

* U包含一个元素：attachment。

强制使用只能组合成兼容的用法列表，可以使API限制在处理内存时何时可能发生数据竞争。这种属性使针对WebGPU编写的应用程序更有可能在不同的平台上运行而无需修改。

通常，当实现处理使用不同方式使用子资源的操作时，它会安排将资源转换为新状态。在某些情况下，例如在打开的GPURenderPassEncoder内部，由于硬件限制，这样的转换是不可能的。我们将这些地方定义为用法范围。

主要的使用规则是，对于任何一个子资源，在一个用法范围内其内部用法列表必须是兼容的用法列表。

例如，在同一GPURenderPassEncoder中将相同的缓冲区绑定为存储和输入会将编码器以及拥有的GPUCommandEncoder置为错误状态。这些用法的组合不构成兼容的用法列表。

注意：允许在单个用法范围内存在多个可写存储缓冲区/纹理用法的竞争条件。

包含在GPURenderPassColorAttachment.view和GPURenderPassColorAttachment.resolveTarget提供的视图中的纹理的子资源被视为用于此渲染通道的用法范围的附件。

### 3.4.4. 同步
对于物理资源的每个子资源，在Queue时间轴上跟踪其内部使用标志集。

在Queue时间轴上，有一个使用范围的有序序列。在每个范围的持续时间内，任何给定子资源的内部使用标志集都是恒定的。子资源可以在使用范围之间的边界处转换到新的用法。

本规范定义以下使用范围：

* 在通道之外（在GPUCommandEncoder中），每个（非状态设置）命令都是一个使用范围（例如，copyBufferToTexture()）。

* 在计算通道中，每个调度命令（dispatchWorkgroups()或dispatchWorkgroupsIndirect()）是一个使用范围。如果可以通过该命令访问，则子资源在使用范围中“使用”。在调度内，对于由当前GPUComputePipeline的[[layout]]使用的每个绑定组槽，每个由该绑定组引用的子资源在使用范围内均“使用”。状态设置计算通道命令（例如setBindGroup()）不直接contributing到使用范围中；它们改变了在调度命令中检查的状态。

* 一个渲染通道是一个使用范围。如果被任何（状态设置或非状态设置）命令引用，则子资源在使用范围内“使用”。例如，在setBindGroup()中，每个绑定组中的每个子资源在渲染通道的使用范围内“使用”。

上述内容可能应该讨论GPU命令。但我们目前还没有一种引用特定GPU命令（如dispatch）的方法。

注意：上述规则意味着以下示例资源用法包含在使用范围验证中：

* 在渲染通道中，用于任何setBindGroup()调用的子资源，无论当前绑定的管道的着色器或布局实际上是否依赖于这些绑定，或者绑定组是否被另一个“set”调用所覆盖。

* 用于任何setVertexBuffer()调用中的缓冲区，无论任何绘制调用是否依赖于该缓冲区，或者该缓冲区是否被另一个“set”调用所覆盖。

* 用于任何setIndexBuffer()调用中的缓冲区，无论任何绘制调用是否依赖于该缓冲区，或者该缓冲区是否被另一个“set”调用所覆盖。

* 由beginRenderPass()在GPURenderPassDescriptor中用作颜色附件、解析附件或深度/模板附件的纹理子资源，无论着色器实际上是否依赖于这些附件。

* 在可见性为0或仅针对计算阶段可见但在渲染通道中使用（或反之亦然）的绑定组条目中使用的资源。

在命令编码期间，每个子资源的每个使用都记录在命令缓冲区中的一个使用范围中。对于每个使用范围，实现通过组合在使用范围中使用的每个子资源的所有内部使用标志的列表来执行使用范围验证。如果其中任何一个列表不是兼容的使用列表，则GPUCommandEncoder.finish()将生成验证错误。

## 3.5. 核心内部对象
### 3.5.1. Adapters
Adapter 标识了系统上 WebGPU 的实现：即基于浏览器的平台上的计算/渲染功能实例，以及在该功能之上的浏览器 WebGPU 实现的实例。

Adapters 并不唯一地表示底层实现：多次调用 requestAdapter() 会返回不同的 adapter 对象。

每个 adapter 对象只能用于创建一个 device，在成功的 requestDevice() 调用后，adapter 就变为无效。此外，adapter 对象也可能随时过期。

注意：这确保应用程序在创建 device 时使用最新的系统状态进行 adapter 选择。这也通过使各种场景看起来相似来鼓励更多的鲁棒性：首次初始化、由于拔掉 adapter 而重新初始化、由于测试 GPUDevice.destroy() 调用而重新初始化等。

如果一个 adapter 具有显著的性能缺陷，以换取更广泛的兼容性、更可预测的行为或更好的隐私，则可将其视为后备 adapter。并不要求每个系统都有后备 adapter 可用。

Adapter 具有以下内部 slot：

[[features]]，类型为 ordered set<GPUFeatureName>，只读  
可用于在此 adapter 上创建设备的特性。

[[limits]]，类型为 supported limits，只读  
可用于在此 adapter 上创建设备的最佳限制。

每个 adapter 限制必须与其 supported limits 中的默认值相同或更好。

[[fallback]]，类型为 boolean  
如果设置为 true，表示该 adapter 是后备 adapter。

[[unmaskedIdentifiers]]，类型为 ordered set<DOMString>  
用户代理选择为此 adapter 报告的 GPUAdapterInfo 字段名称列表。最初填充了用户代理选择未经用户同意报告的任何 GPUAdapterInfo 字段的名称。

Adapter 通过 GPUAdapter 暴露。

3.5.2. Devices  
Device 是 adapter 的逻辑实例化，通过它可以创建内部对象。它可以在多个代理之间共享（例如，专用 worker）。

Device 拥有从它创建的所有内部对象的独占所有权：当 device 变为无效（丢失或销毁）时，直接（例如 createTexture()）或间接（例如 createView()）在其上创建的所有对象都会变得隐式不可用。

Device 具有以下内部 slot：

[[adapter]]，类型为 adapter，只读  
创建此 device 的 adapter。

[[features]]，类型为 ordered set<GPUFeatureName>，只读  
可在此 device 上使用的特性。即使底层 adapter 可以支持它们，也不能使用其他特性。

[[limits]]，类型为 supported limits，只读  
可在此 device 上使用的限制。即使底层 adapter 可以支持更好的限制，也不能使用更好的限制。

当使用 GPUDeviceDescriptor 描述符从 adapter adapter 创建新的设备 device 时：  
将 device.[[adapter]] 设为 adapter。

将 device.[[features]] 设为 descriptor.requiredFeatures 中的值集合。

让 device.[[limits]] 成为一个 supported limits 对象，其中包含默认值。对于 descriptor.requiredLimits 中的每个 (key, value) 对，将 device.[[limits]] 中对应于 key 的成员设置为 value 的更好值或 supported limits 中的默认值。

任何时候，用户代理需要撤销对设备的访问，就会在设备的 device timeline 上调用 lose the device(device, "unknown")，可能超前于当前在该 timeline 上排队的其他操作。

如果一个操作失败并具有可观察地更改设备上对象状态或潜在地损坏内部实现/驱动程序状态的副作用，就应将该设备失效以防止这些更改被观察到。

注意：对于所有非应用程序发起的设备失效（通过 destroy()），用户代理应无条件发出开发者可见的警告，即使已处理了失效的 promise。这些情况应该很少见，并且该信号对开发人员非常重要，因为 WebGPU API 的大多数部分都试图像没有问题一样运行，以避免中断应用程序的运行时流程：不会引发验证错误，大多数 promise 正常解析等等。

要失去设备(device, reason)：  
使设备无效。

让 gpuDevice 成为与 device 对应的 content timeline GPUDevice。

在 gpuDevice 的内容时间轴上执行以下步骤：

使用 reason 设定为 reason 和 message 设定为一个实现定义的值的新 GPUDeviceLostInfo 解析 device.lost。

注意：message 不应透露不必要的用户/系统信息，也不应该由应用程序解析。

完成任何未完成的 mapAsync() 步骤。

完成任何未完成的 onSubmittedWorkDone() 步骤。

注意：设备失效后不会生成任何错误。参见 § 22 错误和调试。

Device 通过 GPUDevice 暴露。