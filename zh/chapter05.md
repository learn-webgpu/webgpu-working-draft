---
theme: juejin
---
# 5.1. GPUBuffer
GPUBuffer表示可以在GPU操作中使用的一块内存块。数据以线性布局存储，这意味着可以通过其相对于GPUBuffer起始位置的偏移量来访问分配的每个字节，但根据操作的对齐限制而定。某些GPUBuffers可以映射，这使得可以通过称为其映射的ArrayBuffer访问内存块。
通过createBuffer()可以创建GPUBuffers，某些Buffers可以在创建时进行映射。
```
[Exposed=(Window, DedicatedWorker), SecureContext]
interface GPUBuffer {
    readonly attribute GPUSize64 size;
    readonly attribute GPUBufferUsageFlags usage;

    readonly attribute GPUBufferMapState mapState;

    Promise<undefined> mapAsync(GPUMapModeFlags mode, optional GPUSize64 offset = 0, optional GPUSize64 size);
    ArrayBuffer getMappedRange(optional GPUSize64 offset = 0, optional GPUSize64 size);
    undefined unmap();

    undefined destroy();
};
GPUBuffer includes GPUObjectBase;

enum GPUBufferMapState {
    "unmapped",
    "pending",
    "mapped",
};

```

### GPUBuffer的不变属性

1. size：类型为GPUSize64，只读。GPUBuffer在字节上的分配长度。
2. usage：类型为GPUBufferUsageFlags，只读。此GPUBuffer允许使用的用途。
3. [[internals]]：类型为buffer internals，只读，覆盖。

### GPUBuffer的内容时间线属性

1. mapState：类型为GPUBufferMapState，只读。缓冲区的当前GPUBufferMapState：
   - "unmapped"：缓冲区未映射用于 `this.getMappedRange()`。
   - "pending"：缓冲区映射已被请求，但是还未完成验证（通过 `mapAsync()`）。
   - "mapped"：缓冲区已映射，可以通过 `this.getMappedRange()` 进行访问。

Getter方法：

**内容时间线步骤：**

1. 如果 `this.[[mapping]]` 不为 `null`，则返回 "mapped"。
2. 如果 `this.[[pending_map]]` 不为 `null`，则返回 "pending"。
3. 返回 "unmapped"。

#### GPUBuffer的活动映射（`[[mapping]]`）属性

1. `[[pending_map]]`：类型为 `Promise<void>` 或 `null`，初始值为 `null`。代表当前待处理的映射请求的 Promise。
2. 永远只存在一个待处理映射请求，因为如果已有请求正在进行中，`mapAsync()` 将立即拒绝。
3. `[[mapping]]`：类型为活动映射或 `null`，初始值为 `null`。仅在缓冲区已映射用于 `getMappedRange()` 时设置。否则为 `null`（即使有 `[[pending_map]]` 存在）。
4. 活动映射是一个结构体，它具有以下字段：
   - `data`（`Data Block` 类型）：表示此 GPUBuffer 的映射。通过 `getMappedRange()` 返回的 ArrayBuffer 视图可以访问此数据，且这些视图存储在 `views` 中。
   - `mode`（`GPUMapModeFlags` 类型）：与映射对应的 `GPUMapModeFlags`。
   - `range`（`[unsigned long long, unsigned long long]` 类型的元组）：已映射此 GPUBuffer 的范围。
   - `views`（list<ArrayBuffer> 类型）：由应用程序通过 `getMappedRange()` 返回的 ArrayBuffers。它们被跟踪，以便在调用 `unmap()` 时可以解除引用。

初始化活动映射然后设置 `mode` 和 `range`：

```javascript
Let size be range[1] - range[0].
Let data be ? CreateByteDataBlock(size).
Return an active mapping with:
 - data set to data.
 - mode set to mode.
 - range set to range.
 - views set to [].
```

### GPUBuffer的内部对象

GPUBuffer的内部对象是 `buffer internals`，它扩展了具有以下设备时间线插槽的内部对象：

1. state：缓冲区的当前内部状态：
   - "available"：缓冲区可以在队列操作中使用（除非它无效）。
   - "unavailable"：由于已映射，缓冲区不能在队列操作中使用。
   - "destroyed"：由于已使用 `destroy()`，缓冲区不能在任何操作中使用。

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/43c4ba83178445ea91b7ae1a934c06db~tplv-k3u1fbpfcp-watermark.image?)
    

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9381491f04774597ac5ea2778364accb~tplv-k3u1fbpfcp-watermark.image?)
    
