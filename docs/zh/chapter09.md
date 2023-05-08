## 9.1. GPUShaderModule

```
[Exposed=(Window, DedicatedWorker), SecureContext]
interface GPUShaderModule {
    Promise<GPUCompilationInfo> getCompilationInfo();
};
GPUShaderModule includes GPUObjectBase;
```
`GPUShaderModule` 是对内部着色器模块对象的引用.



# 9.1.1. 着色器模块创建

```WebIDL
dictionary GPUShaderModuleDescriptor : GPUObjectDescriptorBase {
  required USVString code;
  object sourceMap;
  record<USVString, GPUShaderModuleCompilationHint> hints;
};
```

- `code`，类型为 `USVString`。着色器模块的 WGSL 源代码。
- `sourceMap`，类型为 `object`。如果定义，则可以解释为源映射 v3 格式。
- `hints`，类型为 `record<USVString, GPUShaderModuleCompilationHint>`。如果定义，则将着色器的入口点名称映射到 `GPUShaderModuleCompilationHint`。没有对其中的任何 `GPUShaderModuleCompilationHint` 执行验证。实现应使用 `GPUShaderModuleCompilationHint` 中存在的任何信息，在 `createShaderModule()` 中尽可能多地执行编译。入口点名称遵循在 WGSL 标识符比较中定义的规则。

> 注：在 `hints` 中提供信息除了提高性能外，并没有任何其他可观察的效果。因为单个着色器模块可以包含多个入口点，一个着色器模块可以创建多个管线。实现中为了提高性能，可以在 `createShaderModule()` 中做尽可能多的编译，而不是多次在多个调用 `createComputePipeline()`/`createRenderPipeline()` 中执行编译。

`createShaderModule(descriptor)`：创建一个 `GPUShaderModule`。

- 被调用于：`GPUDevice this`。
- 参数：

    | Parameter | Type                           | Nullable | Optional | Description                      |
    | --------- | ------------------------------ | -------- | -------- | -------------------------------- |
    | descriptor | `GPUShaderModuleDescriptor`    | ✘        | ✘        | 要创建的 `GPUShaderModule` 的描述信息。 |

- 返回值：`GPUShaderModule`

内容时间线步骤：

- 创建一个新的 `GPUShaderModule` 对象 `sm`。
- 在此设备的重建时间线上发出初始化步骤。
- 返回 `sm`。

设备时间线的初始化步骤：

1.  令 `result` 是使用 `descriptor.code` 创建 WGSL 着色器模块的结果。
2.  如果以下任何条件不被满足，则产生验证错误，使 `sm` 无效，并返回。

  - `this` 必须是有效的。
  - `result` 不能是着色器创建程序错误。

