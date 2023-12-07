# Kademlia


## Authority discovery

The **authority-discovery validators set** of a *block header* is defined as the output of the runtime call to `AuthorityDiscoveryApi_authorities`.

TODO: define the format

An **authority-discovery record** is defined as the encoding of the `SignedAuthorityRecord` message in the following protobuf definition file:

```protobuf
syntax = "proto3";

package authority_discovery_v2;

message AuthorityRecord {
	repeated bytes addresses = 1;
}

message PeerSignature {
	bytes signature = 1;
	bytes public_key = 2;
}

message SignedAuthorityRecord {
	bytes record = 1;
	bytes auth_signature = 2;
	PeerSignature peer_signature = 3;
}
```

Where:

- The `record` field of `SignedAuthorityRecord` is the encoding of the `AuthorityRecord` protobuf message.
- Each element of the `addresses` field must end with `/p2p/...`. TODO define better
- The `public_key` field of the `peer_signature` is in the following format:

| Field name         | Type      | Size (bytes)   |
| ------------------ | --------- | -------------- |
| (constant) | 0x08011220 | 4 |
| [Ed25519](https://www.rfc-editor.org/rfc/rfc8032.txt) Public key | Bytes | 32 |

- The *public key* of the *PeerId*s at the end of `addresses`, and the *public key* of `peer_signature` must all be identical.

An *authority-discovery record* is valid if:

- The `auth_signature` is a valid cryptographic signature of the payload `record` with the public key of the authorit-discovery authority. TODO unclear
- The `signature` in `PeerSignature` is a valid cryptographic signature of the payload `record` with the `PeerId`.
