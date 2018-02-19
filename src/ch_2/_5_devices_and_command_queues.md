# Devices and Command Queues
We mentioned earlier briefly about QueueFamilies, Queues and Commands. Ultimately, we'll need a hal::Graphics CommandQueue to send commands for drawing. We'll also need a logical device, 'Device' for Gfx-hal to work with. Similar to how we made a 'Surface' for Gfx-hal from a window, we'll make a 'Device' and 'QueueGroup' with the help of the Adapter and Surface we created earlier. We'll likely only need Queues of one type at a time, so there's a convenience function ```adapter.open_with``` which lets us say what type of queue we want and how many, and it will try to return them.

Before we use that function though, let's have a look at all the queues available through the QueueFamilies on our adapter.

```rust
    for queue_family    in &adapter.queue_families {
        println!("------------------------------------------------------");
        println!("Queue Family Id: {:?}", queue_family.id().0);
        println!("------------------------------------------------------");
        println!("Queue Type: {:?}", queue_family.queue_type());
        println!("Max Queues: {:?}", queue_family.max_queues());
        println!("Graphics Supported: {:?}", queue_family.supports_graphics());
        println!("Compute Supported: {:?}\n", queue_family.supports_compute());
    }
```
![img](https://i.imgur.com/CqwDuvg.png)

Ok, so two QueueFamiles, one General supporting a maximum 16 queues with graphics and compute support and, another Transfer supporting only 1 queue and no graphics/compute. The Transfer queue is used for memory transfer operations like uploading textures to the GPU.

Great! Let's move on and make that Device mentioned earlier and request a queue group that support graphics.



```
    let (device, mut queue_group) =
        adapter.open_with::<_, hal::Graphics>(1,|family| family.supports_graphics())
            .unwrap();

        println!("Queue's Family Id: {:#?}", &queue_group.family().0);
        println!("Number of queues: {:#?}", &queue_group.queues.len());

```

![img](https://i.imgur.com/gSdgMDn.png)

Excellent. Just as expected. We got back a QueueGroup which contains a Vec of one CommandQueue that is capable of supporting graphics, and the QueueFamily ID matches that of our General QueueFamily we saw earlier. We also got back a Device, a logical representation of our physical device which we'll use for setting up our rendering pipeline and general resource management.

But wait... That's not very safe. We know the QueueFamily supports graphics. But we don't know that our Surface supports that QueueFamily! Let's fix that quickly with the following:

```rust
    let (device, mut queue_group) =
        adapter.open_with::<_, hal::Graphics>(1, |family| {
            surface.supports_queue_family(family)
        }).unwrap();
```

#### Command Pools

So before we can send anything to our graphics-supporting CommandQueue, we need a CommandBuffer to store all our vertices/vertex attributes/graphics stuff. CommandBuffers are provided by CommandPools from our Device. So let's make one!

```rust
    let mut command_pool =
    device.create_command_pool_typed(&queue_group,
                                     pool::CommandPoolCreateFlags::empty(), 16);
```
We pass in our QueueGroup, an empty configuration flag, and the max number of buffers the pool will support.

The possible CommandPoolCreateFlags are:

| Flags | Description | Bits |
| ----- | ----------- | ---- |
|TRANSIENT|Indicates short-lived command buffers. Memory optimization hint for implementations.| 0x1|
|RESET_INDIVIDUAL|Allow command buffers to be reset individually.|0x2|
|empty| -| 0x0|



