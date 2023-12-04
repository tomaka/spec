# Networking protocols

After an encrypted multiplexed connection is open between two peers, a **substream** can be opened on the connection through the following steps:

- One side (called "the initiator") opens a substream using the underlying multiplexing protocol.
- The initiator of the substream then starts a **multistream-select** negotiation, as described in [the transport protocol](./network-transport.md) section. The name of the protocol being negotiated must be one of the protocol names found later down this section.
- Once the **multistream-select** negotiation is finished, the rest of the steps depend on the protocol that has been negotiated.

An implementation should reject substreams (using the substream rejection mechanism of the underlying multiplexing protocol) if too many substreams are already opened.

In the protocol names below, the string `<genesis-hash-and-fork-id>` should be replaced with the **lower-case-hexadecimal-encoded 32 bytes hash of the genesis block of the chain**.

> **Example**: For the Polkadot network, this string is `91b171bb158e2d3848fa23a9f1c25182fb8e20313b2c1eb49219da7a70ce90c3`. The "sync" protocol name is thus `/91b171bb158e2d3848fa23a9f1c25182fb8e20313b2c1eb49219da7a70ce90c3/sync/sync/2`.

## Request-response protocols

This sub-section contains a list of **request-response protocols**.

A request-response protocol works as follows:

- After the **multistream-select** negotiation is finished, the initiator of the substream sends a LEB-128-encoded unsigned integer representing the size in bytes of the request.
- The initiator of the substream then sends the request. The format of the request depends on the protocol that has been negotiated.
- Then, either:
  - The other peer sends a LEB-128-encoded unsigned integer representing the size in bytes of the response, then sends the response. The format of the response depends on the protocol that has been negotiated.
  - Or the other peer closes its writing side, indicating that it refuses the answer the request.
- The initiator and the other peer close their writing side. TODO: explain that it can be done in parallel, or draw a diagram or something
- The substream is closed after the two peers have closed their writing side.

No reason is provided to the requester when the responder refuses to answer the request.

### Sync

**Protocol name**: `/<genesis-hash-and-fork-id>/sync/sync/2`

### Warp sync

**Protocol name**: `/<genesis-hash-and-fork-id>/sync/warp`

The **request** is as follows:

| Field name         | Type      | Size (bytes)   |
| ------------------ | --------- | -------------- |
| Starting block hash | Bytes | 32 |

The **response** is as follows:

| Field name         | Type      | Size (bytes)   |
| ------------------ | --------- | -------------- |
| Number of fragments | SCALE-compact-encoded unsigned integer | (variable) |
| (repeated) Fragment | Fragment | (variable) |
| Is finished | Boolean (`true` is any non-zero value) | 1 |

Where a **fragment** is defined as:

| Field name         | Type      | Size (bytes)   |
| ------------------ | --------- | -------------- |
| Header | Block header | (variable) |
| Justification | Justification | (variable) |

TODO: what happens if block hash unknown

### State

**Protocol name**: `/<genesis-hash-and-fork-id>/state/2`

### Light

**Protocol name**: `/<genesis-hash-and-fork-id>/light/2`

### Kademlia

**Protocol name**: `/<genesis-hash-and-fork-id>/kad`

Invalid multiaddresses should be ignored. Implementations should not consider the entire response as invalid just because one multiaddress is invalid.

An implementation of the responsing side should make a reasonable effort to send back only multiaddresses that it thinks are reachable to it.

The requesting side should be aware of the fact that the list of multiaddresses can't be untrusted. A multiaddress might be unreachable, point to a non-conforming implementation, or point to an implementation whose **PeerId** is different from the one indicated.

> **Note**: This protocol is identical to the Kademlia protocol of the libp2p library, apart from the protocol name which differs.

TODO: for protocols below, see config at https://github.com/paritytech/polkadot-sdk/blob/master/polkadot/node/network/protocol/src/request_response/mod.rs#L192

### Chunk fetching

**Protocol name**: `/<genesis-hash-and-fork-id>/req_chunk/1`

### Collation fetching v1

**Protocol name**: `/<genesis-hash-and-fork-id>/req_collation/1`

### Collation fetching v2

**Protocol name**: `/<genesis-hash-and-fork-id>/req_collation/2`

### PoV fetching

**Protocol name**: `/<genesis-hash-and-fork-id>/req_pov/1`

### Available data fetching

**Protocol name**: `/<genesis-hash-and-fork-id>/req_available_data/1`

### Statement fetching

**Protocol name**: `/<genesis-hash-and-fork-id>/req_statement/1`

### Dispute sending

**Protocol name**: `/<genesis-hash-and-fork-id>/send_dispute/1`

### Attested candidate request

**Protocol name**: `/<genesis-hash-and-fork-id>/req_attested_candidate/2`

## Notification protocols

### Block announces

### Transactions

The *transactions* notifications substream should be opened only after a *block announces* notifications substream has been opened.

The *handshake* of this substream is an empty set of bytes.

> **Note**: The handshake phase is identical to the one of the other notification protocols and consists in sending a `0` byte in order to indicate that the handshake payload is empty.

The format of a notification is:

| Field name         | Type      | Size (bytes)   |
| ------------------ | --------- | -------------- |
| Number of transactions | SCALE-compact-encoded unsigned integer | (variable) |
| (repeated) Transaction | Bytes | (variable) |

TODO: transactions aren't decodable, thus this table is meh, would be fixed by https://github.com/polkadot-fellows/RFCs/pull/56

Implementers are encouraged to send only one transaction per notification.
TODO: maybe update for https://github.com/polkadot-fellows/RFCs/pull/56 if it is merged before this spec is finished

### Grandpa

### Validation v1

**Protocol name**: `/<genesis-hash-and-fork-id>/validation/1`

### Validation v2

**Protocol name**: `/<genesis-hash-and-fork-id>/validation/2`

### Collation v1

**Protocol name**: `/<genesis-hash-and-fork-id>/collation/1`

### Collation v2

**Protocol name**: `/<genesis-hash-and-fork-id>/collation/2`

TODO: see https://github.com/paritytech/polkadot-sdk/tree/master/polkadot/node/network/protocol/src
https://github.com/paritytech/polkadot-sdk/blob/master/polkadot/node/network/protocol/src/peer_set.rs#L283

## Other

### Ping

**Protocol name**: `/ipfs/ping/1.0.0`

> **Note**: This protocol is identical to the ping protocol of the libp2p library.

TODO: finish

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
