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

Ok... so let's take a closer look at some of this stuff.

#### Instance Extensions (Vulkan)
Extensions are a way to 'extend' the Vulkan API with new commands, structures and enumerants.

For example:
* VK_KHR_surface      - Allows for Surface objects to be created
* VK_EXT_debug_report - Allows for debug information to be outputted.
* VK_KHR_xcb_surface  - Provides a way to make Surfaces that refer to X11 windows (Linux)






