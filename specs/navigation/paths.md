# Paths

Paths are the human-readable variant of [Generalized Indices](./generalized_indices.md).
To make paths human-readable, and derive SSZ properties like generalized indices, type information is required.

## Syntax

The human-readable form is simply a `/`-separated list of path components, starting with the name of the root type.

Example:
```
MyType/some_field/abc_list/123/foobar
```

The Python-like SSZ spec overloads the `/` operator to present paths:

```python
ssz_path(MyType) / 'some_field' / 'abc_list' / 123 / 'foobar'
```

`ssz_path(ssz_type)` here creates a root path element (type `Path`), to separate path logic from the root (`MyType` here) itself.

`Path` objects can also be appended to other paths, but only if the type of the root matches the type of the leaf we are appending to:

```
a = ssz_path(MyType) / 'some_field' / 'abc_list'
a.leaf_type() == AbcListType

b = ssz_path(AbcListType) / 123 / 'foobar'

combined = a / b
combined == ssz_path(MyType) / 'some_field' / 'abc_list' / 123 / 'foobar'
```

## Interface

Typed paths offer the following interface:

`.leaf_type()`: get the type of the end (leaf) of the path.
`.root_type()`: get the type of the start (root) of the path.
`.parent()`: get the path, without the last component (leaf).
`.generalized_index()`: get the generalized index of the path.

## Paths

Paths are the preferred form to express accesses of leaf values in source code:
 in the rare event of changes to a SSZ type (e.g. a fork), 
 the amount of fields or elements may not be the same, and change the generalized index.
This would break a generalized index, but not the path.

However, such changes should be avoided, or prepared for by padding the type (and thus reserving the merkleization space for expected future changes).

