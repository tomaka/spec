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

A **trie node value** is defined as the concatenation of the following bytes:

- TODO

The **Merkle value of a trie node** is:

- If the **trie node value** is less than 32 bytes, the trie node value itself.
- If the **trie node value** is 32 bytes or more, then the 32 bytes [BLAKE2](https://datatracker.ietf.org/doc/html/rfc7693) hash of the trie node value.
