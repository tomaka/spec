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

TODO https://github.com/paritytech/polkadot-sdk/blob/408af9b32d95acbbac5e18bee66fd1b74230a699/substrate/client/network/sync/src/schema/api.v1.proto#L5

The **request** is defined as the encoding of the `BlockRequest` protobuf struct in the following document:

```protobuf
syntax = "proto3";

package api.v1;

enum Direction {
	Ascending = 0;
	Descending = 1;
}

message BlockRequest {
	uint32 fields = 1;
	oneof from_block {
		bytes hash = 2;
		bytes number = 3;
	}
	Direction direction = 5;
	uint32 max_blocks = 6; // optional
	bool support_multiple_justifications = 7; // optional
}
```

The **response** is defined as the encoding of the `BlockResponse` protobuf struct in the following document:

```protobuf
syntax = "proto3";

package api.v1;

message BlockResponse {
	repeated BlockData blocks = 1;
}

message BlockData {
	// Block header hash.
	bytes hash = 1;
	// Block header if requested.
	bytes header = 2; // optional
	// Block body if requested.
	repeated bytes body = 3; // optional
	// Block receipt if requested.
	bytes receipt = 4; // optional
	// Block message queue if requested.
	bytes message_queue = 5; // optional
	// Justification if requested.
	bytes justification = 6; // optional
	// True if justification should be treated as present but empty.
	// This hack is unfortunately necessary because shortcomings in the protobuf format otherwise
	// doesn't make in possible to differentiate between a lack of justification and an empty
	// justification.
	bool is_empty_justification = 7; // optional, false if absent
	// Justifications if requested.
	// Unlike the field for a single justification, this field does not required an associated
	// boolean to differentiate between the lack of justifications and empty justification(s). This
	// is because empty justifications, like all justifications, are paired with a non-empty
	// consensus engine ID.
	bytes justifications = 8; // optional
	// Indexed block body if requestd.
	repeated bytes indexed_body = 9; // optional
}
```

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
| (repeated) Fragment | Fragment repeated *Number of fragments* times | (variable) |
| Is finished | Boolean (`true` is any non-zero value) | 1 |

Where a **fragment** is defined as:

| Field name         | Type      | Size (bytes)   |
| ------------------ | --------- | -------------- |
| Header | Block header | (variable) |
| Justification | Justification | (variable) |

TODO: what happens if block hash unknown

### State

**Protocol name**: `/<genesis-hash-and-fork-id>/state/2`

The **request** is defined as the encoding of the `StateRequest` protobuf struct in the following document:

```protobuf
syntax = "proto3";

package api.v1;

message StateRequest {
	bytes block = 1;
	repeated bytes start = 2; // optional
	bool no_proof = 3;
}
```

The **response** is defined as the encoding of the `StateResponse` protobuf struct in the following document:

```protobuf
syntax = "proto3";

package api.v1;

message StateResponse {
	// A collection of keys-values states. Only populated if `no_proof` is `true`
	repeated KeyValueStateEntry entries = 1;
	// If `no_proof` is false in request, this contains proof nodes.
	bytes proof = 2;
}

// A key value state.
message KeyValueStateEntry {
	// Root of for this level, empty length bytes
	// if top level.
	bytes state_root = 1;
	// A collection of keys-values.
	repeated StateEntry entries = 2;
	// Set to true when there are no more keys to return.
	bool complete = 3;
}

// A key-value pair.
message StateEntry {
	bytes key = 1;
	bytes value = 2;
}
```

TODO clean up

### Light

**Protocol name**: `/<genesis-hash-and-fork-id>/light/2`

The **request** is defined as the encoding of the `Request` protobuf struct in the following document:

```protobuf
syntax = "proto2";

package api.v1.light;

message Request {
	oneof request {
		RemoteCallRequest remote_call_request = 1;
		RemoteReadRequest remote_read_request = 2;
		RemoteReadChildRequest remote_read_child_request = 4;
	}
}

message RemoteCallRequest {
	required bytes block = 2;
	required string method = 3;
	required bytes data = 4;
}

message RemoteReadRequest {
	required bytes block = 2;
	repeated bytes keys = 3;
}

message RemoteReadChildRequest {
	required bytes block = 2;
	required bytes storage_key = 3;
	repeated bytes keys = 6;
}
```

