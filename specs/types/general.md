# Types

Types define how we interpret and interact with SSZ data.
In addition to this they provide readability, and guard users from mixing up data or processing input beyond intended limits.

## Readability

Types can be aliased to more specific types, good use of type aliasing can make a data-structure much clearer.
E.g. `BLSSignature` instead of `Vector[byte, 96]`.

## Default values

Part of the promise of types is that data structures have defaults, avoiding `null` (a.k.a. "the billion dollar mistake").
Default values are recursive; elements in composite types such as containers are initialized with their respective default initializations.

## Merkle proofs

Every type deterministically describes the shape of the Merkle tree representing the type:
 reasoning about the shape of a proof is abstracted away by the typing layer.
Most types do so statically: the shape can be constructed on compile time, and navigation is stable (See [generalized indices](../navigation/generalized_indices.md)).
Some types (e.g. those based on Sparse Merkle Trees) are not static, but are deterministic based the contents of the proof.

Mapping a *valid (to the type)* merkle tree to that same type is bijective:
- No two different values *of the same type* can merkleize to the same root
- No two roots can be derived for the same value *of the type used for the root*.

Do note that some different types may merkleize to the same root:
 - Intentionally: see [summaries and expansions](../navigation/summaries_expansions.md).
 - Or because of different types with the same structure:
   Two values *of different types* can merkleize to the same root, e.g. a `uint256(123)` and `uint8(123)` have the same root.
   Or more exceptionally, a `Container` with 4 `Bytes32` fields can have the same root as a `Vector[uint64, 16]`.
   Hence, typing is essential to consuming a proof for data, and should not be chosen arbitrarily by another actor (if a different type has any meaning to the application of the proof).

## Representation

Mapping *valid* instances of the same type to a byte sequence is bijective:
- Serialization: Any two different values *of te same type* cannot have the same representation.
- Deserialization: Any *valid* representation *of a given type* cannot be interpreted as two different values *of that same type*.

Mapping *any* instance of a type to any byte sequence is *injective and non-surjective*:
- Serialization: All type instantiations can be serialized to a unique (to the type) value.
- Deserialization: not all byte sequences are a valid representation for a given type, because of constraints such as:
   - representation length (See [fixed length](../representation/fixed_variable_size.md))
   - element count (See [list limits](./complex.md#list-limits))
   - [element offsets](../representation/sequences.md#offsets)
   - delimiters (See [bitlists](./bitfields.md#bitlist))
   - selectors (See [union](./union.md))
   - more, this is not an exhaustive list.
