# Classic merkle proof backing

This format is not optimal for newer multi-proof use cases, but does offer:
- Simplicity
- Backwards compatibility
- Consistency with regular single-leaf merkle proofs

## Contents

- Description: Generalized indices of the leaves `[optional]`
- Leaf data: Leaf chunks `[optional]`
- Witness data: Witness chunks

The separation of content types, and the order of the chunks in the witness data, makes a the backing encoding compatible with a standard single leaf merkle proof. top-to-bottom branch 

### Description
 
The description of contents is kept separate from the actual witness data, and an optional part of the format:
 if the both the proof producing party and and proof consuming part have the same expectation of contents, there is no need to repeat it in communication.

Also this fits with the existing convention for Merkle proofs where the index is separate from the proof.

#### Helper indices

The generalized indices of the witness data (the "helper indices") can be derived from those of the leaves, and are thus not encoded.
The below Python code demonstrates how this can be done:

```python
def get_branch_indices(tree_index: GeneralizedIndex) -> Sequence[GeneralizedIndex]:
    """
    Get the generalized indices of the sister chunks along the path from the chunk with the
    given tree index to the root.
    """
    o = [generalized_index_sibling(tree_index)]
    while o[-1] > 1:
        o.append(generalized_index_sibling(generalized_index_parent(o[-1])))
    return o[:-1]
```

```python
def get_path_indices(tree_index: GeneralizedIndex) -> Sequence[GeneralizedIndex]:
    """
    Get the generalized indices of the chunks along the path from the chunk with the
    given tree index to the root.
    """
    o = [tree_index]
    while o[-1] > 1:
        o.append(generalized_index_parent(o[-1]))
    return o[:-1]
```

```python
def get_helper_indices(indices: Sequence[GeneralizedIndex]) -> Sequence[GeneralizedIndex]:
    """
    Get the generalized indices of all "extra" chunks in the tree needed to prove the chunks with the given
    generalized indices. Note that the decreasing order is chosen deliberately to ensure equivalence to the
    order of hashes in a regular single-item Merkle proof in the single-item case.
    """
    all_helper_indices: Set[GeneralizedIndex] = set()
    all_path_indices: Set[GeneralizedIndex] = set()
    for index in indices:
        all_helper_indices = all_helper_indices.union(set(get_branch_indices(index)))
        all_path_indices = all_path_indices.union(set(get_path_indices(index)))

    return sorted(all_helper_indices.difference(all_path_indices), reverse=True)
```

#### Leaf Indices

The positions of the leaves are encoded in bit-alphabetical left-to-right order:

```python
def split_by_root(ints, depth):
    t, l, r = [], [], []
    for i in ints:
        if i.bit_length() < depth:
            t.append(i)
        elif (i >> (i.bit_length() - depth)) % 2 == 1:
            r.append(i)
        else:
            l.append(i)
    return t, l, r

def alphasort(ints, depth=2):
    if len(ints) <= 1:
        return ints
    t, l, r = split_by_root(ints, depth)
    return t + alphasort(l, depth+1) + alphasort(r, depth+1)
```

##### Example

```
Tree:
   1
 2   3 
4 5 6 7

>>> alphasort([1,2,3,4,5,6,7])
[1, 2, 4, 5, 3, 6, 7]
```

Note that not all generalized indices would be encoded, only those of the leaves the proof is targetting.
Those that do get encoded will be ordered correctly for a single-pass left-to-right tree traversal for verification.

### Leaf data

The leaf data is optional because in some cases the leaf data may also be known by both parties, or encoded elsewhere outside of the proof.

The 32 byte chunks of data are ordered the same as the leaf indices of the description part are.

### Witness data

Witness data is the essential part of the proof, and kept separate from the leaf data.

Like leaf data, witness data is also sorted in bit-alphabetical left-to-right order (in respect to the generalized indices matching the chunks).

## Verification

```python
def calculate_multi_merkle_root(leaves: Sequence[Bytes32],
                                proof: Sequence[Bytes32],
                                indices: Sequence[GeneralizedIndex]) -> Bytes32:
    assert len(leaves) == len(indices)
    helper_indices = get_helper_indices(indices)
    assert len(proof) == len(helper_indices)
    objects = {
        **{index: node for index, node in zip(indices, leaves)},
        **{index: node for index, node in zip(helper_indices, proof)}
    }
    keys = sorted(objects.keys(), reverse=True)
    pos = 0
    while pos < len(keys):
        k = keys[pos]
        if k in objects and k ^ 1 in objects and k // 2 not in objects:
            objects[GeneralizedIndex(k // 2)] = hash(
                objects[GeneralizedIndex((k | 1) ^ 1)] +
                objects[GeneralizedIndex(k | 1)]
            )
            keys.append(GeneralizedIndex(k // 2))
        pos += 1
    return objects[GeneralizedIndex(1)]
```

```python
def verify_merkle_multiproof(leaves: Sequence[Bytes32],
                             proof: Sequence[Bytes32],
                             indices: Sequence[GeneralizedIndex],
                             root: Hash) -> bool:
    return calculate_multi_merkle_root(leaves, proof, indices) == root
```

A challenge for the reader now, but planned as appendix to this spec,
 is to write an optimized single-pass stack based merkleization alternative.
Since even for a multi-proof, it is not necessary to buffer too much at once.
 