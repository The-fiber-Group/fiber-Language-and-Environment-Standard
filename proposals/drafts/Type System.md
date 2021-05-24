Proposal for the Fiber Language Type System
===

This proposal outlines the basic mechanisms of a data typing system for the fiber programming language.
It does not describe the effects on program objects or evaluation in general.

# General Concepts

This type system consists of only type objects. 
Type objects are organized in a hierarchy where some types are "collected" as subtypes of another type. 
The subtypes of a given type are determined through a number of constraints which are defined with that type. 
Constraints are defined through type comparison expressions such as `Nat < 128` which may be a type constraint for a Byte data type which can only contain positive whole numbers below 128.

