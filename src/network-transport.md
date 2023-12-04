# Transport protocol

A host SHOULD listen for incoming TCP connections on one or more TCP ports reachable from the Internet.

TODO: Noise + Yamux

## PeerId

A **PeerId** is defined as:

| Field name         | Type      | Size (bytes)   |
| ------------------ | --------- | -------------- |
| (constant) | Bytes 0x002408011220 | 6 |
| [Ed25519](https://www.rfc-editor.org/rfc/rfc8032.txt) public key | Bytes | 32 |

> **Note**: A **PeerId** used to be defined as a multihash encoding of a protobuf struct containing another protobuf struct containing the public key. In order to prevent the same public key from being representable as multiple different **PeerId**s, all implementations must encode this in a single consistent way that is more easily defined as specific bytes.

> **Note**: A **PeerId** is usually displayed to programmers and end users as the *base58* encoding of its bytes.

## multistream-select

The **multistream-select** protocol is a protocol allowing to negotiate a protocol.

## Noise

The **Noise** protocol is the the Noise protocol defined by <https://noiseprotocol.org/noise.html> and using the XX handshake.
TODO libp2p handshake

## Yamux

TODO: https://github.com/hashicorp/yamux/blob/master/spec.md

## Supported protocols

- TCP/IP
- TCP/IP and WebSocket (secure or not)
