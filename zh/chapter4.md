4.初始化[]（https://www.w3.org/tr/webgpu/#initialization）
4.1. navigator.gpu []（https://www.w3.org/tr/webgpu/#navigator-gpu）
在浏览器的 Window 和 DedicatedWorkerGlobalScope 上通过 Navigator 和 WorkerNavigator 接口可以获得 GPU 对象，通过navigator.gpu可以访问：

interface mixin NavigatorGPU {
    [SameObject, SecureContext] readonly attribute GPU gpu;
};
Navigator 包含 NavigatorGPU;
WorkerNavigator 包含 NavigatorGPU;
4.2. GPU
GPU  类型是 WebGPU 的入口。

[Exposed=(Window, DedicatedWorker), SecureContext]
interface GPU {
    Promise<GPUAdapter?> requestAdapter(optional GPURequestAdapterOptions options = {});
    GPUTextureFormat getPreferredCanvasFormat();
    [SameObject] readonly attribute WGSLLanguageFeatures wgslLanguageFeatures;
};
GPU有如下的方法和属性：

requestAdapter(options)

从用户代理请求适配器。用户代理会选择是否返回适配器，如果是，则根据提供的选项选择并返回。

调用者:  GPU this.

入参:

名称	类型	Nullable	Optional	描述
options	GPURequestAdapterOptions	✘	✔	用于选择适配器的标准条件
出参:  Promise<GPUAdapter?>

内容时间线 步骤:

将 contentTimeline  置为当前的  内容时间线
将 promise 赋值为 一个新的 Promise
在当前的 设备时间线  执行  初始化步骤
返回  promise
设备时间线  的 初始化步骤:

将 adapter  赋值为  null
如果用户代理决定选择返回一个 adapter，它应当:
将adapter设置为有效的adapter，根据的规则来自于§ 4.2.2 选择适配器以及options中定义的条件，参考§ 4.2.1 确保适配器的能力。适配器的能力限制必须符合§ 3.6.2 限制中的要求。

如果adapter满足后备适配器的条件，将adapter.fallback设置为true。

在contentTimeline上继续执行后续步骤。
内容时间线 步骤:

如果 adapter  不为  null:
将封装了adapter的一个新的GPUAdapter作为promise的Resolve参数返回。
否则，使用null来Resolve promise。
getPreferredCanvasFormat()

返回适合在此系统上显示 8 位深度的标准动态范围内容的最佳 GPUTextureFormat。只会返回"rgba8unorm" or "bgra8unorm"。

返回的值可以作为参数 format 传递给GPUCanvasContext对象上的 configure() 方法，以确保相关的画布能够高效地显示其内容。

注意：离屏渲染的画布可能会，也可能不会从这种格式中受益。

调用者:  GPU this.

出参:  GPUTextureFormat

内容时间线 步骤:

返回  "rgba8unorm" 或者 "bgra8unorm"，具体取决于哪种格式最适合此系统上显示的 WebGPU 画布。
wgslLanguageFeatures，类型为WGSLLanguageFeatures，只读属性。

受支持的 WGSL language extensions 扩展的名称，支持的语言拓展会自动启用。

Adapter(适配器) 在某些时候可能会变成无效的(即过时)。一旦系统状态发生任何可能影响到任何requestAdapter()调用结果的变化，用户代理应当使之前返回的所有适配器失效。例如：

一个物理上的适配器被添加/移除掉了(通过插拨插头，设备更新，挂起恢复等等)

系统的电源配置被更改(笔记本电脑电源爬出，电源设置变更等)

注意：用户代理可能会经常性的让适配器失效，即使没有系统状态的变化（例如，在创建适配器后几秒或几分钟内）。这可以帮助屏蔽真实的系统状态变化，并使开发者更加意识到在调用requestDevice()之前总是需要再次调用requestAdapter()。如果应用程序遇到这种情况，标准化设备丢失恢复处理方法应该可以让它恢复正常。

Examples7

请求一个无附加信息的  GPUAdapter:

const gpuAdapter = await navigator.gpu.requestAdapter();
4.2.1.  确保适配器的能力
requestAdapter() 返回的任何 GPUAdapter 必须提供以下保证：

至少满足以下其中之一的条件：

支持"texture-compression-bc"
支持"texture-compression-etc2" 和"texture-compression-astc"
满足的值至少是默认或者更好

所有alignment-class 限制必须是 2 的幂

