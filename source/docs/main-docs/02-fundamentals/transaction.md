Transactions in Substrate can be considered as any piece of data to be included in a block.
They can be one of 3 distinct types, all of which fall under a broader category called  "extrinsics"-i.e. any information that originates from _outside_ a runtime.
These are:

1. **Signed transactions**: must contain the signature of an account sending an inbound request to execute some runtime call.
  With a signed transaction, the account used to submit the request typically pays a transaction fee and must sign it using the account's private key.
1. **Unsigned transactions**: do not identify the sender of the inbound request and do not require any signature.
  With an unsigned transaction, there's no economic deterrent to prevent spam or replay attacks, so custom logic is required to protect the network from these types of transactions being misused.
1. **Inherents**: are a special type of unsigned transaction made by block authors which carry information required to build a block such as timestamps, storage proofs and uncle blocks.

## Transaction lifecylce

Understanding how transactions are formatted, validated and executed provides an important foundation on the design of the transaction types and behaviors available in Substrate.

[ maybe diagram here ? ]
### Transactions in a block produced locally

If a transaction is included in a block produced by the local node, the transaction lifecycle follows a path like this:

1. Some node listens for transactions on the network.
1. The node receives a JSON-RPC request for a signed transaction.
1. The transaction gets formatted in the runtime, with its corresponding function call, signature, nonce, tip and additional information.
1. Some runtime API is used for the client to verify that the transaction meets its requirements and is valid.
1. Only if the transaction is valid will it be put in a transaction queue ready for block authors.
1. Once in the ready queue, the same runtime API is used to order the transaction according to some priority factor.
1. The node uses transactions in this queue to construct a block and propagate it to peers over the network.

Notice that transactions are not removed from the ready queue when blocks are authored, but removed _only_ on block import.
This is due to the possibility that a recently-authored block might not make it into the canonical chain.

### Transactions in a block received from network

1. The node receives notification of the new block.
1. Other nodes also construct this block and publish it to the network.
1. A 2/3 majority of nodes reach consensus that this block is part of the canonical chain.
1. All other nodes on the network receive and build the block.
1. All transactions in that block are executed and state changes are updated in runtime storage.

## Transaction formats

The way the format of an extrinsic is desgined in Substrate takes into account the metadata an extrinsic can expose as well as any additional information required to verify that a transaction is valid.
In order to expose certain checks required for an extrinisic to be valid and correctly constructed, extrinsics are formatted by the runtime as either unchecked, checked or opaque. 

- Unchecked: signed transactions that require some validation check before they can be accepted in the transaction pool.
Any unchecked extrinsic contains the signature for the data being sent plus some extra data.
- Checked: inherent extrinsics which by definition don't require signature verification. 
Instead, they carry information on where the extrinsic comes from and some extra data.
- Opaque: used for cases when an extrinsic hasn't yet been committed to a format but can still be decoded. 

Extra data can be any additional information that would be useful to attach to a transaction or inherent.
For example, the nonce of the transaction, or the tip for the block author.
This information is provided by a specialized extensions that help determine the validity and ordering of extrinsics before they get included in a block.

A signed transaction call could look like:

```rust
node_runtime::UncheckedExtrinsic::new_signed(
		function.clone(),                                      // some call
		sp_runtime::AccountId32::from(sender.public()).into(), // some sending account
		node_runtime::Signature::Sr25519(signature.clone()),   // the account's signature
		extra.clone(),                                         // the signed extensions 
	)
```

Substrate defines these transaction formats generically to allow developers to implement custom ways to define valid transactions. 
Skip to [transaction construction]() to learn more about how a transaction is constructed in a FRAME runtime.

## Validating and queuing transactions

As discussed in [Consensus](), a majority of nodes in the network must agree on the state of the blockchain to continue securely adding blocks.
To reach consensus, two-thirds of the nodes must agree on the order of the transactions executed and the resulting state change. 
To prepare for consensus, transactions are first validated and queued on the local node in a **transaction pool**.
This is where all signed and unsigned transactions that have been received by a local node are placed before they get broadcast to the rest of the network.
Before including a transaction in a block, nodes must first determine which transactions are valid and in what order of priority it should be included.

### Validating transactions

A FRAME runtime defines rules that determine the validity of any transaction type. 
It translates this information by tagging transactions with special fields to help nodes determine whether a transaction is valid or not.
The tag system gives nodes just enough information to know whether a transaction provides the data required by the runtime.

Using these runtime-defined rules, the local node's transaction pool checks that the transactions received meets specific conditions and whether it should be included in a block or not.

For example, the transaction pool might perform the following checks to determine whether a transaction meets certain requirements:

* Is the transaction index—also referred to as the transaction nonce—correct?
* Does the account used to sign the transaction have enough funds to pay the associated fees?
* Is the signature used to sign the transaction valid?

After the initial validity check, the transaction pool periodically checks whether existing transactions in the pool are still valid.
If a transaction is found to be invalid or has expired, it is dropped from the pool.

