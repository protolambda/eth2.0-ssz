# Merkle proofs

## For beginners

Merkle proofs enable users to efficiently prove specific details of some data-structure that is known by a given hash.

The efficiency is achieved with a tree structure of hashes, with the data in the leaves of the tree.
For a proof of a set of leaves, branches to other leaves do not have to be fully encoded or hashed,
the starts of each such branch, together with the values to proof, are sufficient to reconstruct the root of the tree.
Compare the reconstructed root with the trusted root the data is known by, and the proof is complete.

## Accumulator

Merkle trees are a type of cryptographic accumulator, and a root is a binding vector commitment to a set of contents.
I.e. the position of the contents is also committed, not just inclusion. Changing the position of any of the contents would change the Merkle root.

Membership of a leaf value, at a specific position, can be proven with witness data:
 a set of hashes along the way to the root of the tree, taking `O(log(N))` space and computation for a proof, as opposed to `N` for providing the full data.

## Binary merkle tree

The tree structure itself affects the amount of nodes, and thus the amount of hash operations, and size of the proof.

One of the few opinionated choices made by SSZ for Eth2 is the choice for a **binary merkle tree**, as oposed to Merkle Patricia Trees used in Eth1.

Binary trees provide simplicity and efficiency:
- No irregular branch structures
- Any data structure can be translated to a binary tree with minimal effort.
- Less proof witness data in favor of a few more hash operations
- High affinity with bitfields for navigation and description
- Enable a wide range of other binary-tree specific optimizations

## Verification

Claims for leaves of data can be verified by reconstructing the root from these leaves with the help of witness data:
sibling nodes of the branches leading back to the root.
Notice that leaf nodes that share the same subtrees also share more witness nodes, and are thus proven together more efficiently.

### Examples

The numbers used in below examples are [generalized indices](../navigation/generalized_indices.md), not values.
Note that the ordering of witness data is an encoding choice, defined by the [proof backing](#proof-backings).

#### Classic single-leaf inclusion proofs

```
                      1
          2                       3'
    4'          5           6           7
  8    9     10'  11*    12   13     14   15 
```

Leaf: `11`
Witness data: `10, 4, 3`
Proof: `H(H(4, H(10,11)), 3) == 1`

#### Multiples leaves

Also called "multi-proofs".
Note that witness data is shared between leafs; proving multiple values at the same time is more efficient than proving them individually.

```
                      1
          2                       3
    4'          5           6           7'
  8    9     10*  11*    12'  13*    14   15
```

Leaves: `10,11,13`
Witness data: `4, 12, 7`
Proof: `H(H(4, H(10,11)), H(H(12,13), 7)) == 1`

#### Unbalanced trees


```
              1
       2              3
    4    5'        6       7'
  8' 9*        12     13'
            24'   25
                50  51
           100'101* 102*103*
```

Leaves: `9,101,102,103`
Witness data: `8,524,100,13,7`
Proof: `H(H(H(8,9), 5),   H(H(  H(24, H(H(100,101), H(102, 103))),   13), 7)) == 1`


## Proof backings

For an implementation, several choices can be made:
- Ordering of leaf nodes and witness data
- Ordering of operations to reconstruct the root, in case of multi-proofs.
- Inclusion of a description of the proof target leaves, or the complete proof structure.
- Optimizations for fast reading, verification or modifications to the proof.

SSZ is agnostic to this: merkle proofs are an interface to these backings, not an enshrined choice for one approach.

## Interface

For application level usage, describing the proof with a typed structure is recommended, see [SSZ partials](../partials).

For lower level usage, most of the complexity (and implementation freedom) is transferred to the underlying proof backing, and only a bare minimum interface is defined:

`compute_root(proof_backing) -> root`

`verify(proof_backing, root) -> bool: return compute_root(proof_backing) == root`
