# Session keys

TODO: https://github.com/paritytech/polkadot-sdk/blob/408af9b32d95acbbac5e18bee66fd1b74230a699/substrate/primitives/core/src/crypto.rs#L1145

## Aura and Babe

In order to handle an Aura or Babe key, an implementation must:

- Try as hard as possible to gather blocks from the other peers.
- Try to author blocks.
- Whenever it successfully authors a block, send a block announce notification to other peers about this block.
- Serve the blocks it successfully authors through the `sync` protocol.

## Grandpa

In order to handle a Grandpa key, an implementation must:

- Try as hard as possible to gather blocks from the other peers.
- Try as hard as possible to gather grandpa votes from the other peers.
- TODO: finish

The **Grandpa voter** algorithm is defined as:

- If the public key corresponding to the Grandpa key is *not* equal to the primary, wait for either 2 seconds or the round is completable, whatever comes first.

## Authority discovery

TODO
