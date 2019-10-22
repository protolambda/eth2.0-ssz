# Basic types

The basic types all strictly follow the basic-type principles:
- 1 to 32 bytes long, for merkleization purposes
- a power of 2 bytes long, for packing/alignment purposes
- fixed length
- no two ways to describe one value of the same type.

## Unsinged integers

Type: `uintN`, where `N` can be: `8, 16, 32, 64, 128, 256`.

Alias: `uint8 <-> byte`

Default value: `0`

A `N`-bit unsigned integer.

### Representation

The integers have a little-endian representation.

### Merkleization

The integers, represented as bytes, are padded on the right side with zeroed bytes to a total of 32 bytes for merkleization.
Note:
 - Some complex types pack integers in chunks, and reduce the merkleization overhead.
 - Because of the little-endianness and right-padding, equal integers of different bit-sizes all map to the same 32 bytes value.


## Booleans

Type: `boolean`

Alias: `bit`

Default value: `False` 

### Representation

A single byte: 1 (i.e. `0b00000001`) for `True`, and 0 (i.e. `0b00000000`) for `False`.

To have an injective mapping from serialized byte to a value, the non-utilized bits of the byte MUST all be zero bits.

### Merkleization

The boolean represented as byte is merkleized exactly like `byte`, including the ability to pack (but only to `byte` precision, refer to bitfields for more efficient packing).

