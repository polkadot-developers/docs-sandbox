---
title: Transaction Pool
slug: /v3/concepts/tx-pool
version: '3.0'
section: docs
category: concepts
keywords: transaction, pool, order, ordering, sorting, validity
---

## Validity 
[content not used / to remove] 
It does this check in isolation, so it will not catch errors such as the same output being spent twice.
`validate_transaction` does not check whether calls to a pallet will succeed.

[ todo : put this in architecture? or design? ]
It's possible to design a custom runtime to remove the strict ordering requirements of the queue.
This would allow full nodes to implement different strategies on transaction propagation and block inclusion.

## Further Reading

- [Extrinsics](/v3/concepts/extrinsics)
- [Transaction Fees](/v3/runtime/weights-and-fees)
