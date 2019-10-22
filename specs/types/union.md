# Union

Type: `Union[type_0, type_1, ...]`

Default: `default(type_0)`

A [`Union`](https://en.wikipedia.org/wiki/Union_type) provides a the ability represent a set of predetermined types in the same tree and serialization position.

A special `null` type may be used as first type parameter to emulate an `Option`, any other type parameter than the first MUST not be `null`.

An Union is considered to have a dynamic encoding-size, even if all the selectable options have the same type or happen to have the same serialized byte length.

## Representation

Serialization is defined as an `uint32` for the type index, followed by the serialization of the selected option.

Merkleization is defined as `Mix(x, i)` where `x` is the root of the selected option with index `i` (padded to 32 bytes, effectively an `uint256`).
 