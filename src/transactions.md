# Transactions

A **transaction** is defined as an opaque list of bytes.

An implementation should not try to interpret the format of a transaction.

## Validation

Given a *block*, a transaction can be **validated** against this particular block by following these steps:

- Check the version of the `TaggedTransactionQueue` API in the runtime specification. If the version is not equal to either `2` or `3`, the validation fails.
- If the API version is equal to `2`, perform a *runtime call* of the function `Core_initialize_block`. TODO more precise
- Perform a *runtime call* of the function `TaggedTransactionQueue_validate_transaction`. TODO more precise

TODO

## Transactions ordering

TODO: explain relationship with blocks

Given two validated transactions A and B, the **ordering** of A and B is defined as:

- If the intersection of A's `provides` tags list with B's `requires` tags is non-empty, then B must come **after** A. TODO: what if A's requires overlaps with B's provides at the same time?
- Otherwise, if A's `priority` is strictly superior to B's `priority`, then B must come **after** A.
- Otherwise, A and B are **equal**.
