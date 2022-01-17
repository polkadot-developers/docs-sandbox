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

## Signed extensions

[`SignedExtension`](/rustdocs/latest/sp_runtime/traits/trait.SignedExtension.html) is a trait by which a transaction can be extended with additional data or logic.
Signed extensions are used anywhere you want some information about a transaction prior to execution.
This is heavily used in the transaction pool.

The runtime can use some of this data, for example the `Call` that will be dispatched, to calculate transaction fees.
Signed extensions also include an `AdditionalSigned` type that can hold any encodable data, and therefore allow you to perform any custom logic prior to including or dispatching a transaction.
The transaction queue regularly calls functions from `SignedExtension` to validate transactions prior to block construction to avoid including transactions that will fail in blocks.

Despite the name, `SignedExtension` can also be used to verify [unsigned transactions](/v3/concepts/extrinsics#unsigned-transactions).
The `*_unsigned` set of methods can be implemented to encapsulate validation, spam, and replay protection logic that is needed by the transaction pool.

- [Signed Extension API](/rustdocs/latest/sp_runtime/traits/trait.SignedExtension.html)

## Further Reading

- [Extrinsics](/v3/concepts/extrinsics)
- [Transaction Fees](/v3/runtime/weights-and-fees)
