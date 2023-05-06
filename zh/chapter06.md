### 6.1. `GPUTexture`

一张纹理由一个或多个纹理子资源组成，每个子资源由一个mipmap级别和（仅针对2D纹理）阵列层以及方面唯一标识。

一个纹理子资源是一个子资源，每个子资源可以在单个使用范围内的不同内部位置使用。

在一个mipmap级别中的每个子资源大小在每个空间维度上都大约是比较低一级对应的资源的一半（请参见逻辑mipmap级别特定的纹理范围）。级别0中的子资源具有纹理本身的维度。这些通常用来表示纹理的细节级别。`GPUSampler`和`WGSL`提供了选择和插值不同细节级别的函数，可以是显式的或自动的。

一个“2D”纹理可以是一组阵列层。层中的每个子资源与其他层中的对应资源具有相同的大小。对于非 2D 纹理，所有子资源都具有阵列层索引 0。

每个子资源都有一个方面。颜色纹理只有一个方面：颜色。深度-或模板格式的纹理可以具有多个方面:深度方面，模板方面或两者，并可能以特殊方式使用，如在depthStencilAttachment和“depth”绑定中。

一个“3D”纹理可以有多个切片，每个切片是纹理中特定z值的二维图像。切片不是单独的子资源。

```
[Exposed=(Window, DedicatedWorker), SecureContext]
interface GPUTexture {
    GPUTextureView createView(optional GPUTextureViewDescriptor descriptor = {});

    undefined destroy();

    readonly attribute GPUIntegerCoordinate width;
    readonly attribute GPUIntegerCoordinate height;
    readonly attribute GPUIntegerCoordinate depthOrArrayLayers;
    readonly attribute GPUIntegerCoordinate mipLevelCount;
    readonly attribute GPUSize32 sampleCount;
    readonly attribute GPUTextureDimension dimension;
    readonly attribute GPUTextureFormat format;
    readonly attribute GPUTextureUsageFlags usage;
};
GPUTexture includes GPUObjectBase;
```

`GPUTexture`有以下属性：

`width`，类型为`GPUIntegerCoordinate`，只读
此`GPUTexture`的宽度。

`height`，类型为`GPUIntegerCoordinate`，只读
此`GPUTexture`的高度。

`depthOrArrayLayers`，类型为`GPUIntegerCoordinate`，只读
此`GPUTexture`的深度或层数。

`mipLevelCount`，类型为`GPUIntegerCoordinate`，只读
此`GPUTexture`具有的`mip`级别数。

`sampleCount`，类型为`GPUSize32`，只读
此`GPUTexture`的样本计数。

`dimension`，类型为`GPUTextureDimension`，只读
每个此`GPUTexture`子资源的纹素集的维度。

`format`，类型为`GPUTextureFormat`，只读
此`GPUTexture`的格式。

`usage`，类型为`GPUTextureUsageFlags`，只读
此`GPUTexture`允许的用途。

`GPUTexture`具有以下内部插槽：
`[[size]]`，类型为`GPUExtent3D`
纹理的大小（与宽度，高度和`depthOrArrayLayers`属性相同）。

`[[viewFormats]]`，类型为`sequence<GPUTextureFormat>`
可以用于在此GPUTexture上创建视图时使用的GPUTextureFormats集。

`[[destroyed]]`，类型为布尔值，最初为false。
如果纹理被销毁，则无法在任何操作中使用它，并且可以释放其下层内存。

`compute render extent（baseSize， mipLevel）`
参数：

`GPUExtent3D baseSize`
`GPUSize32 mipLevel`

返回值：`GPUExtent3DDict`

1. 让`extent`为新的`GPUExtent3DDict`对象。

2. 将`extent.width`设置为`max(1, baseSize.width ≫ mipLevel)`。

3. 将`extent.height`设置为`max(1, baseSize.height ≫ mipLevel)`。

4. 将`extent.depthOrArrayLayers`设置为1。

5. 返回`extent`。

一张纹理的逻辑 mip 级别特定的纹理范围是指特定 mip 级别下纹理在 texel 中的大小。其计算过程如下：

## 逻辑 mip 级别特定的纹理范围(descriptor, mipLevel)

### 参数：

- `GPUTextureDescriptor` `descriptor`
- `GPUSize32` `mipLevel`

### 返回值：

`GPUExtent3DDict`

### 实现逻辑：

1. 声明一个新的 `GPUExtent3DDict` 对象 `extent`
2. 根据 `descriptor` 中的 `dimension` 属性进行分支：
    - "1d"
        - 设置 `extent.width` 为 `max(1, descriptor.size.width ≫ mipLevel)`
        - 设置 `extent.height` 为 1
        - 设置 `extent.depthOrArrayLayers` 为 1
    - "2d"
        - 设置 `extent.width` 为 `max(1, descriptor.size.width ≫ mipLevel)`
        - 设置 `extent.height` 为 `max(1, descriptor.size.height ≫ mipLevel)`
        - 设置 `extent.depthOrArrayLayers` 为 `descriptor.size.depthOrArrayLayers`
    - "3d"
        - 设置 `extent.width` 为 `max(1, descriptor.size.width ≫ mipLevel)`
        - 设置 `extent.height` 为 `max(1, descriptor.size.height ≫ mipLevel)`
        - 设置 `extent.depthOrArrayLayers` 为 `max(1, descriptor.size.depthOrArrayLayers ≫ mipLevel)`
3. 返回 `extent`


## 物理 mip 级别特定的纹理范围(descriptor, mipLevel)

### 参数：

- `GPUTextureDescriptor` `descriptor`
- `GPUSize32` `mipLevel`

