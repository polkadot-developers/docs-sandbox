Transactions in Substrate can be considered as any piece of data to be included in a block.
They can be one of 3 distinct types, all of which fall under a broader category called  "extrinsics"-i.e. any information that originates from _outside_ a runtime.
These are:

* **Signed transactions**: must contain the signature of an account sending an inbound request to execute some runtime call.
  With a signed transaction, the account used to submit the request typically pays a transaction fee and must sign it using the account's private key.
* **Unsigned transactions**: do not identify the sender of the inbound request and do not require any signature.
  With an unsigned transaction, there's no economic deterrent to prevent spam or replay attacks, so custom logic is required to protect the network from these types of transactions being misused.
* **Inherents**: are a special type of unsigned transaction made by block authors which carry information required to build a block such as timestamps, storage proofs and uncle blocks.

## Transaction lifecylce

Understanding how signed and unsigned transactions are formatted, validated and executed provides an important foundation on the design of the extrinsic types and behaviors available in Substrate.
Any signed or unsigned transaction that's sent to a [non-authoring node]() will just be gossiped to other nodes in the network and enter their transaction pool until it is received by an authoring node.

[ maybe diagram here ? ]

### Extrinsics in a block produced locally

If a signed or unsigned transaction is included in a block produced by the local node, its lifecycle follows a path like this:

1. Some node listens for transactions on the network.
1. The node receives a JSON-RPC request for a signed transaction.
1. The transaction gets formatted in the runtime, with its corresponding function call, signature, nonce, tip and additional information.
1. Some runtime API is used for the client to verify that the transaction meets its requirements and is valid.
1. Only if the transaction is valid will it be put in a transaction queue ready for block authors.
1. Once in the ready queue, the same runtime API is used to order the transaction according to some priority factor.
1. The node uses transactions in this queue to construct a block and propagate it to peers over the network.

Notice that transactions are not removed from the ready queue when blocks are authored, but removed _only_ on block import.
This is due to the possibility that a recently-authored block might not make it into the canonical chain.
Inherents don't follow the same execution path: they are included in every block without fail.

### Extrinsics in a block received from network

1. The node receives notification of the new block.
1. Other nodes also construct this block and publish it to the network.
1. A 2/3 majority of nodes reach consensus that this block is part of the canonical chain.
1. All other nodes on the network receive and build the block.
1. All transactions in that block are executed and state changes are updated in runtime storage.

## Validating and queuing transactions

As discussed in [Consensus](), a majority of nodes in the network must agree on the state of the blockchain to continue securely adding blocks.
To reach consensus, two-thirds of the nodes must agree on the order of the transactions executed and the resulting state change. 
To prepare for consensus, transactions are first validated and queued on the local node in a **transaction pool**.
This is where all signed and unsigned transactions that have been received by a local node are placed before they get broadcast to the rest of the network.
Before including a transaction in a block, nodes must first determine which transactions are valid and in what order of priority it should be included.

### Validating transactions

A runtime defines rules that determine the validity of any transaction type. 
It translates this information by tagging transactions with special fields to help nodes determine whether a transaction is valid or not.
This tagging system gives nodes just enough information to know whether a transaction provides the data required by the runtime.

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

If a transaction is identified as valid, the transaction pool moves the transaction into a queue. 
There are two separate transaction queues for valid transactions.

* The **ready queue**: contains transactions that can be included in a new pending block.
  For runtimes built with FRAME, the transactions must follow the exact order in the ready queue.

* The **future queue**: contains transactions that may become valid in the future.
  For example, a transaction may have a `nonce` that is too high for its account, in which case the transaction will wait in the future queue until the valid preceding transactions are included in the chain, after which it will be either dropped or reconsidered.
  
It's possible to design a custom runtime to remove the transaction ordering requirements.
However, a runtime without strict transaction ordering would allow full nodes to implement different strategies for propagating transactions and including them in blocks. 
### Invalid transactions

When a transaction cannot be added to a block at all, the runtime will return an `Invalid` error 
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

## Transaction formats

The way the format of an extrinsic is desgined in Substrate takes into account the metadata it should expose as well as any additional information required to verify that a transaction is valid.
This provides a means of checking the requirements for an extrinisic to be valid and correctly constructed, which is done in the runtime by formatting any extrinsic as either unchecked, checked or opaque. 

- Unchecked: signed transactions that require some validation check before they can be accepted in the transaction pool.
Any unchecked extrinsic contains the signature for the data being sent plus some extra data.
- Checked: inherent extrinsics which by definition don't require signature verification. 
Instead, they carry information on where the extrinsic comes from and some extra data.
- Opaque: used for cases when an extrinsic hasn't yet been committed to a format but can still be decoded. 

