# Glossary

This page defines and explains many of the domain-specific terms that are common to the Substrate
ecosystem. This is a helpful resource for even the most experienced Substrate developers.

---

## adaptive quorum biasing (AQB)

Provides a mechanism for adjusting the passing threshold for a referendum based on voter turnout.
Adaptive quorum biasing allows for more flexible governance by removing the requirement to have an arbitrary quorum for voting purposes, which create undesirable governance mechanics.
Adaptive quorum biasing is implemented in the [Democracy pallet](/v3/runtime/frame#democracy). 
The Democracy pallet provides the interfaces for on-chain bodies such as a [collective](#council) or individual token holder—to call referenda with positive,
negative, or neutral biases.

With a **positive turnout bias**, the passing threshold _decreases_ as more votes are cast, so that a higher turnout increases the likelihood of a referendum passing. 
With a **negative turnout bias**, the passing threshold _increases_ as more votes are cast. 
Negative turnout bias is also sometimes called a "default carries" position because if there's an apathetic voting body, the referendum passes by default. 
A **neutral turnout bias** specifies a simple majority passing threshold.  

<!--
## AFG

An internal codename for "Al's Finality Gadget," which is named after [Alistair Stewart](https://w3f-research.readthedocs.io/en/latest/team_members/alistair.html)
who invented it. 
AFG is synonymous with [GRANDPA](#grandpa).
-->

## aggregation

Used in the context of [FRAME](#frame), _aggregation_ or _[pallet](#pallet) aggregation_ is the process of combining analogous types from multiple runtime modules into a single type. 
Pallet aggregation allows each module's analogous types to be represented. 
Currently, there are six such data types:

- `Call` for published functions that can be called with a set of arguments.
- `Error` for messages that indicate why a function invocation (`Call`) failed.
- `Event` for pallet-emitted events that describe state changes.
- `Log` for extensible header items.
- `Metadata` for information that allows inspection of the above.
- `Origin` for the source of a function invocation (`Call`).

## approval voting

Voting system where voters can vote for as many candidates as desired. 
The candidate with the highest overall number of votes wins. 
With approval voting, it is worth noting the following:

- Voting for all candidates is the same as voting for none.
- It is possible to vote against a single candidate by voting for all other candidates.

Approval voting is used by the [FRAME Elections Phragmen pallet](/v3/runtime/frame#elections-phragmén) as a governing [Council](#council) on a number of Substrate-based chains.

## author

Describes the [node](#node) that is responsible for the creation of a [block](#block).
Block authors are also referred to as _block producers_. 
In a proof-of-work blockchain, these nodes are called _miners_.

## authority

The [nodes](#node) that act as a collective to manage [consensus](#consensus) on a[blockchain](#blockchain) network. 
In a [proof-of-stake](#nominated-proof-of-stake-npos) blockchain—for example, a blockchain that us the [Staking pallet](/v3/runtime/frame#staking) from [FRAME](#frame)—authorities are determined through a token-weighted nomination and voting system.

> The terms _authorities_ and _[validators](#validator)_ sometimes seem to refer the same thing.
> However, _validators_ is a broader term that can include other aspects of chain maintenance such as parachain validation. 
> In general, authorities are a (non-strict) subset of validators and many validators are authorities.

## authority round (Aura)

Deterministic [consensus](#consensus) protocol where [block](#block) production is limited to a rotating list of [authorities](#authority) that take turns creating blocks.
With authority round (Aura) consensus, the majority of online
authorities are assumed to be honest. 

Learn more by reading [the official wiki article](https://openethereum.github.io/Aura) for the Aura consensus algorithm.

The Aura protocol is often used in combination with GRANDPA as a [hybrid consensus](#hybrid-consensus) protocol where [Aura](#aura) is used for block production and
short-term [probabilistic finality](#probabilistic-finality), with
[deterministic finality](#deterministic-finality) provided by [GRANDPA](#grandpa).

## blind assignment of blockchain extension (BABE)

A [block authoring](#author) protocol similar to [Aura](#aura).
However, with the blind assignment of blockchain extension (BABE) protocol, [authorities](#authority) win [slots](#slot) based on a verifiable random function (VRF) as opposed to the round-robin selection method. 
The winning authority can select a chain and submit a new block for it. 

Learn more about BABE by referring to its [official Web3 Foundation research document](https://research.web3.foundation/en/latest/polkadot/block-production/Babe.html).

## block

Describes a single element of a blockchain that [cryptographically](#cryptographic-primitives) binds a set of [extrinsic](#extrinsic) data—the body—to a [header](#header). 
Blocks are arranged into a tree through parent pointers. 
The pointer to a parent block is a hash of the parent and the tree is pruned into a list using a [fork-choice rule](/v3/advanced/consensus#fork-choice-rules) and an optional [finality](#finality) mechanism.

## blockchain

Describes a distributed network of computers that uses [cryptography](#cryptographic-primitives) to allow a group of participants to trustlessly come to [consensus](#consensus) on the [state](#state) of a system as it evolves over time
The computers that compose the blockchain network are called [nodes](#node).

## Byzantine fault tolerance (BFT)

Defines the ability of a distributed computer network to remain operational if a certain proportion of its [nodes](#node) or [authorities](#authority) are defective or behaving maliciously. 
Typically, a distributed network is considered byzantine fault tolerant if it can remain functional with up to one-third of nodes assumed to defective, offline, actively malicious, and acting as part of a coordinated attack.

### Byzantine failure

The loss of a network service due to node failures that exceed the proprortion of nodes required to reach consensus.

### practical byzantine fault tolerance (pBFT)

An early approach to Byzantine fault tolerance. pBFT systems tolerate Byzantine behavior from up to one-third of participants. 
The communication overhead for such systems is `O(n²)`, where `n` is the number of nodes (participants) in the system.

## consensus

In the context of a [blockchain](#blockchain), consensus is the process nodes use to agree on the canonical [fork](#fork) of a chain. 
Consensus is comprised of [authorship](#author), [finality](#finality), and [fork-choice rule](/v3/advanced/consensus#fork-choice-rules). 

In the Substrate ecosystem, these three components are separated from one another, and the term consensus often refers specifically to authorship.
In the context of a Substrate [node](#node), the term **consensus engine** describes the node subsystem that is responsible for consensus tasks.

See also [hybrid consensus](hybrid-consensus).

## consensus algorithm

An algorithm that ensures that a set of [actors](#authority)—who don't necessarily trust each other—can reach agreement about state as the result of some computation.
Because most consensus algorithms assume that up to one-third of the actors or nodes can  are [Byzantine fault tolerant](#byzantine-fault-tolerance-bft).


Consensus algorithms are generately concerned with ensuring two properties:

- **safety** indicating that all honest nodes eventually agreed on the state of the chain.
- **liveness**" indicating the ability for the chain to keep making progress. 

For detailed information about the consensus strategies of the [Polkadot network](#polkadot-network), see the [Polkadot Consensus](https://polkadot.network/polkadot-consensus-part-1-introduction/) blog series.

See also [hybrid consensus](hybrid-consensus).

## cryptographic primitives

A general term used to describe fundamental cryptographic concepts such as signature schemes and hashing algorithms. 
Cryptographic primitives are essential to many aspects of the Substrate ecosystem. For example:

- Hashing algorithms produce [blocks](#block) of hashed data and each block uses the hash generated by the hashing algorithm to reference its parent block.
- Hashing is used to encode [state](#state) as a [trie](#trie-patricia-merkle-tree) data structure to facilitate efficient verification.
- Digital signature schemes are used to secure different [consensus](#consensus) models such as [authorities](#authority).
- Cryptographic schemes identify and authenticate the [accounts](/v3/concepts/account-abstractions) used to perform [transactions](#transaction) in the Substrate runtime.

## council

Most often used to refer to an instance of the [Collective pallet](/v3/runtime/frame#collective) on Substrate-based networks such as [Kusama](#kusama) or [Polkadot](#polkadot) if the Collective pallet is part of the [FRAME](#frame)-based [runtime](#runtime) for the network. 
A council primarily serves to optimize and balance the more inclusive referendum system.

## database backend

The means by which the [state](#state) of a [blockchain](#blockchain) network is persisted between invocations of the [blockchain node](#node) application. 
For information about how the database backend is implemented and used by Substrate-based chains, see [Advanced storage](/v3/advanced/storage).

## dev phrase

A [mnemonic phrase](https://en.wikipedia.org/wiki/Mnemonic#For_numerical_sequences_and_mathematical_operations) that is intentionally made public. 
All of the[well-known development accounts](/v3/tools/subkey#well-known-keys) (Alice, Bob, Charlie, Dave, Eve, and Ferdie) are generated from the same dev phrase. 
The dev phrase is:
`bottom drive obey lake curtain smoke basket hold race lonely fit walk`

Many tools in the Substrate ecosystem, such as [`subkey`](/v3/tools/subkey), allow users to implicitly specify the dev phrase by only specifying a derivation path such as `//Alice`.

## digest

An extensible field of the [block header](#header) that encodes information needed by several actors in a blockchain network including:

- [Light clients](#light-client) for chain synchronization.
- Consensus engines for block verification.
- The runtime itself in the case of pre-runtime digests.

## dispatch

The execution of a function with a predefined set of arguments. 
In the context of [runtime](#runtime) development with [FRAME](#frame), a dispatch takes pure data—the type is known as `Call` by convention—and uses that data to call a published function in a runtime module ([pallet](#pallet)) with predefined arguments.
The published functions take one additional parameter, known as [`origin`](#origin), that allows the function to securely determine the provenance of its execution.

## equivocating

A type of erroneous or malicious behavior that involves backing multiple mutually-exclusive options within the [consensus](#consensus) mechanism.

## ethash

A function used by some [proof-of-work](#proof-of-work) [consensus](#consensus) systems, such as the Ethereum blockchain. 
It was developed by [a team led by Tim Hughes](https://github.com/ethereum/ethash/graphs/contributors).

## events

A means of recording, for the benefit of the off-chain world, that some particular [state](#state) transition happened. 
In the context of [FRAME](#frame), events are a composable data types that each [pallet](#pallet) can individually define. 
Events in FRAME are implemented as a set of transient storage items that are inspected immediately after a block has executed and reset during block-initialization.

## executor

A means of executing a function call in a given [runtime](#runtime) with a set of dependencies.
There are two [executor](/v3/advanced/executor) implementations present in
Substrate, _WebAssembly_ and _native_.

- The _native executor_ uses a natively compiled runtime embedded in the [node](#node) to execute calls. 
This is a performance optimization that up-to-date nodes can take advantage of.

- The _WebAssembly executor_ uses a [Wasm](#webassembly-wasm) binary and a Wasm interpreter to execute calls. 
The binary is guaranteed to be up-to-date regardless of the version of the blockchain [node](#node) because it is persisted in the [state](#state) of the Substrate-based chain.

## extrinsic

Data that is external to the blockchain and included in a [block](#block). 
In general, there are two types of extrinsics:

- [signed](/v3/concepts/extrinsics#signed-transactions) or
[unsigned](/v3/concepts/extrinsics#unsigned-transactions) transactions.
- [inherents](/v3/concepts/extrinsics#inherents) inserted by
[block authors](#author).

## existential deposit

The minimum balance an account is allowed to have in the [Balances pallet](/v3/runtime/frame#balances). 
Accounts cannot be created with a balance less than the existential deposit amount.
If an account balance drops below this amount, the Balances pallet uses
[a FRAME System API](/rustdocs/latest/frame_system/pallet/struct.Pallet.html#method.dec_ref) to drop its references to that account. 
If all of the references to an account are dropped, the account can be [reaped](/rustdocs/latest/frame_system/pallet/struct.Pallet.html#method.allow_death).

## finality

The part of [consensus](#consensus) that makes the ongoing progress of the blockchain irreversible.
After a [block](#block) is finalized, all of the [state](#state) changes it encapsulates are irreversible without a hard fork.
The consensus algorithm _must_ guarantee that finalized blocks never need reverting.
However, different consensus algorithms can define different finalization methods.

In a consensus protocol that uses **deterministic finality**, each block is guaranteed to be the canonical block for that chain when the block is included. 
Deterministic finality is desirable in situations where the full chain is not available, such as in the case of [light clients](#light-client). 
[GRANDPA](#grandpa) is the deterministic finality protocol that is used by the [Polkadot Network](#polkadot-network).

In a consensus protocol that uses **probabilistic finality**, finality is expressed in terms of a probability, denoted by `p`, that a proposed block, denoted by `B`, will remain in the canonical chain.
As more blocks are produced on top of `B`, `p` approaches 1.

In a consensus protocol that uses **instant finality**, finality is guaranteed  immediately upon block production. 
This type of non-probabilistic consensus tends to use [practical byzantine fault tolerance (pBFT)](#practical-byzantine-fault-tolerance-pbft) and have expensive communication requirements.

## proof-of-finality

Data that can be used to prove that a particular block is finalized.

## fork

Indicates that there are divergent paths a blockchain might take. 
If two or more [blocks](#block) have the same parent but different state, the blockchain cannot continue to progress until the differences are [resolved](/v3/advanced/consensus#fork-choice-rules).
An unresolved fork would split the blockchain into two separate chains.
By resolving divergent forks, you can ensure that only one canonical chain exists.

## Flaming Fir

A Substrate-based [blockchain](#blockchain) test network that exists for developing and testing the Substrate blockchain development framework.
For more information about accessing Substrate networks and flaming fir, see the [Polkadot wiki](https://wiki.polkadot.network/docs/maintain-networks#flaming-fir).

## FRAME

An acronym for the _Framework for Runtime Aggregation of Modularized Entities_ that enables developers to create blockchain [runtime](#runtime) environments from a modular set of components called [pallets](#pallet).

Runtime developers interact with FRAME using [macros](#macro) such as the following:

* [`#[pallet::event]`](/v3/runtime/macros#palletevents),
* [`#[pallet::error]`](/v3/runtime/macros#palleterror),
* [`#[pallet::storage]`](/v3/runtime/macros#palletstorage),
* [`#[frame_support::pallet]`](/v3/runtime/macros#frame_supportpallet)

The macros make it easy to define custom pallets and compose pallets to create a working runtime using the [`construct_runtime!`](/v3/runtime/macros#construct_runtime)
macro to deploy a Substrate-based blockchain. 

The convention used in [the Substrate codebase](https://github.com/paritytech/substrate/tree/master/frame) is to preface core FRAME modules with `frame_` and the optional pallets with `pallet_*`. 
For example, the preceding macros are all defined in the [`frame_support`](/v3/runtime/frame#support-library) module and all FRAME-based runtimes _must_ include the
[`frame_system`](/v3/runtime/frame#system-library) module. 
After the `frame_support::construct_runtime` macro has been used to create a runtime that includes the `frame_system` module, optional pallets such as the [Balances](/v3/runtime/frame#balances) pallet can be used to extend the core capabilities of the runtime.

## full client

A [node](#node) that is able to synchronize a blockchain in a secure manner through execution (and thus verification) of all logic. 
Full clients stand in contrast to [light clients](#light-client).

## genesis configuration

A mechanism for specifying the initial (genesis) [state](#state) of a [blockchain](#blockchain).
Genesis configuration of Substrate-based chains is accomplished by way of a
[chain specification](/v3/runtime/chain-specs) file, which makes it easy to use a
single Substrate codebase to underpin multiple independently configured chains.

## GRANDPA

A [deterministic finality](#deterministic-finality) gadget for [blockchains](#blockchain) that is
implemented in the [Rust](https://www.rust-lang.org/) programming language. The
[formal specification](https://github.com/w3f/consensus/blob/master/pdf/grandpa-old.pdf) is
maintained by the [Web3 Foundation](https://web3.foundation/)

---

## Header

A structure that is used to aggregate pieces of (primarily
[cryptographic](#cryptographic-primitives)) information that summarize a [block](#block). This
information is used by [light-clients](#light-client) to get a minimally-secure but very efficient
synchronization of the chain.

---

## hybrid consensus

A blockchain consensus protocol that consists of independent or loosely coupled mechanisms for
[block production](#author) and [finality](#finality). This allows the chain to grow as fast as
probabilistic consensus protocols, such as [Aura](#aura-aka-authority-round), while still maintaining the same level of
security as [deterministic finality](#deterministic-finality) consensus protocols, such as
[GRANDPA](#grandpa). In general, block production algorithms tend to be faster than finality
mechanisms; separating these concerns gives Substrate developers greater control of their chain's
performance.

## Keystore

A subsystem in Substrate for managing keys for the purpose of producing new blocks.

## Kusama

[Kusama](https://kusama.network/) is a Substrate-based [blockchain](#blockchain) that implements a
design similar to the [Polkadot Network](#polkadot-network). Kusama is a
"[canary](https://en.wiktionary.org/wiki/canary_in_a_coal_mine)" network and is referred to as
[Polkadot's "wild cousin"](https://polkadot.network/kusama-polkadot-comparing-the-cousins/). The
differences between a canary network and a true test network are related to the expectations of
permanence and stability; although Kusama is expected to be more stable than a true test network,
like [Westend](#westend), it should not be expected to be as stable as an enterprise production
network like [Polkadot](#polkadot). Unlike Westend, which is maintained by
[Parity Technologies](https://www.parity.io/), Kusama (like Polkadot) is
[controlled by its network participants](/v3/runtime/frame#democracy). The level of stability offered
by canary networks like Kusama is intended to encourage meaningful experimentation.

---

## libp2p

A peer-to-peer networking stack that allows use of many transport mechanisms, including WebSockets
(usable in a web browser). Substrate uses the
[Rust implementation](https://github.com/libp2p/rust-libp2p) of the libp2p networking stack.

## Light Client

A light client is a type of blockchain [node](#node) that does not store the [chain state](#state)
or produce ([author](#author)) blocks. It encapsulates basic capabilities for verifying
[cryptographic primitives](#cryptographic-primitives) and exposes an
[RPC (remote procedure call)](https://en.wikipedia.org/wiki/Remote_procedure_call) server to allow
blockchain users to interact with the blockchain network.

---

## Macro

Macros are features of some programming languages,
[including Rust](https://doc.rust-lang.org/1.7.0/book/macros.html), that allow developers to "write
code that writes code". [FRAME](#frame) provides a number of [macros](/v3/runtime/macros) that make
it easy to compose a [runtime](#runtime).

## Metadata

Metadata is information about a system, such as a [blockchain](#blockchain), that makes it easier to
interact with that system. Blockchain [runtimes](#runtime) that are built with [FRAME](#frame)
expose [lots of helpful metadata](/v3/runtime/metadata).

---

## Node

A node correlates to a running instance of a blockchain client; it is part of the
[peer-to-peer](https://en.wikipedia.org/wiki/Peer-to-peer) network that allows blockchain
participants to interact with one another. Substrate nodes can fill a number of roles in a
blockchain network. For instance, [validators](#validator) are the block-producing nodes that power
the blockchain, while [light-clients](#light-client) facilitate scalable interactions in
resource-constrained environments like [UIs](https://github.com/paritytech/substrate-connect) or
embedded devices.

## Nominated Proof-of-Stake (NPoS)

A means of determining a set of [validators](#validator) (and thus [authorities](#authority)) from a
number of accounts willing to commit their stake to the proper (non-[Byzantine](#byzantine-failure))
functioning of one or more [authoring](#author)/validator nodes. The Polkadot protocol describes
validator selection as a constraint optimization problem to eventually give a maximally staked set
of validators each with a number of supporting nominators lending their stake. Slashing and rewards
are done in a pro-rata manner.

---

## Origin

A [FRAME](#frame) primitive that identifies the source of a [dispatched](#dispatch) function call
into the [runtime](#runtime). The FRAME System module defines
[three built-in origins](/v3/runtime/origins#raw-origins); [pallet](#pallet)
developers can easily define custom origins, such as those defined by the
[Collective pallet](/rustdocs/latest/pallet_collective/enum.RawOrigin.html).

---

## Parachain

A parachain is a [blockchain](#blockchain) that derives shared infrastructure and security from a
"[relay chain](#relay-chain)". You can
[learn more about parachains on the Polkadot Wiki](https://wiki.polkadot.network/docs/en/learn-parachains).

## Pallet

A module that can be used to extend the capabilities of a [FRAME](#frame)-based [runtime](#runtime).
Pallets bundle domain-specific logic along with related runtime primitives like [events](#event),
and [storage items](#storage-items).

## Polkadot Network

The [Polkadot Network](https://polkadot.network/) is a [blockchain](#blockchain) that serves as the
central hub of a heterogeneous blockchain network. It serves the role of
"[relay chain](#relay-chain)" and supports the other chains (the "[parachains](#parachain)") by
providing shared infrastructure and security. The Polkadot Network is progressing through a
[multi-phase launch process](https://polkadot.network/explaining-the-polkadot-launch-process/) and
does not currently support parachains.

## Proof-of-Work

A [consensus](#consensus) mechanism that deters attacks by requiring work on the part of network
participants. For instance, some proof-of-work systems require participants to use the
[Ethash](#ethash) function to calculate a hash as a proof of completed work.

---

## Relay Chain

The central hub in a heterogenous ("chain-of-chains") network. Relay chains are
[blockchains](#blockchain) that provide shared infrastructure and security to the other blockchains
in the network (the "[parachains](#parachain)"). In addition to providing [consensus](#consensus)
capabilities, relay chains also allow parachains to communicate and exchange digital assets without
needing to trust one another.

## Remote Procedure Call (RPC)

A mechanism for interacting with a computer program that allows developers to easily query the
computer program or even invoke its logic with parameters they supply. Substrate nodes expose an RPC
server on HTTP and WebSocket endpoints.

### JSON-RPC

A standard way to call functions on a remote system by using a JSON protocol. For Substrate, this is
implemented through the [Parity JSON-RPC](https://github.com/paritytech/jsonrpc) crate.

## Rhododendron

An [instant finality](#instant-finality),
[Byzantine fault tolerant (BFT)](#byzantine-fault-tolerance-bft) [consensus](#consensus) algorithm.
One of a number of adaptions of [pBFT](#practical-byzantine-fault-tolerance-pbft) for blockchains.
Refer to its [implementation on GitHub](https://github.com/paritytech/rhododendron).

## Rococo

Rococo is the [Polkadot](#polkadot) Network's [parachain](#parachain) test network. It is a
Substrate-based [blockchain](#blockchain) that is an evolving testbed for the capabilities of
heterogeneous blockchain networks.

## Runtime

The block execution logic of a blockchain, i.e. the
[state transition function](#state-transition-function-stf). In Substrate, this is stored as a
[WebAssembly](#webassembly-wasm) binary in the [chain state](#state).

---

## Slot

A fixed, equal interval of time used by consensus engines such as [Aura](#aura-aka-authority-round)
and [BABE](#blind-assignment-of-blockchain-extension-babe). In each slot, a subset of
[authorities](#authority) is permitted (or obliged, depending on the engine) to [author](#author) a
[block](#block).

## Stake-Weighted Voting

Democratic voting system that uses one-vote-per-token, rather than one-vote-per-head.

## State

In a [blockchain](#blockchain), the state refers to the cryptographically secure data that persists
between blocks and can be used to create new blocks as part of the state transition function. In
Substrate-based blockchains, state is stored in a [trie](#trie-patricia-merkle-tree), a data structure that supports the
efficient creation of incremental digests. This trie is exposed to the [runtime](#runtime) as
[a simple key/value map](/v3/advanced/storage) where both keys and values can be
arbitrary byte arrays.

## State Transition Function (STF)

The logic of a [blockchain](#blockchain) that determines how the state changes when a
[block](#block) is processed. In Substrate, this is effectively equivalent to the
[runtime](#runtime).

## Storage Items

[FRAME](#frame) primitives that provide type-safe data persistence capabilities to the
[runtime](#runtime). Learn more about storage items in this article about
[runtime storage](/v3/runtime/storage).

## Substrate

A flexible framework for building modular, efficient, and upgradeable [blockchains](#blockchain).
Substrate is written in the [Rust](https://www.rust-lang.org/) programming language and is
maintained by [Parity Technologies](https://www.parity.io/).

---

## Transaction

A type of [extrinsic](#extrinsic) that can be safely gossiped between [nodes](#node) on the network
thanks to efficient [verification](#cryptographic-primitives) through
[signatures](/v3/concepts/extrinsics#signed-transactions) or
[signed extensions](/v3/concepts/tx-pool#signed-extension).

## Transaction Era

A definable period, expressed as a range of [block](#block) numbers, where a transaction may be
included in a block. Transaction eras are used to protect against transaction replay attacks in the
event that an account is reaped and its (replay-protecting) nonce is reset to zero.

## Transaction Pool

A collection of transactions that are not yet included in [blocks](#block) but have been determined
to be valid.

### Tagged Transaction Pool

A generic Substrate-based transaction pool implementation that allows the [runtime](#runtime) to
specify whether a given transaction is valid, how it should be prioritized, and how it relates to
other transactions in the pool in terms of dependency and mutual-exclusivity. It is designed to be
easily extensible and general enough to express both the
[UTXO](https://github.com/danforbes/danforbes/blob/master/writings/utxo.md) and account-based
transaction models.

## trie (Patricia Merkle Tree)

An data structure that is used to represent sets of items where:

- a cryptographic digest of the dataset is needed; and/or
- it is cheap to recompute the digest with incremental changes to the dataset even when it is very
  large; and/or
- a concise proof that the dataset contains some item/pair (or lacks it) is needed.

## validator

A semi-trusted (or untrusted but well-incentivized) actor that helps maintain a
[blockchain](#blockchain) network. In Substrate, validators broadly correspond to the
[authorities](#authority) running the [consensus](#consensus) system. In
[Polkadot](#polkadot-network), validators also manage other duties such as guaranteeing data
availability and validating [parachain](#parachain) candidate [blocks](#block).


## WebAssembly (Wasm)

An execution architecture that allows for the efficient, platform-neutral expression of
deterministic, machine-executable logic. [Wasm](https://webassembly.org/) is easily compiled from
the [Rust](http://rust-lang.org/) programming language and is used by Substrate-based chains to
provide portable [runtimes](#runtime) that can be included as part of the chain's [state](#state).

## Westend

Westend is a [Parity](https://www.parity.io/)-maintained, Substrate-based [blockchain](#blockchain)
that serves as the test network for [the Polkadot Network](#polkadot).

