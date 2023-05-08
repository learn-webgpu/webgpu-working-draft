> 基于W3C翻译中文版，如有翻译不准确，欢迎随时提出改进建议，持续更新中。。。
## 一、简介
### WebGPU
[W3C工作草案](https://www.w3.org/standards/types#WD)，2023年4月24日
##### **More details about this document**

**This version:** <https://www.w3.org/TR/2023/WD-webgpu-20230424/>

**Latest** **published** **version:** <https://www.w3.org/TR/webgpu/>

**Editor's** **Draft:** <https://gpuweb.github.io/gpuweb/>

**Previous** **Versions:** [https://www.w3.org/TR/2023/WD-webgpu20230421/](https://www.w3.org/TR/2023/WD-webgpu-20230421/)

**History:** <https://www.w3.org/standards/history/webgpu>

**Feedback:** public-gpu@w3.org with subject line “[webgpu] *… message topic …*” ([archives](https://lists.w3.org/Archives/Public/public-gpu/))[GitHub](https://github.com/gpuweb/gpuweb/issues/)[Inline In Spec](https://www.w3.org/TR/webgpu/#issues-index)

**Editors:** [Kai Ninomiya](mailto:kainino@google.com) ([Google](https://www.google.com/))[Brandon Jones](mailto:bajones@google.com) ([Google](https://www.google.com/))[Myles C. Maxfield](mailto:mmaxfield@apple.com) ([Apple Inc.](https://www.apple.com/))

**Former Editors:** [Dzmitry Malyshau](mailto:dmalyshau@mozilla.com) ([Mozilla](https://www.mozilla.org/))[Justin Fan](mailto:justin_fan@apple.com) ([Apple Inc.](https://www.apple.com/))

**Participate:** [File an issue](https://github.com/gpuweb/gpuweb/issues/new) ([open issues](https://github.com/gpuweb/gpuweb/issues))

[Copyright](https://www.w3.org/Consortium/Legal/ipr-notice#Copyright) © 2023 [World Wide Web Consortium](https://www.w3.org/). W3C((®)) [liability](https://www.w3.org/Consortium/Legal/ipr-notice#Legal_Disclaimer), [trademark](https://www.w3.org/Consortium/Legal/ipr-notice#W3C_Trademarks) and [permissive document license](https://www.w3.org/Consortium/Legal/2015/copyright-software-and-document) rules apply.

* * *

[版权所有](https://www.w3.org/Consortium/Legal/ipr-notice#Copyright) ©2023[World Wide Web Consortium](https://www.w3.org/)。适用W3C®[责任](https://www.w3.org/Consortium/Legal/ipr-notice#Legal_Disclaimer)、[商标](https://www.w3.org/Consortium/Legal/ipr-notice#W3C_Trademarks) 和[许可文件许可](https://www.w3.org/Consortium/Legal/2015/copyright-software-and-document) 规则。

### 摘要

WebGPU公开了一个API，用于在图形处理单元上执行操作，例如渲染和计算。

#### 本文件的状态

本节描述了本文档在发布时的状态。当前W3C出版物列表和本技术报告的最新修订版可在[W3C技术报告索引](https://www.w3.org/TR/) 中找到<https://www.w3.org/TR/>。

欢迎对本规范提出反馈和意见。[问题·gpuweb/gpuweb](https://github.com/gpuweb/gpuweb/issues) 是讨论本规范的首选。或者，您可以将Web工作组邮件列表<public-gpu@w3.org> （[档案](https://lists.w3.org/Archives/Public/public-gpu/)）的评论发送到GPU。该草案突出了工作组仍有待讨论的一些悬而未决的问题。尚未就这些问题的结果做出决定，包括它们是否有效。

本文档由[Web工作组的GPU](https://www.w3.org/groups/wg/gpu) 使用[推荐轨道](https://www.w3.org/2021/Process-20211102/#recs-and-notes)作为工作草案发布。本文档旨在成为W3C推荐。作为工作草案发布并不意味着W3C及其成员的认可。

这是一份草稿文件，随时可能被其他文件更新、替换或废弃。不宜引用本文件为半成品以外的文件。本文件由一个根据[W3C专利政策](https://www.w3.org/Consortium/Patent-Policy/)运作的团体制作。W3C保留了与该团体交付成果相关的[任何专利披露的公开列表](https://www.w3.org/groups/wg/gpu/ipr) ；该页面还包括披露专利的说明。对个人认为包含[基本权利要求](https://www.w3.org/Consortium/Patent-Policy/#def-essential) 的专利有实际了解的个人必须根据[W3C专利政策的第6节](https://www.w3.org/Consortium/Patent-Policy/#sec-Disclosure)披露信息。

  


1.  ### 导言

> 本节是非规范性的。

[图形处理单元](https://en.wikipedia.org/wiki/Graphics_processing_unit)，或简称GPU，对于在个人计算中实现丰富的渲染和计算应用程序至关重要。WebGPU是一种API，它为Web公开了GPU硬件的功能。API从头开始设计，以有效地映射到（2014年后）原生GPU API。WebGPU与[WebGL](https://www.khronos.org/webgl/) 无关，也没有明确针对OpenGLES。

WebGPU将物理GPU硬件视为[GPUAdapter](https://www.w3.org/TR/webgpu/#gpuadapter)s。它通过[GPUDevice](https://www.w3.org/TR/webgpu/#gpudevice)和设备的[GPUQueue](https://www.w3.org/TR/webgpu/#gpuqueue)s提供与适配器的连接，

[GPUDevice](https://www.w3.org/TR/webgpu/#gpudevice) 可以高速访问处理单元。[GPUBuffer](https://www.w3.org/TR/webgpu/#gpubuffer) 和[GPUTexture](https://www.w3.org/TR/webgpu/#gputexture) 是由GPU内存支持的物理资源。[GPUCommand dBuffer](https://www.w3.org/TR/webgpu/#gpucommandbuffer) 和[GPURenderBundle](https://www.w3.org/TR/webgpu/#gpurenderbundle) 是用户录制命令的容器。[GPUShaderModule](https://www.w3.org/TR/webgpu/#gpushadermodule) 包含[着色器](https://www.w3.org/TR/webgpu/#shaders) 代码。其他资源，如[GPUS放大器](https://www.w3.org/TR/webgpu/#gpusampler) 或[GPUBindGroup](https://www.w3.org/TR/webgpu/#gpubindgroup)，配置GPU使用[物理资源](https://www.w3.org/TR/webgpu/#physical-resources) 的方式。

提供数据来执行[GPUCommand dBuffer](https://www.w3.org/TR/webgpu/#gpucommandbuffer)中编码的命令，是固定功能和可编程阶段的混合体。可编程阶段执行着色器，着色器是设计用于在GPU硬件上运行的特殊程序。[管线](https://www.w3.org/TR/webgpu/#pipeline)的大部分状态由[ GPURenderPipeline](https://www.w3.org/TR/webgpu/#gpurenderpipeline)或[ GPUComputePipeline](https://www.w3.org/TR/webgpu/#gpucomputepipeline)对象定义。这些[管线](https://www.w3.org/TR/webgpu/#pipeline)对象中不包含的状态是在编码过程中使用命令设置的，例如[ BeginRenderPass（）setBlendConstant（）](https://www.w3.org/TR/webgpu/#dom-gpucommandencoder-beginrenderpass)