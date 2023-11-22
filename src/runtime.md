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
- It must either export exactly one memory named `memory`, or import exactly one memory named `memory` in module name `env`.
- It must export a global named `__heap_base` of type `i32`.
- TODO: runtime spec

The **runtime WebAssembly** is allowed to import any function, whatever their signature. A host implementation must not consider a **runtime WebAssembly** as invalid only because it imports an unknown function or a known function with a signature that doesn't match the expected one. A host implementation must raise an error only if such a function is called during the execution of the WebAssembly.

## Runtime call

Given a *runtime WebAssembly*, a *number of heap pages*, a [*state*](state.md), a *runtime entry point*, an *offchain functions enabled* boolean, and some input data, the host can **do a runtime call**.

Doing a runtime call consists in the following step:

- Verify that the *runtime entry point* is a function exported by the *runtime WebAssembly*, and whose signature is `(func $runtime_entry_point (param $data i32) (param $len i32) (result i64))`.
- Instantiate the *runtime WebAssembly*, as defined [here](https://webassembly.github.io/spec/core/bikeshed/#instantiation%E2%91%A1). The list of imports is described below.
- Initialize the *overlay*, defined as the list of modifications performed to the state since the beginning of the execution.
- Read the value of the `__heap_base` global exported by the *runtime WebAssembly*, and initialize a new *allocator*.
- Use the allocator in order to allocate a buffer of size equal to the input data. Copy the input data to this buffer.
- Invoke the *runtime entry point*, as defined [here](https://webassembly.github.io/spec/core/bikeshed/#invocation%E2%91%A1), passing as argument a pointer to the buffer containing the input data and the length of the input data.
- Once the invocation is finished, TODO describe how to get output.

TODO: talk about memory consumption and halting problem stuff

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
