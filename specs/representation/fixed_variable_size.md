# Fixed-size and variable-size

SSZ makes a difference between fixed-size and dynamic-size objects, based on a recursive definition to check if the byte-length is variable or not.

An object is considered **fixed-size** if it is:
- a basic-type
- a fixed composition of fixed-size types

This fixed-size property breaks when e.g. there is a variable amount of elements,
 or the exact type of its serialization cannot be determined without reading data.

An object is considered **variable-size** if and only if it is not fixed-size.