maxBindingsPerBindGroup 一定需要大于等于 (每个着色器阶段最大绑定数 × 每个管道的最大着色器阶段数)，这里：

每个着色器阶段最大绑定数 =(maxSampledTexturesPerShaderStage + maxSamplersPerShaderStage + maxStorageBuffersPerShaderStage + maxStorageTexturesPerShaderStage + maxUniformBuffersPerShaderStage).

每个管道的最大着色器阶段数是  2, 因为一个 GPURenderPipeline 支持顶点着色器和片段着色器。

注意：maxBindingsPerBindGroup 不代表最基本的限制。实现应该将其提高以符合此要求，而不是降低其他限制。

maxBindGroups 需要小于等于 maxBindGroupsPlusVertexBuffers.

maxVertexBuffers  需要小于等于 maxBindGroupsPlusVertexBuffers.

minUniformBufferOffsetAlignment 和minStorageBufferOffsetAlignment 都需要大于等于 32 字节.

注意: 32 字节是 vec4<f64> 占用的单位，具体可参考WebGPU Shading Language § 13.4.1 Alignment and Size.

maxUniformBufferBindingSize  需要小于等于  maxBufferSize.

maxStorageBufferBindingSize  需要小于等于  maxBufferSize.

maxStorageBufferBindingSize  需要是 4 字节的整数倍.

maxVertexBufferArrayStride 需要是 4 字节的整数倍.

maxComputeWorkgroupSizeX  需要小于等于  maxComputeInvocationsPerWorkgroup.

maxComputeWorkgroupSizeY  需要小于等于  maxComputeInvocationsPerWorkgroup.

maxComputeWorkgroupSizeZ 需要小于等于  maxComputeInvocationsPerWorkgroup.

maxComputeInvocationsPerWorkgroup 需要小于等于 maxComputeWorkgroupSizeX × maxComputeWorkgroupSizeY × maxComputeWorkgroupSizeZ.

4.2.2.  选择适配器
GPURequestAdapterOptions 向用户代理提供要求，指示哪种配置适合应用程序。

dictionary GPURequestAdapterOptions {
    GPUPowerPreference powerPreference;
    boolean forceFallbackAdapter = false;
};
enum GPUPowerPreference {
    "low-power",
    "high-performance",
};
GPURequestAdapterOptions  有下列参数：

powerPreference,  类型为 GPUPowerPreference

此可选参数提供了一个指示，应该从系统可用的适配器中选择哪个适配器类型。

这个配置的值可能会影响选择哪个适配器，但不应该影响是否返回适配器。

注意：该配置的主要作用是影响在多 GPU 系统中使用哪个 GPU。例如，某些笔记本电脑具有低功耗集成 GPU 和高性能独立 GPU。该配置也可能会影响所选 GPU 的功率配置，以匹配所请求的功率偏好。

注意：根据确切的硬件配置（例如电池状态、连接的显示器、可移动的 GPU 等），用户代理程序可能会在相同的功率偏好条件下选择不同的适配器。通常，在相同的硬件配置、状态以及 powerPreference 的情况下，用户代理可能会选择相同的适配器。

它的值可以为下面其中之一：

undefined (或者不提供)

不为用户代理提供额外的要求

"low-power"

表示请求优先考虑省电而非性能。

注意：通常，如果不太可能受到绘图性能的限制，应该使用此选项。例如，如果它每秒只渲染一帧，只绘制相对简单的几何图形和简单的着色器，或使用较小的 HTML 画布元素。如果内容允许，鼓励开发者使用此值，因为它可能显著提高便携设备的电池寿命。

"high-performance"

表示请求优先考虑性能而非电量消耗。

注意：通过选择此值，开发者应该意识到，用户代理程序更有可能强制设备丢失，以便通过切换到低功率适配器来节省电源。开发者应该只在确信绝对必要时才指定此值，因为它可能会显著降低便携设备的电池寿命。

forceFallbackAdapter,  类型为  boolean, 默认值  false

当设置为true时，表示只能返回一个备用适配器。如果用户代理程序不支持备用适配器，则会导致 requestAdapter() 结果解析为 null。

注意：如果forceFallbackAdapter设置为false并且没有其他适当的 adapter 可用，或者用户代理程序选择返回备用适配器，requestAdapter()仍然可能返回一个 备用适配器。开发者若希望防止他们的应用程序运行在 备用适配器，需要在请求 GPUDevice 之前检查 GPUAdapter 的 isFallbackAdapter 属性。

