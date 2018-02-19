# Instances, Adapters and Surfaces (WIP)

### Instances
An instance is the main connection between your application and the underlying backends and hardware.
It provides access to surfaces and adapters.

### Surfaces
Surfaces are a cross-platform abstraction over windows.
> *"Hey, isn't that basically what Winit/Glutin is?"*

Close! A surface is a general abstraction over windows (which may be created via Winit, Glutin, the Microsoft Windows Api etc.) for Gfx-hal to work with. A surface is created by passing in an existing window or window handle (hwnd in Windows) to the ```create_surface``` or ```create_surface_from_hwnd``` functions on an instance.

### Adapters
Adapters are an abstraction consisting of a handle to a physical (or virtual) devices (ie. your graphics card), info such as a device's name/vendor Id /device Id and, a vector of supported Queue Families.
### Queue Families
In Gfx-hal, when one wishes to interact with a graphics card to perform some task, a command must be added to some queue. Queues are categorized into separate families based on their utility.
Currently, the Queue types include:
* General
* Graphics
* Compute
* Transfer

### Let's dive in!

#### In main.rs:

Add this use item at the top:

```rust
use hal::{Instance, PhysicalDevice};
```

Add the following after a Window and EventsLoop is created, but before the main execution of the EventsLoop:

```rust
  #[cfg(any(feature = "vulkan", feature = "dx12", feature = "metal"))]
    let (_instance, adapters, surface) = {
        let instance = back::Instance::create("Our Awesome Instance Instance", 1);
        let surface = instance.create_surface(&window);
        let adapters = instance.enumerate_adapters();
        (instance, adapters, surface)
    };

```