Extra data can be any additional information that would be useful to attach to a transaction or inherent.
For example, the nonce of the transaction, or the tip for the block author.
This information is provided by a [specialized extensions](#signed-extensions) that help determine the validity and ordering of extrinsics before they get included in a block.

A signed transaction call could look like:

```rust
node_runtime::UncheckedExtrinsic::new_signed(
		function.clone(),                                      // some call
		sp_runtime::AccountId32::from(sender.public()).into(), // some sending account
		node_runtime::Signature::Sr25519(signature.clone()),   // the account's signature
		extra.clone(),                                         // the signed extensions 
	)
```

## How transactions are constructed 

Substrate defines its transaction formats generically to allow developers to implement custom ways to define valid transactions. 
In a runtime built with FRAME however (assuming transaction version 4), a transaction must be constructed by submitting the following encoded data:

`<signing account ID> + <signature> + <additional data>`

When submitting a signed transaction, the signature is constructed by signing:

- The actual call, composed of:
  - The index of the pallet.
  - The index of the function call in the pallet. 
  - The address and balance of the sender. 
  
- Some extra information, verified by the signed extensions of the transaction:
  - What's the era for this transaction, i.e. how long should this call last in the transaction pool before it gets discarded?
  This can either be `Mortal` or `Immortal`.
  - The nonce, i.e. many prior transactions have occurred from this account?
  This helps protect against replay attacks or accidental double-submissions.
  - The tip amount paid to the block producer to help incentive it to include this transaction in the block.

Then, some additional data that's not part of what gets signed is required, which includes:

  - The spec version and the transaction version. 
  This ensures the transation is being submitted to a compatible runtime.
  - The genesis hash. This ensures that the transaction is valid for the correct chain.
  - The block hash. This corresponds to the hash of the checkpoint block, which enables the signature to verify that the transaction doesn't execute on the wrong fork, by checking against the block number provided by the era information.

The SCALE encoded data is then signed (i.e. (`call`, `extra`, `additional`)) and the signature, extra data and call data is attached in the correct order and SCALE encoded, ready to send off to a node that will verify the signed payload.
In order to minimize the size of the signed transaction if a payload is longer than 256 bytes, it gets hashed and the hashed value is what gets signed and serialized.

This process can be broken down into the following steps:

1. Construct the unsigned payload.
1. Create a signing payload.
1. Sign the payload.
1. Serialize the signed payload.
1. Submit the serialized transaction.

The final result has the following bit-fields:

`0x + [ 1 ] + [ 2 ] + [ 3 ] + [ 4 ] + [ 5 ]`

where:

- `[1]` is a `u8` containing the compact encoded length of the encoded data.
- `[2]` is a `u8` containing 1 byte for the transaction version ID.
- `[3]` is a `u8` containing 1 bit to indicate whether the transacton is signed.
- `[4]` is a `[u8; 32]` containing the signature, if signed. If unsigned this is just a `0; u8`.
- `[5]` is a `u128` containing the encoded call data.

The way applications know how to construct a transaction correctly is provided by the [metadata interface](./frontend#metadata).
For instance, an application will know that a `(u8, u8, u8, [u8; 32], u128)` type will encode to the correct bytes to represent the call it wants to make. 
If a call doesn't need to be signed, the application knows to pre-prend a `None` signature to it (`0; u8`). 

<!-- TODO: How are inherents constructed? -->

**Example:**

Balances transfer from Bob to Dave: Bob sends `42` units to Dave.

[ TODO: polkadotjs apps screenshot ]

* Encoded call data: `0x050000306721211d5404bd9da88e0204360a1a9ab8b87c66c1bc2fcdd37f3c2222cc20a8`
* Encoded call hash: `0x52f197be55b1fcd3bd866f19aab6da02a18fd4ee034292e8c9c3b245939eda71`
* Signed call: `0xf4c7bf707e5fee3e7d9938c4a0b27fabf72fa7c1c154cacb02cb74bd1874d219e57a15856884545f0e4c59d79184eb238272a9aab0a03c13edc65774f0a8ce88`
* Compact endcoded length of encoded data: `4a`

* Resulting extrinsic: `0x4a100f4c7bf707e5fee3e7d9938c4a0b27fabf72fa7c1c154cacb02cb74bd1874d219e57a15856884545f0e4c59d79184eb238272a9aab0a03c13edc65774f0a8ce8852f197be55b1fcd3bd866f19aab6da02a18fd4ee034292e8c9c3b245939eda71`

Submitting the resulting constructed extrinsic via RPC returns:

```json
{
  dispatchInfo: {
    weight: 195,952,000
    class: Normal
    paysFee: Yes
  }
  events: [
    {
      phase: {
        ApplyExtrinsic: 1
      }
      event: {
        method: Withdraw
        section: balances
        index: 0x0508
        data: [
          5FHneW46xGXgs5mUiveU4sbTyGBzmstUspZC92UhjJM694ty
          125,000,141
        ]
      }
      topics: []
    }
    {
      phase: {
        ApplyExtrinsic: 1
      }
      event: {
        method: Transfer
        section: balances
        index: 0x0502
        data: [
          5FHneW46xGXgs5mUiveU4sbTyGBzmstUspZC92UhjJM694ty
          5DAAnrj7VHTznn2AWBemMuyBwZWs6FNFjdyVXUeYum3PTXFy
          42
        ]
      }
      topics: []
    }
    {
      phase: {
        ApplyExtrinsic: 1
      }
      event: {
        method: ExtrinsicSuccess
        section: system
        index: 0x0000
        data: [
          {
            weight: 195,952,000
            class: Normal
            paysFee: Yes
          }
        ]
      }
      topics: []
    }
  ]
  status: {
    InBlock: 0x6543a4d4b44f5acc9ad111f0218296f1da5a5493599431ce9eecb55ed0a4d3fb
  }
}
```

## Signed extensions

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

## Executing transactions and producing blocks 
<!-- 
TODO: 
which explains how blocks are build and executed

`BlockBuilder_build_inherents` is first passed to include inherents -> then other extrinsics go to to pool and then -> `BlockBuilder_append_extrinsic` adds both types of extrinsics to build the block. -->

After valid transactions are placed in the transaction queue, a separate **executive module** orchestrates how transactions are executed to produce a block.
The executive module is not a typical pallet that provides specific functionality to the runtime.
Instead, the executive module acts as the orchestration layer that calls functions in the runtime modules and handles the execution of those functions in the proper order as defined in the runtime business logic.

The executive module provides functions to:

* Check transaction validity.
* Initialize a block.
* Apply extrinsics.
* Execute a block.
* Finalize a block.
* Start an off-chain worker.

As a runtime developer, it's important to understand how the executive module interacts with the system pallet and other pallets you use to compose the business logic for your blockchain.
In particular, you should be familiar with how the executive module performs the following operations:

* Initialize a block
* Execute the inherent and transaction extrinsics in a block
* Finalize a block

Here's a test code snippet for a building and executing a valid block:

```rust
	fn test_build_and_execute_block_success() {
    // Create some inherents.
		let xt1 = TestXt::new(Call::CustomPallet(custom_pallet::Call::inherent_call {}), None);
    // Create some signed transaction.
		let xt2 = TestXt::new(call_transfer(33, 0), sign_extra(1, 0, 0));

		let header = new_test_ext(1).execute_with(|| {
			// Build the block header.
			Executive::initialize_block(&Header::new(
				1,
				H256::default(),
				H256::default(),
				[69u8; 32].into(),
				Digest::default(),
			));

      // Apply all extrinsics.
			Executive::apply_extrinsic(xt1.clone()).unwrap().unwrap();
			Executive::apply_extrinsic(xt2.clone()).unwrap().unwrap();

      // Finalize the block.
			Executive::finalize_block()
		});

    // Execute the block
		new_test_ext(1).execute_with(|| {
			Executive::execute_block(Block::new(header, vec![xt1, xt2]));
		});
	}
```

### Initialize a block

To initialize a block, the executive module calls the `on_initialize` function in the System pallet and all other runtime pallets to execute any business logic defined to take
place before transactions are executed. 
The System pallet `on_initialize` function is always executed first.
The remaining pallets are called in the order they are defined in the `construct_runtime!` macro.

After all of the pallet `on_initialize` functions have been executed, the executive module checks the parent hash in the block header and the extrinsics trie root to verify that the information is correct.

### Executing extrinsics

After the block has been initialized, each valid extrinsic is executed in order of transaction priority. 
Extrinsics must not cause a panic in the runtime logic or the system would be vulnerable to attacks where users could trigger computational execution without any punishment.

When an extrinsic executes, the state is not cached prior to execution.
Instead, state changes are written directly to storage during execution. 
If an extrinsic were to fail mid-execution, any state changes that took place before the failure would not be reverted, leaving the block in an unrecoverable state.
Before committing any state changes to storage, the runtime logic should perform all necessary checks to ensure the extrinsic will succeed. 

Note that [events](/v3/runtime/events-and-errors) are also written to storage. 
Therefore, the runtime logic should not emit an event before performing the complementary actions. 
If an extrinsic fails after an event is emitted, the event is not be reverted.

### Finalizing a block

After all queued extrinsics have been executed, the executive module calls into each pallet's `on_idle` and `on_finalize` functions to perform any final business logic that should take place at the end of the block. 
The modules are again executed in the order that they are defined in the `construct_runtime!` macro, but in this case, the `on_finalize` function in the system pallet is executed last.

After all of the `on_finalize` functions have been executed, the executive modulate checks that the digest and storage root in the block header match what was calculated when the block was initialized.

The `on_idle` function also passes through the remaining weight of the block to allow for execution based on the usage of the blockchain.

## Learn more

- Learn about the origin system for different extrinsic types
- Learn more about how transactions are encoded
- Learn how to configure transaction fees for your chain
- Learn about how block execution works
- Submit offline transactions using `tx-wrapper` 
- Submit transactions using `sidecar`