Examples 8

申请一个"high-performance"的 GPUAdapter:

const gpuAdapter = await navigator.gpu.requestAdapter({
    powerPreference: 'high-performance'
});
4.3. GPU适配器
GPU适配器(GPUAdapter)封装了一个适配器对象，并且描述了它具备的能力(包括特性和限制)。

使用requestAdapter()，可以生成一个GPUAdapter。

[Exposed=(Window, DedicatedWorker), SecureContext]
interface GPUAdapter {
    [SameObject] readonly attribute GPUSupportedFeatures features;
    [SameObject] readonly attribute GPUSupportedLimits limits;
    readonly attribute boolean isFallbackAdapter;

    Promise<GPUDevice> requestDevice(optional GPUDeviceDescriptor descriptor = {});
    Promise<GPUAdapterInfo> requestAdapterInfo(optional sequence<DOMString> unmaskHints = []);
};
GPUAdapter  有下面这些属性：

feature，类型是GPUSupportedFeatures，只读

位于this.[[adapter]].[[features]]下的一组值

limits，类型是GPUSupportedLimits，只读

位于this.[[adapter]].[[limits]]下的值

isFallbackAdapter，类型为boolean，只读

返回[[adapter]].[[fallback]]的值

GPUAdapter 有下列内置属性：

[[adapter]], 类型是adapter，只读

即GPUAdapter引用的adapter实例。

GPUAdapter  有下列方法：

requestDevice(descriptor)

从adapter中申请获取一个设备(device)。

这是一个一次性操作，如果一个设备被成功的返回了，适配器状态会变成无效(invalid)。

调用者:  GPUAdapter this.

入参:

名称	类型	Nullable	Optional	描述
descriptor	GPUDeviceDescriptor	✘	✔	获取GPUDevice的描述配置
出参:  Promise<GPUDevice>

内容时间线 步骤:

将 contentTimeline  置为当前的  内容时间线
将 promise 赋值为 一个新的 Promise
将 adapter 置为 this.[[adapter]]
在当前的 设备时间线  执行  初始化步骤
返回  promise
设备时间线 初始化步骤:

如果下列需求任意一项不满足：
descriptor.requiredFeatures 集合中的值必须是adapter.[[features]]的子集。
则执行contentTimeline中下列步骤并返回：

Content timeline 步骤:

用TypeError错误 Reject promise。
注意：如果浏览器完全无法识别一个特性名称（在GPUFeatureName定义），则会产生与此相同的错误。这将使浏览器在不支持某个特性时的行为 和 特定适配器不支持某个特性时 的行为保持一致。

如果下列需求任意一项不满足：
descriptor.requiredLimits中的每一个 key 的名字都需要在supported limits中.

对于 supported limits 的键中的每个限制的key：设置 value 为 descriptor.requiredLimits[key]

value 不得 超过 adapter.[[limits]]中的限制值。

如果限制的 类型 是 alignment， value  需要是 2 的正整数幂。

然后执行contentTimeline后续的步骤并返回：

内容时间线 步骤：

用OperationError错误 Reject promise。
如果 adapter 是 无效的(invalid)，或者用户代理无法满足需求：

将 device  初始化为新的  device。

丢失设备(device, "unknown")

注意：如果adapter还没准备好，这将使它失效

注意：在大多数情况下，用户代理程序应该考虑发出开发者可见的警告。应用程序应该从requestAdapter()开始执行重新初始化逻辑。

否则：

使用descriptor中描述的功能创建一个新的设备，并将其赋值给device。

使 adapter.[[adapter]] 失效。

执行contentTimeline的下列步骤
内容时间线 步骤：

使用一个新的类型为  GPUDevice的 device 作为 promise Resolve  值。
注意：如果设备已经丢失，因为适配器无法满足请求，在promise resolve 之前，device.lost 会已经被resolve。

requestAdapterInfo()

请求此GPUAdapter的GPUAdapterInfo。

注意：适配器信息值返回一个 Promise，以便让用户代理程序在请求未屏蔽的值时有机会执行潜在的长时间运行的检查，例如在返回之前请求用户同意。但是，如果没有指定unmaskHints，则不应向用户显示任何对话框。

调用者:  GPUAdapter this.

入参:

