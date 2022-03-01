A place for content that's not been used anywhere.
Perhaps to scrap, haven't decided yet. 

# Transactions 

## Where transactions are defined

As discussed in [Runtime development](), the Substrate runtime contains the business logic that defines valid transactions, determines whether the transactions are sent as signed or unsigned, and how transactions change the state of the chain.

Typically, you use pallets to compose the runtime functions and to implement the transactions you want your chain to support.
After you compile the runtime, users interact with the blockchain to submit requests that are processed as transactions.
For example, a user might submit a request to transfer funds from one account to another.
The request becomes a signed transaction that contains the signature for that user account and if there are sufficient funds in the user's account to pay for the transaction, the transaction executes successfully, and the transfer is made.

<!-- Although inherents are very different by nature compared to signed and unsigned transaction types, they are treated like any other transaction from the perspective of the transaction pool. -->

<!-- See how FRAME implements and used these types of transactions:
- #[pallet::inherent]: 
- #[pallet::validate_unsigned]:
- dispatch origins: signed, unsigned, none (inherent) -->

A `CheckedExtrinsic` is quite different on the other hand.
It's format requires:

- a `function` corresponding to the call being made.
- a `signed` field which carries information on where this extrinsic comes from (`AccountId`) and the number of extrinsics that have come before from the same source (defined by the `CheckedExtrinsic` signed extension).


<!-- All signed transactions in a FRAME runtime have the following formatted around the `UncheckExtrinsics` type and 

- tip
- period
- current_block 
- era 
- extra 
- raw_payload 
- signature
- address 

And look like this when decoded:

```text
Extra: (CheckNonZeroSender, CheckSpecVersion, CheckTxVersion, CheckGenesis, CheckEra(Era::Mortal(2048, 99)), CheckNonce(0), CheckWeight, ChargeTransactionPayment<0>)
Additional Signed: (259, 1, 0x0000000000000000000000000000000000000000000000000000000000000000, 0x0000000000000000000000000000000000000000000000000000000000000000, (), (), ())
Raw Payload: [6, 0, 255, 212, 53, 147, 199, 21, 253, 211, 28, 97, 20, 26, 189, 4, 169, 159, 214, 130, 44, 133, 88, 133, 76, 205, 227, 154, 86, 132, 231, 165, 109, 162, 125, 145, 1, 58, 6, 0, 0, 3, 1, 0, 0, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0]
```

Notice some other [signed extensions](#signed-extensions) which perform certain checks on the format of the transaction.
 -->

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


A decoded signed transaction from a JSON-RPC call would look like:

```json
"method":{
            "pallet":"balances",
            "method":"transfer"
         },
         "signature":{
            "signature":"0x94b63112648e8e692f0076fa1ccab3a04510c269d1392c1df2560503865e144e3afd578f1e37e98063b64b98a77a89a9cdc8ade579dcac0984e78d90646a052001",
            "signer":{
               "id":"Gr5sBB1EgdmQ7FG3Ud2BdECWQTMDXNgGPfdHMMtDsmT4Dj3"
            }
         },
         "nonce":"12",
         "args":{
            "dest":{
               "id":"J6ksma2jVeHRcRoYPZBkJRzRbckys7oSmgvjKLrVbj1U8bE"
            },
            "value":"100000000"
         },
         "tip":"0",
         "hash":"0xfbc5e5de75d64abe5aa3ee9272a3112b3ce53710664f6f2b9416b2ffda8799c2",
         "info":{
            "weight":"201217000",
            "class":"Normal",
            "partialFee":"2583332634"
         }
}
```

An inherent would look like:

```json
"method":{
            "pallet":"timestamp",
            "method":"set"
         },
         "signature":null,
         "nonce":null,
         "args":{
            "now":"1620636072000"
         },
         "tip":null,
         "hash":"0x8b853f49b6543e4fcbc796ad3574ea5601d2869d80629e080e501da4cb7b74b4",
         "info":{

         }
}
```


The [`ValidTransaction` struct](/rustdocs/latest/sp_runtime/transaction_validity/struct.ValidTransaction.html) defines two parameters used to determine the ordering of transactions in the pool: `requires` and `provides`. 
Together they create a dependency graph which allows the pool to produce a valid linear ordering of transactions.

Substrate supports multiple `provides` and `requires` tags.
This enables runtime engineers to create alternate ordering schemes.


## FRAME implementation

See how the core mechanisms described in this article are implemented in FRAME:
- `pallet_transaction_payment_rpc`: RPC interface for the transaction payment pallet.
- `pallet_transaction_payment`: Transaction Payment Pallet 
- `pallet_transaction_storage`: Transaction storage pallet. Indexes transactions and …
- `pallet_transaction_payment_rpc_runtime_api`: Runtime API definition for transaction payment pallet.

`Operational` transactions are getting additional priority bump (3/4 * `u64::max()`), because that prevents other transactions from getting in front of them in the pool, but now equal to the `OperationalVirtualTip` amount configurable by a runtime developer.

Notes:

- We need some note in the "welcome" section that says that our docs describe building parachains with Substrate, specifically for runtimes built with FRAME.