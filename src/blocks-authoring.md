# Blocks authoring

## Better block

Given two block headers A and B, we can define the **better block** as follows:

- If A's *parent hash* is equal to B, then A is a better block than B.
- The better block is transitive: if A is a better block than a block header C, and that block header C is a better block than block header B, then A is also a better block header than block header B.
- If the total number of Babe primary slots claims on A is superior to the total number of Babe primary slots claims on B, then A is a better block than B. TODO: define Babe primary slots claims

TODO: https://paritytech.github.io/polkadot-sdk/book/protocol-chain-selection.html

TODO: a block is always better if it is in the finalized chain

## Validators set

The **Aura validators set** and the **Babe validators set** of a block can be determined in two different ways: through block headers and through a runtime call. Only the runtime call method is available for genesis blocks (in other words, blocks whose *block number* is equal to 0).

Through block headers:

- For the **Aura validators set**:
    - If the block doesn't have any Aura digest item, the Aura validators set is empty.
    - If the block has an Aura consensus digest item of type "Authorities change", the Aura validators set is equal to that list. TODO: what if there's a OnDisabled in the same block header
    - Otherwise, the Aura validators set is identical to the Aura validators set of the block whose hash is equal to the `parent_hash` of the block.
- TODO babe

Through a runtime call:

- For the **Aura validators set**:
    - AuraApi_authorities

A client implementation can assume that these two methods produce the same result. TODO: what if it doesn't

## Next slot claim

Given a block and an sr25519 private key, the next **claimable Babe primary slot** of a block is:

- If the sr25519 public key correspondoing to the private key is not part of the Babe validators set of that block: never.
- TODO

Given a block and an sr25519 public key, the next **claimable Babe secondary slot** of a block is:

- If the sr25519 public key is not part of the Babe validators set of that block: never.
- TODO

Given a block and an ed25519 public key, the next **claimable Aura slot** of a block is:

- If the ed25519 public key is not part of the Aura validators set of that block: never.
- TODO

## Authoring a block

Given a set of blocks, an ed25519 or an sr25519 private key, and a list of transactions TODO detail what that means, **authoring a block** consists in the following steps:

- Find the *best block* amongst that set using the **better block** algorithm described above.
- Determine the next claimable Aura slot, next claimable Babe primary slot, and the next Babe claimable secondary slot of that block and private key combination.
- Wait until one of these three slots, whichever comes first. TODO: explain how to convert to timestamp
- Do a runtime call for `Core_initialize_block` using the state of the *best block* (see above), passing as parameter a *block header* with the following field values:
  - *Parent_hash* must be equal to the hash of the *best block*.
  - *Block number* must be equal to the *block number* of the *best block* plus one.
  - *State root* must be all zeroes.
  - *Extrinsics root* must be all zeroes.
  - One digest item: if the slot is an Aura slot, TODO finish.
- Call `BlockBuilder_inherent_extrinsics` TODO detail the input  TODO: it's not possible to decode the output without the runtime at the moment
- For each extrinsic returned `BlockBuilder_inherent_extrinsics`, call `BlockBuilder_apply_extrinsic`. TODO detail return value
- For each transaction, call `BlockBuilder_apply_extrinsic`. TODO details and indicate for how long
- Call `BlockBuilder_finalize_block`, with an empty input.
- Create a digital signature of the *block header* using the ed25519 or sr25519 private key.
- Add to the digest items of the *block header* either an *Aura seal* or a *Babe seal* (depending on whether the slot was an Aura slot or a Babe slot) digest item containing the signature. Add one to the *number of digest items* of the *block header*.

If at any point in the future the client authors a block again using the same private key, it **must** include in the set of blocks the newly-authored block.
This constraint is necessary in order to guarantee that no two blocks using the same key are authored using the same slot.

### Inherents

The **inherents** are defined as:

| Field name         | Type      | Size (bytes)   |
| ------------------ | --------- | -------------- |
| (constant) | 0x2 | 1 |
| (constant) | ASCII string `timstap0` | 8 |
TODO: len
| Timestamp | Little-endian unsigned integer | 8 |

The `Timestamp` field must contain the current Unix time, in other words the number of milliseconds since the Unix epoch ignoring leap seconds.

TODO parachain inherents
