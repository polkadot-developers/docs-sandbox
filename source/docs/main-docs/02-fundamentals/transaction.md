The transaction pool contains all [signed](/v3/concepts/extrinsics#signed-transactions) and [unsigned](/v3/concepts/extrinsics#unsigned-transactions) transactions broadcasted to the network that have been received and validated by the local node.

The [`ValidTransaction` struct](/rustdocs/latest/sp_runtime/transaction_validity/struct.ValidTransaction.html) defines two parameters used to determine the ordering of transactions in the pool: `requires` and `provides`. 
Together they create a dependency graph which allows the pool to produce a valid linear ordering of transactions.

Substrate supports multiple `provides` and `requires` tags, so custom runtimes can create alternate dependency (ordering) schemes.

## Requirements

Every signed transaction needs to contain a nonce, which is incremented by 1 every time a new transaction is made from that account.
For example, the first transaction from a new account will have `nonce = 0` and the second transaction will have `nonce = 1`.

At minimum, FRAME transactions have a `provides` tag of `encode(sender ++ nonce)` and a `requires` tag of `encode(sender ++ (nonce -1)) if nonce > 1`.
Transactions do not require anything if `nonce=0`.
As a result, all transactions coming from a single sender will form a sequence in which they should be included.

## Validity

The validity of a transaction is verified by checks defined in the runtime. 
Before a transaction is propagated to other nodes, it is "tagged" and checked for whether:

- The signature is valid.
- The account has enough funds to pay for the associated fees.
- The transaction index (nonce) is correct.

The transaction pool also regularly checks for transaction validity of existing transactions within the pool.
Every transaction is defined in the runtime using the [`ValidTransaction`](https://docs.substrate.io/rustdocs/latest/sp_runtime/transaction_validity/struct.ValidTransaction.html) struct.
A transaction will be dropped from the pool if it is found to be invalid or expired, for example:

- If another transaction contains the same transaction dependencies (see: [`requires`](https://docs.substrate.io/rustdocs/latest/sp_runtime/transaction_validity/struct.ValidTransaction.html#structfield.requires) and [`TransactionTag`](https://docs.substrate.io/rustdocs/latest/sp_runtime/transaction_validity/type.TransactionTag.html)).
- If the transaction has exceeded the [`TransactionLongevity`](https://docs.substrate.io/rustdocs/latest/sp_runtime/transaction_validity/type.TransactionLongevity.html#), i.e. the amount of blocks it can remain valid for.

Other causes for invalid transactions are listed [here](https://docs.substrate.io/rustdocs/latest/sp_runtime/transaction_validity/enum.InvalidTransaction.html).

The [`TaggedTransactionQueue`](https://docs.substrate.io/rustdocs/latest/sp_transaction_pool/runtime_api/trait.TaggedTransactionQueue.html) runtime API provides a [`validate_transaction`](/rustdocs/latest/sp_transaction_pool/runtime_api/trait.TaggedTransactionQueue.html#method.validate_transaction) method whose job is to verify the correctness of the transaction against the current state (i.e. the signature and nonce).
The function will be called frequently, potentially multiple times for the same transaction.
In doing so, it provides the necessary information for the transaction pool to order and prioritize transactions.
It is also possible for `validate_transaction` to reject a dependent transaction that would pass `execute_block` if it were executed in the correct order.

## Ordering 

If the transaction is valid, the transaction queue sorts transactions into two groups:

- *Ready Queue*: Contains transactions that can be included in a new pending block.
  For runtimes built with FRAME, the transactions must follow the exact order in the ready queue.
- *Future Queue*: Contains transactions that may become valid in the future.
  For example, a transaction may have a `nonce` that is too high for its account.
  This transaction will wait in the future queue until the preceding transactions are included in the chain.

## Transaction priority

Transaction `priority` in the `ValidTransaction` struct determines the ordering of transactions that are in the ready queue.
If a node is the next block author, it will order transactions from high to low priority in the next block until it reaches the weight or length limit of the block.

`priority` defines the linear ordering of a graph in the case of one transaction unlocking multiple dependent transactions.
For example, if we have two (or more) transactions that have their dependencies satisfied, then we use `priority` to choose the order for them.

For runtimes built with [FRAME](/v3/runtime/frame), `priority` is defined as the `fee` that the transaction is going to pay.
For example:

- If we receive 2 transactions from _different_ senders (with `nonce=0`), we use `priority` to
  determine which transaction is more important and included in the block first.
- If we receive 2 transactions from the _same_ sender with an identical `nonce`, only one
  transaction can be included on-chain. We use `priority` to choose the transaction with a higher
  `fee` to store in the transaction pool.

Note that the pool does not know about fees, accounts, or signatures &mdash; it only deals with the validity of the transaction and the abstract notion of the `priority`, `requires`, and `provides` parameters.
All other details are defined by the runtime via the `validate_transaction` function.

## Transaction lifecycle

A transaction can follow two paths:

### Block produced by our node

1. Our node listens for transactions on the network.
1. Each transaction is verified and valid transactions are placed in the transaction pool.
1. The pool is responsible for ordering the transactions and returning ones that are ready to be included in the block.
   Transactions in the ready queue are used to construct a block.
1. Transactions are executed and state changes are stored in local memory.
   Transactions from the `ready` queue are also propagated (gossiped) to peers over the network.
   We use the exact ordering as the pending block since transactions in the front of the queue have a higher priority and are more likely to be successfully executed in the next block.
1. The constructed block is published to the network.
   All other nodes on the network receive and execute the block.

Notice that transactions are not removed from the ready queue when blocks are authored, but removed _only_ on block import.
This is due to the possibility that a recently-authored block may not make it into the canonical chain.

### Block received from network

The block is executed and the entire block either succeeds or fails.

## Extending transactions

The [`SignedExtension`](/rustdocs/latest/sp_runtime/traits/trait.SignedExtension.html) trait is used to extend a signed or unsigned transaction with additional data or logic.
For example, if you wanted to attach some information about a transaction prior to execution you could implement a custom `SignedExtension` to do so.

`SignedExtension` are commonly used to prepare transactions for the transaction pool and use the data attached to the transaction for handling their transaction pool logic.

By default, they can hold any of the following types:
- `AccountId`: to encode the sender's identity.
- `Call`: to encode the pallet call to be dispatched. This data is used to calculate transaction fees.
- `AdditionalSigned`: to handle any additional data to go into the signed payload. This makes it possible to attach any custom logic prior to dispatching a transaction.
- `Pre`: to encode the information that can be passed from before a call is dispatch to after it gets dispatched.

The transaction queue regularly calls functions from `SignedExtension` to validate transactions prior to block construction. 
This is useful for verifying that transactions don't fail in a block.

FRAME's [system pallet](todo) provides a set of [useful `SignedExtensions`](https://docs.substrate.io/rustdocs/latest/frame_system/index.html#signed-extensions) out of the box.


`SignedExtension` can also be used to verify [unsigned transactions](/v3/concepts/extrinsics#unsigned-transactions).
The `*_unsigned` set of methods can be implemented to encapsulate validation, spam, and replay protection logic that is needed by the transaction pool.


## Priority calculation

The following `SignedExtension`s are used to calculate transaction priority using the `priority` parameter.

- `CheckWeight`: the `priority` is set to `0` for all dispatch classes.
- `ChargeTransactionPayment`: calculates the overall priority, modifying the `priority` parameter.

The priority depends on the dispatch class and the amount of tip-per-weight or tip-per-length (whatever is more limiting) the sender is willing to pay.

Transactions without a `tip` are using a minimal tip value of `1` for `priority` calculations to make sure we don't end up with all transactions with `0` priority. 
The consequence of this is that "smaller" transactions are more preferred than the "large" ones.

`Operational` transactions are getting additional priority bump (3/4 * `u64::max()`), because that prevents other transactions from getting in front of them in the pool, but now equal to the `OperationalVirtualTip` amount configurable by a runtime developer.

## Transaction encoding

[ TODO ]
See: https://github.com/substrate-developer-hub/substrate-developer-hub.github.io/pull/674/files



