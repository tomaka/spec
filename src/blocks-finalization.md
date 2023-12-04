# Blocks finalization

## Grandpa validators set

Given a *block header*, the **Grandpa validators set** can be determined through the following process:

TODO

TODO: don't forget to remove from the list disabled validators

## Finalizable block

A *block header* is **finalizable** if:

- Its parent block is *finalized* or *finalizable*.
- TODO: parachain stuff

## Grandpa rounds

Given a set of prevote *block headers*, a set of precommit *block headers*, and a *number of validators*:

The **threshold** is defined as `number of validators * 2 / 3 + 1`.

The **prevotes ghost** is defined as the highest block header in the prevotes that has a supermajority of the threshold.
If the number of prevote *block headers* is too small, the prevotes ghost is undefined.
TODO explain better

> **Note**: As more block headers are added, the prevotes ghost can only ever move to higher blocks.

The **estimate** is defined as the block inferior or equal to the *prevotes ghost* that can potentially achieve a supermajority of the threshold.
TODO: define more formally

> **Note**: More informally, the estimate is the highest block that could still potentially achieve a supermajority.

> **Note**: As more precommit block headers are added to the set, the estimate can only ever move to lower blocks. As more prevote block headers are added to the set, the estimate can only ever move to higher blocks.

A round is **completable** if either:

- The *estimate* is strictly inferior to the *prevotes ghost*.
- It becomes impossible for the *estimate* to become strictly superior to the *prevotes ghost*.

The **updated finalized block** is defined as:

- 
