## Making a Window with Winit (WIP)

When working with Gfx-hal, we need a window to display content and interact with input. Typically this is done
with [Winit](https://github.com/tomaka/winit), a cross-platform window creation and management library written in Rust.
When targeting OpenGL with Gfx-hal, there are context requirements that necessitate using [Glutin](https://github.com/tomaka/glutin) (OpenGL, UTilities and INput) instead of Winit. The two APIs are similar. In fact, Winit was created by cutting out the OpenGL related stuff from Glutin.

### Window Setup

Let's start off with a basic 800 x 600 window with a title:

```rust
extern crate winit;
use winit::{Event, WindowEvent};

fn main() {

    // Create a main execution event loop for listening for input/window events.
    let mut events_loop = winit::EventsLoop::new();

    // Specify basic window properties and EventsLoop with WindowBuilder
    // and build/display the window.
    let window = winit::WindowBuilder::new()
        .with_title("Gfx-hal Quickstart Tutorial")
        .with_dimensions(800, 600)
        .build(&events_loop)
        .unwrap();

```

### Window Events

We'll use window events in the main events loop to handle behaviors like closing a window on keypress
and later on in the series, window resized and  mouse input events.
.

WindowEvents in Winit include:

| Window               | Mouse/Cursor  | Other Input / Misc. |
| -------------------  |---------------| ------------------- |
| Resized              | MouseInput    |  KeyboardInput      |
| Moved                | MouseWheel    | TouchpadPressure    |
| Closed               | CursorLeft    |  AxisMotion         |
| DroppedFile          | CursorEntered |  Refresh            |
| HoveredFile          | CursorMoved   |  Touch              |
| HoveredFileCancelled |               |  HiDPIFactorChanged |
| ReceivedCharacter    |               |                     |
| Focused              |               |                     |




### Main Loop

Let's kick off the main events loop and make sure we can close the window.

```rust
  let mut running = true;

    // Keep our main loop running until an event to explicitly stop it.
    while running {

        events_loop.poll_events(|event| {

    // If we get a WindowEvent...
            if let winit::Event::WindowEvent { event, .. } = event {
                match event {

    // And if that WindowEvent is a KeyBoardInput event...
                    winit::WindowEvent::KeyboardInput {
                        input: winit::KeyboardInput {

    // If the KeyboardInput was the Escape key...
                            virtual_keycode: Some(winit::VirtualKeyCode::Escape),
                            ..
                        },
                        ..
                    }
    // Or, if the event is a window closed event,
    // set running to false to end the main loop.
                    | winit::WindowEvent::Closed => running = false,
                    _ => (),
                }
            }
        });
    }

```



With any luck, running this code should produce a basic window. (How exciting!)
Let's carry on and start setting up Gfx-hal backends in the next section.

##### main.rs full listing   - click  ![img](https://i.imgur.com/mjHEqXq.png)
```rust

# extern crate winit;
#
# use winit::WindowEvent;
#
# fn main() {
# // ## Window Setup ##################################################################
#
#     // Create a main execution event loop for listening for input/window events.
#     let mut events_loop = winit::EventsLoop::new();
#
#     // Specify basic window properties and EventsLoop with WindowBuilder
#     // and build/display the window.
#     let window = winit::WindowBuilder::new()
#         .with_title("Gfx-hal Quickstart Tutorial")
#         .with_dimensions(800, 600)
#         .build(&events_loop)
#         .unwrap();
#
# //###################################################################################
#
#
# //## Main Loop Execution #############################################################
#     let mut running = true;
#
#     // Run our main loop indefinitely, checking for events until we explicitly stop it.
#     while running {
#
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
# //##################################################################################
# }
```