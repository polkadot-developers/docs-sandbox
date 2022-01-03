Substrate provides tools to build all types of blockchain networks and protocols. 
Network architectures can fall into the following categories:

- Relay chains: these Substrate blockchains are desgined to provide decentralized security to other chains connected to it. 
Polkadot is an example of such a chain.
- Parachains: parachains are blockchains built to connect to some relay chain. Such chains need to implement the same consenus protocol as the relay chain they target.
- Solo-chains: these Substrate blockchains implement their own security protocol and are independant from any other chain.
They could connect to other Substrate blockchains but would have to implement their own communication channels.

For every type of chain, there are two types of testing networks: economically bearing test networks called canary networks or non-economically bearing test networks.

[ TODO: Diagrams / illustrations for each type of chain ]