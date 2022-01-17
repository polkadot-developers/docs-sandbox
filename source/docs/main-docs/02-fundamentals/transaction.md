The transaction pool contains all [signed](/v3/concepts/extrinsics#signed-transactions) and [unsigned](/v3/concepts/extrinsics#unsigned-transactions) transactions broadcasted to the network that have been received and validated by the local node.

-- what is a tx made of?
-- what makes a tx valid?

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




## Priority 
Closes #5672 
Closes #9317 

Alternative take on #9596
(we can leave the adjuster from previous PR, but I don't think it is going to be useful at all, given no extensions change `priority`).

This PR changes the way Transaction Priority is calculated by following `SignedExtensions`:
1. `CheckWeight`
2. `ChargeTransactionPayment`

For `CheckWeight` it removes the `priority` calculation completely, leaving the priority as `0` for all dispatch classes. Please note that this might cause a bit of an havoc on existing chains without any extra signed extension that would alter `priority` (which is discouraged anyway) -  all transactions would end up having the same priority (`0`) and the inclusion might be done at random (i.e. depending on the transaction pool ordering - currently based on the insertion time), note that the requirements (i.e. nonce) would still be respected though.

`ChargeTransactionPayment` is completely reworked and is now a single place responsible for calculating the overall `priority` (all other `SignedExtensions` in this repo do not alter it).
The priority depends on the dispatch class and the amount of tip-per-weight or tip-per-length (whatever is more limiting) the sender is willing to pay.
Transactions without a `tip` are using a minimal tip value of `1` for `priority` calculations to make sure we don't end up with all transactions with `0` priority. The consequence of this is that "smaller" transactions are more preferred than the "large" ones (note the behavior used to be similar with `CheckWeight`'s prioriy). 

`Operational` transactions are getting additional priority bump - not as high as previously (3/4 * u64::max), because that prevents other transactions from getting in front of them in the pool, but now equal to the `OperationalVirtualTip` amount configurable by a rutnime developer.

Companion PR: https://github.com/paritytech/polkadot/pull/3901

## Transaction encoding

See: https://github.com/substrate-developer-hub/substrate-developer-hub.github.io/pull/674/files


## Transaction lifecycle

## Transaciton pool logic 

What's the life cycle of a transaction? Which transactions are propagated?

see:  https://docs.substrate.io/rustdocs/latest/sp_runtime/traits/trait.SignedExtension.html#method.validate


