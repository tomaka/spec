# WebAssembly runtime

## Runtime code

The **runtime code** is defined as either:

- The **runtime WebAssembly code** (defined below).
- The bytes `0x52BC537646DB8E05`, followed with the [zstd](https://datatracker.ietf.org/doc/html/rfc8878)-encoded **runtime WebAssembly code**.

The *runtime WebAssembly code* always starts with the bytes `0x0061736D`, making it possible to unambiguously differentiate the two situations.

## Runtime WebAssembly code

The **runtime WebAssembly code** is defined as a [WebAssembly](https://webassembly.github.io/spec/) program in binary format, with the following additional constraints:

- The WebAssembly binary must not use any of the WebAssembly extensions that was used after the launch of WebAssembly 1.0.
- It must not have any [*start function*, as defined by the WebAssembly specification](https://webassembly.github.io/spec/core/bikeshed/#start-function%E2%91%A0).
- It must either export exactly one memory named `memory` (with module name `env`), or import exactly one memory.
- It must export a global named `__heap_base` (with module name `env`) of type `i32`.
- TODO: runtime spec
