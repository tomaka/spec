# Networking protocols

## Request-response protocols

### Sync

**Protocol name**: `/91b171bb158e2d3848fa23a9f1c25182fb8e20313b2c1eb49219da7a70ce90c3/sync/sync/2`

### Warp sync

**Protocol name**: `/91b171bb158e2d3848fa23a9f1c25182fb8e20313b2c1eb49219da7a70ce90c3/sync/warp`

### State

**Protocol name**: `/91b171bb158e2d3848fa23a9f1c25182fb8e20313b2c1eb49219da7a70ce90c3/state/2`

### Light

**Protocol name**: `/91b171bb158e2d3848fa23a9f1c25182fb8e20313b2c1eb49219da7a70ce90c3/light/2`

### Kademlia

**Protocol name**: `/91b171bb158e2d3848fa23a9f1c25182fb8e20313b2c1eb49219da7a70ce90c3/kad`

Invalid multiaddresses should be ignored. Implementations should not consider the entire response as invalid just because one multiaddress is invalid.

An implementation of the responsing side should make a reasonable effort to send back only multiaddresses that it thinks are reachable to it.

Implementers should be aware of the fact that the list of multiaddresses can't be untrusted. A multiaddress might be unreachable, point to a non-conforming implementation, or point to an implementation whose **PeerId** is different from the one indicated.

> **Note**: This protocol is identical to the Kademlia protocol of the libp2p library.

TODO: for protocols below, see config at https://github.com/paritytech/polkadot-sdk/blob/master/polkadot/node/network/protocol/src/request_response/mod.rs#L192

### Chunk fetching

**Protocol name**: `/91b171bb158e2d3848fa23a9f1c25182fb8e20313b2c1eb49219da7a70ce90c3/req_chunk/1`

### Collation fetching v1

**Protocol name**: `/91b171bb158e2d3848fa23a9f1c25182fb8e20313b2c1eb49219da7a70ce90c3/req_collation/1`

### Collation fetching v2

**Protocol name**: `/91b171bb158e2d3848fa23a9f1c25182fb8e20313b2c1eb49219da7a70ce90c3/req_collation/2`

### PoV fetching

**Protocol name**: `/91b171bb158e2d3848fa23a9f1c25182fb8e20313b2c1eb49219da7a70ce90c3/req_pov/1`

### Available data fetching

**Protocol name**: `/91b171bb158e2d3848fa23a9f1c25182fb8e20313b2c1eb49219da7a70ce90c3/req_available_data/1`

### Statement fetching

**Protocol name**: `/91b171bb158e2d3848fa23a9f1c25182fb8e20313b2c1eb49219da7a70ce90c3/req_statement/1`

### Dispute sending

**Protocol name**: `/91b171bb158e2d3848fa23a9f1c25182fb8e20313b2c1eb49219da7a70ce90c3/send_dispute/1`

### Attested candidate request

**Protocol name**: `/91b171bb158e2d3848fa23a9f1c25182fb8e20313b2c1eb49219da7a70ce90c3/req_attested_candidate/2`

## Notification protocols

### Block announces

### Transactions

The *transactions* notifications substream should be opened only after a *block announces* notifications substream has been opened.

The *handshake* of this substream is an empty set of bytes.

> **Note**: The handshake phase is identical to the one of the other notification protocols and consists in sending a `0` byte in order to indicate that the handshake payload is empty.

The format of a notification is:

- A SCALE-compact-encoded integer indicating the number of transactions in the message.
- Zero or more transactions.

Implementers are encouraged to send only one transaction per notification.

### Grandpa

### Validation v1

**Protocol name**: `/91b171bb158e2d3848fa23a9f1c25182fb8e20313b2c1eb49219da7a70ce90c3/validation/1`

### Validation v2

**Protocol name**: `/91b171bb158e2d3848fa23a9f1c25182fb8e20313b2c1eb49219da7a70ce90c3/validation/2`

### Collation v1

**Protocol name**: `/91b171bb158e2d3848fa23a9f1c25182fb8e20313b2c1eb49219da7a70ce90c3/collation/1`

### Collation v2

**Protocol name**: `/91b171bb158e2d3848fa23a9f1c25182fb8e20313b2c1eb49219da7a70ce90c3/collation/2`

TODO: see https://github.com/paritytech/polkadot-sdk/tree/master/polkadot/node/network/protocol/src
https://github.com/paritytech/polkadot-sdk/blob/master/polkadot/node/network/protocol/src/peer_set.rs#L283

## Other

### Ping

**Protocol name**: `/ipfs/ping/1.0.0`

> **Note**: This protocol is identical to the ping protocol of the libp2p library.

### Identify

**Protocol name**: `/ipfs/id/1.0.0`

> **Note**: This protocol is identical to the identify protocol of the libp2p library.

After the protocol has been negotiated on a substream, the receiving side of the substream sends back its identification information.

The identification information is the encoding of the following protobuf definition:

```protobuf
syntax = "proto2";
message Identify {
  optional string protocolVersion = 5;
  optional string agentVersion = 6;
  optional bytes publicKey = 1;
  repeated bytes listenAddrs = 2;
  optional bytes observedAddr = 4;
  repeated string protocols = 3;
}
```

The side that opened the substream can close its writing side either before or after receiving the response, at its discretion.

TODO finish
