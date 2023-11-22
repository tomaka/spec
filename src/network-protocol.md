# Networking protocols

## Request-response protocols

### Warp sync

**Protocol name**: `/91b171bb158e2d3848fa23a9f1c25182fb8e20313b2c1eb49219da7a70ce90c3/sync/warp`



TODO: see https://github.com/paritytech/polkadot-sdk/tree/master/polkadot/node/network/protocol/src
https://github.com/paritytech/polkadot-sdk/blob/master/polkadot/node/network/protocol/src/peer_set.rs#L283

### Kademlia

**Protocol name**: `/91b171bb158e2d3848fa23a9f1c25182fb8e20313b2c1eb49219da7a70ce90c3/kad`

Invalid multiaddresses should be ignored. Implementations should not consider the entire response as invalid just because one multiaddress is invalid.

An implementation of the responsing side should make a reasonable effort to send back only multiaddresses that it thinks are reachable to it.

Implementers should be aware of the fact that the list of multiaddresses can't be untrusted. A multiaddress might be unreachable, point to a non-conforming implementation, or point to an implementation whose **PeerId** is different from the one indicated.

## Other

### Ping

### Identify
