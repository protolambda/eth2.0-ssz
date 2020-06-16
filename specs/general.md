Stage: Finished


# General

This document outlines general things such as the principles and design choices behind SSZ.
It does not spec any SSZ behavior, and can be ignored as an implementer of SSZ,
but may provide valuable insight into the bigger picture of SSZ. 

## Principles of SSZ

To describe design choices in SSZ as a short list of properties,
 loosely based on the desires outlined in the [SSZ specs Readme](../README.md):

- Simple
- Bijective
- Compact
- Merkle-first
- Efficient to traverse

These properties are based on experience with Eth1 smart contracts, blockchain protocol engineering in general,
 and feedback from lots of different implementers and stake-holders in Eth2.

Other properties such as Security, readability and common encoding features are desired as well,
 but do not make SSZ principally different from other encodings.

The main use-case of SSZ is to provide a consistent encoding and merkleization framework for the core of the Eth2 protocol.
However, use outside of the core protocol, such as in smart contracts or layer-2 solutions, is also considered and factored into the design.


### Simple

SSZ is meant to map well to common raw datatypes, and avoid twiddling with bits or nibbles in serialization.
- It has common basic data types
- It has fixed-length types to avoid unnecessary lengths/offsets
- Types to describe the structure

And compared to RLP, an encoding previously used in Ethereum:
RLP had some simplicity, but lacks in typing and merkleization features, making it more difficult to use than desired.


### Bijective

This property expands to two conditions:
- No two different representations can exist for the same value of a single type.
- No two different values of the same type can be have the same representation.

This applies to both merkleization and serialization.

However, this is not maintained between different types:
- Different types may still have overlapping representations in merkleization or serialization.
    - Serialization example: `Vector[uint16, 4]`, `Vector[uint32, 2]`, `Vector[uint64, 1]`, `uint64` are all fixed-length and 8 bytes.
      Another more verbose example: a fixed-length container with different fixed-length fields, but the same total size.
    - Merkleization example: A `Container` with a `Vector[uint32, 8]` and `uint64` field has the same merkleization structure as a `List[uint64, 4]`.
      Or focusing on the hash-tree-root only: a `Vector[byte, 32]` can have any hash-tree-root value. 
- Expansions, summaries and partials can be thought of as different types because of their differences in completeness,
  so they may also break bijective. By definition, they break hash-tree-root the beijective property.


### Compact

SSZ achieves aims for compactness in both serialization, as merkle-proofs.

In general in the serialization or merkleization, no information that can be determined from the type already is embedded.
E.g. the names of container fields are not part of the serialized data, unlike a format such as JSON.
This means that the reader of the data has to know the type, but this is almost a given, since input data is untrusted,
 and validated and parsed using a pre-determined type before further processing. 

For serialization of fixed-length types, the elements are packed together, and do not result in any extra bytes when used as elements in dynamic types such as lists.
A good example of this is the validator registry in the Eth2 `BeaconState`: hundreds of thousands of unique validators can be packed into a big list, with 0 overhead. 

For serialization of dynamic types, it is more of a trade-off with traversal speed, but 4-byte offsets were adopted as a way to enable fast random access of list elements,
 while keeping the size relatively low. Offsets are only used for dynamic-length element types, whose contents are often significantly bigger than 4 bytes.

For merkleization, a binary tree backs every merkle structure. Since the branching factor is lower than the previously used Merkle Patricia Tree, less nodes are required to reach into a leaf of a merkle tree.
And on the application level, an arbitrary key-value store is avoided, since a `List` can be packed together better, and have a smaller key depth,
 thus more efficiency in multi-proofs and avoiding the cost of unbalanced tree shapes.


### Merkle-first

The intention of having a custom type system is also to give anything that can be interpreted by the protocol a sound single generalized merkle-root.

And not just a merkle-root, but also features that make proofs small, avoid complexity in merkleization,
 and make it as flexible as possible to build and interpret proofs for a data-structure.

This merkle-first also enables advanced features such as [summaries and expansions](./navigation/summaries_expansions.md),
 [partials](./partials/partials.md), [generalized indices](./navigation/generalized_indices.md) and [multi-proofs](./merkleization/merkle_proofs.md).   


### Efficient to traverse

Efficient traversal is a feature that was later introduced into SSZ with the creation
 of [Simple Offset Serialization (SOS)](https://gist.github.com/karalabe/3a25832b1413ee98daad9f0c47be3632).
This guarantees a `O(log(N))` lookup speed for deeply nested structures. And offsets even enable `O(1)` random access in lists of dynamic-length elements.

The trade-off here is the use of offsets, which are based on the sizes of subsets of the structure, making streaming of dynamic-length SSZ data impossible.

However, since SSZ also has fixed-length types, and is very efficient about typing data structures,
 often the size of a data structure can be computed much faster than actually serializing it.

The primary part of the `BeaconState`, the validator registry, is a good example of this again.
The size is computed with `length * fixed_element_size`, instead of writing every single element.
And access to any validator by index is only a bit of pointer math.

By doing the size computation separately, some kind of streaming where the size is given, or computed pre-computed before encoding, is possible.
Reducing the intermediate memory requirements in components such as the Eth2 networking request-response code.

The merkle tree is also efficient for lookups, compared to other trees, even though it is a binary tree:
 [generalized indices](./navigation/generalized_indices.md) can statically describe the tree node location of any element path.
This allows any merkle-node lookup to be optimized to a `O(log(N))` operation,
 where `N` is the abstract data size (SSZ does not force a uniform data-structure),
 and where `log(N)` matches the length of the generalized index.
And the purely static parts of the path can even be computed at compile time,
 to improve lookup speeds without writing specialized verbose manual lookup routines.
