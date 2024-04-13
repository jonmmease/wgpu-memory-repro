## Overview
This is a repro for a memory/resource leak using the metal backend of wgpu 0.19. This was not present in version 0.18.

## System specs
These are the specs of the machine that demonstrates this repro:

Chip: Apple M1 Pro
macOS: Sonoma 14.4.1
Memory 32 GB

## Instructions
Run the repro with:
```
RUST_BACKTRACE=1 cargo run
``` 

## Result
```
0
10
20
30
40
50
60
70
80
90
Context leak detected, msgtracer returned -1
100
thread 'main' panicked at /Users/jonmmease/.cargo/registry/src/index.crates.io-6f17d22bba15001f/foreign-types-shared-0.3.1/src/lib.rs:72:9:
assertion failed: !ptr.is_null()
stack backtrace:
   0: rust_begin_unwind
             at /rustc/07dca489ac2d933c78d3c5158e3f43beefeb02ce/library/std/src/panicking.rs:645:5
   1: core::panicking::panic_fmt
             at /rustc/07dca489ac2d933c78d3c5158e3f43beefeb02ce/library/core/src/panicking.rs:72:14
   2: core::panicking::panic
             at /rustc/07dca489ac2d933c78d3c5158e3f43beefeb02ce/library/core/src/panicking.rs:144:5
   3: foreign_types_shared::ForeignTypeRef::from_ptr
             at /Users/jonmmease/.cargo/registry/src/index.crates.io-6f17d22bba15001f/foreign-types-shared-0.3.1/src/lib.rs:72:9
   4: <metal::commandqueue::CommandQueue as core::ops::deref::Deref>::deref
             at /Users/jonmmease/.cargo/registry/src/index.crates.io-6f17d22bba15001f/metal-0.27.0/src/commandqueue.rs:13:1
   5: wgpu_hal::metal::command::<impl wgpu_hal::CommandEncoder<wgpu_hal::metal::Api> for wgpu_hal::metal::CommandEncoder>::begin_encoding::{{closure}}
             at /Users/jonmmease/.cargo/registry/src/index.crates.io-6f17d22bba15001f/wgpu-hal-0.19.3/src/metal/command.rs:179:17
   6: objc::rc::autorelease::autoreleasepool
             at /Users/jonmmease/.cargo/registry/src/index.crates.io-6f17d22bba15001f/objc-0.2.7/src/rc/autorelease.rs:29:5
   7: wgpu_hal::metal::command::<impl wgpu_hal::CommandEncoder<wgpu_hal::metal::Api> for wgpu_hal::metal::CommandEncoder>::begin_encoding
             at /Users/jonmmease/.cargo/registry/src/index.crates.io-6f17d22bba15001f/wgpu-hal-0.19.3/src/metal/command.rs:175:19
   8: wgpu_core::device::queue::PendingWrites<A>::activate
             at /Users/jonmmease/.cargo/registry/src/index.crates.io-6f17d22bba15001f/wgpu-core-0.19.3/src/device/queue.rs:265:17
   9: wgpu_core::device::resource::Device<A>::new
             at /Users/jonmmease/.cargo/registry/src/index.crates.io-6f17d22bba15001f/wgpu-core-0.19.3/src/device/resource.rs:230:9
  10: wgpu_core::instance::Adapter<A>::create_device_and_queue_from_hal
             at /Users/jonmmease/.cargo/registry/src/index.crates.io-6f17d22bba15001f/wgpu-core-0.19.3/src/instance.rs:306:29
  11: wgpu_core::instance::Adapter<A>::create_device_and_queue
             at /Users/jonmmease/.cargo/registry/src/index.crates.io-6f17d22bba15001f/wgpu-core-0.19.3/src/instance.rs:381:9
  12: wgpu_core::instance::<impl wgpu_core::global::Global<G>>::adapter_request_device
             at /Users/jonmmease/.cargo/registry/src/index.crates.io-6f17d22bba15001f/wgpu-core-0.19.3/src/instance.rs:1084:23
  13: <wgpu::backend::wgpu_core::ContextWgpuCore as wgpu::context::Context>::adapter_request_device
             at /Users/jonmmease/.cargo/registry/src/index.crates.io-6f17d22bba15001f/wgpu-0.19.3/src/backend/wgpu_core.rs:587:44
  14: <T as wgpu::context::DynContext>::adapter_request_device
             at /Users/jonmmease/.cargo/registry/src/index.crates.io-6f17d22bba15001f/wgpu-0.19.3/src/context.rs:2019:22
  15: wgpu::Adapter::request_device
             at /Users/jonmmease/.cargo/registry/src/index.crates.io-6f17d22bba15001f/wgpu-0.19.3/src/lib.rs:2119:22
  16: wgpu_memory_repro::State::new::{{closure}}
             at ./src/main.rs:103:31
  17: wgpu_memory_repro::run::{{closure}}
             at ./src/main.rs:300:34
  18: pollster::block_on
             at /Users/jonmmease/.cargo/registry/src/index.crates.io-6f17d22bba15001f/pollster-0.3.0/src/lib.rs:128:15
  19: wgpu_memory_repro::main
             at ./src/main.rs:309:9
  20: core::ops::function::FnOnce::call_once
             at /rustc/07dca489ac2d933c78d3c5158e3f43beefeb02ce/library/core/src/ops/function.rs:250:5
note: Some details are omitted, run with `RUST_BACKTRACE=full` for a verbose backtrace.
```

## Run with 0.18
To run with 0.18, swap comments in `Cargo.toml` from

```
#wgpu = {version = "=0.18.0", default-features = false, features = ["wgsl"]}
wgpu = {version = "=0.19.3", default-features = false, features = ["wgsl", "metal"]}
```

to

```
wgpu = {version = "=0.18.0", default-features = false, features = ["wgsl"]}
#wgpu = {version = "=0.19.3", default-features = false, features = ["wgsl", "metal"]}
```

Then in `main.rs`, swap comments from 

```rust
        // // wgpu 0.18
        // let device_descriptor = wgpu::DeviceDescriptor {
        //     label: None,
        //     features: wgpu::Features::empty(),
        //     limits: wgpu::Limits::default(),
        // };

        // wgpu 0.19.3
        let device_descriptor = wgpu::DeviceDescriptor {
            label: None,
            required_features: wgpu::Features::empty(),
            required_limits: wgpu::Limits::default(),
        };
```

to

```rust
        // wgpu 0.18
        let device_descriptor = wgpu::DeviceDescriptor {
            label: None,
            features: wgpu::Features::empty(),
            limits: wgpu::Limits::default(),
        };

        // // wgpu 0.19.3
        // let device_descriptor = wgpu::DeviceDescriptor {
        //     label: None,
        //     required_features: wgpu::Features::empty(),
        //     required_limits: wgpu::Limits::default(),
        // };
```

Then run as above. In this case, the program completes successfully.