### 返回值：

`GPUExtent3DDict`

### 实现逻辑：

1. 声明一个新的 `GPUExtent3DDict` 对象 `extent`。
2. 使用 `logical miplevel-specific texture extent(descriptor, mipLevel)` 函数得到逻辑 mip 级别特定的纹理范围，在一个新的 `GPUExtent3DDict` 对象 `logicalExtent` 中存储。
3. 根据 `descriptor.dimension` 属性进行分支：
    - 当 `descriptor.dimension` 是 "1d" 时：
        - 将 `extent.width` 设置为将 `logicalExtent.width` 上取整到描述符的 texel 块宽度的最近倍数（即将其对齐到 texel 块的宽度）。
        - 将 `extent.height` 设置为 1。
        - 将 `extent.depthOrArrayLayers` 设置为 1。
    - 当 `descriptor.dimension` 是 "2d" 时：
        - 将 `extent.width` 设置为将 `logicalExtent.width` 上取整到描述符的 texel 块宽度的最近倍数（即将其对齐到 texel 块的宽度）。
        - 将 `extent.height` 设置为将 `logicalExtent.height` 上取整到描述符的 texel 块高度的最近倍数（即将其对齐到 texel 块的高度）。
        - 将 `extent.depthOrArrayLayers` 设置为 `logicalExtent.depthOrArrayLayers`。
    - 当 `descriptor.dimension` 是 "3d" 时：
        - 将 `extent.width` 设置为将 `logicalExtent.width` 上取整到描述符的 texel 块宽度的最近倍数（即将其对齐到 texel 块的宽度）。
        - 将 `extent.height` 设置为将 `logicalExtent.height` 上取整到描述符的 texel 块高度的最近倍数（即将其对齐到 texel 块的高度）。
        - 将 `extent.depthOrArrayLayers` 设置为 `logicalExtent.depthOrArrayLayers`。
4. 返回 `extent`。

#### 6.1.1. `GPUTextureDescriptor`
```
dictionary GPUTextureDescriptor
         : GPUObjectDescriptorBase {
    required GPUExtent3D size;
    GPUIntegerCoordinate mipLevelCount = 1;
    GPUSize32 sampleCount = 1;
    GPUTextureDimension dimension = "2d";
    required GPUTextureFormat format;
    required GPUTextureUsageFlags usage;
    sequence<GPUTextureFormat> viewFormats = [];
};
```

`GPUTextureDescriptor` 具有以下成员:

- `size`，类型为 `GPUExtent3D`

  纹理的宽度、高度和深度或层数。

- `mipLevelCount`，类型为 `GPUIntegerCoordinate`，默认为 1

  纹理包含的 mip 级别数量。

- `sampleCount`，类型为 `GPUSize32`，默认为 1

  纹理的采样次数。`sampleCount > 1` 表示纹理进行了多重采样。

- `dimension`，类型为 `GPUTextureDimension`，默认为 "2d"

  纹理是一维的，是一个二维的层数组，还是三维的。

- `format`，类型为 `GPUTextureFormat`

  纹理的格式。

- `usage`，类型为 `GPUTextureUsageFlags`

  纹理的允许使用情况。

- `viewFormats`，类型为 `sequence<GPUTextureFormat>`，默认为 []

  在调用此纹理的 `createView()` 时指定允许的视图格式值（除纹理实际格式外）。
  
  
  >注意：向此列表添加格式可能会对性能产生显著影响，因此最好避免不必要地添加格式。实际的性能影响高度依赖于目标系统；开发人员必须测试各种系统以了解它们特定应用程序的影响。例如，在某些系统上，任何具有格式或 `viewFormats` 条目并包含 `rgba8unorm-srgb` 格式的纹理将比不包含 `-srgb` 后缀的 `rgba8unorm` 纹理的性能更低。在其他系统上，其他格式和格式对之间也存在类似的警告。

此列表中的格式必须与纹理格式兼容。

如果格式相等或仅在是否为 sRGB 格式（具有 `-srgb` 后缀）方面不同，则两种 `GPUTextureFormats` 格式和 `viewFormat` 是纹理视图格式兼容的。

