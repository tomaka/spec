# Blocks finalization

## Grandpa validators set

Given a *block header*, the **Grandpa validators set** can be determined through the following process:

TODO

TODO: don't forget to remove from the list disabled validators

## Finalizable block

A *block header* is **finalizable** if:

- Its parent block is *finalized* or *finalizable*.
- TODO: parachain stuff

## Ghost

Given a set of *block headers* and a *threshold*, the **ghost** of this set is the highest block that has a supermajority of `threshold * 2 / 3 + 1`.
If the number of *block headers* is too small, the ghost is undefined.

> **Note**: As more block headers are added, the ghost can only ever move to higher blocks.

TODO explain better

## Grandpa round estimate

Given a set of prevote *block headers*, a set of precommit *block headers* and a *threshold*, the **estimate** is the block inferior or equal to `ghost(prevotes)` that can potentially achieve a supermajority of `threshold * 2 / 3 + 1`.

TODO: define more formally

> **Note**: As more precommit block headers are added to the set, the estimate can only ever move to lower blocks. As more prevote block headers are added to the set, the estimate can only ever move to higher blocks.

> **Note**: More informally, the estimate is the highest block that could still achieve a supermajority.

## Grandpa round

Given a set of *block headers* and a threshold, the **estimate** is the highest block that can achieve a supermajority.

Given a set of prevote *block headers*, a set of precommit *block headers*, and a *threshold*, a round is completable if either:

- The **estimate** is strictly inferior to the ghost of the prevotes.
- It becomes impossible for the **estimate** to become strictly superior to the ghost of the prevotes.

