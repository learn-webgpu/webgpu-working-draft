## 1\. Introduction[](#intro)

_This section is non-normative._

[Graphics Processing Units](https://en.wikipedia.org/wiki/Graphics_processing_unit), or GPUs for short, have been essential in enabling rich rendering and computational applications in personal computing. WebGPU is an API that exposes the capabilities of GPU hardware for the Web. The API is designed from the ground up to efficiently map to (post-2014) native GPU APIs. WebGPU is not related to [WebGL](https://www.khronos.org/webgl/) and does not explicitly target OpenGL ES.

WebGPU sees physical GPU hardware as `[GPUAdapter](#gpuadapter)`s. It provides a connection to an adapter via `[GPUDevice](#gpudevice)`, which manages resources, and the device’s `[GPUQueue](#gpuqueue)`s, which execute commands. `[GPUDevice](#gpudevice)` may have its own memory with high-speed access to the processing units. `[GPUBuffer](#gpubuffer)` and `[GPUTexture](#gputexture)` are the physical resources

Info about the 'physical resources' reference.[#physical-resources](#physical-resources)**Referenced in:**

- [1\. Introduction](#ref-for-physical-resources)
- [2.1.4. Out-of-bounds access in shaders](#ref-for-physical-resources①) [(2)](#ref-for-physical-resources②) [(3)](#ref-for-physical-resources③)
- [3.4.2. Memory Model](#ref-for-physical-resources④)
- [3.4.3. Resource Usages](#ref-for-physical-resources⑤)
- [3.4.4. Synchronization](#ref-for-physical-resources⑥)

backed by GPU memory. `[GPUCommandBuffer](#gpucommandbuffer)` and `[GPURenderBundle](#gpurenderbundle)` are containers for user-recorded commands. `[GPUShaderModule](#gpushadermodule)` contains [shader](#shaders) code. The other resources, such as `[GPUSampler](#gpusampler)` or `[GPUBindGroup](#gpubindgroup)`, configure the way [physical resources](#physical-resources) are used by the GPU.

GPUs execute commands encoded in `[GPUCommandBuffer](#gpucommandbuffer)`s by feeding data through a [pipeline](#pipeline), which is a mix of fixed-function and programmable stages. Programmable stages execute shaders

Info about the 'shaders' reference.[#shaders](#shaders)**Referenced in:**

- [1\. Introduction](#ref-for-shaders)
- [2.1.2. GPU-based undefined behavior](#ref-for-shaders①)
- [2.1.4. Out-of-bounds access in shaders](#ref-for-shaders②)

, which are special programs designed to run on GPU hardware. Most of the state of a [pipeline](#pipeline) is defined by a `[GPURenderPipeline](#gpurenderpipeline)` or a `[GPUComputePipeline](#gpucomputepipeline)` object. The state not included in these [pipeline](#pipeline) objects is set during encoding with commands, such as `[beginRenderPass()](#dom-gpucommandencoder-beginrenderpass)` or `[setBlendConstant()](#dom-gpurenderpassencoder-setblendconstant)`.
