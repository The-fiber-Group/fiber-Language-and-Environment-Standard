Proposal for the Fiber Language Type System
===

This proposal outlines the basic mechanisms of a data typing system for the fiber programming language.
It does not describe the effects on program objects or evaluation in general.

# General Concepts

This type system consists of only type objects. 
Type objects are organized in a hierarchy where some types are "collected" as subtypes of another type. 
The subtypes of a given type are determined through a number of constraints which are defined with that type. 
Constraints are defined through type comparison expressions such as `Nat < 128` which may be a type constraint for a Byte data type which can only contain positive whole numbers below 128.

---

Types are surrounded by `[]`.
Inside can contain a number of subtypes which may be written inside a comparison expression.

	A-One = [N=1 R=1 Z=1]

where the type *A-One* is a supertype of the value one in multiple different types.

What's supported in type declarations?

boolean comparison expressions: 
`==` equal\
`!=` not equal\
`<<` less than\
`>>` greater than\
`&&` and\
`||` or\
`!!` not\

since values are subtypes, single constants can be included.

	fifty-three = [53]


