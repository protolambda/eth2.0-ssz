# Summaries & Expansions

Let `A` be an object derived from another object `B` by replacing some of the (possibly nested) values of `B` by their `hash_tree_root`.
Because of this substitution, `hash_tree_root(A) == hash_tree_root(B)`.

We say `A` is a **"summary"** of `B`, and that `B` is an **"expansion"** of `A`. The replaced values are the **"details"** of `B` with respect to `A`.

Summary instances expand to at most one instance of a given expansion: 
 the detail of the summary is a strict subset of that of the expansion and the difference cannot be altered without changing the summary root.

## Transparency

Not all expansions may be defined ahead of time; SSZ does not limit a root to be expanded into more detail.

This is a pathway for different use-cases:
- Forward-compatible hash-tree-roots: a summary definition can stay unaltered when an expansion definition is changed.
- Merkle proofs can navigate as far as roots can be expanded:
    - State roots, crosslink-data roots, block roots and any system root can be taken and expanded, and their data can be too.
      All system data can be proven through a single merkle proof one way or another.
    - User-level roots (e.g. beacon block graffiti) may also be given user-level expansion definitions to soft-fork in new functionality.

Warning: this is all safe when working with static expansion definitions, but pre-image attacks should be considered when working with dynamic expansions.
Someone may exploit an unexpected expansion, e.g. querying a binary path beyond the intended depth. See types below to avoid such arbitrary expansions.

## As types

A summary type can be defined by taking the expansion and substituting the types of the elements to summarize with `Bytes32` to reflect their `hash_tree_root`.
Or vice versa an expansion can be defined based on a summary type.

Some details can also be summarized with a `signing_root`. An implementer has two options here:
1. Ignore the final signature field, the root of this container in the summary type can be annotated to do this.
2. Exclude the signature field from the expansion definition to begin with.

### Example

In the Eth2 beacon chain, a [`BeaconBlock`](https://github.com/ethereum/eth2.0-specs/blob/master/specs/core/0_beacon-chain.md#beaconblock)
is an expansion type of [`BeaconBlockHeader`](https://github.com/ethereum/eth2.0-specs/blob/master/specs/core/0_beacon-chain.md#beaconblockheader).
Note that a `BeaconBlockHeader` objects uniquely expands to a `BeaconBlock` object.
