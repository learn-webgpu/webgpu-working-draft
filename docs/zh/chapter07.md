## 7.1. GPUSampler

`GPUSampler` 编码了可以用于解释纹理资源数据的转换和过滤信息，在着色器中使用。

可以通过 `createSampler()` 创建 `GPUSampler`。

```WebIDL
[Exposed=(Window, DedicatedWorker), SecureContext]
interface GPUSampler {
};
GPUSampler includes GPUObjectBase;
GPUSampler has the following internal slots:

[[descriptor]], of type GPUSamplerDescriptor, readonly
The GPUSamplerDescriptor with which the GPUSampler was created.

[[isComparison]], of type boolean
Whether the GPUSampler is used as a comparison sampler.

[[isFiltering]], of type boolean
Whether the GPUSampler weights multiple samples of a texture.
```

## 7.1.1. GPUSamplerDescriptor

`GPUSamplerDescriptor` 指定用于创建 `GPUSampler` 的选项。

```WebIDL
dictionary GPUSamplerDescriptor : GPUObjectDescriptorBase {
  GPUAddressMode addressModeU = "clamp-to-edge";
  GPUAddressMode addressModeV = "clamp-to-edge";
  GPUAddressMode addressModeW = "clamp-to-edge";
  GPUFilterMode magFilter = "nearest";
  GPUFilterMode minFilter = "nearest";
  GPUMipmapFilterMode mipmapFilter = "nearest";
  float lodMinClamp = 0;
  float lodMaxClamp = 32;
  GPUCompareFunction compare;
  [Clamp] unsigned short maxAnisotropy = 1;
};
```

- `addressModeU`，类型为 `GPUAddressMode`，默认为 `"clamp-to-edge"`。
- `addressModeV`，类型为 `GPUAddressMode`，默认为 `"clamp-to-edge"`。
- `addressModeW`，类型为 `GPUAddressMode`，默认为 `"clamp-to-edge"`。分别指定纹理宽度、高度和深度坐标的地址模式。
- `magFilter`，类型为 `GPUFilterMode`，默认为 `"nearest"`。指定采样区域小于或等于一个纹素时的采样行为。
- `minFilter`，类型为 `GPUFilterMode`，默认为 `"nearest"`。指定采样区域大于一个纹素时的采样行为。
- `mipmapFilter`，类型为 `GPUMipmapFilterMode`，默认为 `"nearest"`。指定采样不同层级之间的行为。
- `lodMinClamp`，类型为 `float`，默认为 `0`。`lodMaxClamp`，类型为 `float`，默认为 `32`。分别指定内部使用的纹理采样的最小和最大细节级别。
- `compare`，类型为 `GPUCompareFunction`。如果提供，采样器将是指定的 `GPUCompareFunction` 的比较采样器。
> 注：Comparison samplers可以使用滤波，但取样结果取决于实现，可能与正常的滤波规则有所不同。
- `maxAnisotropy`，类型为 `unsigned short`，默认为 `1`。指定采样器使用的最大各向异性值的裁剪。

> 注：大多数实现支持最大各向异性值在 1 到 16 的范围内。使用的 `maxAnisotropy` 值将被裁剪为平台支持的最大值。


> ISSUE 7
> 
> ## LOD 计算方式及平台间差异
> 
> 采样器需要确定在纹理中进行采样时要使用的特定 Mipmap 级别来获得最终的像素颜色。Mipmap 是预先渲染的级别，可以根据图像的分辨率选择合适的级别。LOD（Level of Detail）就是确定在给定采样区域（也称为采样脚印）需要使用哪个 Mipmap 级别的计算过程。
> 
> 在计算 LOD 时，通常使用的算法是基于纹理采样器的过滤器和取样脚印的大小，以及所需的细节级别。这个计算过程是非常关键的，因为它涉及到后续的采样过程，包括采样器对 Mipmap 级别选择的加权和过滤器的使用。
> 
> 对于各种平台，LOD 的计算方式和级别可能不同，因此计算出的结果可能会因平台而异。一些实现可能采用固定规则的计算方法，而另一些可能会根据硬件和纹理数据细节来动态地优化计算方法。

> ISSUE 8
> ## 各向异性取样是什么
> 
> 各向异性取样是在处理纹理时使用的一种技术，使得更改纹理采样方式时，不会丢失某些像素的细节，即突出显示细小的结构和形状，通常体现为清晰的线段和锐利的边缘。
> 
> 在一般情况下，纹理在各向同性，即它们在所有方向上都具有相同的细节级别。但有些纹理在某些方向上具有更多或更少的细节级别。各向异性采样器可同时采样不同方向上的不同细节级别使细节表现得更自然。

`GPUAddressMode` 描述了采样器在采样脚印超出采样的纹理的范围时的行为方式。

