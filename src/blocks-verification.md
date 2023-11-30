# Blocks verification

There exists two levels of verification of a block: validating the block and executing the block.

A client implementation can assume that a block that successfully executes also successfully validates.
TODO: say that it's a runtime bug it it's not the case or something

## Header validation

A host can *validate* a block header.

A **genesis block header** is always valid and doesn't need to be validated.

Validating a non-genesis block header is done by following these steps:

- Validating the *block header* whose hash is equal to the *parent hash* of the block to validate.
- Verify that the *block number* of the block header to verify is equal to the *block number* of the parent plus one.
TODO

## Block execution

A host can *execute* a block. Executing a block also validates the header of this block.

A genesis block is assumed to always succeed execution.

Executing a block is done by following these steps:

- Executing the block whose hash is equal to the *parent hash* of the block to execute.
- Obtain the *state root* of the block header whose hash is equal to the *parent hash* of the block to execute.
- Obtain the state whose state root is equal to the *state root* obtained at the previous step.
- Obtain the *runtime WebAssembly code* of that state.
- TODO check entry point API version
- Execute the entry point named *BlockBuilder_check_inherents*.
- Execute the entry point named *Core_execute_block*. The input is equal to the concatenation of the *block header* to verify with the body of the block. If execution fails, then the block is invalid. The output must be empty.
- Check whether the new state is valid. TODO means check if new runtime code is valid, something else?

> **Note**: The block execution will fail if the body that was provided doesn't correspond to the *extrinsics root* field of the header. If the block was downloaded from an untrusted source, implementers should be aware that the body might have been modified by this untrusted source. For this reason, in that situation implementers are encouraged to check whether the body matches the *extrinsics root* prior to starting the execution.

Because the block execution requires providing a timestamp, its execution may succeed or fail based on this timestamp. A client implementation can assume that a block execution that has succeeded with a certain timestamp will always succeed as well with any newer timestamp. In other words, once a block has executed successfully once, executing it again in the future will succeed as well.

TODO: should a host try executing again in case the execution succeeds in the future? the answer is no but this must be clarified
