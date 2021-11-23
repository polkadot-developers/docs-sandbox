# Runtime development
Substrate runtime development has the intention of producing lean, performant, and fast nodes. 

You have full control of the underlying logic that each node on your network will run. You have full access to each and every storage item across all of your pallets, which you can modify and control.

With this level of control, you can even brick your chain with incorrect logic or poor error handling. Runtime engineers have much more responsibility for writing robust and correct code, than those deploying Smart Contracts.

When you develop using Substrate runtime, you must provide the protections for the overhead of transaction reverting and implicitly introduce any fee system to the computation of the nodes your chain runs on. This means while you are developing runtime functions, you must correctly assess and apply fees to different parts of your runtime logic so that it will not be abused by malicious actors.

**Substrate Runtime Development:**

- Provides low level access to your entire blockchain.
- Removes the overhead of built-in safety for performance,
  giving developers increased flexibility at the cost of increased responsibility.
- Raises the entry bar for developers, where developers are
  not only responsible for writing working code but must constantly check to avoid writing broken code.
- Has no inherent economic incentives to repel bad actors.

For a comparison of Smart Contract and runtime development, see [link](link).