The transaction pool only deals with the validity of the transaction and the ordering of valid transaction that are placed in a transaction queue.
All other transaction details—including handling for fees, accounts, or signatures—are defined by the runtime using the `validate_transaction` function.
For more detailed information about validating transactions, see the [`validate_transaction`](/rustdocs/latest/sp_transaction_pool/runtime_api/trait.TaggedTransactionQueue.html#method.validate_transaction) runtime API method.

### Adding valid transactions to a transaction queue

If a transaction is identified as valid, the transaction pool moves the transaction into a transaction queue. 
There are two transaction queues for valid transactions.

* The **ready queue**: contains transactions that can be included in a new pending block.
  For runtimes built with FRAME, the transactions must follow the exact order in the ready queue.

* The **future queue**: contains transactions that may become valid in the future.
  For example, a transaction may have a `nonce` that is too high for its account.
  This transaction will wait in the future queue until the valid preceding transactions are included in the chain, after which it will be either dropped or reconsidered.
  
It's possible to design a custom runtime to remove the transaction ordering requirements.
However, a runtime without strict transaction ordering would allow full nodes to implement different strategies for propagating transactions and including them in blocks. 
### Invalid transactions

The runtime will return an `Invalid` error when a transaction cannot be added to a block at all.
Transactions could be invalid for one of the following reasons:

- The transaction was already included in a block (`Stale`).
- The transaction's signature is invalid (`BadProof`).
- The transaction does not fit in the current block (`ExhaustsResources`). 
This implies that it could be valid for the next one.

## Transaction dependency and priority

If a node is the next block author, it will order transactions from high to low priority for the next block until it reaches the block's weight or length limit.

Transaction priority is calculated in the runtime and provided to the client as a tag on the transaction.
In a FRAME runtime, a special pallet is used to calculate priority based on the weights and fees associated with the transaction.
This priority calculation applies to all types of transactions with the exception of inherents, which are always placed first using the [`EnsureInherentsAreFirst`](https://docs.substrate.io/rustdocs/latest/frame_support/traits/trait.EnsureInherentsAreFirst.html) trait.

For any two or more transactions that have their dependencies satisfied, the ordering is calculated by taking into account the fees that the transaction will pay and what dependency on other transactions it contains.

Some scenarios:

- 1 unsigned transaction with `TransactionPriority::max_value()` and some other signed transaction: the unsigned transaction will be at the top of the queue.
- 2 transactions from _different_ senders (with `nonce=0`): `priority` is needed to determine which transaction is more important and should be included in the block first. 
- 2 transactions from the _same_ sender with an identical `nonce`: only one transaction can be included in the block, so only the transaction with the higher fee will be put in the transaction pool.

## Extending transactions

Substrate provides the concept of **signed extensions** to extend an extrinsic with additional data, provided by the [`SignedExtension`](/rustdocs/latest/sp_runtime/traits/trait.SignedExtension.html) trait.

The transaction queue regularly calls signed extensions to keep checking that a transaction is valid before it gets put in the ready queue. 
This is a useful safeguard for verifying that transactions won't fail in a block.
They are commonly used to enforce some validation, spam and replay protection logic needed by the transaction pool.

By default in FRAME, a signed extension can hold any of the following types:

- `AccountId`: to encode the sender's identity.
- `Call`: to encode the pallet call to be dispatched. This data is used to calculate transaction fees.
- `AdditionalSigned`: to handle any additional data to go into the signed payload. This makes it possible to attach any custom logic prior to dispatching a transaction.
- `Pre`: to encode the information that can be passed from before a call is dispatch to after it gets dispatched.

FRAME's [system pallet](todo) provides a set of [useful `SignedExtensions`](https://docs.substrate.io/rustdocs/latest/frame_system/index.html#signed-extensions) out of the box.

### Practical examples

An important signed extension for validating transactions is `CheckSpecVersion`.  
It provides a way for the sender to provide the spec version as a signed payload attached to the transaction.
Since the spec version is already known in the runtime, the signed extension can perform a simple check to verify that the spec versions match.
If they don't, the transaction fail before it gets put in the transaction pool.

Other examples include the signed extensions used to calculate transaction priority.
These are:

- `CheckWeight`: sets the value for priority to `0` for all dispatch classes.
- `ChargeTransactionPayment`: calculates the overall priority, modifying the priority value accordingly.

The priority depends on the dispatch class and the amount of tip-per-weight or tip-per-length (whatever is more limiting) the sender is willing to pay.
Transactions without a tip use a minimal tip value of `1` for priority calculations to make sure that not all transactions end up having a priority of `0`. 
The consequence of this is that "smaller" transactions are preferred over "larger" ones.

<!-- TODO: consider moving elsewhere to more procedural / practical material.

## Transaction encoding 

Any type of exstrinsic uses SCALE codec to encode and decode transactions.

Here's an example of a decoded transaction:

[ TODO ]

Notice how it's constructed:
- `Call`: 
- `Address`: 
- `Extra`: this shows all the signed extensions for this transaction.
- `Additional Signed`: this contains the spec version
- `Raw Payload`:  

See: https://github.com/substrate-developer-hub/substrate-developer-hub.github.io/pull/674/files
 -->

## Learn more

- Learn about the origin system for different extrinsic types
- Learn more about how transactions are encoded
- Learn how to configure transaction fees for your chain
- Learn about how block execution works