通过定义更大的兼容性类来解决该问题。[问题 #gpuweb/gpuweb#168]

```
enum GPUTextureDimension {
    "1d",
    "2d",
    "3d",
};
```
> "1d"
> 
> 指的是一维纹理，即只有一个维度，宽度。
> 
> "2d"
> 
> 指的是二维纹理，即有宽度和高度，并且可能有层。只有 "2d" 纹理可以含有多个 mipmap，可以进行多重采样，可以使用压缩或深度/模板格式，并且可以用作渲染附件。
> 
> "3d"
> 
> 指的是三维纹理，即有宽度、高度和深度。

## 6.1.2. 纹理用途

```
typedef [EnforceRange] unsigned long GPUTextureUsageFlags;

[Exposed=(Window, DedicatedWorker), SecureContext]

namespace GPUTextureUsage {

​    const GPUFlagsConstant COPY_SRC          = 0x01;

​    const GPUFlagsConstant COPY_DST          = 0x02;

​    const GPUFlagsConstant TEXTURE_BINDING   = 0x04;

​    const GPUFlagsConstant STORAGE_BINDING   = 0x08;

​    const GPUFlagsConstant RENDER_ATTACHMENT = 0x10;

};
```

GPUTextureUsage 标志决定了GPUTexture在创建后可以如何使用：

- COPY_SRC

  纹理可用作复制操作的源纹理。(例如：作为 copyTextureToTexture() 或 copyTextureToBuffer() 调用的源参数。)

- COPY_DST

  纹理可用作复制或写入操作的目标纹理。(例如：作为 copyTextureToTexture() 或 copyBufferToTexture() 调用的目标参数，或作为 writeTexture() 调用的目标。)

- TEXTURE_BINDING

  纹理可绑定为着色器中采样的纹理。(例如：作为 GPUTextureBindingLayout 的绑定组条目。)

- STORAGE_BINDING

  纹理可绑定为着色器中的存储纹理。(例如：作为 GPUStorageTextureBindingLayout 的绑定组条目。)

- RENDER_ATTACHMENT

  纹理可用作渲染通道中的颜色或深度/模板附件。(例如：作为 GPURenderPassColorAttachment.view 或 GPURenderPassDepthStencilAttachment.view。)
  
最大 mipLevel 计数（维度，大小）

**Arguments:**

- dimension  维度

- size      大小

计算最大维度值m：

- 如果dimension是

  - "1d"
    返回1。
    
  - "2d"
    让m = max(size.width，size.height)。
    
  - "3d"
    让m = max(max(size.width，size.height)，size.depthOrArrayLayer)。
    
返回floor(log2(m)) + 1。

## 6.1.3. 纹理创建

**createTexture**(descriptor)

创建一个GPUTexture。

调用者：GPUDevice对象。

**Arguments:**

| 参数名                                     | 类型                    | 可空   | 是否必须  | 描述                                     |
| ---------------------------------------- | --------------------- | ---- | ----- | -------------------------------------- |
| descriptorGPUTextureDescriptor | ✘                  | ✘     | 创建GPUTexture的描述符。                                |
| 返回值                                     | GPUTexture            |      | GPUTexture对象 |

内容时间轴步骤：

- 验证 GPUExtent3D 形状 (descriptor.size) 的有效性。

- 使用 this.[[device]] 验证描述符格式所需的功能。

- 使用 this.[[device]] 验证每个 descriptor.viewFormats 元素的所需格式特性。

让 t 成为一个新的 GPUTexture 对象。

将 t.width 设置为 descriptor.size.width。

将 t.height 设置为 descriptor.size.height。

将 t.depthOrArrayLayers 设置为 descriptor.size.depthOrArrayLayers。

将 t.mipLevelCount 设置为 descriptor.mipLevelCount。

将 t.sampleCount 设置为 descriptor.sampleCount。

将 t.dimension 设置为 descriptor.dimension。

将 t.format 设置为 descriptor.format。

将 t.usage 设置为 descriptor.usage。

发出在此设备时间轴上的初始化步骤。

返回 t。

设备时间轴初始化步骤：

如果以下任何条件不满足，则生成验证错误，使 t 无效并停止。

**验证GPUTextureDescriptor(GPUDevice this, GPUTextureDescriptor descriptor)：**

如果满足以下所有要求，则返回True，否则返回False：

- this 必须是一个有效的GPUDevice实例。
- descriptor.usage 必须不为0。
- descriptor.usage 必须仅包含此允许的纹理用途中存在的位。
- descriptor.size.width、descriptor.size.height和descriptor.size.depthOrArrayLayers 必须大于零。
- descriptor.mipLevelCount 必须大于零。
- descriptor.sampleCount 必须为1或4。

如果描述符维度是：

- "1d"
  
  - descriptor.size.width 必须 ≤ this.limits.maxTextureDimension1D。
  
  - descriptor.size.height 必须为1。
  
  - descriptor.size.depthOrArrayLayers 必须为1。
  
  - descriptor.sampleCount 必须为1。
  
  - descriptor.format 不能是压缩格式或深度或模板格式。

- "2d"
  
  - descriptor.size.width 必须 ≤ this.limits.maxTextureDimension2D。
  
  - descriptor.size.height 必须 ≤ this.limits.maxTextureDimension2D。
  
  - descriptor.size.depthOrArrayLayers 必须 ≤ this.limits.maxTextureArrayLayers。

- "3d"
  
  - descriptor.size.width 必须 ≤ this.limits.maxTextureDimension3D。
  
  - descriptor.size.height 必须 ≤ this.limits.maxTextureDimension3D。
  
  - descriptor.size.depthOrArrayLayers 必须 ≤ this.limits.maxTextureDimension3D。
  
  - descriptor.sampleCount 必须为1。
  
  - descriptor.format 不能是压缩格式或深度或模板格式。

- descriptor.size.width 必须是纹理块宽度的倍数。
- descriptor.size.height 必须是纹理块高度的倍数。

如果descriptor.sampleCount >1：

- descriptor.mipLevelCount 必须为1。
- descriptor.size.depthOrArrayLayers 必须为1。
- descriptor.usage 不得包含 STORAGE_BINDING 位。
- descriptor.usage 必须包括 RENDER_ATTACHMENT 位。
- descriptor.format 必须根据§ 26.1纹理格式功能支持多重采样。
- descriptor.mipLevelCount 必须 ≤ maximum mipLevel count(descriptor.dimension, descriptor.size)。

如果descriptor.usage 包括 RENDER_ATTACHMENT 位：

- descriptor.format 必须是可渲染格式。
- descriptor.dimension 必须为 "2d"。

如果descriptor.usage 包括 STORAGE_BINDING 位：

- descriptor.format 必须在 § 26.1.1 纯色格式表格中以 STORAGE_BINDING 能力为列出的格式中。
- 对于 descriptor.viewFormats 中的每个 viewFormat，descriptor.format 和 viewFormat 必须是纹理视图格式兼容的。

> EXAMPLE 14
> 
> 创建一个大小为16x16，RGBA格式，1个数组层，1个mip级别的2D纹理：
> 
> ```
> const texture = gpuDevice.createTexture({
>   size: { width: 16, height: 16 },
>   format: 'rgba8unorm',
>   usage: GPUTextureUsage.TEXTURE_BINDING,
> });
> ```
## 6.1.4. 纹理销毁

当应用程序不再需要一个GPUTexture时，可以通过调用destroy()在垃圾收集之前失去对它的访问。
> 
> 注意：一旦所有之前提交使用它的操作都完成，这允许用户代理回收与GPUTexture相关联的GPU内存。

**destroy()**

销毁GPUTexture。

调用者：GPUTexture this。

返回值：无。

内容时间轴步骤：

将 this.[[destroyed]] 设置为 true。

## 6.2. GPUTextureView

GPUTextureView 是对 GPUTexture 定义的一些纹理子资源的视图。

```javascript
[Exposed=(Window, DedicatedWorker), SecureContext]
interface GPUTextureView {
};

GPUTextureView includes GPUObjectBase;
```
GPUTextureView 具有以下内部插槽:

**[[texture]]**
此纹理视图所在的GPUTexture。

**[[descriptor]]**
描述此纹理视图的GPUTextureViewDescriptor描述符对象。

所有GPUTextureViewDescriptor的可选字段都定义。

**[[renderExtent]]**
对于可呈现的纹理视图，这是呈现的有效GPUExtent3DDict对象。

> 注意：此区域取决于baseMipLevel。

一个纹理视图的子资源集合，具有描述符 desc，是视图子资源中满足以下条件的子资源的子集：

s的 mipmap级别≥ desc.baseMipLevel 并且<desc.baseMipLevel + desc.mipLevelCount。

s的数组层≥ desc.baseArrayLayer并且<desc.baseArrayLayer + desc.arrayLayerCount。

s的 aspect 在 desc.aspect 集合中。

仅当两个GPUTextureView对象的子资源集合相交时，它们才是纹理视图别名。

## 6.2.1. 纹理视图创建

定义 GPUTextureViewDescriptor 字典类型，继承自 GPUObjectDescriptorBase 接口：

```javascript
dictionary GPUTextureViewDescriptor : GPUObjectDescriptorBase {
  GPUTextureFormat format;
  GPUTextureViewDimension dimension;
  GPUTextureAspect aspect = "all";
  GPUIntegerCoordinate baseMipLevel = 0;
  GPUIntegerCoordinate mipLevelCount;
  GPUIntegerCoordinate baseArrayLayer = 0;
  GPUIntegerCoordinate arrayLayerCount;
};
```

GPUTextureViewDescriptor 具有以下成员：

**format**，类型为 GPUTextureFormat

纹理视图的格式。必须是纹理的格式之一或在其创建期间指定的 viewFormats 之一。

**dimension**，类型为 GPUTextureViewDimension

要查看纹理的维度。

**aspect**，类型为 GPUTextureAspect，默认值为 “all”

可访问纹理视图的方面是哪些。

**baseMipLevel**，类型为 GPUIntegerCoordinate，默认值为 0

纹理视图可访问的第一个（最详细的）mipmap 级别。

**mipLevelCount**，类型为 GPUIntegerCoordinate

从 baseMipLevel 开始，有多少个 mipmap 级别可供纹理视图访问。

**baseArrayLayer**，类型为 GPUIntegerCoordinate，默认值为 0

可访问纹理视图的第一个数组层的索引。

**arrayLayerCount**，类型为 GPUIntegerCoordinate

从 baseArrayLayer 开始，有多少个数组层可供纹理视图访问。

```javascript
enum GPUTextureViewDimension {
  "1d",
  "2d",
  "2d-array",
  "cube",
  "cube-array",
  "3d",
};
```

- "1d"

  将纹理视图视为 1D 形式的图像。

  对应的 WGSL 类型为：

  ```
  texture_1d
  texture_storage_1d
  ```

- "2d"

  纹理视图被视为单个 2D 形式的图像。

  对应的 WGSL 类型为：

  ```
  texture_2d
  texture_storage_2d
  texture_multisampled_2d
  texture_depth_2d
  texture_depth_multisampled_2d
  ```

- "2d-array"

  纹理视图被视为一组 2D 形式的图像。

  对应的 WGSL 类型为：

  ```
  texture_2d_array
  texture_storage_2d_array
  texture_depth_2d_array
  ```

- "cube"

  纹理被视为立方体贴图。这个视图有 6 个数组层，对应于立方体的 [+X，-X，+Y，-Y，+Z，-Z] 面。在立方体贴图中无缝采样。

  对应的 WGSL 类型为：

  ```
  texture_cube
  texture_depth_cube
  ```

- "cube-array"

  将纹理视为 n 个立方体贴图的压缩数组，每个贴图有 6 个数组层，对应于立方体的 [+X，-X，+Y，-Y，+Z，-Z] 面。在立方体贴图中无缝采样。

  对应的 WGSL 类型为：

  ```
  texture_cube_array
  texture_depth_cube_array
  ```

- "3d"

  将纹理视为 3D 图像。

  对应的 WGSL 类型为：

  ```
  texture_3d
  texture_storage_3d
  ```

枚举 GPUTextureAspect 定义子资源方面：

```javascript
enum GPUTextureAspect {
  "all",
  "stencil-only",
  "depth-only",
};
```

- "all"

包括所有的方面。

- "stencil-only"

只包括模板方面。

- "depth-only"

只包括深度方面。

## createView(descriptor)

创建一个 GPUTextureView 视图对象。

注意：默认情况下，若未指定纹理视图的维度，将创建一个可表示整个纹理的视图。例如，在具有多个层的 "2d" 纹理上调用 createView() 时，即使指定了 arrayLayerCount 为 1，仍会创建一个 "2d-array" 的 GPUTextureView 我们建议对于在开发时层数未知的纹理资源，调用 createView() 时要提供显式的维度以确保着色器兼容性。

该方法作用于 GPUTexture 对象，具有以下参数：

| 参数         | 类型                                | 可空 | 可选 | 描述                                                                                      |
| ------------ | ----------------------------------- | ---- | ---- | ----------------------------------------------------------------------------------------- |
| descriptor   | GPUTextureViewDescriptor 类型的对象 | ✘    | ✔    | 用于创建 GPUTextureView 的描述对象。                                                      |
| 返回结果     | GPUTextureView 类型的对象           |      |      | 新的 GPUTextureView 视识对象。                                                            |
| 时间轴步骤   |                                      |      |      |                                                                                           |
|              | 验证 descriptor.format 所需特性     |      |      | 使用 this.[[device]] 验证纹理格式所需的特性。                                            |
|              | 创建 GPUTextureView 对象             |      |      | 创建一个新的 GPUTextureView 对象。                                                        |
|              | 执行初始化步骤                        |      |      | 执行此对象的初始化步骤，该步骤已记录在设备(Device)的时间轴中。                          |
|              | 返回 GPUTextureView 对象             |      |      | 返回新创建的 GPUTextureView 视识对象。                                                   |

**返回值**

返回新创建的 GPUTextureView 对象。

**时间轴**

- 验证 texture format：

  使用 this.[[device]] 验证纹理格式所需的特性。

- 创建 GPUTextureView 对象：

  创建一个新的 GPUTextureView 对象。

- 执行初始化步骤：

  执行此对象的初始化步骤，该步骤已记录在设备(Device)的时间轴中。

- 返回 GPUTextureView 对象：

  返回新创建的 GPUTextureView 视识对象。
  
## 设备时间轴（）初始化步骤

设备(Device)时间轴初始化步骤用于在设备设备(Device)时间轴上执行 GPUTextureView 对象的初始化。此过程包括以下子步骤：

1. 使用默认描述符解析 GPUTextureViewDescriptor 默认值并设置 descriptor。

2. 如果以下任何条件不满足，则生成验证错误，使视图视觉无效，并停止初始化：

- this 是有效的。

- descriptor.aspect 必须存在于 this.format 中。

- 如果 descriptor.aspect 是 “all”：

  - descriptor.format 必须等于 this.format 中的格式之一。

  - descriptor.format 必须等于 this.[[viewFormats]] 中的格式之一。

- 否则：

  - descriptor.format 必须等于 resolveGPUTextureAspect(this.format,descriptor.aspect) 的结果。

- descriptor.mipLevelCount 必须大于 0。

- descriptor.baseMipLevel + descriptor.mipLevelCount 必须小于或等于 this.mipLevelCount。

- descriptor.arrayLayerCount 必须大于 0。

- descriptor.baseArrayLayer + descriptor.arrayLayerCount 必须小于或等于 this 的数组层数。

- 如果 this.sampleCount > 1，则 descriptor.dimension 必须为 "2d"。

- 如果 descriptor.dimension 为：

  - “1d”

    - this.dimension 必须为 "1d"。

    - descriptor.arrayLayerCount 必须为 1。

  - "2d"

    - this.dimension 必须为 "2d"。

    - descriptor.arrayLayerCount 必须为 1。

  - "2d-array"

    - this.dimension 必须为 "2d"。

  - "cube"

    - this.dimension 必须为 "2d"。

    - descriptor.arrayLayerCount 必须为 6。

    - this.width 必须等于 this.height。

  - "cube-array"

    - this.dimension 必须为 "2d"。

    - descriptor.arrayLayerCount 必须为 6 的倍数。

    - this.width 必须等于 this.height。

  - "3d"

    - this.dimension 必须为 "3d"。

    - descriptor.arrayLayerCount 必须为 1。

3. 创建一个新的 GPUTextureView 对象 view。

4. 将 view.[[texture]] 设置为 this。

5. 将 view.[[descriptor]] 设置为 descriptor。

6.  如果 this.usage 包含 RENDER_ATTACHMENT：

- 计算渲染区域，设置为 renderExtent。

- 将 view.[[renderExtent]] 设置为 renderExtent。

完成上述步骤之后，设备(Device)时间轴初始化步骤返回已初始化的 GPUTextureView 对象 view。

---

对于 GPUTextureView 纹理(即 `texture`) 的 GPUTextureViewDescriptor 描述符解析默认值时，运行以下步骤：

1. 复制一份 descriptor 为 resolved。

2. 如果 resolved.format 未提供：

- 计算格式 format 的结果，使用 GPUTextureAspect( format, descriptor.aspect) 解析之。

- 如果 format 为 null，则将 resolved.format 设置为 texture.format；否则将 resolved.format 设置为 format。

3. 如果 resolved.mipLevelCount 未提供，则将其设置为 texture.mipLevelCount - resolved.baseMipLevel。

4. 如果 resolved.dimension 未提供：

- 如果 texture.dimension 为 "1d"，则将 resolved.dimension 设置为 "1d"。

- 如果 texture.dimension 为 "2d"：

  - 如果 texture 的数组层数为 1，则将 resolved.dimension 设置为 "2d"。

  - 否则，将 resolved.dimension 设置为 "2d-array"。

- 如果 texture.dimension 为 "3d"，则将 resolved.dimension 设置为 "3d"。

5. 如果 resolved.arrayLayerCount 未提供，并且 resolved.dimension 为：

- "1d"、"2d" 或 "3d"，则将 resolved.arrayLayerCount 设置为 1。

- "cube"，则将 resolved.arrayLayerCount 设置为 6。

- "2d-array" 或 "cube-array"，则将 resolved.arrayLayerCount 设置为 texture 的数组层数减去 resolved.baseArrayLayer。

6. return resolved。

要确定 GPUTexture texture 的数组层数，请按以下步骤执行：

1. 如果 texture.dimension 为 "1d" 或 "3d"，返回 1。

2. 如果 texture.dimension 为 "2d"，则返回 texture.depthOrArrayLayers。

## 6.3 纹理格式

纹理格式的名称指定了组件的顺序、每个组件的位数以及组件的数据类型。

- r、g、b、a 分别表示红、绿、蓝、透明度。

- unorm 表示无符号规范化；

- snorm 表示符号规范化；

- uint 表示无符号整数；

- sint 表示带符号整数；

- float 表示浮点数。

如果格式名称具有 -srgb 后缀，则在着色器读取和写入颜色值时会应用从 gamma 到线性和反之的 sRGB 转换。纹理能力(feature)提供了压缩纹理格式。它们的命名应遵循此处的惯例，使用纹理名称作为前缀。例如：etc2-rgba8unorm。

纹素块(texel block)是像素基础的 GPUTextureFormats 纹理中的单个可寻址元素，以及块压缩的 GPUTextureFormats 纹理中的单个压缩块。

纹素块的宽度和高度规定了一个纹素块的维度。

对于像素基础的 GPUTextureFormats，纹素块的宽度和高度始终为1。

对于块压缩的 GPUTextureFormats，纹素块的宽度是一个纹素块中每一行的纹素数，纹素块的高度是一个纹素块中的纹理行数。有关每个纹理格式的所有值的详尽列表，请参见“§ 26.1 纹理格式功能”。

一个 GPUTextureFormat 的一个面(aspect)的纹素块复制占用(footprint)是在图像复制中单个纹素块占用的字节数（如果适用）。

>注意：一个 GPUTextureFormat 的纹素块(memory cost)占用大小是存储一个纹素块所需的字节数。它并不完全定义所有格式。这个值是信息性和非规范性的。

```
enum GPUTextureFormat {
    // 8-bit formats
    "r8unorm",
    "r8snorm",
    "r8uint",
    "r8sint",

    // 16-bit formats
    "r16uint",
    "r16sint",
    "r16float",
    "rg8unorm",
    "rg8snorm",
    "rg8uint",
    "rg8sint",

    // 32-bit formats
    "r32uint",
    "r32sint",
    "r32float",
    "rg16uint",
    "rg16sint",
    "rg16float",
    "rgba8unorm",
    "rgba8unorm-srgb",
    "rgba8snorm",
    "rgba8uint",
    "rgba8sint",
    "bgra8unorm",
    "bgra8unorm-srgb",
    // Packed 32-bit formats
    "rgb9e5ufloat",
    "rgb10a2unorm",
    "rg11b10ufloat",

    // 64-bit formats
    "rg32uint",
    "rg32sint",
    "rg32float",
    "rgba16uint",
    "rgba16sint",
    "rgba16float",

    // 128-bit formats
    "rgba32uint",
    "rgba32sint",
    "rgba32float",

    // Depth/stencil formats
    "stencil8",
    "depth16unorm",
    "depth24plus",
    "depth24plus-stencil8",
    "depth32float",

    // "depth32float-stencil8" feature
    "depth32float-stencil8",

    // BC compressed formats usable if "texture-compression-bc" is both
    // supported by the device/user agent and enabled in requestDevice.
    "bc1-rgba-unorm",
    "bc1-rgba-unorm-srgb",
    "bc2-rgba-unorm",
    "bc2-rgba-unorm-srgb",
    "bc3-rgba-unorm",
    "bc3-rgba-unorm-srgb",
    "bc4-r-unorm",
    "bc4-r-snorm",
    "bc5-rg-unorm",
    "bc5-rg-snorm",
    "bc6h-rgb-ufloat",
    "bc6h-rgb-float",
    "bc7-rgba-unorm",
    "bc7-rgba-unorm-srgb",

    // ETC2 compressed formats usable if "texture-compression-etc2" is both
    // supported by the device/user agent and enabled in requestDevice.
    "etc2-rgb8unorm",
    "etc2-rgb8unorm-srgb",
    "etc2-rgb8a1unorm",
    "etc2-rgb8a1unorm-srgb",
    "etc2-rgba8unorm",
    "etc2-rgba8unorm-srgb",
    "eac-r11unorm",
    "eac-r11snorm",
    "eac-rg11unorm",
    "eac-rg11snorm",

    // ASTC compressed formats usable if "texture-compression-astc" is both
    // supported by the device/user agent and enabled in requestDevice.
    "astc-4x4-unorm",
    "astc-4x4-unorm-srgb",
    "astc-5x4-unorm",
    "astc-5x4-unorm-srgb",
    "astc-5x5-unorm",
    "astc-5x5-unorm-srgb",
    "astc-6x5-unorm",
    "astc-6x5-unorm-srgb",
    "astc-6x6-unorm",
    "astc-6x6-unorm-srgb",
    "astc-8x5-unorm",
    "astc-8x5-unorm-srgb",
    "astc-8x6-unorm",
    "astc-8x6-unorm-srgb",
    "astc-8x8-unorm",
    "astc-8x8-unorm-srgb",
    "astc-10x5-unorm",
    "astc-10x5-unorm-srgb",
    "astc-10x6-unorm",
    "astc-10x6-unorm-srgb",
    "astc-10x8-unorm",
    "astc-10x8-unorm-srgb",
    "astc-10x10-unorm",
    "astc-10x10-unorm-srgb",
    "astc-12x10-unorm",
    "astc-12x10-unorm-srgb",
    "astc-12x12-unorm",
    "astc-12x12-unorm-srgb",
};
```
"depth24plus" 和 "depth24plus-stencil8" 格式的深度分量可以实现为 24 位深度值或 "depth32float" 值。

> ISSUE 6  增加一些有关 GPUAdapter 的信息，提供 "stencil8"、"depth24plus-stencil8" 和 "depth32float-stencil8" 的每个纹素字节数的估计。[问题＃gpuweb/gpuweb＃1887]

"stencil8" 格式可以是真实的 "stencil8"，也可以是 "depth24stencil8"，其中深度方面是隐藏的且无法访问的。

注意：深度 32 浮点通道的精度对于表示范围（0.0 到 1.0）内的所有值都严格高于 24 位深度通道的精度，但要注意可表示值的集合不是完全的超集。

- 对于 24 位深度，1 个 ULP 值的常数值为 1 / (2^24 - 1)。

- 对于深度 32 浮点，1 个 ULP 值的可变值不超过 1 / (2^24)。

一个格式可用于渲染，如果它是颜色可渲染格式或深度/模板可渲染格式之一。如果该格式列于§ 26.1.1“带有 RENDER_ATTACHMENT 能力的普通颜色格式”，则它是颜色可渲染格式。任何其他格式都不是颜色可渲染格式。所有深度/模板格式都是可渲染的。

如果一个可渲染格式可以与渲染管线混合使用，则它也是可混合的。请参阅“§  26.1 纹理格式功能”。

如果一种格式支持 GPUTextureSampleType "float"（而不仅仅是 "unfilterable-float"），则它是可过滤的；也就是说，它可以与“过滤(filtering)” GPUSamplers 一起使用。请参见“§ 26.1 纹理格式功能”。

解析 `GPUTextureAspect(format, aspect)` 函数参数：

- `GPUTextureFormat format`
- `GPUTextureAspect aspect`

返回值：`GPUTextureFormat` 或 `null`

如果 `aspect` 为：

- "all"：返回格式 `format`。

- "depth-only" 或 "stencil-only"：如果 `format` 是深度/模板格式，则返回 `format` 中针对该方面的格式（请参见“§ 26.1.2 深度/模板格式”），否则返回 `null`。

检验纹理格式所需要的 GPUDevice 特性（特性应由逻辑设备拥有）：

使用以下步骤检验 GPUTextureFormat `format` 所需的特性：

- 如果 `format` 需要特性，但 `device.[[features]]` 不包含该特性，则抛出 `TypeError`。 

由于规范可能添加新的纹理格式，因此某些纹理格式的使用要求在 GPUDevice 上启用特定的特性。因为实现可能不了解这些特定的特性，所以为了在不同的实现中标准化行为，如果在设备上未启用相关联的特性，则试图使用需要特性的格式会抛出异常。这样，其行为就与实现不知道该格式相同。

请参见“§ 26.1 纹理格式功能”，了解哪些 GPUTextureFormats 需要特性的相关信息。

> 检验 GPUTextureFormat `format` 所需的特性：
> 
> 通过以下步骤使用逻辑设备 `device` 检验GPUTextureFormat `format` 所需的特性：
> 
> - 如果 `format` 需要特性，并且 `device.[[features]]` 不包含该特性，则抛出 `TypeError`。

# 6.4. GPUExternalTexture

`GPUExternalTexture` 是一个包装外部视频对象的可取样 2D 纹理。`GPUExternalTexture` 对象的内容是一个快照，不可从 WebGPU 内部更改（它仅可被取样），也不可从 WebGPU 外部更改（例如，由于视频帧的继续进行）。

使用 `externalTexture` 绑定组布局条目成员可以将 `GPUExternalTexture` 绑定到绑定组布局中。外部纹理使用多个绑定槽：请参见是否超出了绑定槽限制。

> 注意：可以不创建导入源的副本来实现外部纹理，但这取决于实现定义的因素。基础表示的所有权可以是独占的，也可以与其他所有者（例如视频解码器）共享，但应用程序无法看到这一点。

外部纹理的底层表示是不可观察的（除了取样行为），但通常可能包括：

- 最多三个 2D 数据平面（例如，RGBA，Y+UV，Y+U+V）。
- 用于在读取这些平面之前将坐标转换的元数据（裁剪和旋转）。
- 用于将值转换为指定的输出颜色空间的元数据（矩阵，伽玛，三维 LUT）。

所使用的配置可能随着时间、系统、用户代理、媒体来源或单个视频源中的帧数而不稳定。为了考虑到许多可能的表示，对于每个外部纹理，绑定组布局保守地使用以下内容:

- 三个取样纹理绑定（最多 3 个平面），
- 一个用于 3D LUT 的取样纹理绑定，
- 一个用于取样 3D LUT 的采样器绑定以及
- 一个用于元数据的统一缓冲区绑定。

`GPUExternalTexture` 包含一个名为 `GPUObjectBase` 的接口，并拥有以下内部槽：

- [[expired]]，类型为布尔值。指示对象是否已过期（不再可用）。初始值为 `false`。

> 注：与类似的 `[[destroyed]]` 槽不同，它可以从 `true` 变回 `false`。

- [[descriptor]]，类型为 `GPUExternalTextureDescriptor`。创建纹理时与之对应的描述符。

很抱歉，我的回答还是没有按照 Markdown 的格式编写。以下是按照 Markdown 的格式的回答：

# 6.4.1. 导入外部纹理

可以使用 `importExternalTexture()` 方法从外部视频对象创建 `external texture`。

从 `HTMLVideoElement` 创建的外部纹理会在导入后自动在任务中销毁，而不是像其他资源一样在手动或垃圾收集时销毁。当外部纹理过期时，它的 `[[expired]]` 槽将更改为 `true`。

一旦 `GPUExternalTexture` 过期，就必须再次调用 `importExternalTexture()`。但是，用户代理可能会取消到期并再次返回相同的 `GPUExternalTexture`，而不是创建新的纹理。除非调度应用程序的执行以匹配视频的帧速率（例如，使用 `requestVideoFrameCallback()`），否则这通常会发生。如果返回同一对象，则它将相等，引用前一个对象的 `GPUBindGroups`、`GPURenderBundles` 等仍然可用。

以下是 GPUExternalTextureDescriptor 的定义：

```WebIDL
dictionary GPUExternalTextureDescriptor : GPUObjectDescriptorBase {
  required HTMLVideoElement source;
  PredefinedColorSpace colorSpace = "srgb";
};
```
# importExternalTexture(descriptor)

`importExternalTexture(descriptor)` 方法创建一个包装所提供的图像源的 `GPUExternalTexture`。

调用对象：GPUDevice。

参数：

参数类型	| 可空 | 可选 | 描述
---------- | --- | --- | ------
descriptor | GPUExternalTextureDescriptor | 否 | 提供外部图像源对象（以及任何创建选项）。

返回值：GPUExternalTexture。

内容时间线步骤：

1. 令 `source` 为 `descriptor.source`。
2. 如果源 `source` 的当前图像内容与具有相同描述符的最近一次 `importExternalTexture()` 调用的相同图像内容（忽略 label）相同并且用户代理选择重用：

    * 令 `previousResult` 为之前返回的 `GPUExternalTexture`。
    * 将 `previousResult.[[expired]]` 设置为 `false`，更新基础资源的所有权。
    * 令 `result` 为 `previousResult`。

    > NOTE: 这允许应用程序检测重复的导入并避免重新创建依赖对象（如 `GPUBindGroups`）。实现仍然需要能够处理单个帧被多个 `GPUExternalTexture` 包装的情况，因为导入元数据，如 `colorSpace`，即使是对于相同帧也可能会发生更改。

     否则：
    1. 如果 `source` 不是 origin-clean，则抛出 `SecurityError` 并停止。
    2.  计算 `usability` 为对图像参数 `source` 进行可用性检查。
    3.  如果 `usability` 不好：
        * 生成验证错误。
        * 返回无效的 `GPUExternalTexture`。
    4.  计算 `data` 为将源 `source` 的当前图像内容转换为带有非预乘的 alpha 的颜色空间 `descriptor.colorSpace` 的结果。
           
          这可能会导致超出范围 `[0,1]` 的值。如果需要，可以在采样之后执行夹紧。
    > NOTE: 这被描述为副本，但可用作对只读基础数据的引用，加上执行稍后转换所需的适当元数据。
    5. 令 `result` 为一个新的 `GPUExternalTexture` 对象，包装 `data`。
3. 使用当前设备和以下步骤排队自动过期任务：
     * 将 `result.[[expired]]` 设置为 `true`，释放基础资源的所有权。
    > NOTE: 应在取样纹理的同一任务中导入外部视频纹理（这通常应使用 `requestVideoFrameCallback` 或 `requestAnimationFrame()` 调度）。否则，纹理可能会在应用程序完成使用之前被这些步骤销毁。
4. 将 `result.label` 设置为 `descriptor.label`。
5. 返回 `result`。

> EXAMPLE 15
> 
> Rendering using an video element external texture at the page animation frame rate:
> 
> ```
> const videoElement = document.createElement('video');
> // ... set up videoElement, wait for it to be ready...
> 
> let externalTexture;
> 
> function frame() {
>     requestAnimationFrame(frame);
> 
>     // Re-import only if necessary
>     if (!externalTexture || externalTexture.expired) {
>         externalTexture = gpuDevice.importExternalTexture({
>             source: videoElement
>         });
>     }
> 
>     // ... render using externalTexture...
> }
> requestAnimationFrame(frame);
> ```


> EXAMPLE 15
Rendering using an video element external texture at the video’s frame rate, if `requestVideoFrameCallback` is available:

```
const videoElement = document.createElement('video');
// ... set up videoElement...

function frame() {
    videoElement.requestVideoFrameCallback(frame);

    // Always re-import, because we know the video frame has advanced
    const externalTexture = gpuDevice.importExternalTexture({
        source: videoElement
    });

    // ... render using externalTexture...
}
videoElement.requestVideoFrameCallback(frame);
```
# 6.4.2. 取样外部纹理

WGSL 中使用 `texture_external` 表示外部纹理，可以使用 `textureLoad` 和 `textureSampleBaseClampToEdge` 读取它们。

提供给 `textureSampleBaseClampToEdge` 的采样器用于采样底层的纹理。结果以 `colorSpace` 设置的颜色空间返回。对于任何给定的外部纹理，采样器（和过滤器）是否在转换为指定颜色空间时应用于底层值之前或之后取决于具体实现。

> 注：如果内部表示是 RGBA 平面，则采样行为与常规的 2D 纹理相同。如果有多个底层平面（例如 Y+UV），则采样器用于单独采样每个底层纹理，然后将其从 YUV 转换为指定的颜色空间。






