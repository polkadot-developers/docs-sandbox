Section: Design
Sub-section: Economic models
Type: reference + conceptual

[ WIP ]
## Hybrid consensus

See: https://github.com/paritytech/substrate/issues/1304
Describes considerations for swappable consenus.
A good starting point for designing hybrid consesnus systems.

Note: block authoring engines must be made aware of blocks that are finalized so that they don't waste time building on top of blocks that will never be in the canonical chain.
