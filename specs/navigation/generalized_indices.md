# Generalized Merkle tree index

The hash-tree-root of all SSZ types merkleizes the contents as a binary tree.
In such a binary tree, the path to any node from the root can be described by a bitfield.

This bitfield can also be expressed as an integer, called a "generalized index" in SSZ.
The generalized index value for a node in a binary tree is `2**depth + index`, starting with a 1 for the root.
Visually, this looks as follows:

```
    1
 2     3
4 5   6 7
   ...
```

Like the bitfield form that is extended with a `0` or `1` for each child,
 the generalized index has the convenient property that the two children of node `k` are `2k` and `2k+1`.

## Combination and slicing

To navigate from `A` to `B` to `C`, where `B` is in the subtree of `A` and `C` in the subtree of `B`, the generalized indices can be composed and sliced:

A generalized index is composed of a leading bit `1` for the root, and the remainder navigates the path in binary tree.

These navigation parts `AB` and `BC` can be concatenated to get the navigation part `AC`: `AB ++ BC <-> AC`.
And then a `1` is prepended again to delimit the exact length of the path and represent the root.  


## Flat indexing
 
For implementation purposes, the generalized index matches the position of a node in the linear representation of the Merkle tree, as computed by this function:

```python
def merkle_tree(leaves: Sequence[Bytes32]) -> Sequence[Bytes32]:
    padded_length = get_next_power_of_two(len(leaves))
    o = [Bytes32()] * padded_length + list(leaves) + [Bytes32()] * (padded_length - len(leaves))
    for i in range(padded_length - 1, 0, -1):
        o[i] = hash(o[i * 2] + o[i * 2 + 1])
    return o
```

## Representation

In the SSZ spec, a generalized index is represented as a custom integer type: `GeneralizedIndex` (of arbitrary bitlength).
It can be also be represented as a Bitvector/Bitlist object.

Note that for bitfields, the root bit is not encoded in SSZ:
- bitlists: SSZ already naturally appends 1 to the serialized bits, to make a difference in bitlengths.
- bitvectors: vectors are of fixed length, and do not need the delimiting bit.
