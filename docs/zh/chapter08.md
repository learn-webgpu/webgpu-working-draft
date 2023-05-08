### 8.1. GPUBindGroupLayout

GPUBindGroupLayout 定义了一组资源在 GPUBindGroup 中的绑定方式，以及在着色器阶段中它们的可访问性。

```WebIDL
[Exposed=(Window, DedicatedWorker), SecureContext]
interface GPUBindGroupLayout {
};
```

GPUBindGroupLayout 包含了 GPUObjectBase，它有以下内部属性:

- `[[descriptor]]`: 类型为 `GPUBindGroupLayoutDescriptor`。

### 8.1.1. 创建绑定组布局

通过 GPUDevice.createBindGroupLayout() 方法创建 GPUBindGroupLayout。

```WebIDL
dictionary GPUBindGroupLayoutDescriptor : GPUObjectDescriptorBase {
  required sequence<GPUBindGroupLayoutEntry> entries;
};

dictionary GPUBindGroupLayoutEntry {
  required GPUIndex32 binding;
  required GPUShaderStageFlags visibility;

  GPUBufferBindingLayout buffer;
  GPUSamplerBindingLayout sampler;
  GPUTextureBindingLayout texture;
  GPUStorageTextureBindingLayout storageTexture;
  GPUExternalTextureBindingLayout externalTexture;
};
```

- `GPUBindGroupLayoutDescriptor` 描述符用于定义 GPUBindGroupLayout。
- `GPUBindGroupLayoutEntry` 用于描述要包含在 GPUBindGroupLayout 中的单个着色器资源绑定。

`GPUBindGroupLayoutEntry` 字典包含以下成员：

- `binding`，类型为 `GPUIndex32`。

GPUBindGroupLayout 中资源绑定的唯一标识符，对应于 `GPUBindGroupEntry.binding` 和 `GPUShaderModule` 中的 `@binding` 属性。

- `visibility`，类型为 `GPUShaderStageFlags`。

`GPUShaderStage` 的成员的位集。每个设定位表示 `GPUBindGroupLayoutEntry` 的资源将可以在关联的着色器阶段中被访问。

- `buffer`，类型为 `GPUBufferBindingLayout`。

如果提供，则表示此 `GPUBindGroupLayoutEntry` 的绑定资源类型为 `GPUBufferBinding`。

- `sampler`，类型为 `GPUSamplerBindingLayout`。

如果提供，则表示此 `GPUBindGroupLayoutEntry` 的绑定资源类型为 `GPUSampler`。

- `texture`，类型为 `GPUTextureBindingLayout`。

如果提供，则表示此 `GPUBindGroupLayoutEntry` 的绑定资源类型为 `GPUTextureView`。

- `storageTexture`，类型为 `GPUStorageTextureBindingLayout`。

如果提供，则表示此 `GPUBindGroupLayoutEntry` 的绑定资源类型为 `GPUTextureView`。

- `externalTexture`，类型为 `GPUExternalTextureBindingLayout`。

如果提供，则表示此 `GPUBindGroupLayoutEntry` 的绑定资源类型为 `GPUExternalTexture`。

```WebIDL
typedef [EnforceRange] unsigned long GPUShaderStageFlags;

[Exposed=(Window, DedicatedWorker), SecureContext]
namespace GPUShaderStage {
  const GPUFlagsConstant VERTEX   = 0x1;
  const GPUFlagsConstant FRAGMENT = 0x2;
  const GPUFlagsConstant COMPUTE  = 0x4;
};
```

`GPUShaderStage` 包含以下标志，用于描述对应的 `GPUBindGroupLayoutEntry` 的 `GPUBindGroupEntry` 将可被哪些着色器阶段访问：

- `VERTEX`

绑定组条目将可被顶点着色器访问。

- `FRAGMENT`

绑定组条目将可被片段着色器访问。

- `COMPUTE`

绑定组条目将可被计算着色器访问。

`GPUBindGroupLayoutEntry` 的 `binding` 成员通过其所定义的 `GPUBindGroupLayoutEntry` 成员来确定：`buffer`、`sampler`、`texture`、`storageTexture` 或 `externalTexture`。对于任何给定的 `GPUBindGroupLayoutEntry`，只能定义其中一个成员。每个成员都有一个关联的 `GPUBindingResource` 类型，每个绑定类型都有一个关联的内部



这个表格显示了支持的绑定类型和它们的使用方式：

