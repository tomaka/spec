# Blocks

A **block** consists in the following properties:

- A *header*.
- A *hash*, which must be equal to the 32 bytes [BLAKE2](https://datatracker.ietf.org/doc/html/rfc7693) hash of the header.
- An ordered list of transactions TODO or is it extrinsics called the *body* of the block.
- A *state*, as described in [the State](state.html) section.

TODO: this section is weird

## Block header

A *block header* is defined as:

| Field name         | Type      | Size (bytes)   |
| ------------------ | --------- | -------------- |
| Parent hash | Bytes | 32 |
| Block number | Little endian unsigned integer | 4 |
| State root | Bytes | 32 |
| Extrinsics root | Bytes | 32 |
| Number of digest items | SCALE-compact-encoded unsigned integer | (variable) |
| (repeated) Digest items | Digest item repeated *Number of digest items* times | (variable) |

### Digest item

A digest item is defined as one of:

TODO: finish here

#### Aura consensus authorities change

| Field name         | Type      | Size (bytes)   |
| ------------------ | --------- | -------------- |
| (constant) | 0x4 | 1 |
| (constant) | ASCII string `aura` | 4 |
| (constant) | 0x1 | 1 |
| Number of authorities | SCALE-compact-encoded unsigned integer | (variable) |
| (repeated) Authorities | Bytes repeated *Number of authorities* times | 32 times `Number of authorities` |

#### Aura consensus authority disabled

| Field name         | Type      | Size (bytes)   |
| ------------------ | --------- | -------------- |
| (constant) | 0x4 | 1 |
| (constant) | ASCII string `aura` | 4 |
| (constant) | 0x2 | 1 |
| Authority index | Little endian unsigned integer | 4 |

#### Aura seal

| Field name         | Type      | Size (bytes)   |
| ------------------ | --------- | -------------- |
| (constant) | 0x5 | 1 |
| (constant) | ASCII string `aura` | 4 |
| Signature | Bytes | 32 |

#### Aura slot number

| Field name         | Type      | Size (bytes)   |
| ------------------ | --------- | -------------- |
| (constant) | 0x6 | 1 |
| (constant) | ASCII string `aura` | 4 |
| Slot number | Little endian unsigned integer | 8 |

#### Babe seal

| Field name         | Type      | Size (bytes)   |
| ------------------ | --------- | -------------- |
| (constant) | 0x5 | 1 |
| (constant) | ASCII string `BABE` | 4 |
| Signature | Bytes | 32 |

#### Runtime environment update

| Field name         | Type      | Size (bytes)   |
| ------------------ | --------- | -------------- |
| (constant) | 0x8 | 1 |

#### Other

| Field name         | Type      | Size (bytes)   |
| ------------------ | --------- | -------------- |
| (constant) | 0x0 | 1 |
| Data length | SCALE-compact-encoded unsigned integer | (variable) |
| Data | Bytes | (indicated by `Data length`) |

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

## Tree of blocks

A **tree of blocks** is defined as a set of *block headers* where the *parent hash* of every block header in the set except for one is the hash of a block header that is also found in the set.

The **best block** of a *tree of blocks* is the block in the set for which there exists no *better block*. If multiple blocks are equal, which one is the *best block* is unspecified and implementation-defined.