TODO describe fields

The **response** is defined as the encoding of the `Response` protobuf struct in the following document:

```protobuf
syntax = "proto2";

package api.v1.light;

message Response {
	oneof response {
		RemoteCallResponse remote_call_response = 1;
		RemoteReadResponse remote_read_response = 2;
	}
}

message RemoteCallResponse {
	optional bytes proof = 2;
}

message RemoteReadResponse {
	optional bytes proof = 2;
}
```

The `proof` is TODO
TODO: the proof can be missing if the remote refuses to answer, which is redundant with the closing of the substream, maybe clean this with an RFC

### Kademlia

**Protocol name**: `/<genesis-hash-and-fork-id>/kad`

Invalid multiaddresses should be ignored. Implementations should not consider the entire response as invalid just because one multiaddress is invalid.

An implementation of the responsing side should make a reasonable effort to send back only multiaddresses that it thinks are reachable to it.

The requesting side should be aware of the fact that the list of multiaddresses can't be untrusted. A multiaddress might be unreachable, point to a non-conforming implementation, or point to an implementation whose **PeerId** is different from the one indicated.

> **Note**: This protocol is identical to the Kademlia protocol of the libp2p library, apart from the protocol name which differs.

TODO: for protocols below, see config at https://github.com/paritytech/polkadot-sdk/blob/master/polkadot/node/network/protocol/src/request_response/mod.rs#L192

### Chunk fetching

**Protocol name**: `/<genesis-hash-and-fork-id>/req_chunk/1`

TODO

### Collation fetching v1

**Protocol name**: `/<genesis-hash-and-fork-id>/req_collation/1`

TODO

### Collation fetching v2

**Protocol name**: `/<genesis-hash-and-fork-id>/req_collation/2`

TODO

### PoV fetching

**Protocol name**: `/<genesis-hash-and-fork-id>/req_pov/1`

TODO

### Available data fetching

**Protocol name**: `/<genesis-hash-and-fork-id>/req_available_data/1`

TODO

### Statement fetching

**Protocol name**: `/<genesis-hash-and-fork-id>/req_statement/1`

TODO

### Dispute sending

**Protocol name**: `/<genesis-hash-and-fork-id>/send_dispute/1`

TODO

### Attested candidate request

**Protocol name**: `/<genesis-hash-and-fork-id>/req_attested_candidate/2`

TODO

## Notification protocols

### Block announces

The format of the handshake is defined as:

| Field name         | Type      | Size (bytes)   |
| ------------------ | --------- | -------------- |
| Role | Role | 1 |
| Best block number | Little endian unsigned integer | 4 TODO or 8 |
| Best block hash | Bytes | 32 |
| Genesis block hash | Bytes | 32 |

> **Note**: The genesis block hash field is a redundant information.  TODO write an RFC about that

Where `Role` is one of:

- `0x1`: Full node capabilities.
- `0x2`: Light client capabilities.
- `0x4`: Authorities capabilities.

TODO explain Role and consider renaming through an RFC

The format of a notification is defined as:

| Field name         | Type      | Size (bytes)   |
| ------------------ | --------- | -------------- |
| Header | Block header | (variable) |
| Is new best | Boolean (`true` if non-zero value) | 1 |

TODO: missing a field

An implementation should send a block announce notification only if it would be capable of later answering a *Sync* protocol request concerning this block.

An implementation must not send a block annouce notification concerning a block header that hasn't been [*validated*](./blocks-verification.md).

### Transactions

The *transactions* notifications substream should be opened only after a *block announces* notifications substream has been opened.

The *handshake* of this substream is an empty set of bytes.

> **Note**: The handshake phase is identical to the one of the other notification protocols and consists in sending a `0` byte in order to indicate that the handshake payload is empty.

The format of a notification is:

