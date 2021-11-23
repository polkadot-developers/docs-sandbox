
# Smart contracts development
This article gives an overview of the different ways to implement smart contracts for Substrate-based blockchains.

A traditional smart contract platform allows users to publish additional logic on top of some core blockchain logic. Because smart contract logic can be published by anyone, including malicious actors and inexperienced developers, there are a number of intentional safe guards built around these public smart contract platform.

Some examples are:

- **Fees**: Ensuring that contract execution incurs fees for the computation and storage it forces on the computers running it, so it can't abuse block creators.

- **Sandbox**: A contract is not able to modify core blockchain storage or the storage of other contracts directly. It's power is limited to only modifying it's own state, and the ability to make outside calls to other contracts or runtime functions.

- **Storage Deposit**: A contract takes up space on the blockchain, and should be charged for taking up space on the hard drives of the nodes. This ensures that people don't take advantage of "free, unlimited storage."

- **Reversion**: A contract can be prone to logical errors. The expectations of a contract developer are low, so extra overhead is added to support reverting transactions when they fail, so no state is updated when things go wrong.

These different overheads makes running contracts slower and more costly, but attractive to certain developers.

Contracts allow your community to extend and develop on top of your runtime logic without needing to go through all of the proposals, runtime upgrades, etc... . It can even be used
as a testing ground for future runtime changes, but done in a way that isolates your network from any of the growing pains or errors which might occur.

**Substrate Smart Contracts:**

- Are inherently safer to the network.
- Have built in economic incentives against abuse.
- Have computational overhead to support graceful failures in logic.
- Have a lower bar to entry for development.
- Enable fast-pace community interaction through a playground to write new logic.

For a comparison of Smart Contract and runtime development, see [link](link).