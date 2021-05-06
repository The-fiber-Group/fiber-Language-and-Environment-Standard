Revised Proposal for Delimited Continuations as Control Flow Primitives
===
This proposal outlines a delimited continuation system that can be used as the 
central and only control flow utility in the Fiber programming language.
---

# Concepts
Delimited Continuations provide a means to segment an entire program continuation into any number of individual blocks which are reified as callable functions.

This proposal extends the paremtheses with the ability to enclose multiple expressions as a block.

# Operators and Syntax

This proposal introduces the following syntax:

* Continuation Declaration Delimiters: \
`(=` *Expressions** `)`

* Continuation Call Expression:\
`(` *Continuation-Name* *Values** `)`

* Continuation "Hole" Declaration:\
`(?` *Data-Type*? `)`

# Operation

## Declaring Continuations

Continuations are declared as a number of expressions surrounded by the Continuation Declaration delimiters. The expression returns a Continuation object without evaluating the expressions in the body.