> ISSUE 9
> ## 取样脚印（Sample Footprint）的详细说明
> 
> 取样脚印是采样器读取纹理时扫描的区域。它是采样器根据纹理坐标相邻的纹素计算出来的，可以理解为代表了采样过程中纹理中所要考虑的像素区域。
> 
> `GPUAddressMode` 描述了如果采样脚印超出采样的纹理的范围，采样器的行为方式。
> 
> - `"clamp-to-edge"`：超出范围的纹素将被限制为边缘的纹素。
> - `"repeat"`：纹理在取样坐标方向上无限重复。
> - `"mirror-repeat"`：与 `"repeat"` 类似，但是纹素被翻转并隔行重复。

```
enum GPUAddressMode {
    "clamp-to-edge",
    "repeat",
    "mirror-repeat",
};
```
- `"clamp-to-edge"`: 纹理坐标被夹紧在 [0.0, 1.0] 范围内。如果采样器的标准化纹理坐标小于 0.0，则采样器返回纹理坐标采样最近的纹理像素颜色；如果标准化纹理坐标超出 1.0，则采样器返回最边缘的像素颜色。
- `"repeat"`: 纹理被水平和垂直地平铺，超出 [0, 1] 范围的纹理坐标循环重复。
- `"mirror-repeat"`: 纹理被水平和垂直地重复平铺，超出 [0, 1] 范围的纹理坐标循环反转并重复。即当整数部分为奇数时，会反转。



GPUFilterMode 和 GPUMipmapFilterMode 用于描述采样器的行为，当采样脚印不完全匹配一个纹素时，采样器会采用这些行为。

```
enum GPUFilterMode {
    "nearest",
    "linear",
};

enum GPUMipmapFilterMode {
    "nearest",
    "linear",
};
```

>"nearest": 返回最靠近纹理坐标的纹素值。

>"linear": 在每个维度上选择两个纹素，并进行线性插值，然后返回插值结果。

GPUCompareFunction 确定比较采样器的行为。如果在着色器中使用了比较采样器，则将输入值与纹理采样值进行比较，在过滤操作中使用比较测试的结果（0.0f 表示通过，1.0f 表示失败）。

> ISSUE 10 describe how filtering interacts with comparison sampling.

```
enum GPUCompareFunction {
    "never",
    "less",
    "equal",
    "less-equal",
    "greater",
    "not-equal",
    "greater-equal",
    "always",
};
```

-   `"never"`: 比较测试永远不会通过。
-   `"less"`: 如果提供的值小于采样值，则提供的值通过比较测试。
-   `"equal"`: 如果提供的值等于采样值，则提供的值通过比较测试。
-   `"less-equal"`: 如果提供的值小于或等于采样值，则提供的值通过比较测试。
-   `"greater"`: 如果提供的值大于采样值，则提供的值通过比较测试。
-   `"not-equal"`: 如果提供的值不等于采样值，则提供的值通过比较测试。
-   `"greater-equal"`: 如果提供的值大于或等于采样值，则提供的值通过比较测试。
-   `"always"`: 比较测试总是通过。

## 7.1.2. 创建采样器

### createSampler(descriptor)

创建 GPUSampler。

在 GPUDevice 对象上调用该方法。

#### 参数

参数类型|可空|可选|描述
-|-|-|-
descriptor|GPUSamplerDescriptor|✘|✔|创建 GPUSampler 的描述符。

#### 返回值

类型：GPUSampler。

#### 时间线

* 令 s 为一个新的 GPUSampler 对象。
* 在 this 的设备上，执行初始化步骤。
* 返回 s。

#### 设备初始化步骤

如果下列任何一项未满足，则生成一个验证错误，使 s 无效，并停止执行：

* this 是有效的。
* descriptor.lodMinClamp ≥ 0。
* descriptor.lodMaxClamp ≥ descriptor.lodMinClamp。
* descriptor.maxAnisotropy ≥ 1。

> 注意：大多数实现支持 maxAnisotropy 值在 1 到 16 的范围内，包含边界。maxAnisotropy 的值将会被夹紧在平台支持的最大值。

如果 descriptor.maxAnisotropy > 1：

* descriptor.magFilter、descriptor.minFilter 和 descriptor.mipmapFilter 必须为 "linear"。

将 s.[[descriptor]] 设为 descriptor。

如果 s.[[descriptor]] 的 compare 属性为空或 undefined，则将 s.[[isComparison]] 设为 false，否则将其设为 true。

如果 minFilter、magFilter 或 mipmapFilter 中的任意一个都不等于 "linear"，则将 s.[[isFiltering]] 设为 false，否则将其设为 true。

创建一个执行三线性过滤，并且以重复的方式对纹理坐标进行采样的 GPUSampler：

```
const sampler = gpuDevice.createSampler({
  addressModeU: 'repeat',
  addressModeV: 'repeat',
  magFilter: 'linear',
  minFilter: 'linear',
  mipmapFilter: 'linear'
});
```







