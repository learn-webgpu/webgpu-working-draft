GPUBuffer有以下不可变属性:

- size：类型为GPUSize64，只读。GPUBuffer在字节上的分配长度。
- usage：类型为GPUBufferUsageFlags，只读。此GPUBuffer允许使用的用途。
- [[internals]]：类型为buffer internals，只读，覆盖。

GPUBuffer有以下内容时间线属性：

- mapState：类型为GPUBufferMapState，只读。缓冲区的当前GPUBufferMapState：
    - "unmapped"：该缓冲区未映射用于this.getMappedRange()。
    - "pending"：已请求对缓冲区进行映射，但仍在等待响应。它可以成功，也可能因mapAsync()的验证而失败。
    - "mapped"：缓冲区已映射，可以使用this.getMappedRange()。

getter步骤：

内容时间线步骤：

- 如果this.[[mapping]]不为null，则返回"mapped"。
- 如果this.[[pending_map]]不为null，则返回"pending"。
- 返回"unmapped"。

- [[pending_map]]：类型为Promise&lt;void&gt;或null，初始值为null。当前正在等待mapAsync()调用返回的Promise。
- 永远不会存在多个等待映射的请求，因为如果有请求正在进行中，mapAsync()将立即拒绝。
- [[mapping]]：类型为活动缓冲区映射或null，初始值为null。仅在缓冲区当前已映射用于getMappedRange()时设置。否则为Null（即使存在[[pending_map]]）。
- 活动缓冲区映射是具有以下字段的结构：
    * data：类型为Data Block。此GPUBuffer的映射。可以通过ArrayBuffers访问此数据，这些ArrayBuffers是通过getMappedRange()返回的数据视图，并存储在views中。
    * mode：类型为GPUMapModeFlags。映射的GPUMapModeFlags，如在mapAsync()或createBuffer()中指定。
    * range：类型为元祖[unsigned long long, unsigned long long]。映射此GPUBuffer的范围。
    * views：类型为list&lt;ArrayBuffer&gt;。由应用程序通过getMappedRange()返回的ArrayBuffers。它们被跟踪，以便在调用unmap()时可以分离它们。

使用模式和范围初始化活动缓冲区映射：

- 让size等于range[1]-range[0]。
- 让data等于? CreateByteDataBlock(size)。

注意：这可能会导致抛出RangeError。为了保持一致性和可预测性：

- 对于在给定时刻new ArrayBuffer()成功的任何大小，此分配都应在该时刻成功。
- 对于new ArrayBuffer()在给定大小上确定抛出RangeError的情况，此分配也应抛出RangeError。

返回具有以下内容的活动缓冲区映射：

- 将数据设置为data。
- 将模式设置为mode。
- 将范围设置为range。
- 将视图设置为空列表[]。

GPUBuffer的内部对象是buffer internals，它扩展了具有以下设备时间线插槽的内部对象：

- state：缓冲区的当前内部状态：
    - "available"：缓冲区可以在队列操作中使用（除非它无效）。
    - "unavailable"：由于已映射，缓冲区不能在队列操作中使用。
    - "destroyed"：由于已使用destroy()，缓冲区不能在任何操作中使用。
