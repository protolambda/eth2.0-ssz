# Representation of sequences

Representation of sequences can be thought of as two parts: the fixed-size part, and the variable-size part.

Fixed-size types do not have a variable part. 

Note that if all elements have the same type, the two parts can be specialized. However, conceptually it is all the same.

## Fixed part

For each of the elements in order, if the element type is:
- fixed-size: serialize the element and append it to the fixed-size part.
- variable-size:
  - Append an offset to the fixed-size part, pointing to the start of the element data in the variable-size part.
  - Serialize the element and append it to the variable size part.

### Offsets

Within the fixed-size part offsets may be encoded to locate elements in the variable-size part.

Offsets are 4 bytes each, typed as `uint32`, and can range from `[bytelen(fixed_part), bytelen(fixed_part) + bytelen(variable_part)]`. I.e. the fixed-part byte length is included as part of the offset.

Each offset is pointing to the start of the serialized data, the index of the first byte of the element.

For each offset, it MUST hold that `offsets[i-1] <= offsets[i] <= offsets[i+1]`, so that elements can be read from the byte stream following the offsets in order.

Some elements in the variable-size part may be empty, this can result in:
- sequential equal offsets
- the last offset being equal to the end of the scope.

There may be a dynamic number of variable-size elements all of the same type,
 in this case the element count can be derived from the *first offset*: `offset / 4`, as each offset matches an element and is 4 bytes. 

## Variable part

For variable-size elements, the elements are serialized, tightly packed, appended in order to the variable-size part.