* We exclude OpenGL via a cfg attribute for the time being! (Sorry old guy... We'll Come back to you later.)
* We create an Instance, collection of adapters and a surface


Let's use our new found friends, flex our inner OCD and print out some sweet data!
```rust
 #[cfg(any(feature = "vulkan"))] {
    println!("------------------------------------------------------");
    println!("Instance Extensions");
    println!("------------------------------------------------------");
    println!("{:#?}\n", _instance.extensions);
    println!("------------------------------------------------------");

for adapter in &adapters {
    println!("------------------------------------------------------");
    println!("Adapter Info");
    println!("------------------------------------------------------");
    println!("{:#?}", adapter.info);
    println!("------------------------------------------------------");
    println!("Queue Families");
    println!("------------------------------------------------------");
    println!("{:#?}", adapter.queue_families);
    println!("------------------------------------------------------");
    println!("Physical Device Features");
    println!("------------------------------------------------------");
    println!("{:#?}", adapter.physical_device.get_features());
    println!("------------------------------------------------------");
    println!("Physical Device Memory Properties");
    println!("------------------------------------------------------");
    println!("{:#?}", adapter.physical_device.memory_properties());
    println!("------------------------------------------------------");
    println!("Physical Device Limits");
    println!("------------------------------------------------------");
    println!("{:#?}", adapter.physical_device.get_limits());
    println!("------------------------------------------------------");

    println!("------------------------------------------------------");
    println!("Surface");
    println!("------------------------------------------------------");
    println!("{:#?}\n", surface.capabilities_and_formats(&adapters[0].physical_device));
    println!("------------------------------------------------------");
    }
```

With any luck, if you're running with Vulkan, you should see something like the following:

![img](https://i.imgur.com/veVedZq.png)

All that sweet data! Just imagine the possibilities!

Before we get in to the nitty gritty of what some of that data means, let's first pick an adapter to work with. You'll see two or more adapters if you have for example: a CPU integrated adapter and a dedicated onboard adapter in a laptop, or 2+ graphics cards in a desktop. For simplicity's sake, we'll just go with the first one listed. Feel free to pick whichever works best for you.
```rust
let adapter = adapters.remove(0);
```

Ok... so let's take a closer look at some of this stuff.

#### Instance Extensions (Vulkan)
![img](https://i.imgur.com/zHeKNoK.png)

Extensions are a way to 'extend' the Vulkan API with new commands, structures and enumerants.

For example:
* VK_KHR_surface      - Allows for Surface objects to be created
* VK_EXT_debug_report - Allows for debug information to be outputted.
* VK_KHR_xcb_surface  - Provides a way to make Surfaces that refer to X11 windows (Linux)

#### Adapter Info
![img](https://i.imgur.com/ymiNkOz.png)

Basic information about the device like name and, PCI vendor and device IDs. It also lists whether the device is currently doing software rendering (as opposed to hardware) which should be false in most cases.

#### Queue Families
![img](https://i.imgur.com/KVJnw3q.png)

#### Device Features (Vulkan)
![img](https://i.imgur.com/dYVsiLJ.png)


Device features indicate what a device is capable of performing. This information is useful for knowing whether or not you can turn on a given graphics feature for a user or if the application will even run to begin with.

The following is a table summarizing some of the features from the authors device. Feel free to scroll right past. Or, for more details, [check out the official Khronos spec](https://www.khronos.org/registry/vulkan/specs/1.0/man/html/VkPhysicalDeviceFeatures.html)

| Feature |Description |
| ------- |----------- |
|ALPHA_TO_ONE| indicates whether the implementation is able to replace the alpha value of the color fragment output from the fragment shader with the maximum representable alpha value for fixed-point colors or 1.0 for floating-point colors. If this feature is not enabled, then the alphaToOneEnable member of the VkPipelineMultisampleStateCreateInfo structure must be set to VK_FALSE. Otherwise setting alphaToOneEnable to VK_TRUE will enable alpha-to-one behavior.|
|DEPTH_BIAS_CLAMP| indicates whether depth bias clamping is supported. If this feature is not enabled, the depthBiasClamp member of the VkPipelineRasterizationStateCreateInfo structure must be set to 0.0 unless the VK_DYNAMIC_STATE_DEPTH_BIAS dynamic state is enabled, and the depthBiasClamp parameter to vkCmdSetDepthBias must be set to 0.0.|
|DEPTH_BOUNDS| indicates whether depth bounds tests are supported. If this feature is not enabled, the depthBoundsTestEnable member of the VkPipelineDepthStencilStateCreateInfo structure must be set to VK_FALSE. When depthBoundsTestEnable is set to VK_FALSE, the minDepthBounds and maxDepthBounds members of the VkPipelineDepthStencilStateCreateInfo structure are ignored.|
|DEPTH_CLAMP| indicates whether depth clamping is supported. If this feature is not enabled, the depthClampEnable member of the VkPipelineRasterizationStateCreateInfo structure must be set to VK_FALSE. Otherwise, setting depthClampEnable to VK_TRUE will enable depth clamping.|
|DRAW_INDIRECT_FIRST_INSTANCE| indicates whether indirect draw calls support the firstInstance parameter. If this feature is not enabled, the firstInstance member of all VkDrawIndirectCommand and VkDrawIndexedIndirectCommand structures that are provided to the vkCmdDrawIndirect and vkCmdDrawIndexedIndirect commands must be 0.|
|DUAL_SRC_BLENDING| indicates whether blend operations which take two sources are supported. If this feature is not enabled, the VK_BLEND_FACTOR_SRC1_COLOR, VK_BLEND_FACTOR_ONE_MINUS_SRC1_COLOR, VK_BLEND_FACTOR_SRC1_ALPHA, and VK_BLEND_FACTOR_ONE_MINUS_SRC1_ALPHA enum values must not be used as source or destination blending factors.|
|FORMAT_BC|indicates whether more than one viewport is supported. If this feature is not enabled, the viewportCount and scissorCount members of the VkPipelineViewportStateCreateInfo structure must be set to 1. Similarly, the viewportCount parameter to the vkCmdSetViewport command and the scissorCount parameter to the vkCmdSetScissor command must be 1, and the firstViewport parameter to the vkCmdSetViewport command and the firstScissor parameter to the vkCmdSetScissor command must be 0.|
|FRAGMENT_STORES_AND_ATOMICS|indicates whether storage buffers and images support stores and atomic operations in the fragment shader stage. If this feature is not enabled, all storage image, storage texel buffers, and storage buffer variables used by the fragment stage in shader modules must be decorated with the NonWriteable decoration (or the readonly memory qualifier in GLSL).|
|FULL_DRAW_INDEX_U32|indicates the full 32-bit range of indices is supported for indexed draw calls when using a VkIndexType of VK_INDEX_TYPE_UINT32|
|GEOMETRY_SHADER| indicates whether geometry shaders are supported. If this feature is not enabled, the VK_SHADER_STAGE_GEOMETRY_BIT and VK_PIPELINE_STAGE_GEOMETRY_SHADER_BIT enum values must not be used. This also indicates whether shader modules can declare the Geometry capability.|
|IMAGE_CUBE_ARRAY| indicates whether image views with a VkImageViewType of VK_IMAGE_VIEW_TYPE_CUBE_ARRAY can be created, and that the corresponding SampledCubeArray and ImageCubeArray SPIR-V capabilities can be used in shader code.|
|INDEPENDENT_BLENDING| indicates whether the VkPipelineColorBlendAttachmentState settings are controlled independently per-attachment. If this feature is not enabled, the VkPipelineColorBlendAttachmentState settings for all color attachments must be identical. Otherwise, a different VkPipelineColorBlendAttachmentState can be provided for each bound color attachment.|
|LOGIC_OP|indicates whether logic operations are supported. If this feature is not enabled, the logicOpEnable member of the VkPipelineColorBlendStateCreateInfo structure must be set to VK_FALSE, and the logicOp member is ignored.|
|MULTI_DRAW_INDIRECT| indicates whether multiple draw indirect is supported. If this feature is not enabled, the drawCount parameter to the vkCmdDrawIndirect and vkCmdDrawIndexedIndirect commands must be 0 or 1. The maxDrawIndirectCount member of the VkPhysicalDeviceLimits structure must also be 1 if this feature is not supported.|
|MULTI_VIEWPORTS|indicates whether more than one viewport is supported. If this feature is not enabled, the viewportCount and scissorCount members of the VkPipelineViewportStateCreateInfo structure must be set to 1. Similarly, the viewportCount parameter to the vkCmdSetViewport command and the scissorCount parameter to the vkCmdSetScissor command must be 1, and the firstViewport parameter to the vkCmdSetViewport command and the firstScissor parameter to the vkCmdSetScissor command must be 0.|
|PIPELINE_STATISTICS_QUERY|indicates whether the pipeline statistics queries are supported. If this feature is not enabled, queries of type VK_QUERY_TYPE_PIPELINE_STATISTICS cannot be created, and none of the VkQueryPipelineStatisticFlagBits bits can be set in the pipelineStatistics member of the VkQueryPoolCreateInfo structure.|
|PRECISE_OCCLUSION_QUERY|indicates whether occlusion queries returning actual sample counts are supported. Occlusion queries are created in a VkQueryPool by specifying the queryType of VK_QUERY_TYPE_OCCLUSION in the VkQueryPoolCreateInfo structure which is passed to vkCreateQueryPool. If this feature is enabled, queries of this type can enable VK_QUERY_CONTROL_PRECISE_BIT in the flags parameter to vkCmdBeginQuery. If this feature is not supported, the implementation supports only boolean occlusion queries. When any samples are passed, boolean queries will return a non-zero result value, otherwise a result value of zero is returned. When this feature is enabled and VK_QUERY_CONTROL_PRECISE_BIT is set, occlusion queries will report the actual number of samples passed.|
|ROBUST_BUFFER_ACCESS| Indicates that accesses to buffers are bounds-checked against the range of the buffer descriptor |
|SAMPLE_RATE_SHADING| indicates whether per-sample shading and multisample interpolation are supported. If this feature is not enabled, the sampleShadingEnable member of the VkPipelineMultisampleStateCreateInfo structure must be set to VK_FALSE and the minSampleShading member is ignored. This also indicates whether shader modules can declare the SampleRateShading capability.|
|SAMPLER_ANISOTROPY| indicates whether anisotropic filtering is supported. If this feature is not enabled, the anisotropyEnable member of the VkSamplerCreateInfo structure must be VK_FALSE.|
|TESSELLATION_SHADER|indicates whether tessellation control and evaluation shaders are supported. If this feature is not enabled, the VK_SHADER_STAGE_TESSELLATION_CONTROL_BIT, VK_SHADER_STAGE_TESSELLATION_EVALUATION_BIT, VK_PIPELINE_STAGE_TESSELLATION_CONTROL_SHADER_BIT, VK_PIPELINE_STAGE_TESSELLATION_EVALUATION_SHADER_BIT, and VK_STRUCTURE_TYPE_PIPELINE_TESSELLATION_STATE_CREATE_INFO enum values must not be used. This also indicates whether shader modules can declare the Tessellation capability.|
|VERTEX_STORES_AND_ATOMICS| indicates whether storage buffers and images support stores and atomic operations in the vertex, tessellation, and geometry shader stages. If this feature is not enabled, all storage image, storage texel buffers, and storage buffer variables used by these stages in shader modules must be decorated with the NonWriteable decoration (or the readonly memory qualifier in GLSL).|





