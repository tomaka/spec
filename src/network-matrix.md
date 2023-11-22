# Validators matrix

## Topology

Given a *block*, the **validators topology** can be calculated by:

- Perform a runtime call to `Babe_currentEpoch`. TODO more precise
- Calculate the [BLAKE2](https://datatracker.ietf.org/doc/html/rfc7693) hash of the concatenation of the ASCII string `gossipsu` with the Babe randomness.
- Get the list of authorities. (how?)
- Do this lol https://docs.rs/rand/latest/src/rand/seq/mod.rs.html#244-245
- Calculate the square root (rounded down) of the number of authorities.
- TODO https://github.com/paritytech/polkadot-sdk/blob/master/polkadot/node/network/protocol/src/grid_topology.rs#L122

TODO: https://github.com/paritytech/polkadot-sdk/blob/master/polkadot/node/network/gossip-support/src/lib.rs#L579
