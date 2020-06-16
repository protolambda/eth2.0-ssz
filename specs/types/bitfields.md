# Bitfields

Bitfields are collections of booleans, backed by sequences of bytes: a bit at sequence index `i` is put into byte `i // 8` and matches `1 << (i % 8)` within that byte.

## Bitfields vs Collections

Bitfields represent boolean data in a packed form, as opposed to lists or vectors of booleans that do not efficiently utilize space.

E.g. the representation of a `Bitvector[N]` is 8 times smaller than a `Vector[boolean, N]`.
And because of [chunkification](../merkleization/chunkify.md) adapting to bitfields better than boolean collections, the Merkle tree is also 8 times smaller.  


## Bitvector

Type: `Bitvector[N]`

Default value: `N` bits, all set to `0`

Note that a `Bitvector[0]` is an illegal type, since fixed-length types many not have 0 byte-length representations.

### Representation

A fixed-length sequence of `N` bits, packed into `(N + 7) // 8` bytes.

If `N` is not a multiple of 8, the remainder of bits (high end of byte) in the *last* byte in the sequence MUST be zeroed.

### Merkleization

`root = merkle_subtree(chunkify(bitvector_value))`,
 see [`merkle_subtree`](../merkleization/subtree_merkleization.md), [`chunkify`](../merkleization/chunkify.md).

Note: A bitvector is merkleized the same as serializing it, and then merkleizing it as a `Vector[byte, ((N + 7) // 8)]`.


## Bitlist

Type: `Bitlist[N]`

Default value: `0` bits, i.e. an empty bitlist.

A dynamic-length sequence, with a limit of `N` bits, packed into bytes.
The limit here reflects the limit behavior as described by `List`: it enforces input constraints, and stabilizes merkleization depth. 

### Representation

From the offset coding the length (in bytes) of the bitlist is known; the bitlist cannot have redundant zero bytes.
An additional `1` bit is appended so that the length in bits will also be known: "the delimiting bit".

This delimiting `1` bit is put in what would effectively be the bitfield index `bit_length(bitlist_value)`.
Note that for an empty bitlist that would be the first bit at index 0: A single zeroed byte, or empty bytes, is illegal as bitlist representation. 

Because of this delimiting bit, the total byte length for serialization purposes is: `(((N + 1) + 7) // 8) == ((N // 8) + 1)`

### Merkleization

For merkleization, the length of the bitlist is mixed in with the root, and hence the delimiting bit is not used for merkleization.
Similarly to a `List`, the subtree is padded to fit the limit of the bitlist.

`root = mix_in_num(merkle_subtree(chunkify(bitlist_value), limit=chunk_count(Bitlist[N])), bit_length(bitlist_value))`,
 see [`merkle_subtree`](../merkleization/subtree_merkleization.md),
  [`chunkify, chunk_count`](../merkleization/chunkify.md) and [`mix_in_num`](../merkleization/mixin.md). 