# 5.1.1. GPUBufferDescriptor

```
dictionary GPUBufferDescriptor : GPUObjectDescriptorBase {
  required GPUSize64 size;
  required GPUBufferUsageFlags usage;
  boolean mappedAtCreation = false;
};
```
GPUBufferDescriptor 有以下成员:

* size：类型为 GPUSize64，表示缓冲区的大小（以字节为单位）。
* usage：类型为 GPUBufferUsageFlags，表示缓冲区所允许的使用方式。
* mappedAtCreation：类型为 boolean，默认为 false。如果设置为 true，创建缓冲区时就将其映射，可以立即调用 `getMappedRange()`。即使缓冲区的创建最终失败，确保在缓冲区未被映射时写入/读取映射范围仍然是有效的。

即使缓冲区创建最终失败，确保在缓冲区未被映射时写入/读取映射范围仍然是有效的。
 
# 5.1.2. Buffer Usages

`GPUBufferUsageFlags` 是一个用于确定可在缓冲区创建之后进行哪些操作的标志集。每个标志都代表不同的操作。以下是可用的标志：

```javascript
typedef [EnforceRange] unsigned long GPUBufferUsageFlags;
[Exposed=(Window, DedicatedWorker), SecureContext]
namespace GPUBufferUsage {
  const GPUFlagsConstant MAP_READ      = 0x0001;
  const GPUFlagsConstant MAP_WRITE     = 0x0002;
  const GPUFlagsConstant COPY_SRC      = 0x0004;
  const GPUFlagsConstant COPY_DST      = 0x0008;
  const GPUFlagsConstant INDEX         = 0x0010;
  const GPUFlagsConstant VERTEX        = 0x0020;
  const GPUFlagsConstant UNIFORM       = 0x0040;
  const GPUFlagsConstant STORAGE       = 0x0080;
  const GPUFlagsConstant INDIRECT      = 0x0100;
  const GPUFlagsConstant QUERY_RESOLVE = 0x0200;
};
```

以下是可用标志的详细说明：

- `MAP_READ`：可将缓冲区映射为只读模式，例如调用 `mapAsync()` 时使用 `GPUMapMode.READ`。
  + 只能与 `COPY_DST` 组合。
- `MAP_WRITE`：可将缓冲区映射为可写入模式，例如调用 `mapAsync()` 时使用 `GPUMapMode.WRITE`。
  + 只能与 `COPY_SRC` 组合。
- `COPY_SRC`：可作为复制操作的源缓冲区使用，例如作为 `copyBufferToBuffer()` 或 `copyBufferToTexture()` 方法的源参数。
- `COPY_DST`：可作为复制或写入操作的目标缓冲区使用，例如作为 `copyBufferToBuffer()`、`copyTextureToBuffer()` 方法的目标参数，或作为 `writeBuffer()` 方法的目标参数。
- `INDEX`：可作为索引缓冲区使用，例如传递给 `setIndexBuffer()` 方法。
- `VERTEX`：可作为顶点缓冲区使用，例如传递给 `setVertexBuffer()` 方法。
- `UNIFORM`：可作为 uniform 缓冲区使用，例如作为具有 `buffer.type` 为 "uniform" 的 `GPUBufferBindingLayout` 的绑定组条目。
- `STORAGE`：可作为存储缓冲区使用，例如作为具有 `buffer.type` 为 "storage" 或 "read-only-storage" 的 `GPUBufferBindingLayout` 的绑定组条目。
- `INDIRECT`：可用于存储间接命令参数，例如作为 `drawIndirect()` 或 `dispatchWorkgroupsIndirect()` 方法的 `indirectBuffer` 参数。
- `QUERY_RESOLVE`：可用于捕获查询结果，例如作为 `resolveQuerySet()` 方法的目标参数。
    
# 5.1.3. Buffer Creation

`createBuffer(descriptor)` 方法用于创建一个 `GPUBuffer`。它是在 `GPUDevice` 对象上调用的，接受一个 `GPUBufferDescriptor` 对象作为参数，并返回创建出的 `GPUBuffer` 对象。

```javascript
createBuffer(descriptor: GPUBufferDescriptor) => GPUBuffer
```

在创建缓冲区时，可以为其指定以下属性：

