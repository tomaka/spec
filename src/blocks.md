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

## Better block

Given two block headers A and B, we can define the **better block** as follows:

- If A's *parent hash* is equal to B, then A is a better block than B.
- The better block is transitive: if A is a better block than a block header C, and that block header C is a better block than block header B, then A is also a better block header than block header B.
- If the total number of Babe primary slots claims on A is superior to the total number of Babe primary slots claims on B, then A is a better block than B. TODO: define Babe primary slots claims

TODO: https://paritytech.github.io/polkadot-sdk/book/protocol-chain-selection.html