名称	类型	Nullable	Optional	描述
unmaskHints	sequence<DOMString>	✘	✔	如果可用，希望返回未屏蔽值的GPUAdapterInfo属性名称列表。
出参: Promise<GPUAdapterInfo>

内容时间线 步骤:

将 promise 赋值为 一个新的 Promise
将 adapter 置为 this.[[adapter]]
如果 this 相关的全局变量被短暂激活，则将hasActivation置为true，否则为false
并行运行下列步骤：
如果unmaskHints.length > 0:

如果hasActivation值为false，来自抛出NotAllowedError错误的promise的reject，终止这些步骤。

如果可能的话，让unmaskedKeys成为unmaskHints中指定的字段的列表，用户代理程序决定要解除掩码的字段。

注意：用户代理程序可以自由地使用其认为合适的任何方法来决定哪些字段需要解除掩码。

将unmaskedKeys 添加 到adapter.[[unmaskedIdentifiers]]
使用adapter创建一个新的适配器信息 并使用该信息 Resolve promise。

返回promise

Example 9

使用默认的特性和限制申请一个GPUDevice：

const gpuAdapter = await navigator.gpu.requestAdapter();
const gpuDevice = await gpuAdapter.requestDevice();
4.3.1. GPUDeviceDescriptor
GPUDeviceDescriptor 用来描述申请设备的需求。

dictionary GPUDeviceDescriptor
         : GPUObjectDescriptorBase {
    sequence<GPUFeatureName> requiredFeatures = [];
    record<DOMString, GPUSize64> requiredLimits = {};
    GPUQueueDescriptor defaultQueue = {};
};
GPUDeviceDescriptor 有下列成员：

requiredFeatures, 类型属于<GPUFeatureName>，默认是 []

指定设备请求所需的功能。如果适配器无法提供这些功能，则请求将失败。

在生成的设备上对验证可调用的API时，功能集应该恰好和指定一致，不多也不少。

requiredLimits, 类型为 Record<DOMString, GPUSize64>, defaulting to {}

指定设备请求所需的限制(limits)。如果适配器无法提供这些限制，则请求将失败。

每个键必须是supported limits中的成员名称。在生成的设备上对API调用进行验证时，限制会恰好满足条件，不会更好或更差。

defaultQueue, 类型为GPUQueueDescriptor, 默认值是 {}

对默认的GPUQueue的描述

Example 10

如果支持，申请一个具有"texture-compression-astc"特性的GPUDevice：

const gpuAdapter = await navigator.gpu.requestAdapter();

const requiredFeatures = [];
if (gpuAdapter.features.has('texture-compression-astc')) {
    requiredFeatures.push('texture-compression-astc')
}

const gpuDevice = await gpuAdapter.requestDevice({
    requiredFeatures
});
4.3.1.1. GPUFeatureName
每个GPUFeatureName标识了一组功能，如果可用，将允许使用WebGPU附加的用法，否则这些用法是无效的。

enum GPUFeatureName {
    "depth-clip-control",
    "depth32float-stencil8",
    "texture-compression-bc",
    "texture-compression-etc2",
    "texture-compression-astc",
    "timestamp-query",
    "indirect-first-instance",
    "shader-f16",
    "rg11b10ufloat-renderable",
    "bgra8unorm-storage",
    "float32-filterable",
};
4.4. GPUDevice
GPUDevice封装了一个device，并公开了该设备的功能函数。

GPUDevice是创建WebGPU接口的最上层接口。

要获取GPUDevice，使用requestDevice()。

[Exposed=(Window, DedicatedWorker), SecureContext]
interface GPUDevice : EventTarget {
    [SameObject] readonly attribute GPUSupportedFeatures features;
    [SameObject] readonly attribute GPUSupportedLimits limits;

    [SameObject] readonly attribute GPUQueue queue;

    undefined destroy();

    GPUBuffer createBuffer(GPUBufferDescriptor descriptor);
    GPUTexture createTexture(GPUTextureDescriptor descriptor);
    GPUSampler createSampler(optional GPUSamplerDescriptor descriptor = {});
    GPUExternalTexture importExternalTexture(GPUExternalTextureDescriptor descriptor);

    GPUBindGroupLayout createBindGroupLayout(GPUBindGroupLayoutDescriptor descriptor);
    GPUPipelineLayout createPipelineLayout(GPUPipelineLayoutDescriptor descriptor);
    GPUBindGroup createBindGroup(GPUBindGroupDescriptor descriptor);

