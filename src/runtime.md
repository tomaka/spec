# WebAssembly runtime

## Runtime code

The **runtime code** is defined as either:

- The **runtime WebAssembly** (defined below).
- The bytes `0x52BC537646DB8E05`, followed with the [zstd](https://datatracker.ietf.org/doc/html/rfc8878)-encoded **runtime WebAssembly**.

TODO: limit to the zstd size for zip bombs

The *runtime WebAssembly* always starts with the bytes `0x0061736D`, making it possible to unambiguously differentiate the two situations.

## Runtime WebAssembly

The **runtime WebAssembly** is defined as a [WebAssembly](https://webassembly.github.io/spec/) program in binary format, with the following additional constraints:

- The WebAssembly binary must not use any of the WebAssembly extensions that was used after the launch of WebAssembly 1.0.
- It must not have any [*start function*, as defined by the WebAssembly specification](https://webassembly.github.io/spec/core/bikeshed/#start-function%E2%91%A0).
- It must either export exactly one memory named `memory` (with module name `env`), or import exactly one memory.
- It must export a global named `__heap_base` (with module name `env`) of type `i32`.
- TODO: runtime spec

The **runtime WebAssembly** is allowed to import any function, whatever their signature. A host implementation must not consider a **runtime WebAssembly** as invalid because it imports an unknown function or a known function with a signature that doesn't match the expected one. When that is the case, a host implementation must raise an error only if that function is called during the execution of the WebAssembly.

## Entry point

A **runtime entry point** is a function exported by the **runtime WebAssembly**, as defined [in the WebAssembly specification](https://webassembly.github.io/spec/core/bikeshed/#exports%E2%91%A0), that has the following signature:

> (func $runtime_entry_point (param $data i32) (param $len i32) (result i64))

> **Note**: A **runtime WebAssembly** is allowed to export items that do not match the conditions here, as long as it doesn't conflict with some of the other requirements found in this specification.

TODO: do we really need to define entry point

## WebAssembly execution

Given a *runtime WebAssembly*, a *runtime entry point*, and some input data, the host can **execute** the WebAssembly.
