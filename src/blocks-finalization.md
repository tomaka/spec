# Blocks finalization

## Grandpa validators set

Given a *block header*, the **Grandpa validators set** can be determined through the following process:

TODO finish

- Find the first ancestor that has a Grandpa digest item.

TODO make it clear that for forced changes the block must have been executed, otherwise security issue

> **Note**: The *Grandpa validators set* of a block corresponds to the list of validators that can finalize blocks that are children of the block in question. The block itself must be finalized by a validator of the validators set of the parent block.

The **validators set id** of a block can be determined through:

TODO

## Grandpa commits and justifications

### Grandpa commit

A **Grandpa commit** is defined as:

| Field name         | Type      | Size (bytes)   |
| ------------------ | --------- | -------------- |
| Round number | Little endian unsigned integer | 8 |
| Set ID | Little endian unsigned integer | 8 |
| Target block hash | Bytes | 32 |
| Target block number | Little endian unsigned integer | 4 TODO or 8 due to block number |
| Number of precommits | SCALE-compact-encoded unsigned integer | (variable) |
| (repeated) Precommits | Vote | *Number of precommits* * 36 or 40 TODO |
| Number of authentication data | SCALE-compact-encoded unsigned integer | (variable) |
| (repeated) Authentication data | Authentication data | *Number of authentication data* * 96 |

Where a **Vote** is defined as:

| Field name         | Type      | Size (bytes)   |
| ------------------ | --------- | -------------- |
| Target block hash | Bytes | 32 |
| Target block number | Little endian unsigned integer | 4 TODO or 8 due to block number |

And **Authentication data** is defined as:

| Field name         | Type      | Size (bytes)   |
| ------------------ | --------- | -------------- |
| Signature | Bytes | 64 |
| Authority public key | Bytes | 32 |

### Grandpa justification

A **Grandpa justification** is defined as:

| Field name         | Type      | Size (bytes)   |
| ------------------ | --------- | -------------- |
| Round number | Little endian unsigned integer | 8 |
| Target block hash | Bytes | 32 |
| Target block number | Little endian unsigned integer | 4 TODO or 8 due to block number |
| Number of precommits | SCALE-compact-encoded unsigned integer | (variable) |
| (repeated) Precommits | Vote | *Number of precommits* * 36 or 40 TODO |
| Number of vote ancestries | SCALE-compact-encoded unsigned integer | (variable) |
| (repeated) Vote ancestries | Block header | *Number of vote ancestries* * (variable) |

Where a **Vote** is defined as:

| Field name         | Type      | Size (bytes)   |
| ------------------ | --------- | -------------- |
| Target block hash | Bytes | 32 |
| Target block number | Little endian unsigned integer | 4 TODO or 8 due to block number |
| Signature | Bytes | 64 |
| Authority public key | Bytes | 32 |

### Verification

A *Grandpa commit* or *Grandpa justification* can be **verified** by following these steps:

- TODO

## Finalized block

Given a *tree of blocks*, a set of *verified Grandpa commit*s, and a set of *verified Grandpa justification*s, the **latest finalized block** is defined as the block with the highest *block number* TODO.

An implementation can assume that when it adds a new *Grandpa commit* or *Grandpa justification* to the set, the *latest finalized block* can only ever become a descendant of the previous latest finalized block. TODO unclear

> **Note**: Thanks to this assumption, an implementation generally only needs to keep track of the charateristics of the latest finalized block and its descendants. Any block that isn't the latest finalized block or one of its descendants may be discarded.

A block is **finalized** either if it is equal to the *latest finalized block*, or if it is the parent of a *finalized* block.

> **Note**: In other words, the *latest finalized block* and all of its ancestors (and no other block besides them) are *finalized*.

## Finalizable block

A *block header* is **finalizable** if:

- Its parent block is *finalized* or *finalizable*.
- TODO: parachain stuff

## Grandpa rounds

Given a set of prevote *block headers*, a set of precommit *block headers*, and a *number of validators*:

### Supermajority threshold

The **supermajority threshold** is defined as `(number of validators - number of validators that have equivocated) * 2 / 3 + 1`.

### Prevotes ghost

The **prevotes ghost** is defined as the highest block header in the prevotes that has a supermajority of the threshold.
If the number of prevote *block headers* is too small, the prevotes ghost is undefined.
TODO explain better

> **Note**: As more block headers are added, the prevotes ghost can only ever move to higher blocks.

### Estimate

The **estimate** is defined as the precommit block header inferior or equal to the *prevotes ghost* that can potentially achieve a supermajority of the threshold.
TODO: define more formally

> **Note**: More informally, the estimate is the highest block that could still potentially achieve a supermajority.

> **Note**: As more precommit block headers are added to the set, the estimate can only ever move to lower blocks. As more prevote block headers are added to the set, the estimate can only ever move to higher blocks.

### Completable

A round is **completable** if either:

- The *estimate* is strictly inferior to the *prevotes ghost*.
- It becomes impossible for the *estimate* to become strictly superior to the *prevotes ghost*.

### Finality update

The **updated finalized block** is defined as:

- 