- `size`：缓冲区的大小。
- `usage`：缓冲区的用途。
- `mappedAtCreation`：一个布尔值，默认为 `false`，表示创建缓冲区时是否应该将其映射。如果设置为 `true`，将创建一个已映射缓冲区。这样可以立即对其进行读写操作。

在进行缓冲区创建时，按照以下步骤执行：

- 为缓冲区创建一个 `WebGPU` 对象。
- 将缓冲区的 `size` 和 `usage` 属性设置为所提供的描述器中的属性值。
- 如果 `mappedAtCreation` 为 `true`，使用 `initialize an active buffer mapping` 方法创建一个活动的缓冲区映射，其模式为 WRITE，范围为 [0, descriptor.size]。
- 执行设备任务时间轴的初始化步骤。
- 返回创建的 `GPUBuffer` 对象。

在设备任务时间轴中，对以下条件进行验证：

- `device` 应该是有效的。
- `descriptor.usage` 不能为 0。
- `descriptor.usage` 应该是 `device` 允许的缓冲区用途的子集。
- 如果 `descriptor.usage` 包含 `MAP_READ`，则 `descriptor.usage` 不应包含除 `COPY_DST` 之外的其他标志。
- 如果 `descriptor.usage` 包含 `MAP_WRITE`，则 `descriptor.usage` 不应包含除 `COPY_SRC` 之外的其他标志。
- `descriptor.size` 必须小于或等于 `device.[[device]].[[limits]].maxBufferSize`。
- 如果 `descriptor.mappedAtCreation` 为 `true`，则 `descriptor.size` 必须为 4 的倍数。
- 如果缓冲区创建失败，并且 `descriptor.mappedAtCreation` 为 `false`，则任何调用 `mapAsync()` 的请求将拒绝，因此为实现映射而分配的任何资源都可以被丢弃或回收。

在创建成功后，根据 `descriptor.mappedAtCreation` 的值，将缓冲区的状态设置为 `available`（如果为 `false`）或 `unavailable`（如果为 `true`）。然后在设备上为缓冲区分配内存。

以下是创建一个可写入的 128 字节 uniform 缓冲区的示例代码：

```javascript
const buffer = gpuDevice.createBuffer({
  size: 128,
  usage: GPUBufferUsage.UNIFORM | GPUBufferUsage.COPY_DST
});
```
# 5.1.4. Buffer Destruction

当应用程序不再需要 `GPUBuffer`，可以通过调用 `destroy()` 方法来失去对它的访问权限，以便在垃圾回收之前释放它。销毁缓冲区也会取消映射，释放分配给映射内存的任何内存。

```javascript
destroy() => undefined
```

在执行 `destroy()` 方法时，会按照以下步骤执行：

- 首先调用 `unmap()` 方法取消映射。
- 在设备任务时间轴上执行以下步骤：

- 将 `[[internals]].state` 设为 `"destroyed"`。

注意：由于不能再使用此缓冲区队列更多的操作，因此实现可以释放资源分配，包括刚刚取消映射的映射内存。注意，该方法可以多次调用。
# 5.2. Buffer Mapping

应用程序可以通过映射 `GPUBuffer` 来请求访问其内容，可以通过代表 GPUBuffer 分配的一部分的 ArrayBuffer 来访问。使用 `mapAsync()` 异步请求映射缓冲区，以便用户代理可以确保 GPU 在应用程序访问其内容之前完成使用 GPUBuffer。已映射的 GPUBuffer 不能被 GPU 使用，并且必须在提交使用其工作之前使用 `unmap()` 进行取消映射。

一旦 `GPUBuffer` 被映射，应用程序就可以使用 `getMappedRange()` 同步地获取其内容的范围访问权限。返回的 ArrayBuffer 只能由 `unmap()`（直接或通过 `GPUBuffer.destroy()` 或 `GPUDevice.destroy()`）取消关联，不能被转移。任何尝试进行其他操作的操作都会抛出 TypeError。

`GPUMapModeFlags` 枚举定义了一些标志：

- `GPUMapMode.READ`：只在使用 `MAP_READ` 用途创建的缓冲区上有效。读取缓冲区的当前值。
- `GPUMapMode.WRITE`：只在使用 `MAP_WRITE` 用途创建的缓冲区上有效，用于写入缓冲区。

`GPUBuffer.mapAsync()` 方法用于映射 `GPUBuffer`，其接受以下参数：