    GPUShaderModule createShaderModule(GPUShaderModuleDescriptor descriptor);
    GPUComputePipeline createComputePipeline(GPUComputePipelineDescriptor descriptor);
    GPURenderPipeline createRenderPipeline(GPURenderPipelineDescriptor descriptor);
    Promise<GPUComputePipeline> createComputePipelineAsync(GPUComputePipelineDescriptor descriptor);
    Promise<GPURenderPipeline> createRenderPipelineAsync(GPURenderPipelineDescriptor descriptor);

    GPUCommandEncoder createCommandEncoder(optional GPUCommandEncoderDescriptor descriptor = {});
    GPURenderBundleEncoder createRenderBundleEncoder(GPURenderBundleEncoderDescriptor descriptor);

    GPUQuerySet createQuerySet(GPUQuerySetDescriptor descriptor);
};
GPUDevice includes GPUObjectBase;
GPUDevice 有下列属性：

features, 类型为GPUSupportedFeatures，只读

包含设备支持的GPUFeatureName值的集合（即使用这些值创建设备的功能）

limits, 类型为GPUSupportedLimits，只读

公开设备支持的限制（即使用这些限制创建设备）。

queue，类型为GPUQueue，只读

该设备首要的GPUQueue

GPUDevice的[[device]]是GPUDevice引用的device

GPUDevice具有其上面的WebIDL定义中列出的方法。未在此处定义的方法，会在本文档的其他位置定义。

destroy()

销毁device，防止对其进行进一步的操作，未完成的异步操作将失败。

注意：可对设备进行多次销毁。

调用者:  GPUDevice this.

内容时间线 步骤:

从此设备中unmap()所有GPUBuffer。
在this的Device timeline执行后续步骤。
设备时间线 步骤:

一旦此设备上任何队列中的操作都完成了，就在当前时间线上执行后续步骤。
丢失此设备(this.[[device]], "destroyed")
注意：由于无法添加进一步操作到设备任务队列中，因此实现上可以立即中止未完成的异步操作并释放资源分配，包括刚刚取消映射的内存。

GPUDevice允许的缓冲区用法为：

总是允许: MAP_READ, MAP_WRITE, COPY_SRC, COPY_DST, INDEX, VERTEX, UNIFORM, STORAGE, INDIRECT, QUERY_RESOLVE
GPUDevice允许的纹理用法为：

总是允许: COPY_SRC, COPY_DST, TEXTURE_BINDING, STORAGE_BINDING, RENDER_ATTACHMENT
4.5. 案例
一个强大的，带有错误处理请求的 GPUAdapter 和 GPUDevice 的示例

let gpuDevice = null;

async function initializeWebGPU() {
    // 检查用户代理对WebGPU的支持情况
    if (!('gpu' in navigator)) {
        console.error("User agent doesn’t support WebGPU.");
        return false;
    }

    // 请求一个适配器
    const gpuAdapter = await navigator.gpu.requestAdapter();

    // 如果找不到合适的适配器，requestAdapter 可能会resolve 为 null。
    if (!gpuAdapter) {
        console.error('No WebGPU adapters found.');
        return false;
    }

    // Request a device.
    // Note that the promise will reject if invalid options are passed to the optional
    // dictionary. To avoid the promise rejecting always check any features and limits
    // against the adapters features and limits prior to calling requestDevice().
    gpuDevice = await gpuAdapter.requestDevice();

    // requestDevice will never return null, but if a valid device request can’t be
    // fulfilled for some reason it may resolve to a device which has already been lost.
    // Additionally, devices can be lost at any time after creation for a variety of reasons
    // (ie: browser resource management, driver updates), so it’s a good idea to always
    // handle lost devices gracefully.
    gpuDevice.lost.then((info) => {
        console.error(`WebGPU device was lost: ${info.message}`);

        gpuDevice = null;

        // Many causes for lost devices are transient, so applications should try getting a
        // new device once a previous one has been lost unless the loss was caused by the
        // application intentionally destroying the device. Note that any WebGPU resources
        // created with the previous device (buffers, textures, etc) will need to be
        // re-created with the new one.
        if (info.reason != 'destroyed') {
            initializeWebGPU();
        }
    });

    onWebGPUInitialized();

    return true;
}

function onWebGPUInitialized() {
    // Begin creating WebGPU resources here...
}

initializeWebGPU();