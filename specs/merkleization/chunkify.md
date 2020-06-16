# Chunkify

`chunks` are `Bytes32` intermediate merkle values, used for e.g. [subtree merkleization](./subtree_merkleization.md) leafs.

## `chunkify`

### Complex sequences

Sequences that are not homogeneously typed, or not of basic values or bits, are not packed.
Instead, the `chunks` are the `hash_tree_root`s for each of the values.  

### Basic sequences / bitfields

To convert a homogeneously typed sequence of basic values or bits into chunks, the values are packed.

Chunkification of `elements` of a sequence type `T` is defined as following:

#### For basic elements:

Given ordered `elements` of the same basic type:
 - Partition the elements into chunks: split the elements in groups of consecutive `32 / size_of(B)` elements.
    - The last partition may not be full.
 - Serialize the elements in each partition, and tightly pack the partition into a chunk (no padding between elements).
    - If the last-partition is not full, it is right-padded with zero bytes.

#### For bitfields

 - Serialize the Bitlist or bitvector.
 - The length-delimiting bit for bitlists is excluded: bitlists mix-in the bit-length and do not need the delimiting bit.
 - Right-pad the serialized bytes to a multiple of 32.
 - Partition into chunks: split the bytes into groups of consecutive `32` bytes

## `chunk_count`

`chunk_count(type)`: calculate the amount of leafs for merkleization of the type.
 * all basic types: `1`
 * `Bitlist[N]` and `Bitvector[N]`: `(N + 255) // 256` (dividing by chunk size, rounding up)
 * `List[B, N]` and `Vector[B, N]`, where `B` is a basic type: `(N * size_of(B) + 31) // 32` (dividing by chunk size, rounding up)
 * `List[C, N]` and `Vector[C, N]`, where `C` is a composite type: `N`
 * containers: `len(fields)`

