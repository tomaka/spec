# State

The so-called **state** is made of one or more **trie**s:

- One **main trie**.
- Zero or more **child tries**.

## Trie

A **trie** is defined as a (potentially empty) set of *keys*.
Each key has a (potentially empty) value associated to it.

The **Merkle value of a trie** is defined as:

- If the trie is not empty, the **Merkle value of the trie node** that serves as the common ancestor of all other trie nodes.
- If the trie is empty, it is equal to the hash of TODO

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
If the 4 most significant bits of *Header* are equal to `0000`, then the value *Header* must be equal to `0`.

The *Partial key length in header* is defined as:

- If the two most significant bits of *Header* are equal to `01`, `10`, or `11`, it is equal to the 6 least significant bits of *Header*.
- If the three most significant bits of *Header* are equal to `001`, it is equal to the 5 least significant bits of *Header*.
- If the four most significant bits of *Header* are equal to `0001`, it is equal to the 4 least significant bits of *Header*.
- If the *Header* is equal to `0`, is it equal to `0`.

The *Partial key extra length* is defined as:

- If the *Partial key length in header* is equal to `63`, it is a serie of zero or more bytes equal to `255` followed with one byte that is not equal to `255`.
- If the *Partial key length in header* is not equal to `63`, it is empty (the field is not present).

The *Partial key length* is defined as the sum of each byte of *Partial key extra length*, plus the value of *Partial key length in header*.

The *Partial key* is defined as:

- If *Partial key length* is an even number, a list of bytes whose length is *Partial key length* divided by 2.
- If *Partial key length* is an uneven number, a list of bytes whose length is (1 plus *Partial key length*) divided by 2. The four most significant bits of the first byte must be equal to `0000`.

*Children bitmap* is present only if the most significant bit of *Header* is `1`, or if the four most significant bits of *Header* are `0001`. If *Children bitmap* is missing, it implicitly has a value of 0.

*Storage value* is present only if the second most significant bit of *Header* is `1`, or if the three most significant bits of *Header* are `001`, or if the four most significant bits of *Header* are `0001`.

If the second most significant bit of *Header* is `1`, then the *Storage value version* is `0` and the *Storage value* field is defined as:

| Field name         | Type      | Size (bytes)   |
| ------------------ | --------- | -------------- |
| Storage value size | SCALE-compact-encoded unsigned integer | (variable) |
| Storage value | Bytes | *Storage value size* |

If the three most significant bits of *Header* are `001`, or if the four most significant bits of *Header* are `0001`, then the *Storage value version* is `1` and the *Storage value* field is defined as:

| Field name         | Type      | Size (bytes)   |
| ------------------ | --------- | -------------- |
| Storage value | Bytes | 32 |

Each of the 16 elements of *Children* corresponds to a bit in *Children bitmap*. The first child corresponds to the least significant bit, while the last child corresponds to the most significant bit.

For each element of *Children*, if the corresponding bit in *Children bitmap* is `0`, then it is empty. Otherwise, it is defined as:

| Field name         | Type      | Size (bytes)   |
| ------------------ | --------- | -------------- |
| Child Merkle value size | SCALE-compact-encoded unsigned integer | (variable) |
| Child Merkle value | Bytes | *Child Merkle value size* |

*Child Merkle value size* is always inferior or equal to 32.

### Merkle value

The **Merkle value of a trie node** is:

- If the **trie node value** is less than 32 bytes, the trie node value itself.
- If the **trie node value** is 32 bytes or more, then the 32 bytes [BLAKE2](https://datatracker.ietf.org/doc/html/rfc7693) hash of the trie node value.