| Binding member  | Resource type          | Binding type                                      | Binding usage        |
|-----------------|------------------------|--------------------------------------------------|----------------------|
| buffer          | GPUBufferBinding       | "uniform"                                         | `constant`           |
|                 |                        | "storage"                                         | `storage`            |
|                 |                        | "read-only-storage"                               | `storage-read`       |
| sampler         | GPUSampler             | "filtering"                                       | `constant`           |
|                 |                        | "non-filtering"                                   |                      |
|                 |                        | "comparison"                                      |                      |
| texture         | GPUTextureView         | "float"                                           | `constant`           |
|                 |                        | "unfilterable-float"                              |                      |
|                 |                        | "depth"                                           |                      |
|                 |                        | "sint"                                            |                      |
|                 |                        | "uint"                                            |                      |
| storageTexture  | GPUTextureView         | "write-only"                                      | `storage`            |
| externalTexture | GPUExternalTexture     |                                                  | `constant`           |

---

描述了在 GPUBindGroupLayout 中的每个条目所使用的插槽数量，以及这些插槽对应的限制。具体如下：

- 如果条目数量超出了支持的 binding slot 限制，则列表中的 GPUBindGroupLayoutEntry 值数会超出限制。每个 entry 可以使用多个插槽，用于多个各自的限制。

1. 对于 `entries` 中的每个 `entry`：

- 如果 `entry.buffer?.type` 是 `"uniform"`，并且 `entry.buffer?.hasDynamicOffset` 为 `true`，则将考虑使用 1 个 `maxDynamicUniformBuffersPerPipelineLayout` 插槽。
- 如果 `entry.buffer?.type` 是 `"storage"`，并且 `entry.buffer?.hasDynamicOffset` 为 `true`，则将考虑使用 1 个 `maxDynamicStorageBuffersPerPipelineLayout` 插槽。

2. 对于每个着色器阶段 `stage`（`VERTEX`、`FRAGMENT`、`COMPUTE`）：

> 1.  对于 `entry.visibility` 包括 `stage` 的 `entries` 中的每个 `entry`：
>   - 如果 `entry.buffer?.type` 是 `"uniform"`，则将考虑使用 1 个 `maxUniformBuffersPerShaderStage` 插槽。
>   - 如果 `entry.buffer?.type` 是 `"storage"` 或 `"read-only-storage"`，则将考虑使用 1 个 `maxStorageBuffersPerShaderStage` 插槽。
>   - 如果 `entry.sampler` 存在，则将考虑使用 1 个 `maxSamplersPerShaderStage` 插槽。
>   - 如果 `entry.texture` 存在，则将考虑使用 1 个 `maxSampledTexturesPerShaderStage` 插槽。
>   - 如果 `entry.storageTexture` 存在，则将考虑使用 1 个 `maxStorageTexturesPerShaderStage` 插槽。
>   - 如果 `entry.externalTexture` 存在，则将考虑使用 4 个 `maxSampledTexturesPerShaderStage` 插槽、1 个 `maxSamplersPerShaderStage` 插槽和 1 个 `maxUniformBuffersPerShaderStage` 插槽。


```
enum GPUBufferBindingType {
    "uniform",
    "storage",
    "read-only-storage",
};

dictionary GPUBufferBindingLayout {
    GPUBufferBindingType type = "uniform";
    boolean hasDynamicOffset = false;
    GPUSize64 minBindingSize = 0;
};
```
GPUBufferBindingLayout 字典具有以下成员：

- `type`，类型为 `GPUBufferBindingType`，默认为 `"uniform"`，指定绑定到该绑定点的缓冲区所需的类型。
- `hasDynamicOffset`，类型为布尔型，默认值为 `false`，指示此绑定是否需要使用动态偏移量。
- `minBindingSize`，类型为 `GPUSize64`，默认为 `0`，指示与此绑定点一起使用的缓冲区绑定的最小大小。在创建 `BindGroup` 时，始终会根据此大小验证绑定。如果该值不为 `0`，则在管道创建时会验证该值 ≥ 变量的最小缓冲区绑定大小。如果它为 `0`，则管道创建将忽略它，而是绘制/调度命令将验证 `GPUBindGroup` 中的每个绑定是否满足变量的最小缓冲区绑定大小。
>注意:理论上也可以针对早期验证指定的其他与绑定相关的字段（例如 `sampleType` 和格式）进行类似的执行时验证，但这样的执行时验证可能会很昂贵或过于复杂，因此仅适用于 `minBindingSize`，因为它预计对人体工程学影响最大。

```
enum GPUSamplerBindingType {
    "filtering",
    "non-filtering",
    "comparison",
};

dictionary GPUSamplerBindingLayout {
    GPUSamplerBindingType type = "filtering";
};
```

