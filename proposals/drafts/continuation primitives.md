Proposal to Implement all Control Flow Using Delimited Continuations
===

This proposal putlines a control flow primitive system based around the concept of continuations which can be used to compose more complex control flow structures.
---

# Data Types

This proposal introduces the data type "Context"
The context data type supports equality comparisons.
Two contexts are equal if they refer to the same location within a program.

# Operators

* Continuation delimiters: `:{` ... `}`

* Continuation call delimiters: `(>>` ... `)`

* Continuation "Hole" declaration: `()`

---

Continuation looks like this:

	cont : { ... }
it's surrounded by parentheses and
it has a number of expressions inside

it can take a number of "holes" with default values. Holes always require a data type

	cont: {cont 1 + Int() = 1 - Int() = 5 }

when you call it, you supply values to the holes in order of lexical occurance

	(>>cont -1 3)

a continuation that is a single expression will return a value.
a continuation that is not a single expression will not return a value.