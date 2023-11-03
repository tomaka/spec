# Blocks

A **block** consists in the following properties:

- A *header*.
- A *hash*, which must be equal to the 32 bytes [BLAKE2](https://datatracker.ietf.org/doc/html/rfc7693) hash of the header.
- An ordered list of transactions TODO or is it extrinsics called the *body* of the block.
- A *state*, as described in [the State](state.html) section.

## Block header

A *block header* is defined as the concatenation of:

- An *unsigned block header* (see below).
- A 32 bytes *signature*.

An *unsigned block header* is defined as the concatenation of the following bytes:

- *Parent hash*: 32 bytes.
- *Block number*: A little-endian 32 bits unsigned integer.
- *State root*: 32 bytes.
- *Extrinsics root*: 32 bytes.
- *Number of digest items*: A SCALE-compact-encoded number equal to the number of digest items.
- *Digest items*: Zero or more digest items. See below.

### Digest item

A digest item is defined as one of:

TODO: 

## Header authentication

A host can *authenticate* a block.

## Header validation

A host can *validate* a block header. Validating a block header also authenticates the block header.

Validating a block is done by following these steps:

- Validating the block header whose hash is equal to the *parent hash* of the block to validate.
- Obtain the *state root* of the block header whose hash is equal to the *parent hash* of the block to validate.
- Obtain the state whose state root is equal to the *state root* obtained at the previous step.
- Obtain the *runtime WebAssembly code* of that state.
- TODO check entry point API version
- Execute the entry point named *BlockBuilder_check_inherents*.
- Execute the entry point named *Core_execute_block*. The input is equal to the concatenation of the *block header* to verify with the body of the block. If execution fails, then the block is invalid. The output must be empty.
- Check whether the new state is valid. TODO means check if new runtime code is valid, something else?

> **Note**: The header validation will fail if the body that was provided doesn't correspond to the *extrinsics root* field of the header. If the block was downloaded from an untrusted source, implementers should be aware that the body might have been modified by this untrusted source. For this reason, in that situation implementers are encouraged to check whether the body matches the *extrinsics root* prior to starting the validation.
