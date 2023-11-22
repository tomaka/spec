# WebAssembly runtime

## Runtime code

The **runtime code** is defined as either:

- The **runtime WebAssembly** (defined below).
- The bytes `0x52BC537646DB8E05`, followed with the [zstd](https://datatracker.ietf.org/doc/html/rfc8878)-encoded **runtime WebAssembly**. The decoded **runtime WebAssembly** must not be larger than 52428800 bytes (i.e. 50 MiB).

The *runtime WebAssembly* always starts with the bytes `0x0061736D`, making it possible to unambiguously differentiate the two situations.

> **Note**: Implementers should be aware of the danger of so-called "zip bombs" where a zstd-encoded payload decodes to a huge output. Implementers should make sure to cap the size of the output while decoding is in progress.

## Runtime WebAssembly

The **runtime WebAssembly** is defined as a [WebAssembly](https://webassembly.github.io/spec/) program in binary format, with the following additional constraints:

- The WebAssembly binary must not use any of the WebAssembly extensions that was used after the launch of WebAssembly 1.0.
- It must not have any [*start function*, as defined by the WebAssembly specification](https://webassembly.github.io/spec/core/bikeshed/#start-function%E2%91%A0).
- It must either export exactly one memory named `memory` (with module name `env`), or import exactly one memory.
- It must export a global named `__heap_base` (with module name `env`) of type `i32`.
- TODO: runtime spec

The **runtime WebAssembly** is allowed to import any function, whatever their signature. A host implementation must not consider a **runtime WebAssembly** as invalid only because it imports an unknown function or a known function with a signature that doesn't match the expected one. A host implementation must raise an error only if such a function is called during the execution of the WebAssembly.

## Entry point

A **runtime entry point** is a function exported by the **runtime WebAssembly**, as defined [in the WebAssembly specification](https://webassembly.github.io/spec/core/bikeshed/#exports%E2%91%A0), that has the following signature:

> (func $runtime_entry_point (param $data i32) (param $len i32) (result i64))

> **Note**: A **runtime WebAssembly** is allowed to export items that do not match the conditions here, as long as it doesn't conflict with some of the other requirements found in this specification.

TODO: do we really need to define entry point

## WebAssembly execution

Given a *runtime WebAssembly*, a *state*, a *runtime entry point*, and some input data, the host can **execute** the WebAssembly.

While execution is in progress, an implementation must maintain an *overlay*, defined as the list of modifications performed to the state since the beginning of the execution.

The following functions can be imported by the **runtime WebAssembly**:

### ext_allocator_malloc_version_1

```
(func $ext_allocator_malloc_version_1
    (param $size i32) (result i32))
```

### ext_allocator_free_version_1

```
(func $ext_allocator_free_version_1 (param $ptr i32))
```

### ext_storage_set_version_1

```
(func $ext_storage_set_version_1
    (param $key i64) (param $value i64))
```

With:

- `key`: A pointer-size to the memory location containing the key.
- `value`: A pointer-size to the memory location containing the value.

The implementation must set the given *key* to the given *value* in the *overlay*.

### ext_storage_get_version_1

```
(func $ext_storage_get_version_1
    (param $key i64) (result i64))
```

With:

- `key`: A pointer-size to the memory location containing the key.
- The result: a pointer-size.

If the given *key* is present in the overlay, TODO

### ext_storage_clear_version_1

```
(func $ext_storage_clear_version_1
    (param $key i64))
```

With:

- `key`: A pointer-size to the memory location containing the key.

The implementation must set the given *key* in the *overlay* to being non-existent.

### ext_storage_root_version_1

```
(func $ext_storage_root_version_1
    (return i64))
```

With:

- The result: a pointer-size.

The implementation must build a temporary *state* by applying the *overlay* to the state provided as input, then calculate the *Merkle value* of this temporary state.
The implementation then allocates a 32 bytes buffer using the same mechanism as `ext_allocator_malloc_version_1`, write the calculated Merkle value in it, and return a pointer-size to it.

> **Note**: Runtimes are encouraged to not perform any further modification to the storage after having called this function. An implementation can use this hint in order to cache the result of its calculation for later.

### ext_storage_root_version_2

```
(func $ext_storage_root_version_2
    (param $version i32) (return i64))
```

This function is deprecated and shouldn't be used by runtimes.
