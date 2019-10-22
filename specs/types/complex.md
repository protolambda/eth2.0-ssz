| Standardization stage | 4 | 

# Complex types

Complex types are types that can hold multiple values at the same time, with a familiar usage.

The complex types are defined based on purer types called "kinds", primarily `Series` and `Compound`, described in the [Base model](../base/model.md). 

A compound object is considered fixed size if all of the contained elements are fixed size, and the container type has a fixed element count (e.g. Lists cannot be fixed size).

An element within a compound object can however be fixed-size; the relation is one way only.

## Vector

Type: `Vector[T, N]`

Default value: `[default(T)] * N`, i.e. all elements set to their default value.

A Vector is a sequence of elements, all of the same type `T`, and of fixed length `N`.

Empty vectors (`N = 0`) are illegal.

Elements can be read and written in `O(1)`, as they are indexed (using an offsets prologue if `T` is variable size).

### Functionality

For all Vector model purposes, `len(values)` MUST be equal to `N`.

- **serialize**: as a `Series[T, N](values)`, i.e. each element value as type `T`
- **deserialize** as a `Series[T, N](values)`
- **hash-tree-root**: as a `Series[T, N](values)`


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

For small limits, the type information may help to optimize for a single full list allocation.
However, list limits can be arbitrarily high as the cost for serialization and merkleization is `O(n)`:
 - the limit is not padded to in serialization
 - `O(log(N))` [zero-hashes](TODO) may need to be merged during merkleization.
Hence, lists should not be allocated to their full limit for larger numbers.

### Functionality

Note that the typing is constant (`T, N` are both constants) to enforce limits on compile time.
  
- **serialize**: as a `Series[T, N](values)`, i.e. each element value as type `T`.
- **deserialize** as a `Series[T, N](values)`
- **hash-tree-root**: as a `Mix(Series[T, N](values), length)`, i.e. the length is mixed with the hash-tree-root of the list as a `Series`.


## Container

Type: `Container[(<K_i>: <T_i>)+]`

Default value: `Container[(<K_i>: <T_i>)+](default(T_i)...)`, i.e. all fields set to their default value.

A Container is a predefined sequence of fields, each field can be defined as any type `T_i` independently from the other fields, and is identified by a unique (relative to the other fields) name `K_i`.

Note that field names are not included in serialization or merkleization: a Container is not self-describing.

An empty container, i.e. 0 fields, is illegal.

### Functionality

- **serialize**: as a `Compound[T_f...](field_values)`, i.e. each of the fields with the respective type.
- **deserialize** as a `Compound[T_f...](field_values)`
- **hash-tree-root**: as a `Compound[T_f...](field_values)`

