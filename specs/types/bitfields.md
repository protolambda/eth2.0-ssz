# Bitfields

Bitfields represent boolean data in a packed form, as opposed to lists or vectors of booleans that do not efficiently utilize space.

Bitfields are backed by sequences of bytes: a bit at sequence index `i` is put into byte `i // 8` and matches `1 << (i % 8)` within that byte.

## Bitvector

Type: `Bitvector[N]`

Default value: `N` bits, all set to `0`

### Representation

A fixed-length sequence of `N` bits, packed into `(N + 7) // 8` bytes.

If `N` is not a multiple of 8, the remainder of bits (high end of byte) in the *last* byte in the sequence MUST be zeroed.

The resulting bytes are serialized and merkleized the same as a `Vector[byte, ((N + 7) // 8)]`.


## Bitlist

Type: `Bitlist[N]`

Default value: `0` bits, i.e. an empty bitlist.

A dynamic-length sequence, with a limit of `N` bits, packed into bytes.
The limit here reflects the limit behavior as described by `List`: it enforces input constraints, and stabilizes merkleization depth. 

### Representation

#### Serialization

From the offset coding the length (in bytes) of the bitlist is known; the bitlist cannot have redundant zero bytes.
An additional `1` bit is appended so that the length in bits will also be known: "the delimiting bit".

This delimiting `1` bit is put in what would effectively be the bitfield index `bit_length(bitlist_value)`.
Note that for an empty bitlist that would be the first bit at index 0: A single zeroed byte, or empty bytes, is illegal as bitlist representation. 

Because of this delimiting bit, the total byte length for serialization purposes is: `(((N + 1) + 7) // 8) == ((N // 8) + 1)`

#### Merkleization

For merkleization, the length of the bitlist is mixed in with the root, and hence the delimiting bit is not used for merkleization.

Merkleization is performed as in `Mix(Series[byte, ((N + 7) // 8)](values), length)` 
 where `values` would be the bits packed into bytes without the delimiting bit, as in `Bitvector[bit_length(bitlist_value)]`.

