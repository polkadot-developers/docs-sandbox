Section: Design
Sub-section: Economic models
Type: reference + conceptual

All blockchains require resources—processors, memory, storage, and network bandwidth—to perform operations.
The computers that participate in the network—the authoring nodes that produce blocks—provide these resources to blockchain users.
Together, these computers create a distributed, decentralized network that serves the needs of a community of participants.

To support a community and make a blockchain sustainable, most blockchains require users to pay for the network resources they use in the form of transaction fees.
The payment of transaction fees requires user identities to be associated with accounts that hold assets of some type.
Blockchains typically use cryptographic tokens to represent different assets account can hold and these tokens are made available for purchase outside the chain through an exchange.
Network participants can then deposit the tokens to participate as validators by staking their funds or can use them make transactions on the network by paying fees.

Common use cases for an underlying network token:

- **To keep an account alive.** Any account requires a minimum deposit-called the existential deposit-to prevent users from draining resources without the ability to pay for consuming those resources.

- **To participate in governance.** Most blockchains also enable network participants to submit and vote on proposals that affect network operations or the blockchain community.
By submitting and voting on proposals—referenda—the blockchain community can determine how the blockchain evolves in an essentially democratic process.

- **To participate in proof-of-stake**. Participants that act as block authoring nodes for the network must maintain a significant stake of funds in a special account called the **stash account**.

## Hybrid consensus

See: https://github.com/paritytech/substrate/issues/1304
Describes considerations for swappable consenus.
A good starting point for designing hybrid consesnus systems.

Note: block authoring engines must be made aware of blocks that are finalized so that they don't waste time building on top of blocks that will never be in the canonical chain.
