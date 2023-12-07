# State

## Trie

A **trie** is defined as:

- A *hash algorithm*
- A possibly empty tree of nodes.

Each **node** is defined as:

- A *partial key* made of 4 bits **nibbles**.
- Between zero and sixteen children nodes.
- Optionally, a **storage value** and a **storage value version** (either both present or both absent). The storage value version is equal to either `0` or `1`.

Nodes with exactly one child and no storage value are invalid.

Any given *trie* is either empty, or has one *node* with no parent and a set of nodes whose parent is also in the trie. Cycles are forbidden.

The **key** of a node is defined as the concatenation of the key of the parent, the index of the node within its parent's children list, and the partial key of the node.

The **Merkle value** of a trie is defined as:

- If the trie is not empty, it is equal to the hash of the *trie node value* of the trie node that serves as the common ancestor of all other trie nodes.
- If the trie is empty, it is equal to the hash of a trie node value whose *Header* is equal to `0`.

> **Note**: The Merkle value of a BLAKE2 empty trie is always equal to `0x03170a2e7597b7b7e3d84c05391d139a62b157e78786d8c082f29dcf4c111314`. The Merkle value of a Keccak256 empty trie is `0xbc36789e7a1e281436464229828f817d6612f7b477d66591ff96a9e064bcc98a`.

## State

The **state** is composed of:

- One main *trie* whose hash algorithm is [BLAKE2](https://datatracker.ietf.org/doc/html/rfc7693).
- Zero or more child *trie*s whose hash algorithm is [BLAKE2](https://datatracker.ietf.org/doc/html/rfc7693).

TODO: talk about the Merkle value of child tries within the main trie

## Trie node value and Merkle value

### Trie node value

A **trie node value** is defined as:

| Field name         | Type      | Size (bytes)   |
| ------------------ | --------- | -------------- |
| Header | (see below) | 1 |
| Partial key extra length | (see below) | (variable) |
| Partial key | (see below) | (variable) |
| Children bitmap | (optional) Little endian unsigned integer | 0 or 2 |
| Storage value | (see below) | (variable) |
| Children | (see below) repeated 16 times | (variable) |

Where:

*Header* is one byte that is used to determine the format of the rest of the trie node value.
If the 4 most significant bits of *Header* are equal to `0000`, then the value of all the bits of *Header* must be equal to `0`.

The *Partial key length in header* is defined as:

- If the two most significant bits of *Header* are equal to `01`, `10`, or `11`, it is equal to the 6 least significant bits of *Header*.
- If the three most significant bits of *Header* are equal to `001`, it is equal to the 5 least significant bits of *Header*.
- If the four most significant bits of *Header* are equal to `0001`, it is equal to the 4 least significant bits of *Header*.
- If the *Header* is equal to `0`, is it equal to `0`.

The *Partial key extra length* is defined as:

- If the *Partial key length in header* is equal to `63`, it is a serie of zero or more bytes equal to `255` followed with one byte that is not equal to `255`.
- If the *Partial key length in header* is not equal to `63`, it is empty (the field is not present).

The *Partial key length* is defined as the sum of the value of each byte of *Partial key extra length*, plus the value of *Partial key length in header*.

The *Partial key* is defined as:

- If *Partial key length* is an even number, a list of bytes whose length is *Partial key length* divided by 2.
- If *Partial key length* is an uneven number, a list of bytes whose length is (1 plus *Partial key length*) divided by 2. The four most significant bits of the first byte must be equal to `0000`.
TODO: convert to nibbles

*Children bitmap* is present only if the most significant bit of *Header* is `1`, or if the four most significant bits of *Header* are `0001`. If *Children bitmap* is missing, it implicitly has a value of `0`.

*Storage value* is present only if the second most significant bit of *Header* is `1`, if the three most significant bits of *Header* are `001`, or if the four most significant bits of *Header* are `0001`.

The *Storage value version* is defined as:

- `0` if the second most significant bit of *Header* is `1`.
- `1` if the three most significant bits of *Header* are `001`, or if the four most significant bits of *Header* are `0001`.
- Undefined otherwise.

If the *Storage value version* is `0`, then the *Storage value* field is defined as:

| Field name         | Type      | Size (bytes)   |
| ------------------ | --------- | -------------- |
| Storage value size | SCALE-compact-encoded unsigned integer | (variable) |
| Storage value | Bytes | *Storage value size* |

If the *Storage value version* is `1`, then the *Storage value* field is defined as:

| Field name         | Type      | Size (bytes)   |
| ------------------ | --------- | -------------- |
| Storage value | Bytes | 32 |

Each of the 16 elements of *Children* corresponds to a bit in *Children bitmap*. The first child corresponds to the least significant bit, the second child corresponds to the second least significant bit, and so on. The last child corresponds to the most significant bit.

For each element of *Children*, if the corresponding bit in *Children bitmap* is `0`, then it is empty. Otherwise, it is defined as:

| Field name         | Type      | Size (bytes)   |
| ------------------ | --------- | -------------- |
| Child Merkle value size | SCALE-compact-encoded unsigned integer | (variable) |
| Child Merkle value | Bytes | *Child Merkle value size* |

*Child Merkle value size* is always inferior or equal to 32.