| Field name         | Type      | Size (bytes)   |
| ------------------ | --------- | -------------- |
| Number of transactions | SCALE-compact-encoded unsigned integer | (variable) |
| (repeated) Transaction | Bytes repeated *Number of transactions* times | (variable) |

TODO: transactions aren't decodable, thus this table is meh, would be fixed by https://github.com/polkadot-fellows/RFCs/pull/56

Implementers are encouraged to send only one transaction per notification.
TODO: maybe update for https://github.com/polkadot-fellows/RFCs/pull/56 if it is merged before this spec is finished

When it receives a notification, an implementation should add the transaction to TODO.

### Grandpa

The format of the handshake is defined as:

| Field name         | Type      | Size (bytes)   |
| ------------------ | --------- | -------------- |
| Role | Role | 1 |

> **Note**: `Role` is defined in the *Block announces* section.

An implementation can use the `Role` of a peer as a hint in order to determine the priority level at which it broadcasts notifications to that peer.

The format of a notification is one of the following:

#### Neighbor packet

| Field name         | Type      | Size (bytes)   |
| ------------------ | --------- | -------------- |
| (constant) | 0x2 | 1 |
| (constant) | 0x1 | 1 |
| Round number | Little endian unsigned integer | 8 |
| Set ID | Little endian unsigned integer | 8 |
| Commit finalized height | Little endian unsigned integer | 4 TODO or 8 due to block number |

#### Commit message

| Field name         | Type      | Size (bytes)   |
| ------------------ | --------- | -------------- |
| (constant) | 0x1 | 1 |
| Grandpa commit | Grandpa commit TODO link | (variable) |

#### Vote

| Field name         | Type      | Size (bytes)   |
| ------------------ | --------- | -------------- |
| (constant) | 0x0 | 1 |
| Round number | Little endian unsigned integer | 8 |
| Set ID | Little endian unsigned integer | 8 |
| Vote type | Vote type | 1 |
| Target block hash | Bytes | 32 |
| Target block number | Little endian unsigned integer | 4 TODO or 8 due to block number |
| Signature | Bytes | 64 |
| Authority public key | Bytes | 32 |

Where `Vote type` is one of:

- `0x0`: Prevote
- `0x1`: Precommit
- `0x2`: Primary propose

An implementation should send a vote grandpa notification only if it would be capable of later answering a *Sync* protocol request concerning the target block.

#### Catch-up request

| Field name         | Type      | Size (bytes)   |
| ------------------ | --------- | -------------- |
| (constant) | 0x3 | 1 |
| Round number | Little endian unsigned integer | 8 |
| Set ID | Little endian unsigned integer | 8 |

### Catch-up response

TODO: according to smoldot the set id is before the round number, whereas everywhere else it's the other way around, check whether smoldot is wrong or if it's actually like that

| Field name         | Type      | Size (bytes)   |
| ------------------ | --------- | -------------- |
| (constant) | 0x4 | 1 |
| Set ID | Little endian unsigned integer | 8 |
| Round number | Little endian unsigned integer | 8 |
| Number of prevotes | SCALE-compact-encoded unsigned integer | (variable) |
| (repeated) Prevotes | Vote (see below) repeated *Number of prevotes* times | 36 or 40 times *Number of prevotes* TODO |
| Number of precommits | SCALE-compact-encoded unsigned integer | (variable) |
| (repeated) Precommits | Vote (see below) repeated *Number of precommits* times | 36 or 40 times *Number of precommits*  TODO |
| Base block hash | Bytes | 32 |
| Base block number | Little endian unsigned integer | 4 TODO or 8 due to block number |

Where a **Vote** is defined as:

| Target block hash | Bytes | 32 |
| Target block number | Little endian unsigned integer | 4 TODO or 8 due to block number |
| Signature | Bytes | 64 |
| Authority public key | Bytes | 32 |

### Validation v1

**Protocol name**: `/<genesis-hash-and-fork-id>/validation/1`

TODO

### Validation v2

**Protocol name**: `/<genesis-hash-and-fork-id>/validation/2`

TODO

### Collation v1

**Protocol name**: `/<genesis-hash-and-fork-id>/collation/1`

TODO

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