```javascript
mapAsync(mode: GPUMapModeFlags, offset: GPUSize64, size: GPUSize64) => Promise<undefined>
```

- `mode`：缓冲区被映射时应该如何映射。
- `offset`：要映射的缓冲区中的起始偏移量。
- `size`：要映射的字节数的长度。

返回一个 `Promise` 对象，该对象将在缓冲区内容准备好能够通过 `getMappedRange()` 访问时进行解析。

注意：此 `Promise` 的解析仅表示已映射缓冲区。它不保证任何在内容时间线上可见的其他操作的完成，特别是不意味着任何从 `onSubmittedWorkDone()` 或其他 `GPUBuffers` 的调用返回的 Promise 已经解析。返回 `onSubmittedWorkDone()` 的 Promise 的解析意味着先前调用的那些 GPUBuffers 上的 `mapAsync()` 调用的完成。

以下是 `GPUBuffer.mapAsync()` 的具体步骤：

- 首先，检测以前是否调用过 `mapAsync()`。
- 如果 `[[pending_map]]` 不为 `null`，则返回一个拒绝 Promise，该 Promise 的理由为 `OperationError`。
- 创建一个新的 Promise 对象 `p`，并将 `[[pending_map]]` 设置为它。
- 在设备任务时间轴上执行验证步骤。
- 返回 Promise 对象 `p`。

在设备任务时间轴上的验证步骤如下：

1. 如果 `size` 未定义，则将 `rangeSize` 设置为 `max(0, this.size - offset)`。
- 否则，将 `rangeSize` 设置为 `size`。
2. 如果以下任一条件未满足，则执行映射失败步骤，并生成验证错误：

- `this` 是有效的 `GPUBuffer` 对象。
- `this.[[internals]].state` 的值为 `"available"`。
- `offset` 是 8 的倍数。
- `rangeSize` 是 4 的倍数。
- `offset + rangeSize` 不超过 `this.size`。
- `mode` 中的标志位仅由 `GPUMapMode` 中定义的位组成。
- `mode` 中恰好包含 `READ` 或 `WRITE` 中的一个。
- 如果 `mode` 中包含 `READ`，则 `this.usage` 必须包含 `MAP_READ`。
- 如果 `mode` 中包含 `WRITE`，则 `this.usage` 必须包含 `MAP_WRITE`。
    
接着：

    1. 在内容时间轴上触发映射失败步骤。

    2. 生成一个验证错误。

    3. 返回。

3.  `this.[[internals]].state` 设置为 `"unavailable"`。

> 注意：由于缓冲区已经映射，因此在此完成和取消映射的过程中，缓冲区的内容不会更改。

4. 如果 `this.[[device]]` 已经失去或即将失去：

    1. 触发内容时间轴的映射失败步骤。

否则，在设备任务时间轴上执行以下步骤：
    
- 当前所有已排队的使用此缓冲区的操作在处理完成后（无论它们是否使用此缓冲区），
- 并在设备任务时间轴被告知所有已排队的操作都已完成之前（无论它们是否使用此缓冲区，也无论它们使用其它缓冲区）执行以下步骤：
    1. 将 `internalStateAtCompletion` 设置为 `this.[[internals]].state`。
     > 备注：仅当此时缓冲区尚未通过 `unmap()` 取消映射时，才会出现 `[[pending_map]] != p` 的情况。
    2. 从缓冲区的 `offset` 位置开始，以 `rangeSize` 字节长的长度读取缓冲区的内容，并将结果存储到 `dataForMappedRegion` 中。
    3. 触发内容时间轴的映射成功步骤。

 如果成功映射了缓冲区，应用程序可以使用 `getMappedRange()` 方法同步访问缓冲区的内容范围。映射成功时实际执行的步骤如下：

 1.  如果 `this.[[pending_map]] != p`：
  > 注意：映射已被取消，由 `unmap()` 导致。
    1. 断言 Promise `p` 被拒绝. 
    2. 返回
    
2. 断言 Promise `p` 处于等待状态。
3. 断言 `internalStateAtCompletion` 等于 `"unavailable"`。
4. 创建一个包含模式 `mode` 和范围 `[offset, offset + rangeSize]` 的已初始化活动缓冲区映射，并将其保存到 `mapping` 变量中。
如果分配失败：
  * 将 `this.[[pending_map]]` 设置为 `null`，并使用 RangeError 拒绝 Promise `p`。
  * 返回。
