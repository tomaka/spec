# Transport protocol

A host SHOULD listen for incoming TCP connections on one or more TCP ports reachable from the Internet.

TODO: Noise + Yamux

## PeerId

A **PeerId** is defined as the concatenation of the bytes `0x002408011220` with a 32 bytes [ed25519](https://www.rfc-editor.org/rfc/rfc8032.txt) public key.

> **Note**: A **PeerId** used to be defined as a multihash encoding of a protobuf struct containing another protobuf struct containing the public key. In order to prevent the same public key from being representable as multiple different **PeerId**s, all implementations must encode this in a single consistent way.

## multistream-select

The **multistream-select** protocol is a protocol allowing to negotiate a protocol.

## Noise

The **Noise** protocol is the the Noise protocol defined <https://noiseprotocol.org/noise.html> and using the XX handshake.
TODO libp2p handshake

## Supported protocols

- TCP/IP
- TCP/IP and WebSocket (secure or not)
