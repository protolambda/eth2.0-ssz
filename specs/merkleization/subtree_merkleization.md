# Subtree Merkleization

Merkleization does not have to be irregular, many types are designed to map to a stable complete tree.

Subtree merkleization `merkle_subtree(chunks, limit=None)` (see [chunkification](./chunkify.md)) is defined with the following functions:

* `next_pow_of_two(i)`: get the next power of 2 of `i`, if not already a power of 2, with 0 mapping to 1. Examples: `0->1, 1->1, 2->2, 3->4, 4->4, 6->8, 9->16`
* __`merkleize(chunks, limit=None)`__: Given ordered chunks, merkleize the chunks, and return the root:
    * The merkleization depends on the effective input, which can be padded/limited:
        - if no limit: pad the `chunks` with zeroed chunks to `next_pow_of_two(len(chunks))` (virtually for memory efficiency).
        - if `limit > len(chunks)`, pad the `chunks` with zeroed chunks to `next_pow_of_two(limit)` (virtually for memory efficiency).
        - if `limit < len(chunks)`: do not merkleize, input exceeds limit. Raise an error instead.
    * Then, merkleize the chunks (empty input is padded to 1 zero chunk):
        - If `1` chunk: the root is the chunk itself.
        - If `> 1` chunks: merkleize as binary tree.

For virtual padding, two subtrees of zero-padding merkleize into a pre-computable [zero-hash](./hashing.md#zero-hashes).