> ISSUE 12 Should internal errors ([uncategorized errors](https://gpuweb.github.io/gpuweb/wgsl/#uncategorized-error)) be allowed here or should we force them to be deferred to pipeline creation? [[Issue #gpuweb/gpuweb#2308]](https://github.com/gpuweb/gpuweb/issues/2308)

> ISSUE 13
> [](https://www.w3.org/TR/2023/WD-webgpu-20230425/#issue-f8aca447) Describe remaining `createShaderModule()` validation and algorithm steps.

> 注：用户代理不应在此处包含详细的编译器错误消息或着色器文本，这些细节可以通过 `getCompilationInfo()` 方法获得。用户代理应为开发人员提供易于调试的人类可读格式的错误详细信息（例如，作为浏览器开发者控制台中的警告信息，可扩展以显示完整的着色器源）。
> 
> 由于在生产应用程序中着色器编译错误应很少发生，用户代理可以选择在不考虑错误处理（GPU 错误范围或未捕获的 `error` 事件处理程序）的情况下将其显示给开发人员，例如作为可展开的警告。如果不这样做，则应提供和记录另一种方法来让开发人员访问人类可读的错误详细信息，例如通过添加一个复选框以无条件显示错误，或者在将 `GPUCompilationInfo` 对象记录到控制台时显示人类可读的详细信息。


Create a `GPUShaderModule` from WGSL code:

```
// A simple vertex and fragment shader pair that 用于将视口填充为红色
const shaderSource = `
    var<private> pos : array<vec2<f32>, 3> = array<vec2<f32>, 3>(
        vec2(-1.0, -1.0), vec2(-1.0, 3.0), vec2(3.0, -1.0));

    @vertex
    fn vertexMain(@builtin(vertex_index) vertexIndex : u32) -> @builtin(position) vec4<f32> {
        return vec4(pos[vertexIndex], 1.0, 1.0);
    }

    @fragment
    fn fragmentMain() -> @location(0) vec4<f32> {
        return vec4(1.0, 0.0, 0.0, 1.0);
    }
`;

const shaderModule = gpuDevice.createShaderModule({
    code: shaderSource,
});
```

# 9.1.1.1. 着色器模块编译提示

着色器模块编译提示是可选的，可根据给定的 `GPUShaderModule` 入口点在未来的使用中如何使用指定的额外信息。对于某些实现，此信息可能有助于更早地编译着色器模块，从而可能提高性能。

以下是用于定义 `GPUShaderModuleCompilationHint` 的字典：

```WebIDL
dictionary GPUShaderModuleCompilationHint {
  (GPUPipelineLayout or GPUAutoLayoutMode) layout;
};

```

- `layout`，类型为 `GPUPipelineLayout` 或 `GPUAutoLayoutMode`。`GPUShaderModule` 可能将来在 `createComputePipeline()` 或 `createRenderPipeline()` 调用中使用的 `GPUPipelineLayout`，或者如果设置为 "auto"，则使用此提示所关联入口点的默认管线布局。

注：如果可能，作者应在 `createShaderModule()` 和 `createComputePipeline()`/`createRenderPipeline()` 中提供相同的信息。

如果作者在调用 `createShaderModule()` 时无法提供提示信息，通常不应将 `createShaderModule()` 的调用延迟，而应从提示或 `GPUShaderModuleCompilationHint` 中省略未知信息。省略此信息可能会导致编译被推迟到 `createComputePipeline()`/`createRenderPipeline()`。

如果作者不确定传递给 `createShaderModule()` 的提示信息是否与稍后使用相同模块传递给 `createComputePipeline()`/`createRenderPipeline()` 的信息匹配，他们应避免在 `createShaderModule()` 中传递该信息，因为将不匹配的信息传递到 `createShaderModule()` 中可能会导致不必要的编译。


# 9.1.2. 着色器模块编译信息

枚举 `GPUCompilationMessageType`：

```WebIDL
enum GPUCompilationMessageType {
  "error",
  "warning",
  "info",
};
```

定义 `GPUCompilationMessage` 和 `GPUCompilationInfo` 接口来描述 GPU 着色器模块编译过程中生成的错误，警告或其他信息：

```WebIDL
[Exposed=(Window, DedicatedWorker), Serializable, SecureContext]
interface GPUCompilationMessage {
  readonly attribute DOMString message;
  readonly attribute GPUCompilationMessageType type;
  readonly attribute unsigned long long lineNum;
  readonly attribute unsigned long long linePos;
  readonly attribute unsigned long long offset;
  readonly attribute unsigned long long length;
};

[Exposed=(Window, DedicatedWorker), Serializable, SecureContext]
interface GPUCompilationInfo {
  readonly attribute FrozenArray<GPUCompilationMessage> messages;
};
```

`GPUCompilationMessage` 描述由 GPU 着色器模块编译器生成的消息。这些消息旨在供开发人员阅读，以帮助诊断其着色器代码的问题。每个消息可以对应于着色器代码中的一个点，一个子字符串或根本不对应于代码中的任何特定点。

`GPUCompilationMessage` 具有以下属性：

- `message`，类型为 `DOMString`，只读。这个编译消息的人类可读的本地化文本。

注意：该消息应遵循语言和方向信息的最佳实践。这包括利用任何未来标准，这些标准涉及报告字符串语言和方向元数据。

- `type`，类型为 `GPUCompilationMessageType`，只读。消息的严重级别。如果类型为 "error"，则对应于着色器创建错误。

- `lineNum`，类型为 `unsigned long long`，只读。消息对应的着色器代码中的行号。该值基于 1，因此 `lineNum` 为 1 表示着色器代码的第一行。换行符分隔行。

如果消息对应于子字符串，则此属性指向子字符串开始的行。如果消息不对应于着色器代码中的任何特定点，则必须为 0。

- `linePos`，类型为 `unsigned long long`，只读。从着色器代码中行 `lineNum` 的开头到消息对应点或子字符串开头处的 UTF-16 代码单元的偏移量。该值基于 1，因此 `linePos` 为 1 表示该行的第一个代码单元。

如果消息对应于子字符串，则此属性指向子字符串的第一个 UTF-16 代码单元。如果消息不对应于着色器代码中的任何特定点，则必须为 0。

- `offset`，类型为 `unsigned long long`，只读。从着色器代码的开头开始的 UTF-16 代码单元的偏移量，到消息对应的点或子字符串的开始。必须引用与 `lineNum` 和 `linePos` 相同的位置。如果消息不对应于着色器代码中的任何特定点，则必须为 0。

- `length`，类型为 `unsigned long long`，只读。消息所对应的子字符串中的 UTF-16 代码单元数。如果消息不对应于子字符串，则 `length` 必须为 0。

> 注意：`GPUCompilationMessage.lineNum` 和 `GPUCompilationMessage.linePos` 基于 1，因为它们的最常见用途是打印人类可读的消息，可以与许多文本编辑器中显示的行和列编号对应。

> 注意：`GPUCompilationMessage.offset` 和 `GPUCompilationMessage.length` 可以适当地传递给 `substr()`，以检索消息所对应的着色器代码的子字符串。

### getCompilationInfo()

`getCompilationInfo()` 方法返回在`GPUShaderModule`的编译期间生成的任何消息。消息的位置，顺序和内容由实现定义。特别地，消息可能不是按行号排序的。

在 `GPUShaderModule` 上调用此方法，
**Returns:**  `Promise`<`GPUCompilationInfo`>
其中包含有关编译期间生成的任何错误，警告或信息的消息。

按以下方式执行：

1. 将 `contentTimeline` 设为当前 Content 时间线。
2. 创建一个新的 `Promise`。
3. 在此设备时间轴上执行同步步骤。
4. 返回该 `Promise`。

设备时间轴同步步骤：
1. 当设备时间轴收到通知，此着色器模块创建过程已经完成：
    1. 将 `messages` 初始化为在此着色器模块创建期间生成的任何错误、警告或信息消息的列表。
    2. 在 `contentTimeline` 上执行后续步骤。
  
[Content timeline](https://www.w3.org/TR/2023/WD-webgpu-20230425/#content-timeline) steps:

1. 创建一个新的 `GPUCompilationInfo`。
2. 针对 messages 列表中的每个消息进行以下操作：
   1. 创建一个新的 `GPUCompilationMessage`为m。
   2. 设置 `m.message` 为 `message` 的文本内容。
   3. 如果 `message` 是着色器创建错误，则将 `m.type` 设置为 "error"。
     - 如果 `message` 是警告，则将 `m.type` 设置为 "warning"。
     - 否则将 `m.type` 设置为 "info"。
   4. 如果消息与着色器代码中的特定子串或位置相关联：
      1. 将 `m.lineNum` 设置为消息所引用的第一行的基于 1 的行号。
      2. 将 `m.linePos` 设置为消息所引用的第一行上的基于 1 的第一个 UTF-16 代码单元的编号，如果消息引用整行则设置为 1。
      3. 将 `m.offset` 设置为消息所引用的子串或位置的开头到着色器开头的 Unicode 代码单元数。
      4. 将 `m.length` 设置为消息所引用的子串的长度，以 UTF-16 代码单元为单位，如果消息引用位置则将其设置为 0。
      否则：将 `m.lineNum`、`m.linePos`、`m.offset` 和 `m.length` 都设置为 0。
   5. 将 `m` 添加到 `info.messages` 列表中。
3. 将 `promise` 解析为 `info`。
