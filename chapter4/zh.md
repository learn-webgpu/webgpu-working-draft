## 4.初始化[]（https://www.w3.org/tr/webgpu/#initialization）

### 4.1. navigator.gpu []（https://www.w3.org/tr/webgpu/#navigator-gpu）

在浏览器的 [Window](https://html.spec.whatwg.org/multipage/nav-history-apis.html#window) 和 [DedicatedWorkerGlobalScope](https://html.spec.whatwg.org/multipage/workers.html#dedicatedworkerglobalscope) 上通过 [Navigator](https://html.spec.whatwg.org/multipage/system-state.html#navigator) 和 [WorkerNavigator](https://html.spec.whatwg.org/multipage/workers.html#workernavigator) 接口可以获得 [GPU](https://www.w3.org/TR/webgpu/#gpu) 对象，通过`navigator.gpu`可以访问：

```
interface mixin NavigatorGPU {
    [SameObject, SecureContext] readonly attribute GPU gpu;
};
Navigator 包含 NavigatorGPU;
WorkerNavigator 包含 NavigatorGPU;
```

### 4.2. GPU[](https://www.w3.org/TR/webgpu/#gpu-interface)

`GPU`  类型是 WebGPU 的入口。

```
[Exposed=(Window, DedicatedWorker), SecureContext]
interface GPU {
    Promise<GPUAdapter?> requestAdapter(optional GPURequestAdapterOptions options = {});
    GPUTextureFormat getPreferredCanvasFormat();
    [SameObject] readonly attribute WGSLLanguageFeatures wgslLanguageFeatures;
};
```

`GPU`有如下的方法和属性：

- `requestAdapter(options)`

  从用户代理请求[适配器](https://www.w3.org/TR/webgpu/#adapter)。用户代理会选择是否返回适配器，如果是，则根据提供的选项选择并返回。

  **调用者:**  `GPU` this.

  **入参:**

  | 名称      | 类型                                                                                       | Nullable | Optional | 描述                     |
  | --------- | ------------------------------------------------------------------------------------------ | -------- | -------- | ------------------------ |
  | `options` | [GPURequestAdapterOptions](https://www.w3.org/TR/webgpu/#dictdef-gpurequestadapteroptions) | ✘        | ✔        | 用于选择适配器的标准条件 |

  **出参:**  `Promise`<`GPUAdapter`?>

  [内容时间线](https://www.w3.org/TR/webgpu/#content-timeline) 步骤:

  1.  将 `contentTimeline`  置为当前的  [内容时间线](https://www.w3.org/TR/webgpu/#content-timeline)
  2.  将 `promise` 赋值为 [一个新的 Promise](https://webidl.spec.whatwg.org/#a-new-promise)
  3.  在当前的 [设备时间线](https://www.w3.org/TR/webgpu/#device-timeline)  执行  *初始化步骤*
  4.  返回  `promise`

  [设备时间线](https://www.w3.org/TR/webgpu/#device-timeline)  的 _初始化步骤_:

  1. 将 `adapter`  赋值为  `null`
  2. 如果用户代理决定选择返回一个 adapter，它应当:

  - 将`adapter`设置为[有效](https://www.w3.org/TR/webgpu/#valid)的[adapter](https://www.w3.org/TR/webgpu/#adapter)，根据的规则来自于[§ 4.2.2 选择适配器](https://www.w3.org/TR/webgpu/#adapter-selection)以及`options`中定义的条件，参考[§ 4.2.1 确保适配器的能力](https://www.w3.org/TR/webgpu/#adapter-capability-guarantees)。适配器的[能力限制](https://www.w3.org/TR/webgpu/#supported-limits)必须符合[§ 3.6.2 限制](https://www.w3.org/TR/webgpu/#limits)中的要求。

  - 如果`adapter`满足[后备适配器](https://www.w3.org/TR/webgpu/#fallback-adapter)的条件，将`adapter.fallback`设置为`true`。

  3. 在`contentTimeline`上继续执行后续步骤。

  [内容时间线](https://www.w3.org/TR/webgpu/#content-timeline) 步骤:

  1. 如果 `adapter`  不为  `null`:

  - 将封装了`adapter`的一个新的`GPUAdapter`作为`promise`的[Resolve](https://webidl.spec.whatwg.org/#resolve)参数返回。

  2. 否则，使用`null`来[Resolve](https://webidl.spec.whatwg.org/#resolve) `promise`。

- `getPreferredCanvasFormat()`

  返回适合在此系统上显示 8 位深度的标准动态范围内容的最佳 `GPUTextureFormat`。只会返回`"rgba8unorm"` or `"bgra8unorm"`。

  返回的值可以作为参数 `format` 传递给`GPUCanvasContext`对象上的 `configure()` 方法，以确保相关的画布能够高效地显示其内容。

  注意：离屏渲染的画布可能会，也可能不会从这种格式中受益。

  **调用者:**  `GPU` this.

  **出参:**  `GPUTextureFormat`

  [内容时间线](https://www.w3.org/TR/webgpu/#content-timeline) 步骤:

  1. 返回  `"rgba8unorm"` 或者 `"bgra8unorm"`，具体取决于哪种格式最适合此系统上显示的 WebGPU 画布。

- `wgslLanguageFeatures`，类型为[WGSLLanguageFeatures](https://www.w3.org/TR/webgpu/#gpuwgsllanguagefeatures)，只读属性。

  受支持的 WGSL [language extensions](https://gpuweb.github.io/gpuweb/wgsl/#language-extension) 扩展的名称，支持的语言拓展会自动启用。

[Adapter(适配器)](https://www.w3.org/TR/webgpu/#adapter) 在某些时候**可能**会变成[无效的](https://www.w3.org/TR/webgpu/#invalid)(即过时)。一旦系统状态发生任何可能影响到任何`requestAdapter()`调用结果的变化，用户代理**应当**使之前返回的所有适配器失效。例如：

- 一个物理上的适配器被添加/移除掉了(通过插拨插头，设备更新，挂起恢复等等)

- 系统的电源配置被更改(笔记本电脑电源爬出，电源设置变更等)

注意：用户代理可能会经常性的让适配器[失效](https://www.w3.org/TR/webgpu/#adapter-expire)，即使没有系统状态的变化（例如，在创建适配器后几秒或几分钟内）。这可以帮助屏蔽真实的系统状态变化，并使开发者更加意识到在调用`requestDevice()`之前总是需要再次调用`requestAdapter()`。如果应用程序遇到这种情况，标准化设备丢失恢复处理方法应该可以让它恢复正常。

[Examples7](https://www.w3.org/TR/webgpu/#example-78e28ede)

请求一个无附加信息的  `GPUAdapter`:

```
const gpuAdapter = await navigator.gpu.requestAdapter();
```

#### 4.2.1.  确保适配器的能力[](https://www.w3.org/TR/webgpu/#adapter-capability-guarantees)

`requestAdapter()` 返回的任何 `GPUAdapter` 必须提供以下保证：

- 至少满足以下其中之一的条件：

  - 支持`"texture-compression-bc"`
  - 支持`"texture-compression-etc2"` 和`"texture-compression-astc"`

- 满足的值至少是[默认](https://www.w3.org/TR/webgpu/#limit-default)或者[更好](https://www.w3.org/TR/webgpu/#limit-better)

- 所有[alignment-class](https://www.w3.org/TR/webgpu/#limit-class-alignment) 限制必须是 2 的幂

- `maxBindingsPerBindGroup` 一定需要大于等于 ([每个着色器阶段最大绑定数](https://www.w3.org/TR/webgpu/#max-bindings-per-shader-stage) × [每个管道的最大着色器阶段数](https://www.w3.org/TR/webgpu/#max-shader-stages-per-pipeline))，这里：

  - 每个着色器阶段最大绑定数 =(`maxSampledTexturesPerShaderStage` + `maxSamplersPerShaderStage` + `maxStorageBuffersPerShaderStage` + `maxStorageTexturesPerShaderStage` + `maxUniformBuffersPerShaderStage`).

  - 每个管道的最大着色器阶段数是  `2`, 因为一个 `GPURenderPipeline` 支持顶点着色器和片段着色器。

  注意：`maxBindingsPerBindGroup` 不代表最基本的限制。实现应该将其提高以符合此要求，而不是降低其他限制。

- `maxBindGroups` 需要小于等于 `maxBindGroupsPlusVertexBuffers`.

- `maxVertexBuffers`  需要小于等于 `maxBindGroupsPlusVertexBuffers`.

- `minUniformBufferOffsetAlignment` 和`minStorageBufferOffsetAlignment` 都需要大于等于 32 字节.

  注意: 32 字节是 `vec4<f64>` 占用的单位，具体可参考[WebGPU Shading Language § 13.4.1 Alignment and Size](https://www.w3.org/TR/WGSL/#alignment-and-size).

- `maxUniformBufferBindingSize`  需要小于等于  `maxBufferSize`.

- `maxStorageBufferBindingSize`  需要小于等于  `maxBufferSize`.

- `maxStorageBufferBindingSize`  需要是 4 字节的整数倍.

- `maxVertexBufferArrayStride` 需要是 4 字节的整数倍.

- `maxComputeWorkgroupSizeX`  需要小于等于  `maxComputeInvocationsPerWorkgroup`.

- `maxComputeWorkgroupSizeY`  需要小于等于  `maxComputeInvocationsPerWorkgroup`.

- `maxComputeWorkgroupSizeZ` 需要小于等于  `maxComputeInvocationsPerWorkgroup`.

- `maxComputeInvocationsPerWorkgroup` 需要小于等于 `maxComputeWorkgroupSizeX` × `maxComputeWorkgroupSizeY` × `maxComputeWorkgroupSizeZ`.

#### 4.2.2.  选择适配器[](https://www.w3.org/TR/webgpu/#adapter-selection)

`GPURequestAdapterOptions` 向用户代理提供要求，指示哪种配置适合应用程序。

```
dictionary GPURequestAdapterOptions {
    GPUPowerPreference powerPreference;
    boolean forceFallbackAdapter = false;
};
```

```
enum GPUPowerPreference {
    "low-power",
    "high-performance",
};
```

`GPURequestAdapterOptions`  有下列参数：

- `powerPreference`,  类型为 [GPUPowerPreference](https://www.w3.org/TR/webgpu/#enumdef-gpupowerpreference)

  此可选参数提供了一个指示，应该从系统可用的适配器中选择哪个[适配器](https://www.w3.org/TR/webgpu/#adapter)类型。

  这个配置的值可能会影响选择哪个适配器，但不应该影响是否返回适配器。

  注意：该配置的主要作用是影响在多 GPU 系统中使用哪个 GPU。例如，某些笔记本电脑具有低功耗集成 GPU 和高性能独立 GPU。该配置也可能会影响所选 GPU 的功率配置，以匹配所请求的功率偏好。

  注意：根据确切的硬件配置（例如电池状态、连接的显示器、可移动的 GPU 等），用户代理程序可能会在相同的功率偏好条件下选择不同的[适配器](https://www.w3.org/TR/webgpu/#adapter)。通常，在相同的硬件配置、状态以及 `powerPreference` 的情况下，用户代理可能会选择相同的适配器。

  它的值可以为下面其中之一：

  - `undefined` (或者不提供)

    不为用户代理提供额外的要求

  - `"low-power"`

    表示请求优先考虑省电而非性能。

    注意：通常，如果不太可能受到绘图性能的限制，应该使用此选项。例如，如果它每秒只渲染一帧，只绘制相对简单的几何图形和简单的着色器，或使用较小的 HTML 画布元素。如果内容允许，鼓励开发者使用此值，因为它可能显著提高便携设备的电池寿命。

  - `"high-performance"`

    表示请求优先考虑性能而非电量消耗。

    注意：通过选择此值，开发者应该意识到，用户代理程序更有可能强制[设备](https://www.w3.org/TR/webgpu/#device)丢失，以便通过切换到低功率适配器来节省电源。开发者应该只在确信绝对必要时才指定此值，因为它可能会显著降低便携设备的电池寿命。

- `forceFallbackAdapter`,  类型为  [boolean](https://webidl.spec.whatwg.org/#idl-boolean), 默认值  `false`

  当设置为`true`时，表示只能返回一个[备用适配器](https://www.w3.org/TR/webgpu/#fallback-adapter)。如果用户代理程序不支持[备用适配器](https://www.w3.org/TR/webgpu/#fallback-adapter)，则会导致 `requestAdapter()` 结果解析为 `null`。

  注意：如果`forceFallbackAdapter`设置为`false`并且没有其他适当的 [adapter](https://www.w3.org/TR/webgpu/#adapter) 可用，或者用户代理程序选择返回[备用适配器](https://www.w3.org/TR/webgpu/#fallback-adapter)，`requestAdapter()`仍然可能返回一个 [备用适配器](https://www.w3.org/TR/webgpu/#fallback-adapter)。开发者若希望防止他们的应用程序运行在 [备用适配器](https://www.w3.org/TR/webgpu/#fallback-adapter)，需要在请求 `GPUDevice` 之前检查 `GPUAdapter` 的 `isFallbackAdapter` 属性。

[Examples 8](https://www.w3.org/TR/webgpu/#example-6808af9c)

申请一个`"high-performance"`的 `GPUAdapter`:

```
const gpuAdapter = await navigator.gpu.requestAdapter({
    powerPreference: 'high-performance'
});
```

### 4.3. `GPU适配器`[](https://www.w3.org/TR/webgpu/#gpuadapter)

`GPU适配器(GPUAdapter)`封装了一个[适配器对象](https://www.w3.org/TR/webgpu/#adapter)，并且描述了它具备的能力(包括[特性](https://www.w3.org/TR/webgpu/#feature)和[限制](https://www.w3.org/TR/webgpu/#limit))。

使用`requestAdapter()`，可以生成一个`GPUAdapter`。

```
[Exposed=(Window, DedicatedWorker), SecureContext]
interface GPUAdapter {
    [SameObject] readonly attribute GPUSupportedFeatures features;
    [SameObject] readonly attribute GPUSupportedLimits limits;
    readonly attribute boolean isFallbackAdapter;

    Promise<GPUDevice> requestDevice(optional GPUDeviceDescriptor descriptor = {});
    Promise<GPUAdapterInfo> requestAdapterInfo(optional sequence<DOMString> unmaskHints = []);
};
```

`GPUAdapter`  有下面这些属性：

- `feature`，类型是[GPUSupportedFeatures](https://www.w3.org/TR/webgpu/#gpusupportedfeatures)，只读

  位于`this.[[adapter]].[[features]]`下的一组值

- `limits`，类型是[GPUSupportedLimits](https://www.w3.org/TR/webgpu/#gpusupportedlimits)，只读

  位于`this.[[adapter]].[[limits]]`下的值

- `isFallbackAdapter`，类型为[boolean](https://webidl.spec.whatwg.org/#idl-boolean)，只读

  返回`[[adapter]]`.`[[fallback]]`的值

`GPUAdapter` 有下列内置属性：

- `[[adapter]]`, 类型是[adapter](https://www.w3.org/TR/webgpu/#adapter)，只读

  即`GPUAdapter`引用的[adapter](https://www.w3.org/TR/webgpu/#adapter)实例。

`GPUAdapter`  有下列方法：

- `requestDevice(descriptor)`

  从[adapter](https://www.w3.org/TR/webgpu/#adapter)中申请获取一个[设备(device)](https://www.w3.org/TR/webgpu/#device)。

  这是一个一次性操作，如果一个设备被成功的返回了，适配器状态会变成[无效(invalid)](https://www.w3.org/TR/webgpu/#invalid)。

  **调用者:**  `GPUAdapter` this.

  **入参:**

  | 名称         | 类型                  | Nullable | Optional | 描述                      |
  | ------------ | --------------------- | -------- | -------- | ------------------------- |
  | `descriptor` | `GPUDeviceDescriptor` | ✘        | ✔        | 获取`GPUDevice`的描述配置 |

  **出参:**  `Promise`<`GPUDevice`>

  [内容时间线](https://www.w3.org/TR/webgpu/#content-timeline) 步骤:

  1.  将 `contentTimeline`  置为当前的  [内容时间线](https://www.w3.org/TR/webgpu/#content-timeline)
  2.  将 `promise` 赋值为 [一个新的 Promise](https://webidl.spec.whatwg.org/#a-new-promise)
  3.  将 `adapter` 置为 `this`.`[[adapter]]`
  4.  在当前的 [设备时间线](https://www.w3.org/TR/webgpu/#device-timeline)  执行  *初始化步骤*
  5.  返回  `promise`

  [设备时间线](https://www.w3.org/TR/webgpu/#device-timeline) `初始化步骤`:

  1.  如果下列需求任意一项不满足：

  - `descriptor`.`requiredFeatures` 集合中的值必须是`adapter`.`[[features]]`的子集。

  则执行`contentTimeline`中下列步骤并返回：

  [Content timeline](https://www.w3.org/TR/webgpu/#content-timeline) 步骤:

  1.  用`TypeError`错误 [Reject](https://webidl.spec.whatwg.org/#reject) `promise`。

  注意：如果浏览器完全无法识别一个特性名称（在`GPUFeatureName`定义），则会产生与此相同的错误。这将使浏览器在不支持某个特性时的行为 和 特定适配器不支持某个特性时 的行为保持一致。

  2.  如果下列需求任意一项不满足：

  - `descriptor`.`requiredLimits`中的每一个 key 的名字都需要在[supported limits](https://www.w3.org/TR/webgpu/#supported-limits)中.

  - 对于 [supported limits](https://www.w3.org/TR/webgpu/#supported-limits) 的键中的每个限制的`key`：设置 `value` 为 `descriptor`.`requiredLimits`[`key`]

    - `value` 不得 [超过](https://www.w3.org/TR/webgpu/#limit-better) `adapter`.`[[limits]]`中的限制值。

    - 如果限制的 [类型](https://www.w3.org/TR/webgpu/#limit-class) 是 [alignment](https://www.w3.org/TR/webgpu/#limit-class-alignment)， `value`  需要是 2 的正整数幂。

    然后执行`contentTimeline`后续的步骤并返回：

    [内容时间线](https://www.w3.org/TR/webgpu/#content-timeline) 步骤：

    - 用`OperationError`错误 [Reject](https://webidl.spec.whatwg.org/#reject) `promise`。

  1.  如果 `adapter` 是 [无效的(invalid)](https://www.w3.org/TR/webgpu/#invalid)，或者用户代理无法满足需求：

  1.  将 `device`  初始化为新的  [device](https://www.w3.org/TR/webgpu/#device)。

  1.  [丢失设备](https://www.w3.org/TR/webgpu/#lose-the-device)(`device`, `"unknown"`)

  注意：如果`adapter`还没准备好，这将使它[失效](https://www.w3.org/TR/webgpu/#invalid)

  注意：在大多数情况下，用户代理程序应该考虑发出开发者可见的警告。应用程序应该从`requestAdapter()`开始执行重新初始化逻辑。

  否则：

  1. 使用`descriptor`中描述的功能创建一个[新的设备](https://www.w3.org/TR/webgpu/#a-new-device)，并将其赋值给`device`。

  2. 使 `adapter.[[adapter]]` [失效](https://www.w3.org/TR/webgpu/#invalid)。

  ***

  1. 执行`contentTimeline`的下列步骤

  [内容时间线](https://www.w3.org/TR/webgpu/#content-timeline) 步骤：

  1. 使用一个新的类型为  `GPUDevice`的 `device` 作为 `promise` [Resolve](https://webidl.spec.whatwg.org/#resolve)  值。

  注意：如果设备已经丢失，因为适配器无法满足请求，在`promise` resolve 之前，`device`.`lost` 会已经被`resolve`。

- `requestAdapterInfo()`

  请求此`GPUAdapter`的`GPUAdapterInfo`。

  注意：适配器信息值返回一个 Promise，以便让用户代理程序在请求未屏蔽的值时有机会执行潜在的长时间运行的检查，例如在返回之前请求用户同意。但是，如果没有指定`unmaskHints`，则不应向用户显示任何对话框。

  **调用者:**  `GPUAdapter` this.

  **入参:**

  | 名称          | 类型                  | Nullable | Optional | 描述                                                       |
  | ------------- | --------------------- | -------- | -------- | ---------------------------------------------------------- |
  | `unmaskHints` | `sequence<DOMString>` | ✘        | ✔        | 如果可用，希望返回未屏蔽值的`GPUAdapterInfo`属性名称列表。 |

  **出参:** `Promise`<`GPUAdapterInfo`>

  [内容时间线](https://www.w3.org/TR/webgpu/#content-timeline) 步骤:

  1.  将 `promise` 赋值为 [一个新的 Promise](https://webidl.spec.whatwg.org/#a-new-promise)
  2.  将 `adapter` 置为 `this`.`[[adapter]]`
  3.  如果 this [相关的全局变量](https://html.spec.whatwg.org/multipage/webappapis.html#concept-relevant-global)被[短暂激活](https://html.spec.whatwg.org/multipage/interaction.html#transient-activation)，则将`hasActivation`置为`true`，否则为`false`
  4.  [并行](https://html.spec.whatwg.org/multipage/infrastructure.html#in-parallel)运行下列步骤：

  - 如果`unmaskHints`.length > `0`:

    - 如果`hasActivation`值为`false`，来自抛出`NotAllowedError`错误的`promise`的[reject](https://webidl.spec.whatwg.org/#reject)，终止这些步骤。

    - 如果可能的话，让`unmaskedKeys`成为`unmaskHints`中指定的字段的[列表](https://infra.spec.whatwg.org/#list)，用户代理程序决定要解除掩码的字段。

    注意：用户代理程序可以自由地使用其认为合适的任何方法来决定哪些字段需要解除掩码。

    - 将`unmaskedKeys` [添加](https://infra.spec.whatwg.org/#set-append) 到`adapter`.`[[unmaskedIdentifiers]]`

  4.  使用`adapter`创建一个[新的适配器信息](https://www.w3.org/TR/webgpu/#abstract-opdef-new-adapter-info) 并使用该信息 [Resolve](https://webidl.spec.whatwg.org/#resolve) `promise`。

  5.  返回`promise`

[Example 9](https://www.w3.org/TR/webgpu/#example-c8b5b8a0)

使用默认的特性和限制申请一个`GPUDevice`：

```
const gpuAdapter = await navigator.gpu.requestAdapter();
const gpuDevice = await gpuAdapter.requestDevice();
```

### 4.5. 案例[](https://www.w3.org/TR/webgpu/#initialization-examples)

[](https://www.w3.org/TR/webgpu/#example-abcf3590)
一个强大的，带有错误处理请求的 `GPUAdapter` 和 `GPUDevice` 的示例

```
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
```