`GPUSamplerBindingLayout` 字典具有以下成员：

- `type`，类型为 `GPUSamplerBindingType`，默认值为 `"filtering"`，指示绑定到此绑定点的采样器的要求类型。

```
enum GPUTextureSampleType {
    "float",
    "unfilterable-float",
    "depth",
    "sint",
    "uint",
};

dictionary GPUTextureBindingLayout {
    GPUTextureSampleType sampleType = "float";
    GPUTextureViewDimension viewDimension = "2d";
    boolean multisampled = false;
};
```

`GPUTextureBindingLayout`字典有以下成员：

**`sampleType`** **，类型为** **[GPUTexture SampleType](https://www.w3.org/TR/2023/WD-webgpu-20230425/#enumdef-gputexturesampletype)** **，默认为** **`“浮动”`**

指示绑定到此绑定的纹理视图所需的类型。

**`viewDimension`** **，类型为** **[GPUTextureViewDimension](https://www.w3.org/TR/2023/WD-webgpu-20230425/#enumdef-gputextureviewdimension)** **，默认为** **`"2d"`**

指示绑定到此绑定的纹理视图所需的`维度`。

**`多重采样`** **，** **[布尔类型](https://webidl.spec.whatwg.org/#idl-boolean)** **，默认为** **`false`**

指示绑定到此绑定的纹理视图是否必须进行多采样。

```
enum GPUStorageTextureAccess {
    "write-only" ,
};

dictionary GPUStorageTextureBindingLayout {
    GPUStorageTextureAccess access = "write-only";
    required GPUTextureFormat format;
    GPUTextureViewDimension viewDimension = "2d";
};
```

`GPUStorageTextureBindingLayout`字典有以下成员：

**`access`** **，类型为** **[GPUStorageTexture Access](https://www.w3.org/TR/2023/WD-webgpu-20230425/#enumdef-gpustoragetextureaccess)** **，默认为** **`"write-only"`**

此绑定的访问模式，指示易读性和可写性。

注意：目前只有一种访问模式，`“只写”`，但这将在未来扩展。

**`格式`** **，类型为** **[GPUTextureFormat](https://www.w3.org/TR/2023/WD-webgpu-20230425/#enumdef-gputextureformat)**

绑定到此绑定的纹理视图所需的`格式`。

**`viewDimension`** **，类型为** **[GPUTextureViewDimension](https://www.w3.org/TR/2023/WD-webgpu-20230425/#enumdef-gputextureviewdimension)** **，默认为** **`"2d"`**

指示绑定到此绑定的纹理视图所需的`维度`。

```
dictionary GPUExternalTextureBindingLayout {
};
```


一个`GPUBindGroupLayout`对象有以下内部槽：

**`[[entryMap]]`** **，类型为** **[有序映射](https://infra.spec.whatwg.org/#ordered-map)** **<** **`GPUSize32`** **，** **`GPUBindGroupLayoutEntry`** **>**

`绑定索引的映射，指向GPUBindGroupLayoutEntrys，此``GPUBindGroupLayout`描述。

**`[[DynamicOffsetCount]]`** **，类型为** **`GPUSize32`**

此`GPUBindGroupLayout`中具有动态偏移量的缓冲区绑定数。

**`[[expsivePipeline]]`** **，类型为** **`GPUPipelineBase`** **？，最初** **`为null`**

创建此`GPUBindGroupLayout`的管道，如果它是作为[默认管道布局](https://www.w3.org/TR/2023/WD-webgpu-20230425/#default-pipeline-layout)的一部分创建的。如果不`为空``，则使用此``GPUBindGroupLayout创建`的GPUBindGroup只能与指定的`GPUPipelineBase`一起使用。

**`创建绑定组布局（描述符）`**

创建一个`GPUBindGroupLayout`。

**调用：** `GPUDevice`this.

**论据：**

| Parameter  | Type                                                                                                                | Nullable | Optional | Description                                                                                                           |
| ---------- | ------------------------------------------------------------------------------------------------------------------- | -------- | -------- | --------------------------------------------------------------------------------------------------------------------- |
| descriptor | [GPUBindGroupLayoutDescriptor](https://www.w3.org/TR/2023/WD-webgpu-20230425/#dictdef-gpubindgrouplayoutdescriptor) | ✘        | ✘        | Description of the [GPUBindGroupLayout](https://www.w3.org/TR/2023/WD-webgpu-20230425/#gpubindgrouplayout) to create. |

**返回：** `GPUBindGroupLayout`

[内容时间线](https://www.w3.org/TR/2023/WD-webgpu-20230425/#content-timeline)步骤：

1.  对于描述符中的每个`GPUBindGroupLayoutEntry`条目。`条目`：

    1.  如果[提供](https://infra.spec.whatwg.org/#map-exists)了`entry. storageTexture`：

        1.  [？](https://tc39.es/ecma262/#sec-returnifabrupt-shorthands) [验证纹理格式所需功能](https://www.w3.org/TR/2023/WD-webgpu-20230425/#abstract-opdef-validate-texture-format-required-features)的条目。`存储纹理`。`格式化`与此。`[[设备]]`。

1.  让布局成为一个新的`GPUBindGroupLayout`对象。

1.  在this的[设备时间线上](https://www.w3.org/TR/2023/WD-webgpu-20230425/#device-timeline)发出初始化步骤。

1.  返回布局。

[设备时间线](https://www.w3.org/TR/2023/WD-webgpu-20230425/#device-timeline)初始化步骤：

1.  如果以下任一条件不满足，[将生成一个验证错误](https://www.w3.org/TR/2023/WD-webgpu-20230425/#abstract-opdef-generate-a-validation-error)，使布局[无效](https://www.w3.org/TR/2023/WD-webgpu-20230425/#invalid)，并停止。

    *   这是[有效的](https://www.w3.org/TR/2023/WD-webgpu-20230425/#valid)。

    *   让限制是这样的。`[[设备]]`。`[[限制]]`。

    *   描述符中每个条目的`绑定`是唯一的。

    *   描述符中每个条目的`绑定`必须是<`限制. maxBindingsPerBindGroup`。

    *   描述符。`条目`不得[超过绑定槽限制](https://www.w3.org/TR/2023/WD-webgpu-20230425/#exceeds-the-binding-slot-limits)的限制。

    *   对于描述符中的每个`GPUBindGroupLayoutEntry`条目。`条目`：

        -   正是其中的一个条目。`缓冲区`，条目。`采样器`，条目。`纹理`和条目。`存储纹理`被[提供](https://infra.spec.whatwg.org/#map-exists)。

        -   entry.`可见性`仅包含`GPUShaderStage`中定义的位。
        -   如果entry.`可见性`包括`VERTEX`：

            -   条目。`缓冲区`？。`类型`不能是`“存储”`。
            -   entry.`storageTexture`？。`访问`不能是`“只写”的`。
        -   如果入口.`纹理`？.`多采样`为`真`：

            -   entry.`纹理`.`viewDimension`是`"2d"`。
            -   entry.`纹理`.`sampleType`不是`"浮点数"`。

        -   如果[提供](https://infra.spec.whatwg.org/#map-exists)了`entry. storageTexture`：
            -   entry.`storageTexture`.`viewDimension`不是`"多维数据集"`或`"多维数据集数组"`。
            -   entry.`storageTexture`.`format`必须是能够支持存储使用的格式。

1.  设置布局。`[[描述符]]`到描述符。

1.  设置布局。`[[DynamicOffsetCount]]`为描述符中[提供](https://infra.spec.whatwg.org/#map-exists)了`缓冲区`和`缓冲区`的条目数。`hasDynamicOffset`为`真`。

1.  对于描述符中的每个`GPUBindGroupLayoutEntry`条目。`条目`：

    1.  插入进入布局。`[[entryMap]]`与条目的关键。`绑定`。

## 8.1.2. 兼容性

当且仅当以下所有条件均满足时，`GPUBindGroupLayout` 中的两个对象 `a` 和 `b` 被视为组等价：

- `a.[[exclusivePipeline]] == b.[[exclusivePipeline]]`
- 对于任何绑定编号 `binding`，满足以下一个条件之一：
  - 既缺失于 `a.[[entryMap]]` 中，也缺失于 `b.[[entryMap]]` 中。
  - `a.[[entryMap]][binding] == b.[[entryMap]][binding]`
  
如果绑定组布局是组等价的，它们可以在所有内容中互换使用。


# 8.2. GPUBindGroup

`GPUBindGroup` 定义了要一起绑定的资源集以及这些资源在着色器阶段中的使用方式。

```webidl
[Exposed=(Window, DedicatedWorker), SecureContext]
interface GPUBindGroup {
};
```

`GPUBindGroup` 包括 `GPUObjectBase`。

`GPUBindGroup` 对象具有以下内部属性：

- `[[layout]]`，类型为 `GPUBindGroupLayout`，只读，表示与此 `GPUBindGroup` 相关联的 `GPUBindGroupLayout`。
- `[[entries]]`，类型为数组 `sequence<GPUBindGroupEntry>`，只读，表示此 `GPUBindGroup` 描述的 `GPUBindGroupEntry` 的集合。
- `[[usedResources]]`，类型为有序映射 `ordered map<subresource, list<internal usage>>`，只读，表示此绑定组使用的缓冲区和纹理子资源，与内部使用标志列表相关联。

## 8.2.1. 绑定组创建

`GPUBindGroup` 可通过 `GPUDevice.createBindGroup()` 方法创建。

`GPUBindGroupDescriptor` 字典具有以下成员：

- `layout`，类型为 `GPUBindGroupLayout`，必填项，表示此绑定组的布局。
- `entries`，类型为数组 `sequence<GPUBindGroupEntry>`，必填项，表示此绑定组所描述的资源列表，每个资源绑定由布局描述。

```webidl
dictionary GPUBindGroupDescriptor : GPUObjectDescriptorBase {
  required GPUBindGroupLayout layout;
  required sequence<GPUBindGroupEntry> entries;
};
```

`GPUBindGroupEntry` 字典描述要在 `GPUBindGroup` 中绑定的单个资源，它包括以下成员：

- `binding`，类型为 `GPUIndex32`，必填项，表示 `GPUBindGroup` 内部的资源绑定的唯一标识符，对应于 `GPUBindGroupLayoutEntry.binding` 和 `GPUShaderModule` 中的 `@binding` 属性。
- `resource`，类型为 `GPUBindingResource`，必填项，表示要绑定的资源，可以是 `GPUSampler`、`GPUTextureView`、`GPUExternalTexture` 或 `GPUBufferBinding`。

```webidl
typedef (GPUSampler or GPUTextureView or GPUBufferBinding or GPUExternalTexture) GPUBindingResource;

dictionary GPUBindGroupEntry {
    required GPUIndex32 binding;
    required GPUBindingResource resource;
};
```

`GPUBufferBinding` 字典描述要绑定为资源的缓冲区和可选范围，包括以下成员：

- `buffer`，类型为 `GPUBuffer`，必填项，表示要绑定的 `GPUBuffer`。
- `offset`，类型为 `GPUSize64`，默认值为 0，表示缓冲绑定在着色器中公开的范围的字节偏移量，从缓冲区开头计算。
- `size`，类型为 `GPUSize64`，必填项，表示缓冲绑定的大小，以字节为单位。如果未提供，则指定从偏移量开始到缓冲区结尾的范围。

```webidl
dictionary GPUBufferBinding {
    required GPUBuffer buffer;
    GPUSize64 offset = 0;
    GPUSize64 size;
}
```

`createBindGroup(descriptor)` 方法创建一个 `GPUBindGroup`。

```webidl
GPUBindGroup createBindGroup(GPUBindGroupDescriptor descriptor);
```

- 参数 `descriptor` 类型为 `GPUBindGroupDescriptor`，表示要创建的 `GPUBindGroup` 的描述。
- 返回值为 `GPUBindGroup`。

在 `GPUDevice` 时间轴上，按以下步骤对 `GPUBindGroup` 进行初始化。

该过程的时间轴步骤如下：

- 创建一个名为 `bindGroup` 的新的 `GPUBindGroup` 对象。
- 在 `GPUBindGroup` 对象的内部属性 `[[layout]]` 上设置传递给 `createBindGroup()` 的描述符的布局 `layout`。
- 在 `GPUBindGroup` 对象的内部属性 `[[entries]]` 上设置传递给 `createBindGroup()` 的描述符的 `entries`。
- 遍历 `GPUBindGroup` 对象的内部属性 `[[entries]]`，检查其中的每个 `GPUBindGroupEntry.resource`，验证其符合要求规范，并且在此过程中记录其内部使用标记。
- 验证 `GPUBindGroup` 对象的内部属性 `[[usedResources]]` 是否与 `GPUBindGroup` 的布局 `layout` 相容。
- 发出 `GPUDevice` 时间轴上的初始化步骤。
- 返回 `bindGroup`。

---

[设备时间线](https://www.w3.org/TR/2023/WD-webgpu-20230425/#device-timeline)初始化步骤：

1.  限制是这样的。`[[device]]`。`[[limits]]`。

1.  如果以下任一条件不满足，[将生成一个验证错误](https://www.w3.org/TR/2023/WD-webgpu-20230425/#abstract-opdef-generate-a-validation-error)，使bindGroup[无效](https://www.w3.org/TR/2023/WD-webgpu-20230425/#invalid)，并停止。

>    -   `descriptor.layout`是[有效的](https://www.w3.org/TR/2023/WD-webgpu-20230425/#abstract-opdef-valid-to-use-with)。
>    -  描述符`descriptor.layout`的条目数量与 `descriptor.entries` 的条目数量完全相等。
>    
>   对于描述符中的每个`GPUBindGroupEntry`绑定`descriptor.entries`：
> - 使用`bindingDescriptor.resource`绑定资源。
> - 描述符 `descriptor.layout.entries` 中恰好有一个 `GPUBindGroupLayoutEntry` `layoutBinding`，使得 `layoutBinding.binding` 等于 `bindingDescriptor.binding`。 
> - 如果定义的[binding member](https://www.w3.org/TR/2023/WD-webgpu-20230425/#binding-member)是
> 
>      **[sampler:](https://www.w3.org/TR/2023/WD-webgpu-20230425/#dom-gpubindgrouplayoutentry-sampler)**
> 
>   -   资源是一个`GPUSampler`.
>   -   资源是[有效的使用与](https://www.w3.org/TR/2023/WD-webgpu-20230425/#abstract-opdef-valid-to-use-with)此。
>   -   如果`layoutBinding.sampler.type`是：
>   -  ["filtering"（过滤）](https://www.w3.org/TR/2023/WD-webgpu-20230425/#dom-gpusamplerbindingtype-filtering) 
>           `resource.[[isComparison]]` is `false`.
>   -   ["non-filtering"（非过滤）](https://www.w3.org/TR/2023/WD-webgpu-20230425/#dom-gpusamplerbindingtype-non-filtering)
>   -  `resource.[[isFiltering]]` is `false`. `resource.[[isComparison]]` is `false`.
>   -  ["comparison"（比较）](https://www.w3.org/TR/2023/WD-webgpu-20230425/#dom-gpusamplerbindingtype-comparison) 
>   -   `resource.[[isComparison]]` is `true`.
> 
>      **[texture:](https://www.w3.org/TR/2023/WD-webgpu-20230425/#dom-gpubindgrouplayoutentry-texture)**
>    如果`layoutBinding` 的定义绑定成员为 `texture`：
>    
>   - `resource` 是一个 `GPUTextureView`。
>   - `resource` 可以与此对象一起使用。
>   - 令纹理为 `texture = resource.[[texture]]`。
>   - `layoutBinding.texture.viewDimension` 等于资源的维数。
>   - `layoutBinding.texture.sampleType` 与资源的格式兼容。
>   - `texture` 的使用包括 `TEXTURE_BINDING`。
>   - 如果 `layoutBinding.texture.multisampled` 为 `true`，则 `texture` 的 `sampleCount > 1`；否则，`texture` 的 `sampleCount` 为 `1`。
>   
>    **[storageTexture:](https://www.w3.org/TR/2023/WD-webgpu-20230425/#dom-gpubindgrouplayoutentry-storagetexture)** 如果 `layoutBinding` 的定义绑定成员为 `storageTexture`：
>
>   - `resource` 是一个 `GPUTextureView`。
>   - `resource` 可以与此对象一起使用。
>   - 令纹理为 `texture = resource.[[texture]]`。
>   - `layoutBinding.storageTexture.viewDimension` 等于资源的维数。
>   - `layoutBinding.storageTexture.format` 等于 `resource.[[descriptor]].format`。
>   - `texture` 的使用包括 `STORAGE_BINDING`。
>   - `resource.[[descriptor]].mipLevelCount]` 必须为 `1`。
> 
>    **[buffer](https://www.w3.org/TR/2023/WD-webgpu-20230425/#dom-gpubindgrouplayoutentry-buffer)** 如果 `layoutBinding` 的定义绑定成员为 `buffer`：
> 
>    - `resource` 是一个 `GPUBufferBinding`。
>    - `resource.buffer` 可以与此对象一起使用。
>    - `resource.offset` 和 `resource.size` 指定的绑定部分位于缓冲区内，并且大小大于 `0`。
>    - 有效的缓冲绑定大小(`effective buffer binding size(resource)`) ≥ `layoutBinding.buffer.minBindingSize`。
>    - 如果 `layoutBinding.buffer.type` 
>    
>      - uniform
>         - `"uniform"`，则 `resource.buffer.usage` 包括 `UNIFORM`。
> 
>         - `effective buffer binding size(resource)` ≤ `limits.maxUniformBufferBindingSize`。
>         - `resource.offset` 是 `limits.minUniformBufferOffsetAlignment` 的倍数。
> 
>      - storage
>        - 如果 `layoutBinding.buffer.type` 为 `"storage"` 或 `"read-only-storage"`，则 `resource.buffer.usage` 包括 `STORAGE`。
> 
>        - `effective buffer binding size(resource)` ≤ `limits.maxStorageBufferBindingSize`。
>        - `effective buffer binding size(resource)` 是 `4` 的倍数。
>        - `resource.offset` 是 `limits.minStorageBufferOffsetAlignment` 的倍数。
> 
> - 对于定义绑定成员为 `externalTexture` 的情况，
>   - `resource` 是一个 `GPUExternalTexture`，
>   - 可以与此对象一起使用。
> 
> 3.  设bindGroup.`[[布局]]`=描述符.`布局`。
> 
> 4.  设bindGroup.`[[条目]]`=描述符.`条目`。
> 
> 5.  让bindGroup.`[[usedResources]]`= {}.
> 
> 6.  对于描述符中的每个`GPUBindGroupEntry`绑定描述符。`条目`：
> 
>     1.  让interentalUsage成为layoutBind的[绑定用法](https://www.w3.org/TR/2023/WD-webgpu-20230425/#binding-usage)。
>     1.  资源看到的每个[子资源](https://www.w3.org/TR/2023/WD-webgpu-20230425/#subresource)都被添加到`[[usedResources]]`中。


有效的缓冲绑定大小(`effective buffer binding size(binding)`)如下计算：

1.  如果 `binding.size` 未提供，则返回 `max(0, binding.buffer.size - binding.offset)`。
2.  如果 `binding.size` 已提供，则返回 `binding.size`。

如果两个 `GPUBufferBinding` 对象 `a` 和 `b` 满足以下所有条件，则它们被视为是缓冲区绑定别名(`buffer-binding-aliasing`)：

- `a.buffer == b.buffer`。
- 由 `a.offset` 和 `a.size` 组成的范围与由 `b.offset` 和 `b.size` 组成的范围相交。

> ISSUE 11 Define how a range is formed by offset/size when size can be undefined.


# 8.3. GPUPipelineLayout

`GPUPipelineLayout` 定义了在 `setBindGroup()` 命令编码期间设置的所有 `GPUBindGroup` 对象的资源与由 `GPURenderCommandsMixin.setPipeline` 或 `GPUComputePassEncoder.setPipeline` 设置的管线着色器之间的映射关系。

资源的完整绑定地址可以定义为：

- 着色器阶段掩码，该资源对其可见。
- 绑定组索引。
- 绑定号。

这个地址的组成部分也可以看作是管线的绑定空间(`binding space`)。`GPUBindGroup`(与相应的`GPUBindGroupLayout`)为固定的绑定组索引覆盖了这个空间。所包含的绑定需要是着色器在该绑定组索引处使用的资源的超集。

```WebIDL
[Exposed=(Window, DedicatedWorker), SecureContext]
interface GPUPipelineLayout {
};
```

`GPUPipelineLayout` 包含 `GPUObjectBase` 接口。

`GPUPipelineLayout` 具有以下内部插槽：

- `[[bindGroupLayouts]]`，类型为 `list<GPUBindGroupLayout>`。这是在 `GPUPipelineLayoutDescriptor.bindGroupLayouts` 中提供的 `GPUBindGroupLayout` 对象。

注意：在许多 `GPURenderPipeline` 或 `GPUComputePipeline` 管线中使用相同的 `GPUPipelineLayout` 保证了当这些管线之间切换时，用户代理不需要在内部重新绑定任何资源。

假设使用以下绑定组布局配置创建了 `GPUComputePipeline` 对象 X: A、B、C；使用以下绑定组布局配置创建了 `GPUComputePipeline` 对象 Y: A、D、C。编码命令序列中执行了两次 `dispatches`：

```
setBindGroup(0, ...)

setBindGroup(1, ...)

setBindGroup(2, ...)

setPipeline(X)

dispatchWorkgroups()

setBindGroup(1, ...)

setPipeline(Y)

dispatchWorkgroups()
```

在这种情况下，即使 `GPUPipelineLayout.bindGroupLayouts` 索引 2 处的 `GPUBindGroupLayout` 或插槽 2 的 `GPUBindGroup` 没有更改，用户代理也必须重新绑定插槽 2。

注意：`GPUPipelineLayout` 的预期用法是将最常见和最不经常更改的绑定组放在布局的“底部”，即较低的绑定组插槽编号，如 0 或 1。绑定组需要在绘制调用之间更频繁地更改，则其索引应该更高。这个通用指南允许用户代理最小化绘制调用之间的状态更改，从而降低 CPU 开销。


# 8.3.1. 管线布局创建( Pipeline Layout Creation)

可以通过 `GPUDevice.createPipelineLayout()` 来创建 `GPUPipelineLayout`。

```WebIDL
dictionary GPUPipelineLayoutDescriptor : GPUObjectDescriptorBase {
  required sequence<GPUBindGroupLayout> bindGroupLayouts;
};
```

`GPUPipelineLayoutDescriptor` 字典定义了管线使用的所有 `GPUBindGroupLayout` 并具有以下成员：

- `bindGroupLayouts`，类型为 `sequence<GPUBindGroupLayout>`。这是管线将使用的 `GPUBindGroupLayout` 列表。每个元素都对应于 `GPUShaderModule` 中的 `@group` 属性，其中第 `N` 个元素对应于 `@group(N)`。

`createPipelineLayout(descriptor)`：创建一个 `GPUPipelineLayout`。

- 被调用于：`GPUDevice this`。
- 参数：

    | Parameter | Type                              | Nullable | Optional | Description                                                |
    | --------- | --------------------------------- | -------- | -------- | ---------------------------------------------------------- |
    | descriptor | `GPUPipelineLayoutDescriptor`     | ✘        | ✘        | 要创建的 `GPUPipelineLayout` 的描述信息。                     |

- 返回值：`GPUPipelineLayout`

内容时间线步骤：

- 创建一个新的 `GPUPipelineLayout` 对象 `pl`。
- 在此设备的重建时间线上发出初始化步骤。
- 返回 `pl`。

设备时间线的初始化步骤：

- 令 `limits` 为 `this.[[device]].[[limits]]`。
- 让 `allEntries` 成为对于所有 `descriptor.bindGroupLayouts` 中的 `bgl`，将 `bgl.[[descriptor]].entries` 进行串联的结果。
- 如果以下任何条件不被满足，则产生验证错误，使 `pl` 无效，并停止。

  - `descriptor.bindGroupLayouts` 中的每个 `GPUBindGroupLayout` 必须可以与此一起使用，并且其 `[[exclusivePipeline]]` 为 null。
  - `descriptor.bindGroupLayouts` 的大小必须 ≤ `limits.maxBindGroups`。
  - `allEntries` 必须不超过 `limits` 的绑定槽限制。

- 将 `pl.[[bindGroupLayouts]]` 设置为 `descriptor.bindGroupLayouts`。

注意：如果两个 `GPUPipelineLayout` 对象的内部 `[[bindGroupLayouts]]` 序列包含了组相等的 `GPUBindGroupLayout` 对象，则在任何用途上都被认为是等效的。

# 8.4. 示例

创建一个 `GPUBindGroupLayout`，该组布局包含一个 Uniform 缓冲区、一个纹理和一个采样器。然后使用 `GPUBindGroupLayout` 创建 `GPUBindGroup` 和 `GPUPipelineLayout`。

```javascript
const bindGroupLayout = gpuDevice.createBindGroupLayout({
  entries: [{
    binding: 0,
    visibility: GPUShaderStage.VERTEX | GPUShaderStage.FRAGMENT,
    buffer: {}
  }, {
    binding: 1,
    visibility: GPUShaderStage.FRAGMENT,
    texture: {}
  }, {
    binding: 2,
    visibility: GPUShaderStage.FRAGMENT,
    sampler: {}
  }]
});

const bindGroup = gpuDevice.createBindGroup({
    layout: bindGroupLayout,
    entries: [{
        binding: 0,
        resource: { buffer: buffer },
    }, {
        binding: 1,
        resource: texture
    }, {
        binding: 2,
        resource: sampler
    }]
});

const pipelineLayout = gpuDevice.createPipelineLayout({
  bindGroupLayouts: [bindGroupLayout]
});
```

首先，使用 `GPUDevice.createBindGroupLayout()` 创建一个 `GPUBindGroupLayout`，其中包含了三个绑定：

- 绑定 0：Uniform 缓冲区，可在顶点着色器和片元着色器中使用。
- 绑定 1：纹理，仅可在片元着色器中使用。
- 绑定 2：采样器，仅可在片元着色器中使用。

接下来，使用 `GPUDevice.createBindGroup()` 创建一个 `GPUBindGroup`，将上一步中创建的 `GPUBindGroupLayout` 用于布局，并向绑定中添加实际资源。这里使用了一个 Uniform 缓冲区 `buffer`，一张纹理 `texture`，和一个采样器 `sampler`。

最后，使用 `GPUDevice.createPipelineLayout()` 创建 `GPUPipelineLayout`，并将该 `GPUBindGroupLayout` 添加到 `bindGroupLayouts` 列表中。

