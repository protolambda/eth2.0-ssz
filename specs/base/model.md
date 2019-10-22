# Base model

SSZ concepts are defined with just a few base constructs (or "kinds"), to keep it minimal and simple.

Note that these constructs are purely for conceptual understanding, implementations can optimize more for individual types.


## Composition of types

### Fixed-size and dynamic-size

SSZ makes a difference between fixed-size and dynamic-size objects, based on a recursive definition to check if the byte-length is variable or not.

An object is considered **fixed-size** if it:
- is a basic-type
- is a static composition of basic-types
- is a static composition of a static composition

This static property breaks when e.g. there is a variable amount of elements, or the exact type of its serialization cannot be determined without reading data.
The below constructs, and the types themselves go into detail on this. 


### Series

Construct: `Series[ElemType, Max](values)`

A Series has a single element type and a maximum length.

- if `len(values) > Max`: the contents SHOULD not be serialized, as they contradict the type.

The serialization can be thought of as two parts: the fixed size part, and the variable size part.

If `ElemType` is a fixed-size type:
 - the fixed size part is used for the elements, as element positions can be predetermined because of their fixed size.
 - no variable size part
If it is not, then:
 - the fixed size part is an *offset prologue* that describes the byte positions of each of the elements
 - the variable size part is used for the elements

The elements themselves are serialized independently, and tightly packed in order.

#### Offsets prologue

Offsets are a sequence of `uint32` numbers, pointing to the byte index, relative to the start of the serialized data (including offsets), of the first byte of the element.

For each offset, it MUST hold that `offsets[i-1] <= offsets[i] <= offsets[i+1]`, so that elements can be read from the byte stream following the offsets in order.


### Compound

Construct: `Compound[FieldTypes](values)`

A Compound is like a series, but each of the values is allowed to be different from the other types. `len(FieldTypes)` MUST be `len(values)`.

Like a Series, a Compound is serialized with a fixed and variable size part.

For each of the fields:
- if the field type is a fixed-size type: append the serialized field
- if not, then:
  - append an offset to the fixed part, pointing to the serialized element in the variable size part
  - append the serialized element to the variable size part, after completing the fixed size part, in order relative to possible other variable size elements.


## Merkle Mix-ins

Construct: `Mix(core, tag)`

To tag specific information ("`tag`" here) to a root value ("`core`"), e.g. a length of the contents the root represents, a mix-in is used.

SSZ is consistent with merkleizing in these tags to the right: it avoids a kink of branch nodes in common proof encodings.
I.e. the left-most contents value and its branch of witness nodes go straight to the root. 


