TODO
**Trie abstraction**

Substrate uses a Base-16 Modified Merkle Patricia tree ("trie") from [`paritytech/trie`](https://github.com/paritytech/trie) to provide a trie structure whose contents can be modified and whose root hash is recalculated efficiently.

Substrate-based chains have a single main trie, called the state trie, whose root hash is placed in each block header. 
This is used to easily verify the state of the blockchain and provide a basis for light clients to verify proofs.

This trie only stores content for the canonical chain, not forks. 
There is a separate [`state_db` layer](/rustdocs/latest/sc_state_db/index.html) that maintains the trie state with references counted in memory for all non-canonical blocks.
All trie nodes are stored in the database and part of the trie state can get pruned, i.e. a key-value pair can be deleted from storage when it is out of pruning range for non-archive nodes. 

Substrate also provides an API to generate new [child tries](./todo-link-design) with their own root hashes that can be used in the runtime.
Learn about ways to design and implement the storage for your chain [here](./todo-link-design).
