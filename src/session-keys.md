# Session keys

TODO: https://github.com/paritytech/polkadot-sdk/blob/408af9b32d95acbbac5e18bee66fd1b74230a699/substrate/primitives/core/src/crypto.rs#L1145

When the `ext_crypto_ed25519_generate_version_1`, `ext_crypto_sr25519_generate_version_1`, or `ext_crypto_ecdsa_generate_version_1` host functions are called by the runtime during a runtime call, the implementation must persist the private key. In other words, the private key should be stored on disk.

> **Note**: These host functions are not always legal to call. See the appropriate section for more information.

Additionally, an implementation must start performing some actions based on the `key_type` parameter that was provided when calling the host function. The `key_types` that require additional actions are:

- `aura`
- `babe`
- `gran`
- `audi`
- `para`

TODO: parachain stuff?

Any other `key_type` doesn't require any further action.

## Aura and Babe

In order to handle a key of type `aura` or `babe`, an implementation must:

TODO: find a way to add links to the rest of the spec

- Maintain a tree of blocks that successfully pass block execution.
- Run offchain workers. TODO clarify
- Maintain a set of valid transactions.
- Try as hard as possible to gather blocks from the other peers.
- Try as hard as possible to gather transactions from the other peers.
- Try to author blocks, providing the tree of blocks, key, and set of valid transactions. TODO match aura/babe key types with Aura and Babe algorithms
- Whenever a block is successfully authored, send a block announce notification to other peers about this block.
- Serve to other peers the blocks it successfully authors through the `sync` protocol.

## Grandpa

In order to handle a key of type `gran`, an implementation must:

- Maintain a tree of blocks that successfully pass block execution.
- Run offchain workers. TODO clarify
- Maintain set of prevotes, precommits, Grandpa commits, and Grandpa justifications.
- Try as hard as possible to gather blocks from the other peers.
- Try as hard as possible to gather prevotes, precommits, Grandpa commits, and Grandpa justifications from the other peers.
- Run the Grandpa voter algorithm.
- Whenever a prevote or a precommit is emitted by the Grandpa voter algorithm, send it to other peers.

> **Note**: This specification intentionally leaves some freedom in the choice of the block to vote for. Historically, implementations have been voting for the ancestor of the best block whose number is equal to the best block's number minus 3. Adding a gap between the best block and the block to finalize might avoid finalizing blocks that have accidentally modified the state in a way that ultimately leads to the chain stalling. However, this gap is not enforced in any way by the protocol and is intentionally not part of this specification. The exact algorithm that chooses which block to vote for should be the result of an informal dialogue within the Polkadot community.

TODO finish

## Authority discovery

In order to handle a key of type `audi`, an implementation must:

- Maintain a tree of blocks.
- Try as hard as possible to gather blocks from the other peers.
- If and only if the key is present in the *authority-discovery validators set* of the best block of the tree, publish a valid *authority-discovery record*.

TODO

# Parachain validation

TODO
