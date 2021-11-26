# EVM pallet

The FRAME Ethereum Virtual Machine (EVM) provides an execution environment for Substrate's Ethereum compatibility layer, known as Frontier. Frontier allows unmodified EVM code to be executed in a Substrate-based blockchain, designed to closely emulate the functionality of executing contracts on the Ethereum mainnet within the Substrate runtime. 

For more information on the FRAME EVM, see [FRAME EVM pallet reference](https://docs.rs/pallet_evm/).

To download Frontier, go to https://github.com/paritytech/frontier.

The Substrate runtime works alongside the [Ethereum pallet](https://docs.rs/pallet-ethereum) and the [Dynamic Fee pallet](https://docs.rs/pallet-dynamic-fee) to enable the creation of runtimes capable of fully emulating Ethereum block production and transaction processing.

<AccentButton
  text={`Start the Frontier workshop`}
  link={`/tutorials/v3/frontier`}
/>

## EVM engine

The EVM pallet uses [SputnikVM](https://github.com/rust-blockchain/evm) as the underlying EVM engine. The engine is overhauled to be modular. For more information see [Core Paper Project of EVM](https://github.com/corepaper/evm)

The EVM is a good theoretical execution environment, but it is not very practical to use with modern hardware. For example, manipulation of 256 bit integers on modern architectures is significantly more complex than standard types. The Ethereum team has investigated the use of [Wasm](https://github.com/ewasm/design) for the next generation of the network. 

An alternative to using the EVM is adding the Contracts pallet. The Contracts pallet iterates on existing ideas in the smart contract ecosystem, particularly Ethereum and the EVM.

The most obvious difference between the Contracts pallet and the EVM is the underlying execution engine used to run smart contracts. To learn more about the Contracts pallet, see [link](link) 

## Cost

The EVM charges for storage fees only at the time of storage. This one-time cost results in some permanent amount of storage being used on the blockchain, forever, which is not economically sound.

## Execution lifecycle

Substrate-based accounts can call the EVM pallet to deposit or withdraw a balance from the Substrate base-currency into a different balance managed and used by the EVM pallet. Once a user has populated their balance, they can create and call smart contracts using this pallet.

There's one-to-one mapping from Substrate accounts and EVM external accounts defined by a conversion function.

## EVM pallet vs. Ethereum network

The EVM pallet should be able to produce nearly identical results as the Ethereum mainnet, including gas cost and balance changes.

Observable differences include:

- The available length of block hashes may not be 256 depending on the configuration of the Substrate System pallet in the Substrate runtime. For more information on the System pallet, see the [System pallet rust documentation](/rustdocs/latest/frame_system/index.html#system-pallet).
  
- Difficulty and coinbase, which do not make sense in this pallet and are currently hard-coded to zero.

Unobservable behaviors, such as state root are not the same when using the EVM pallet. The transaction / receipt format is also different. However, given one Ethereum transaction and one Substrate account private key, you should be able to convert any Ethereum transaction into a transaction compatible with this pallet.

The gas configurations are currently hard-coded to the Istanbul hard fork. It can later be expanded to support earlier hard fork configurations.

Substrate is built to enable you to extend what's provided out of the box. We encourage further development of alternative smart contract platforms on top of the Substrate runtime. Use these pre-built pallets to inform how you might design your own system or how you could port over an existing system to work on a Substrate-based chain.

## Resources

- To view the reference documentation for the EVM pallet, see https://docs.rs/pallet_evm and https://docs.rs/fp-evm/.
  
- To view the reference documentation fir SputnikVM's `evm` crate, see https://docs.rs/evm/.