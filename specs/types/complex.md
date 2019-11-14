# Complex types

Complex types are types that can hold multiple values at the same time, with usage similar to that of a `struct` in popular programming languages.

The complex types are all serialized like [Sequences](../representation/sequences.md).

A complex object is considered fixed size if all of the contained elements are fixed size, and type has a fixed element count (e.g. Lists cannot be fixed size).

Elements can be read and written in `O(1)`, as they are indexed (using an [offsets prologue](../representation/sequences.md#offsets) if `T` is variable size).

## Vector

Type: `Vector[T, N]`

Default value: `[default(T)] * N`, i.e. all elements set to their default value.

A Vector is a sequence of elements, all of the same type `T`, and of fixed length `N`.

Empty vectors (`N = 0`) are illegal.

### Representation

Serialized and deserialized like a [Sequence](../representation/sequences.md) of the `values`, all of type `T`.

### Merkleization

`root = merkle_subtree(chunkify(values))`, see [`merkle_subtree`](../merkleization/subtree_merkleization.md) and [`chunkify`](../merkleization/chunkify.md)


## Lists

Type: `List[T, N]`

Default value: `[]`, i.e. empty list.

A List is a sequence of elements, all of the same type `T`, and can be any length from `0` to `N`.

### List Limits

The maximum list length is preset as `N` and called the "list limit".

This limit is preset for two primary reasons:
- Stable merkleization: there are no variable numbers in the hash-tree-root definition.
- Strong garantuees on inputs: lists should never contain more elements than their limit was designed for.

#### Allocation

For small list limits, the limit type information may help to optimize for a single allocation of full list capacity.
However, list limits can be arbitrarily high as the cost for serialization and merkleization is `O(n)`:
 - the limit is not padded to in serialization
 - `O(log(N))` [zero-hashes](../merkleization/hashing.md#zero-hashes) may need to be merged during merkleization.
Hence, lists should not be allocated to their full limit for larger numbers.

### Representation

Serialized and deserialized like a [Sequence](../representation/sequences.md) of the `values`, all of type `T`.

Note: A list is by definition variable-size, but this does not necessarily mean its elements are.

### Merkleization

Note: the contents subtree (not including the length mix-in) is padded to fit the limit of the bitlist.

`root = mix_in_num(merkle_subtree(chunkify(values), limit=chunk_count(Lit[T,N])), length)`,
 see [`merkle_subtree`](../merkleization/subtree_merkleization.md),
  [`chunkify, chunk_count`](../merkleization/chunkify.md) and [`mix_in_num`](../merkleization/mixin.md). 


## Container

Type: `Container[(<K_i>: <T_i>)+]`

Default value: `Container[(<K_i>: <T_i>)+](default(T_i)...)`, i.e. all fields set to their default value.

A Container is a predefined sequence of fields, each field can be defined as any type `T_i` independently from the other fields, and is identified by a unique (relative to the other fields) name `K_i`.

Note that field names are not included in serialization or merkleization: a Container is not self-describing.

An empty container, i.e. 0 fields, is illegal.

### Representation

Serialized and deserialized like a [Sequence](../representation/sequences.md) of the `fields`, each of their own type `T_i`.

### Merkleization

`root = merkle_subtree(chunkify(fields))`,
 see [`merkle_subtree`](../merkleization/subtree_merkleization.md), [`chunkify`](../merkleization/chunkify.md).

