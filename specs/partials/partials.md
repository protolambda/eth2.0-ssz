Stage: Proposal


# Partials

One of the main uses of multi-proofs are interactions with verifiable pre-states: a merkle multi-proof verifies the data, and the data is used in a computation.

With SSZ, this data can be (partially) typed, and interacted with, without dealing with the underling merkleization.

This separation of the typed interface and the multi-proof enables different abstractions and use-cases:
- The multi-proof is abstracted away, merkleization and lookups may be implemented by specialized proof-backings.
- The verification, lookups and modifications are all just an interface feature;
   environments can decide on when and how to implement this.
   - E.g. an aggregate proof can be verified first, and subsequent individual processes can be given a scoped interface to interact with the proof data.
- The typing interface can be extended, and new proof-backings may be introduced in the future.

Note that the concept of [Summaries and expansions](../navigation/summaries_expansions.md) is a subset of that of partials:
instead of summarizing a type nicely at the edges of the type (i.e. summarizing fields or elements),
arbitrary subsets of an element can be summarised as a whole.

Deciding between a partial and a summary is not difficult:
- If you need a small subset, or a very arbitrary subset, utilize the expressive flexibility of a partial.
- If you only need to leave out a single part, leave it out by summarizing it, and keep the type as static as possible.

## Core functionality

### `PartialType(base_type: SSZType, paths: Set[Path])`

The idiomatic syntax here differs strongly per implementation, as meta-programming is not a first-class citizen in every language.
The core idea however, is to take an existing type, define the subset (e.g. a list of [ssz paths](../navigation/paths.md)) of information you need,
 and construct a type structure to interact with multi-proofs. 

### `interface(partial_type: PartialType, proof_backing: ProofBacking) -> Partial`

Given a partial type structure, a proof-backing can be wrapped and create a partial.

### `scope(partial_type: PartialType, paths: Set[Path]) -> PartialType`

Since not every `base_type` can be provided in the same level of detail to create a partial-type
 (e.g. only select a specific set of indices of a list), additional changes to the scope should be possible to make. 
The scope may may also be implemented with language-specific features (e.g. annotations, struct-tags, etc.)

### `compute_root(partial: Partial)`

Construct the hash-tree-root (or possibly signing-root); effectively computing the root of the proof-backing. Used for verification purposes.

Note that there is no signing root function; a Partial without the last Container field can be used to ignore the last field. 
A proof can also not back both a signing-root and hash-tree-root unless the last field would be included as witness or leaf, which is handled better by scoping the type manually. 

## Read-only partials

Some partials may not be meaningful to modify; in this case a proof-backing optimized for reads, and a read-only partial could be implemented.

