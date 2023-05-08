## 4. Initialization[](https://www.w3.org/TR/webgpu/#initialization)

### 4.1. navigator.gpu[](https://www.w3.org/TR/webgpu/#navigator-gpu)

A `GPU` object is available in the `Window` and `DedicatedWorkerGlobalScope` contexts through the `Navigator` and `WorkerNavigator` interfaces respectively and is exposed via `navigator.gpu`:

```
interface mixin NavigatorGPU {
    [SameObject, SecureContext] readonly attribute GPU gpu;
};
Navigator includes NavigatorGPU;
WorkerNavigator includes NavigatorGPU;
```

### 4.2. GPU[](https://www.w3.org/TR/webgpu/#gpu-interface)

`GPU` is the entry point to WebGPU.

```
[Exposed=(Window, DedicatedWorker), SecureContext]
interface GPU {
    Promise<GPUAdapter?> requestAdapter(optional GPURequestAdapterOptions options = {});
    GPUTextureFormat getPreferredCanvasFormat();
    [SameObject] readonly attribute WGSLLanguageFeatures wgslLanguageFeatures;
};
```

`GPU` has the following methods and attributes:

- `requestAdapter(options)`

  Requests an [adapter](https://www.w3.org/TR/webgpu/#adapter) from the user agent. The user agent chooses whether to return an adapter, and, if so, chooses according to the provided options.

  **Called on:**  `GPU` `this`.

  **Arguments:**

  | Parameter | Type                       | Nullable | Optional | Description                          |
  | --------- | -------------------------- | -------- | -------- | ------------------------------------ |
  | `options` | `GPURequestAdapterOptions` | ✘        | ✔        | Criteria used to select the adapter. |

  **Returns:**  `Promise`<`GPUAdapter`?>

  [Content timeline](https://www.w3.org/TR/webgpu/#content-timeline) steps:

  1.  Let `contentTimeline` be the current [Content timeline](https://www.w3.org/TR/webgpu/#content-timeline).
  1.  Let `promise` be [a new promise](https://webidl.spec.whatwg.org/#a-new-promise).
  1.  Issue the `initialization steps` on the [Device timeline](https://www.w3.org/TR/webgpu/#device-timeline) of `this`.
  1.  Return `promise`.

  [Device timeline](https://www.w3.org/TR/webgpu/#device-timeline) `initialization steps`:

  1.  Let `adapter` be `null`.

  1.  If the user agent chooses to return an adapter, it should:

      1.  Set `adapter` to a [valid](https://www.w3.org/TR/webgpu/#valid) [adapter](https://www.w3.org/TR/webgpu/#adapter), chosen according to the rules in [§ 4.2.2 Adapter Selection](https://www.w3.org/TR/webgpu/#adapter-selection) and the criteria in `options`, adhering to [§ 4.2.1 Adapter Capability Guarantees](https://www.w3.org/TR/webgpu/#adapter-capability-guarantees).

          The [supported limits](https://www.w3.org/TR/webgpu/#supported-limits) of the adapter must adhere to the requirements defined in [§ 3.6.2 Limits](https://www.w3.org/TR/webgpu/#limits).

      1.  If `adapter` meets the criteria of a [fallback adapter](https://www.w3.org/TR/webgpu/#fallback-adapter) set `adapter`.`[[fallback]]` to `true`.

  1.  Issue the subsequent steps on `contentTimeline`.

  [Content timeline](https://www.w3.org/TR/webgpu/#content-timeline) steps:

  1.  If `adapter` is not `null`:

      1.  [Resolve](https://webidl.spec.whatwg.org/#resolve) `promise` with a new `GPUAdapter` encapsulating `adapter`.

  1.  Otherwise, [Resolve](https://webidl.spec.whatwg.org/#resolve) `promise` with `null`.

- `getPreferredCanvasFormat()`

  Returns an optimal `GPUTextureFormat` for displaying 8-bit depth, standard dynamic range content on this system. Must only return `"rgba8unorm"` or `"bgra8unorm"`.

  The returned value can be passed as the `format` to `configure()` calls on a `GPUCanvasContext` to ensure the associated canvas is able to display its contents efficiently.

  NOTE: Canvases which are not displayed to the screen may or may not benefit from using this format.

  **Called on:**  `GPU` this.

  **Returns:**  `GPUTextureFormat`

  [Content timeline](https://www.w3.org/TR/webgpu/#content-timeline) steps:

  1.  Return either `"rgba8unorm"` or `"bgra8unorm"`, depending on which format is optimal for displaying WebGPU canvases on this system.

- `wgslLanguageFeatures`, of type [WGSLLanguageFeatures](https://www.w3.org/TR/webgpu/#gpuwgsllanguagefeatures), readonly

  The names of supported WGSL [language extensions](https://gpuweb.github.io/gpuweb/wgsl/#language-extension). Supported language extensions are automatically enabled.

[Adapters](https://www.w3.org/TR/webgpu/#adapter) **may** become [invalid](https://www.w3.org/TR/webgpu/#invalid) ("expire") at any time. Upon any change in the system’s state that could affect the result of any `requestAdapter()` call, the user agent **should** expire all previously-returned adapters. For example:

- A physical adapter is added/removed (via plug/unplug, driver update, hang recovery, etc.)
- The system’s power configuration has changed (laptop unplugged, power settings changed, etc.)

NOTE: User agents may choose to [expire](https://www.w3.org/TR/webgpu/#adapter-expire) adapters often, even when there has been no system state change (e.g. seconds or minutes after the adapter was created). This can help obfuscate real system state changes, and make developers more aware that calling `requestAdapter()` again is always necessary before calling `requestDevice()`. If an application does encounter this situation, standard device-loss recovery handling should allow it to recover.

[Examples7](https://www.w3.org/TR/webgpu/#example-78e28ede)Requesting a `GPUAdapter` with no hints:

```
const gpuAdapter = await navigator.gpu.requestAdapter();
```

#### 4.2.1. Adapter Capability Guarantees[](https://www.w3.org/TR/webgpu/#adapter-capability-guarantees)

Any `GPUAdapter` returned by `requestAdapter()` must provide the following guarantees:

- At least one of the following must be true:

  - `"texture-compression-bc"` is supported.
  - Both `"texture-compression-etc2"` and `"texture-compression-astc"` are supported.

- All supported limits must be either the [default](https://www.w3.org/TR/webgpu/#limit-default) value or [better](https://www.w3.org/TR/webgpu/#limit-better).

- All [alignment-class](https://www.w3.org/TR/webgpu/#limit-class-alignment) limits must be powers of 2.

- `maxBindingsPerBindGroup` must be must be ≥ ([max bindings per shader stage](https://www.w3.org/TR/webgpu/#max-bindings-per-shader-stage) × [max shader stages per pipeline](https://www.w3.org/TR/webgpu/#max-shader-stages-per-pipeline)), where:

  - max bindings per shader stage is (`maxSampledTexturesPerShaderStage` + `maxSamplersPerShaderStage` + `maxStorageBuffersPerShaderStage` + `maxStorageTexturesPerShaderStage` + `maxUniformBuffersPerShaderStage`).
  - max shader stages per pipeline is `2`, because a `GPURenderPipeline` supports both a vertex and fragment shader.

  NOTE: `maxBindingsPerBindGroup` does not reflect a fundamental limit; implementations should raise it to conform to this requirement, rather than lowering the other limits.

- `maxBindGroups` must be ≤ `maxBindGroupsPlusVertexBuffers`.

- `maxVertexBuffers` must be ≤ `maxBindGroupsPlusVertexBuffers`.

- `minUniformBufferOffsetAlignment` and `minStorageBufferOffsetAlignment` must both be ≥ 32 bytes.

  NOTE: 32 bytes would be the alignment of `vec4<f64>`. See [WebGPU Shading Language § 13.4.1 Alignment and Size](https://www.w3.org/TR/WGSL/#alignment-and-size).

- `maxUniformBufferBindingSize` must be ≤ `maxBufferSize`.

- `maxStorageBufferBindingSize` must be ≤ `maxBufferSize`.

- `maxStorageBufferBindingSize` must be a multiple of 4 bytes.

- `maxVertexBufferArrayStride` must be a multiple of 4 bytes.

- `maxComputeWorkgroupSizeX` must be ≤ `maxComputeInvocationsPerWorkgroup`.

- `maxComputeWorkgroupSizeY` must be ≤ `maxComputeInvocationsPerWorkgroup`.

- `maxComputeWorkgroupSizeZ` must be ≤ `maxComputeInvocationsPerWorkgroup`.

- `maxComputeInvocationsPerWorkgroup` must be ≤ `maxComputeWorkgroupSizeX` × `maxComputeWorkgroupSizeY` × `maxComputeWorkgroupSizeZ`.

#### 4.2.2. Adapter Selection[](https://www.w3.org/TR/webgpu/#adapter-selection)

`GPURequestAdapterOptions` provides hints to the user agent indicating what configuration is suitable for the application.

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

`GPURequestAdapterOptions` has the following members:

- `powerPreference`, of type [GPUPowerPreference](https://www.w3.org/TR/webgpu/#enumdef-gpupowerpreference)

  Optionally provides a hint indicating what class of [adapter](https://www.w3.org/TR/webgpu/#adapter) should be selected from the system’s available adapters.

  The value of this hint may influence which adapter is chosen, but it must not influence whether an adapter is returned or not.

  NOTE: The primary utility of this hint is to influence which GPU is used in a multi-GPU system. For instance, some laptops have a low-power integrated GPU and a high-performance discrete GPU. This hint may also affect the power configuration of the selected GPU to match the requested power preference.

  NOTE: Depending on the exact hardware configuration, such as battery status and attached displays or removable GPUs, the user agent may select different [adapters](https://www.w3.org/TR/webgpu/#adapter) given the same power preference. Typically, given the same hardware configuration and state and `powerPreference`, the user agent is likely to select the same adapter.

  It must be one of the following values:

  - `undefined` (or not present)

    Provides no hint to the user agent.

  - `"low-power"`

    Indicates a request to prioritize power savings over performance.

    NOTE: Generally, content should use this if it is unlikely to be constrained by drawing performance; for example, if it renders only one frame per second, draws only relatively simple geometry with simple shaders, or uses a small HTML canvas element. Developers are encouraged to use this value if their content allows, since it may significantly improve battery life on portable devices.

  - `"high-performance"`

    Indicates a request to prioritize performance over power consumption.

    NOTE: By choosing this value, developers should be aware that, for [devices](https://www.w3.org/TR/webgpu/#device) created on the resulting adapter, user agents are more likely to force device loss, in order to save power by switching to a lower-power adapter. Developers are encouraged to only specify this value if they believe it is absolutely necessary, since it may significantly decrease battery life on portable devices.

- `forceFallbackAdapter`, of type [boolean](https://webidl.spec.whatwg.org/#idl-boolean), defaulting to `false`

  When set to `true` indicates that only a [fallback adapter](https://www.w3.org/TR/webgpu/#fallback-adapter) may be returned. If the user agent does not support a [fallback adapter](https://www.w3.org/TR/webgpu/#fallback-adapter), will cause `requestAdapter()` to resolve to `null`.

  NOTE: `requestAdapter()` may still return a [fallback adapter](https://www.w3.org/TR/webgpu/#fallback-adapter) if `forceFallbackAdapter` is set to `false` and either no other appropriate [adapter](https://www.w3.org/TR/webgpu/#adapter) is available or the user agent chooses to return a [fallback adapter](https://www.w3.org/TR/webgpu/#fallback-adapter). Developers that wish to prevent their applications from running on [fallback adapters](https://www.w3.org/TR/webgpu/#fallback-adapter) should check the `GPUAdapter`.`isFallbackAdapter` attribute prior to requesting a `GPUDevice`.

[Examples 8](https://www.w3.org/TR/webgpu/#example-6808af9c)Requesting a `"high-performance"` `GPUAdapter`:

```
const gpuAdapter = await navigator.gpu.requestAdapter({
    powerPreference: 'high-performance'
});
```

### 4.3. `GPUAdapter`[](https://www.w3.org/TR/webgpu/#gpuadapter)

A `GPUAdapter` encapsulates an [adapter](https://www.w3.org/TR/webgpu/#adapter), and describes its capabilities ([features](https://www.w3.org/TR/webgpu/#feature) and [limits](https://www.w3.org/TR/webgpu/#limit)).

To get a `GPUAdapter`, use `requestAdapter()`.

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

`GPUAdapter` has the following attributes:

- `features`, of type [GPUSupportedFeatures](https://www.w3.org/TR/webgpu/#gpusupportedfeatures), readonly

  The set of values in `this`.`[[adapter]]`.`[[features]]`.

- `limits`, of type [GPUSupportedLimits](https://www.w3.org/TR/webgpu/#gpusupportedlimits), readonly

  The limits in `this`.`[[adapter]]`.`[[limits]]`.

- `isFallbackAdapter`, of type [boolean](https://webidl.spec.whatwg.org/#idl-boolean), readonly

  Returns the value of `[[adapter]]`.`[[fallback]]`.

`GPUAdapter` has the following internal slots:

- `[[adapter]]`, of type [adapter](https://www.w3.org/TR/webgpu/#adapter), readonly

  The [adapter](https://www.w3.org/TR/webgpu/#adapter) to which this `GPUAdapter` refers.

`GPUAdapter` has the following methods:

- `requestDevice(descriptor)`

  Requests a [device](https://www.w3.org/TR/webgpu/#device) from the [adapter](https://www.w3.org/TR/webgpu/#adapter).

  This is a one-time action: if a device is returned successfully, the adapter becomes [invalid](https://www.w3.org/TR/webgpu/#invalid).

  **Called on:**  `GPUAdapter` this.

  **Arguments:**

  | Parameter    | Type                  | Nullable | Optional | Description                                |
  | ------------ | --------------------- | -------- | -------- | ------------------------------------------ |
  | `descriptor` | `GPUDeviceDescriptor` | ✘        | ✔        | Description of the `GPUDevice` to request. |

  **Returns:**  `Promise`<`GPUDevice`>

  [Content timeline](https://www.w3.org/TR/webgpu/#content-timeline) steps:

  1.  Let `contentTimeline` be the current [Content timeline](https://www.w3.org/TR/webgpu/#content-timeline).
  1.  Let `promise` be [a new promise](https://webidl.spec.whatwg.org/#a-new-promise).
  1.  Let `adapter` be `this`.`[[adapter]]`.
  1.  Issue the `initialization steps` to the [Device timeline](https://www.w3.org/TR/webgpu/#device-timeline) of `this`.
  1.  Return `promise`.

  [Device timeline](https://www.w3.org/TR/webgpu/#device-timeline) `initialization steps`:

  1.  If any of the following requirements are unmet:

      - The set of values in `descriptor`.`requiredFeatures` must be a subset of those in `adapter`.`[[features]]`.

      Then issue the following steps on `contentTimeline` and return:

      [Content timeline](https://www.w3.org/TR/webgpu/#content-timeline) steps:

      1.  [Reject](https://webidl.spec.whatwg.org/#reject) `promise` with a `TypeError`.

      NOTE: This is the same error that is produced if a feature name isn’t known by the browser at all (in its `GPUFeatureName` definition). This converges the behavior when the browser doesn’t support a feature with the behavior when a particular adapter doesn’t support a feature.

  1.  If any of the following requirements are unmet:

      - Each key in `descriptor`.`requiredLimits` must be the name of a member of [supported limits](https://www.w3.org/TR/webgpu/#supported-limits).

      - For each limit name `key` in the keys of [supported limits](https://www.w3.org/TR/webgpu/#supported-limits): Let `value` be `descriptor`.`requiredLimits`[`key`].

        - `value` must be no [better](https://www.w3.org/TR/webgpu/#limit-better) than the value of that limit in `adapter`.`[[limits]]`.
        - If the limit’s [class](https://www.w3.org/TR/webgpu/#limit-class) is [alignment](https://www.w3.org/TR/webgpu/#limit-class-alignment), `value` must be a power of 2.

      Then issue the following steps on `contentTimeline` and return:

      [Content timeline](https://www.w3.org/TR/webgpu/#content-timeline) steps:

      1.  [Reject](https://webidl.spec.whatwg.org/#reject) `promise` with an `OperationError`.

  1.  If `adapter` is [invalid](https://www.w3.org/TR/webgpu/#invalid), or the user agent otherwise cannot fulfill the request:

      1.  Let `device` be a new [device](https://www.w3.org/TR/webgpu/#device).

      1.  [Lose the device](https://www.w3.org/TR/webgpu/#lose-the-device)(`device`, `"unknown"`).

          NOTE: This makes `adapter` [invalid](https://www.w3.org/TR/webgpu/#invalid), if it wasn’t already.

          NOTE: User agents should consider issuing developer-visible warnings in most or all cases when this occurs. Applications should perform reinitialization logic starting with `requestAdapter()`.

      Otherwise:

      1.  Let `device` be [a new device](https://www.w3.org/TR/webgpu/#a-new-device) with the capabilities described by `descriptor`.
      1.  Make `adapter`.`[[adapter]]` [invalid](https://www.w3.org/TR/webgpu/#invalid).

  1.  Issue the subsequent steps on `contentTimeline`.

  [Content timeline](https://www.w3.org/TR/webgpu/#content-timeline) steps:

  1.  [Resolve](https://webidl.spec.whatwg.org/#resolve) `promise` with a new `GPUDevice` object `device`.

      NOTE: If the device is already lost because the adapter could not fulfill the request, `device`.`lost` has already resolved before `promise` resolves.

- `requestAdapterInfo()`

  Requests the `GPUAdapterInfo` for this `GPUAdapter`.

  NOTE: Adapter info values are returned with a Promise to give user agents an opportunity to perform potentially long-running checks when requesting unmasked values, such as asking for user consent before returning. If no `unmaskHints` are specified, however, no dialogs should be displayed to the user.

  **Called on:**  `GPUAdapter` `this`.

  **Arguments:**

  | Parameter     | Type                  | Nullable | Optional | Description                                                                                    |
  | ------------- | --------------------- | -------- | -------- | ---------------------------------------------------------------------------------------------- |
  | `unmaskHints` | `sequence<DOMString>` | ✘        | ✔        | A list of `GPUAdapterInfo` attribute names for which unmasked values are desired if available. |

  **Returns:**  `Promise`<`GPUAdapterInfo`>

  [Content timeline](https://www.w3.org/TR/webgpu/#content-timeline) steps:

  1.  Let `promise` be [a new promise](https://webidl.spec.whatwg.org/#a-new-promise).

  1.  Let `adapter` be `this`.`[[adapter]]`.

  1.  Let `hasActivation` be `true` if the [relevant global object](https://html.spec.whatwg.org/multipage/webappapis.html#concept-relevant-global) for `this` has [transient activation](https://html.spec.whatwg.org/multipage/interaction.html#transient-activation), and `false` otherwise.

  1.  Run the following steps [in parallel](https://html.spec.whatwg.org/multipage/infrastructure.html#in-parallel):

      1.  If `unmaskHints`.length > `0`:

          1.  If `hasActivation` is `false` [reject](https://webidl.spec.whatwg.org/#reject) `promise` with a `NotAllowedError` and abort these steps.

          1.  Let `unmaskedKeys` be a [list](https://infra.spec.whatwg.org/#list) of the fields specified in `unmaskHints` which the user agent decides to unmask, if any.

              NOTE: The user agent is free to use any method it deems appropriate to decide which fields to unmask.

          1.  [Append](https://infra.spec.whatwg.org/#set-append) `unmaskedKeys` to `adapter`.`[[unmaskedIdentifiers]]`.

      1.  [Resolve](https://webidl.spec.whatwg.org/#resolve) `promise` with a [new adapter info](https://www.w3.org/TR/webgpu/#abstract-opdef-new-adapter-info) for `adapter`.

  1.  Return `promise`.

[](https://www.w3.org/TR/webgpu/#example-c8b5b8a0)Requesting a `GPUDevice` with default features and limits:

```
const gpuAdapter = await navigator.gpu.requestAdapter();
const gpuDevice = await gpuAdapter.requestDevice();
```

#### 4.3.1. `GPUDeviceDescriptor`[](https://www.w3.org/TR/webgpu/#gpudevicedescriptor)

`GPUDeviceDescriptor` describes a device request.

```
dictionary GPUDeviceDescriptor
         : GPUObjectDescriptorBase {
    sequence<GPUFeatureName> requiredFeatures = [];
    record<DOMString, GPUSize64> requiredLimits = {};
    GPUQueueDescriptor defaultQueue = {};
};
```

`GPUDeviceDescriptor` has the following members:

- `requiredFeatures`, of type sequence<[GPUFeatureName](https://www.w3.org/TR/webgpu/#gpufeaturename)>, defaulting to `[]`

  Specifies the [features](https://www.w3.org/TR/webgpu/#feature) that are required by the device request. The request will fail if the adapter cannot provide these features.

  Exactly the specified set of features, and no more or less, will be allowed in validation of API calls on the resulting device.

- `requiredLimits`, of type record<[DOMString](https://webidl.spec.whatwg.org/#idl-DOMString), [GPUSize64](https://www.w3.org/TR/webgpu/#typedefdef-gpusize64)>, defaulting to `{}`

  Specifies the [limits](https://www.w3.org/TR/webgpu/#limit) that are required by the device request. The request will fail if the adapter cannot provide these limits.

  Each key must be the name of a member of [supported limits](https://www.w3.org/TR/webgpu/#supported-limits). Exactly the specified limits, and no [better](https://www.w3.org/TR/webgpu/#limit-better) or worse, will be allowed in validation of API calls on the resulting device.

- `defaultQueue`, of type [GPUQueueDescriptor](https://www.w3.org/TR/webgpu/#gpuqueuedescriptor), defaulting to `{}`

  The descriptor for the default `GPUQueue`.

[](https://www.w3.org/TR/webgpu/#example-8a22c3e3)Requesting a `GPUDevice` with the `"texture-compression-astc"` feature if supported:

```
const gpuAdapter = await navigator.gpu.requestAdapter();

const requiredFeatures = [];
if (gpuAdapter.features.has('texture-compression-astc')) {
    requiredFeatures.push('texture-compression-astc')
}

const gpuDevice = await gpuAdapter.requestDevice({
    requiredFeatures
});
```

##### 4.3.1.1. `GPUFeatureName`[](https://www.w3.org/TR/webgpu/#gpufeaturename)

Each `GPUFeatureName` identifies a set of functionality which, if available, allows additional usages of WebGPU that would have otherwise been invalid.

```
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
```

### 4.4. `GPUDevice`[](https://www.w3.org/TR/webgpu/#gpudevice)

A `GPUDevice` encapsulates a [device](https://www.w3.org/TR/webgpu/#device) and exposes the functionality of that device.

`GPUDevice` is the top-level interface through which [WebGPU interfaces](https://www.w3.org/TR/webgpu/#webgpu-interface) are created.

To get a `GPUDevice`, use `requestDevice()`.

```
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
```

`GPUDevice` has the following attributes:

- `features`, of type [GPUSupportedFeatures](https://www.w3.org/TR/webgpu/#gpusupportedfeatures), readonly

  A set containing the `GPUFeatureName` values of the features supported by the device (i.e. the ones with which it was created).

- `limits`, of type [GPUSupportedLimits](https://www.w3.org/TR/webgpu/#gpusupportedlimits), readonly

  Exposes the limits supported by the device (which are exactly the ones with which it was created).

- `queue`, of type [GPUQueue](https://www.w3.org/TR/webgpu/#gpuqueue), readonly

  The primary `GPUQueue` for this device.

The `[[device]]` for a `GPUDevice` is the [device](https://www.w3.org/TR/webgpu/#device) that the `GPUDevice` refers to.

`GPUDevice` has the methods listed in its WebIDL definition above. Those not defined here are defined elsewhere in this document.

- `destroy()`

  Destroys the [device](https://www.w3.org/TR/webgpu/#device), preventing further operations on it. Outstanding asynchronous operations will fail.

  NOTE: It is valid to destroy a device multiple times.

  **Called on:**  `GPUDevice` `this`.

  [Content timeline](https://www.w3.org/TR/webgpu/#content-timeline) steps:

  1.  `unmap()` all `GPUBuffer`s from this device.
  1.  Issue the subsequent steps on the [Device timeline](https://www.w3.org/TR/webgpu/#device-timeline) of `this`.

  [Device timeline](https://www.w3.org/TR/webgpu/#device-timeline) steps:

  1.  Once all currently-enqueued operations on any queue on this device are completed, issue the subsequent steps on the current timeline.

  <!---->

  1.  [Lose the device](https://www.w3.org/TR/webgpu/#lose-the-device)(`this`.`[[device]]`, `"destroyed"`).

  NOTE: Since no further operations can be enqueued on this device, implementations can abort outstanding asynchronous operations immediately and free resource allocations, including mapped memory that was just unmapped.

A `GPUDevice`'s allowed buffer usages are:

- Always allowed: `MAP_READ`, `MAP_WRITE`, `COPY_SRC`, `COPY_DST`, `INDEX`, `VERTEX`, `UNIFORM`, `STORAGE`, `INDIRECT`, `QUERY_RESOLVE`

A `GPUDevice`'s allowed texture usages are:

- Always allowed: `COPY_SRC`, `COPY_DST`, `TEXTURE_BINDING`, `STORAGE_BINDING`, `RENDER_ATTACHMENT`

### 4.5. Example[](https://www.w3.org/TR/webgpu/#initialization-examples)

[](https://www.w3.org/TR/webgpu/#example-abcf3590)A more robust example of requesting a `GPUAdapter` and `GPUDevice` with error handling:

```
let gpuDevice = null;

async function initializeWebGPU() {
    // Check to ensure the user agent supports WebGPU.
    if (!('gpu' in navigator)) {
        console.error("User agent doesn’t support WebGPU.");
        return false;
    }

    // Request an adapter.
    const gpuAdapter = await navigator.gpu.requestAdapter();

    // requestAdapter may resolve with null if no suitable adapters are found.
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
