# Basic types

The basic types all strictly follow the basic-type principles:
- 1 to 32 bytes long, for merkleization purposes.
- A power of 2 bytes long, for packing/alignment purposes. See [chunkification](../merkleization/chunkify.md).
- [Fixed length](../representation/fixed_variable_size.md)

## Unsinged integers

Type: `uintN`, where `N` can be: `8, 16, 32, 64, 128, 256`.

Alias: `uint8 <-> byte`

Default value: `0`

A `N`-bit unsigned integer.

### Representation

The integers have a little-endian representation, and represented in their respective byte sizes.

### Merkleization

The integers, represented as bytes, are padded on the right side with zeroed bytes to a total of 32 bytes for merkleization.
Note:
 - Some complex types pack smaller integers together into 32 bytes, to reduce the merkleization cost.
 - Because of the little-endianness and right-padding, equal integers of different bit-sizes all map to the same 32 bytes value.

### Size

`size_of(uintN): N / 8`


## Booleans

Type: `boolean`

Alias: `bit`

Default value: `False` 

### Representation

A single byte: 1 (i.e. `0b00000001`) for `True`, and 0 (i.e. `0b00000000`) for `False`.

To have a one-to-one correspondence from value to serialized byte, the non-utilized bits of the byte MUST all be zero bits.

### Merkleization

The boolean represented as byte is merkleized exactly like `byte`, including the ability to pack (but only to `byte` precision, refer to bitfields for more efficient packing).

### Size

`size_of(bool): 1`
