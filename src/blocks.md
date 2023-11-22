# Blocks

A **block** consists in the following properties:

- A *header*.
- A *hash*, which must be equal to the 32 bytes [BLAKE2](https://datatracker.ietf.org/doc/html/rfc7693) hash of the header.
- An ordered list of transactions TODO or is it extrinsics called the *body* of the block.
- A *state*, as described in [the State](state.html) section.

## Block header

A *block header* is a collection of bytes defined as the concatenation of:

- An *unsigned block header* (see below).
- A 32 bytes *signature*.

An *unsigned block header* is a collection of bytes defined as the concatenation of:

- *Parent hash*: 32 bytes.
- *Block number*: Little-endian 32 bits unsigned integer.
- *State root*: 32 bytes.
- *Extrinsics root*: 32 bytes.
- *Number of digest items*: SCALE-compact-encoded number equal to the number of digest items.
- *Digest items*: Zero or more digest items. See below.

### Digest item

A digest item is defined as one of:

TODO: 

## Genesis block header

A **genesis block header** is a **block header** that respects the following constraints:

- The *parent hash* is equal to all 0s.
- The *block number* is equal to 0.
- The *extrinsics root* is equal to the *Merkle value* of an empty trie.
- The list of digest items is empty.

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