5.  将 `dataForMappedRegion` 作为映射数据存储在 `mapping.data` 中。
6.  将 `mapping` 保存到 `this.[[mapping]]` 中。
7.  将 `this.[[pending_map]]` 设置为 `null`，并解析 Promise `p`。
    
    ---
    
Content timeline map failure steps
 1. 如果this.[[pending_map]] != p：
    1. 注意：地图已被unmap（）取消。
    2. 断言p已被拒绝。
    3. 返回。
2.  断言p仍处于待定状态。
3.  将this.[[pending_map]]设置为null，并使用OperationError拒绝p。

## getMappedRange(offset, size)
获取给定映射范围中GPUBuffer的内容，并将其返回为ArrayBuffer。

### 调用方式
`GPUBuffer this`

### 参数
| 参数 | 类型 | 可选 | 描述 |
|-----------|------|----------|-------------|
| offset    | `GPUSize64` | ✘ | 要返回缓冲区内容的偏移量（以字节为单位）。|
| size      | `GPUSize64` | ✘ | 要返回的ArrayBuffer的大小（以字节为单位）。|

### 返回值
`ArrayBuffer`

### 时间线步骤 
1. 如果size为missing：
   1. 让rangeSize为max（0，this.size - offset）。
    否则，让rangeSize为size。
2. 如果以下任一条件未满足，则抛出OperationError并停止。
   1. this.[[mapping]]不为null。
   2. offset是8的倍数。
   3. rangeSize是4的倍数。
   4. offset ≥ this.[[mapping]].range[0]。
   5. offset + rangeSize ≤ this.[[mapping]].range[1]。
   6. [offset，offset + rangeSize）不与this.[[mapping]].views中的另一范围重叠。
   >注意：即使mapAtCreation的GPUBuffer无效，获取映射范围也始终有效，因为Content timeline可能不知道它无效。
3. 让`data`为`this.[[mapping]].data`。
4. 让`view`为`一个大小为`rangeSize`的ArrayBuffer，但其指针可变地引用`data`的内容，该内容为`offset`(`offset - [[mapping]].range[0])`。
   >注意：此处可能不会引发RangeError，因为数据已经在`mapAsync()`或`createBuffer()`中分配。
5. 将`view.[[ArrayBufferDetachKey]]`设置为“WebGPUBufferMapping”。
   >注意：如果尝试`DetachArrayBuffer`，除了`unmap()`之外，这将导致抛出TypeError。
6. 将`view`添加到`this.[[mapping]].views`中。
7. 返回`view`。
    
> 注意：如果在未检查地图状态的情况下成功调用`getMappedRange()`，用户代理应该考虑发出可见的开发人员警告，方法是等待`mapAsync()`成功，查询`mapState`为“mapped”，或等待稍后的`onSubmittedWorkDone()`调用成功。
    
## unmap()
取消GPUBuffer映射范围并使其内容再次可以被GPU使用。

Called on: `GPUBuffer this`.
Returns: `undefined`

### 时间线步骤 

1. 如果`this.[[pending_map]]`不为null：
   1. 用`AbortError`拒绝`this.[[pending_map]]`。
   2. 将`this.[[pending_map]]`设置为null。
2. 如果`this.[[mapping]]`为null：
   1. 返回。
3. 对于`this.[[mapping]].views`中的每个`ArrayBuffer ab`：
   1. 执行`DetachArrayBuffer(ab, "WebGPUBufferMapping")`。
4. 让`bufferUpdate`为null。
5. 如果`this.[[mapping]].mode`包含`WRITE`：
   1. 将`bufferUpdate`设置为`{ data: this.[[mapping]].data, offset: this.[[mapping]].range[0] }`。
   注意：当没有使用`WRITE`模式映射缓冲区时，再取消映射，应用程序对已映射范围的`ArrayBuffer`进行的任何本地修改将被丢弃，并且不会影响稍后映射的内容。
6. 将`this.[[mapping]]`设置为null。
7. 在`this.[[device]]`的设备时间轴上执行以下步骤。

### 设备时间轴步骤： 
1. 如果`this.[[device]]`无效，则返回。
2. 如果`bufferUpdate`不为null：
   1. 在`this.[[device]].queue`的队列时间轴上执行以下步骤：

### 队列时间轴步骤：
更新偏移量为`bufferUpdate.offset`上的内容为`bufferUpdate.data`。

设置`this.[[internals]].state`值为 `"available"`。