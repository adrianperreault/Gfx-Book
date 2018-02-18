# Graphics Backends Prep (WIP)

Gfx-rs supports multiple graphics backends:
* Vulkan,
* Direct3D 12,
* Metal,
* OpenGL 2.1+/ES2+

OpenGL (first released in 1992!) stands apart from the others by not natively support "close to the metal", low-level graphics control. As such, Gfx-hal must work around these limitations in order to "emulate" the behavior of the more modern graphics APIs. For best performance/ease of use, it is recommended to prefer Vulkan or Metal when possible.

 Throughout the tutorial series, there will be blocks of code unique to OpenGL that will be tagged with the "gl" feature flag and other blocks of code that will be tagged with the others ("vulkan", "dx12", "metal").
### Setting up the Backends

> Note: Since the previous section, we have refactored the window setup and loop
 execution into helper functions and moved them to the bottom of the file.
 We'll be doing this throughout the series (when possible) to keep the example files focused
 on the content being delivered. Click the "show hidden lines" icon ![icon](https://i.imgur.com/mjHEqXq.png)
 in the full listings at the bottom to see the changes made.

Let's begin by adding some feature flags and specifying dependencies for them:

##### Cargo.toml
```
[features]
default = [] ### <-- Add a preferred default if you'd like, eg. ["vulkan"]

vulkan = ["gfx-backend-vulkan"]
metal = ["gfx-backend-metal"]
dx12 = ["gfx-backend-dx12"]
gl = ["gfx-backend-gl"]
empty = ["&"]

[dependencies.gfx-backend-dx12]
git = "https://github.com/gfx-rs/gfx"
optional = true

[dependencies.gfx-backend-empty]
git = "https://github.com/gfx-rs/gfx"

[dependencies.gfx-backend-gl]
git = "https://github.com/gfx-rs/gfx"
optional = true

[dependencies.gfx-backend-metal]
git = "https://github.com/gfx-rs/gfx"
optional = true

[dependencies.gfx-backend-vulkan]
git = "https://github.com/gfx-rs/gfx"
optional = true

[dependencies.gfx-hal]
git = "https://github.com/gfx-rs/gfx"

```

With the feature flags in place, we can now do [conditional compilation](https://doc.rust-lang.org/1.14.0/book/conditional-compilation.html) based on what graphics backend we choose.

For example, ``` cargo run --features vulkan ``` to build/run a project's main with the Vulkan graphics backend selected. If a default is set in Cargo.toml,  ``` --features...```  may be left off.

Now, add the following to the top of main.rs:
##### main.rs
```rust
extern crate gfx_hal as hal;
#[cfg(feature = "dx12")]
extern crate gfx_backend_dx12 as back;
#[cfg(feature = "vulkan")]
extern crate gfx_backend_vulkan as back;
#[cfg(feature = "metal")]
extern crate gfx_backend_metal as back;
#[cfg(feature = "gl")]
extern crate gfx_backend_gl as back;

#[cfg(any(feature = "vulkan", feature = "dx12", feature = "metal", feature = "gl"))]
fn main() {... *Snip*}
```

What did we get done? We declared a conditional identifier 'back' to point to a specific backend
based on a feature flag. We also made our main function dependent on a feature flag being set.

Great job!
Go have a jam sandwich then check out the next section on Instances, Adapters and Surfaces.


##### Cargo.toml full listing
```rust

#
########### Cargo.toml begins here. Discard above ---^
#
# [package]
# authors = ["Adrian Perreault"]
# name = "vulkan_tutorial_in_gfx"
# version = "0.1.0"
# [dependencies]
# env_logger = "*"
# log = "*"
# winit = "*"
#
# [features]
# default = ["vulkan"]
# dx12 = ["gfx-backend-dx12"]
# gl = ["gfx-backend-gl"]
# metal = ["gfx-backend-metal"]
# vulkan = ["gfx-backend-vulkan"]
#
# [dependencies.gfx-backend-dx12]
# git = "https://github.com/gfx-rs/gfx"
# optional = true
#
# [dependencies.gfx-backend-empty]
# git = "https://github.com/gfx-rs/gfx"
#
# [dependencies.gfx-backend-gl]
# git = "https://github.com/gfx-rs/gfx"
# optional = true
#
# [dependencies.gfx-backend-metal]
# git = "https://github.com/gfx-rs/gfx"
# optional = true
#
# [dependencies.gfx-backend-vulkan]
# git = "https://github.com/gfx-rs/gfx"
# optional = true
#
# [dependencies.gfx-hal]
# git = "https://github.com/gfx-rs/gfx"
#
# [[bin]]
# name = "ch2_2"
# path = "src/ch2/_2_making_a_window.rs"
#
# [[bin]]
# name = "ch2_3"
# path = "src/ch2/_2_backends.rs"
#
########### Cargo.toml ends here. Discard below ---v
```


##### main.rs full listing
```rust
# extern crate gfx_hal as hal;
# #[cfg(feature = "dx12")]
# extern crate gfx_backend_dx12;
# #[cfg(feature = "vulkan")]
# extern crate gfx_backend_vulkan as back;
# #[cfg(feature = "metal")]
# extern crate gfx_backend_metal as back;
# #[cfg(feature = "gl")]
# extern crate gfx_backend_gl as back;
#
# #[cfg(any(feature = "vulkan", feature = "dx12", feature = "metal", feature = "gl"))]
# fn main() {
#     let (events_loop, window) = setup_window();
#     run_main_events_loop(events_loop);
# }
#
# //// # Previous Tutorial Content ///////////////////////////////////////////////////
# extern crate winit;
#
# use winit::{Event, WindowEvent, EventsLoop};
# use winit::Window;
# use std::cell::Cell;
#
# // ## Window Setup #################################################################
# fn setup_window() -> ( Cell<EventsLoop>, Window) {
#
#     // Create a main execution event loop for listening for input/window events.
#     let events_loop=  winit::EventsLoop::new();
#
#     // Specify basic window properties and EventsLoop with WindowBuilder
#     // and build/display the window.
#     let window = winit::WindowBuilder::new()
#         .with_title("Gfx-hal Quickstart Tutorial")
#         .with_dimensions(800, 600)
#         .build(&events_loop)
#         .unwrap();
#
#     (Cell::new(events_loop), window)
#
# }
# //##################################################################################
#
#
# //## Main Loop Execution ###########################################################
# fn run_main_events_loop(events_loop: Cell<EventsLoop>) {
#
#     let mut running = true;
#     // Taking the EventsLoop object out of the Cell<EventLoop> so we can use it normally.
#     let mut events_loop = events_loop.into_inner();
#     // Run our main loop indefinitely, checking for events until we explicitly stop it.
#     while running {
#         events_loop.poll_events(|event| {
#
#             // If we get a WindowEvent...
#             if let winit::Event::WindowEvent { event, .. } = event {
#                 match event {
#
#                     // And if that WindowEvent is a KeyBoardInput event...
#                     winit::WindowEvent::KeyboardInput {
#                         input: winit::KeyboardInput {
#
#                             // If the KeyboardInput was the Escape key...
#                             virtual_keycode: Some(winit::VirtualKeyCode::Escape),
#                             ..
#                         },
#                         ..
#                     }
#                     // Or, if the event is a window closed event,
#                     // set running to false to end the main loop.
#                     | winit::WindowEvent::Closed => running = false,
#                     _ => (),
#                 }
#             }
#         });
#     }
# }
# //##################################################################################
#
# ////////////////////////////////////////////////////////////////////////////////////

```

