WebGPU
W3C工作草案，2023年4月24日
More details about this document
This version:https://www.w3.org/TR/2023/WD-webgpu-20230424/
Latest published version:https://www.w3.org/TR/webgpu/
Editor's Draft:https://gpuweb.github.io/gpuweb/
Previous Versions:https://www.w3.org/TR/2023/WD-webgpu20230421/
History:https://www.w3.org/standards/history/webgpu
Feedback:public-gpu@w3.org with subject line “[webgpu] … message topic …” (archives)GitHubInline In Spec
Editors:Kai Ninomiya (Google)Brandon Jones (Google)Myles C. Maxfield (Apple Inc.)
Former Editors:Dzmitry Malyshau (Mozilla)Justin Fan (Apple Inc.)
Participate:File an issue (open issues)
Copyright © 2023 World Wide Web Consortium. W3C((®)) liability, trademark and permissive document license rules apply.
---
版权所有 ©2023World Wide Web Consortium。适用W3C®责任、商标 和许可文件许可 规则。
摘要
WebGPU公开了一个API，用于在图形处理单元上执行操作，例如渲染和计算。
本文件的状态
本节描述了本文档在发布时的状态。当前W3C出版物列表和本技术报告的最新修订版可在W3C技术报告索引 中找到https://www.w3.org/TR/。
欢迎对本规范提出反馈和意见。问题·gpuweb/gpuweb 是讨论本规范的首选。或者，您可以将Web工作组邮件列表public-gpu@w3.org （档案）的评论发送到GPU。该草案突出了工作组仍有待讨论的一些悬而未决的问题。尚未就这些问题的结果做出决定，包括它们是否有效。
本文档由Web工作组的GPU 使用推荐轨道作为工作草案发布。本文档旨在成为W3C推荐。作为工作草案发布并不意味着W3C及其成员的认可。
这是一份草稿文件，随时可能被其他文件更新、替换或废弃。不宜引用本文件为半成品以外的文件。本文件由一个根据W3C专利政策运作的团体制作。W3C保留了与该团体交付成果相关的任何专利披露的公开列表 ；该页面还包括披露专利的说明。对个人认为包含基本权利要求 的专利有实际了解的个人必须根据W3C专利政策的第6节披露信息。

1. 导言
本节是非规范性的。
图形处理单元，或简称GPU，对于在个人计算中实现丰富的渲染和计算应用程序至关重要。WebGPU是一种API，它为Web公开了GPU硬件的功能。API从头开始设计，以有效地映射到（2014年后）原生GPU API。WebGPU与WebGL 无关，也没有明确针对OpenGLES。
WebGPU将物理GPU硬件视为GPUAdapters。它通过GPUDevice和设备的GPUQueues提供与适配器的连接，
GPUDevice 可以高速访问处理单元。GPUBuffer 和GPUTexture 是由GPU内存支持的物理资源。GPUCommand dBuffer 和GPURenderBundle 是用户录制命令的容器。GPUShaderModule 包含着色器 代码。其他资源，如GPUS放大器 或GPUBindGroup，配置GPU使用物理资源 的方式。
提供数据来执行GPUCommand dBuffer中编码的命令，是固定功能和可编程阶段的混合体。可编程阶段执行着色器，着色器是设计用于在GPU硬件上运行的特殊程序。管线的大部分状态由 GPURenderPipeline或 GPUComputePipeline对象定义。这些管线对象中不包含的状态是在编码过程中使用命令设置的，例如 BeginRenderPass（）setBlendConstant（）
