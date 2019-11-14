# Merkle Mix-ins

Construct: `Mix(core, tag)`

To tag specific information ("`tag`" here) to a root value ("`core`"), e.g. a length of the contents the `core` represents.

This merkleization step is simply: `H(core, tag)`, with both inputs left-padded to to a full 32 bytes.

SSZ is consistent with merkleizing in these tags to the right: it avoids a kink of branch nodes in common proof encodings.
I.e. the left-most contents value and its branch of witness nodes go straight to the root. 

In the spec the following function definition is used when mixing in a number:

__`mix_in_num(root, num)`__: Given a `root` and a `num` (`"uint256"` little-endian serialization) return `hash(root ++ num)` (the `++` is concatenation).
