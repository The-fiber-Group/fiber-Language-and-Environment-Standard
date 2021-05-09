Revised Proposal for Delimited Continuations as Control Flow Primitives
===
This proposal outlines a delimited continuation system that can be used as the 
central and only control flow utility in the Fiber programming language.
---

# Concepts
Delimited Continuations provide a means to segment an entire program continuation into any number of individual blocks which are reified as callable functions.

This proposal extends the parentheses expression delimiters with the ability to enclose multiple expressions as a block.

# Operators and Syntax

This proposal introduces the following syntax:

* Continuation Declaration Delimiters: \
`(` `|` *Expressions** `)`

* Continuation Call Expression:\
`(` *Continuation-Name*`,` *Values*`,`* `)`

* Continuation "Hole" Declaration:\
`(/` *Data-Type* `)` or
`(/)`

# Data Types

This proposal introduces the `Continuation` data type. The only operation defined for the Continuation data type is the equality comparision, `=`.
Two continuation objects may be equal if they are the same continuation object.

# Operation

## Declaring Continuations

Continuations are declared as a number of expressions surrounded by the Continuation Declaration delimiters. The expression returns a Continuation object without evaluating the continuation's body, the expressions contained within the delimiters. 

Below, a continuation with a single expression is bound to the variable, *cont*.

	cont = ( | 2 + 3)

The body of a continuation can contain any number of "Hole" Declarations which may be optionally type constrained.

Below a continuation with a hole is declared and called in the next line to produce the value 10.

	x = ( | 5 * (/Int) )
	(x, 2)

## Calling Continuations

Continuations are called using the function call syntax. The continuation being called as well as any values being provided to holes are surrounded by a closed pair of parentheses.

	y = ( | (/) + 5 )
	(y, 3)

If a continuation is called that contains holes in its body, a value is required to be provided for each hole in every call to that continuation.

## Continuation Holes

A continuation hole is used to persist various program state across arbitrary context boundaries. An untyped hole is treated as a dynamically typed function parameter. A type-constrained hole is treated as a typed function parameter. If a type constrained hole is passed a value that is not of an equivaleent type, that call will result in a type error.

If a continuation body contains multiple hole declarations, the order in which the values are provided to the holes in the continuation call correspond to the order in which the holes were lexically declared.

	nesting = ( | 
		{ f x = x | x = x - (/) }
		{ g x = x | x = x - (/) }
		x = (g (f (/) ) )
	)
	(nesting, 2, 7, 17)

The result of the call to the continuation above is equivalent to the expression `17 - 7 - 2` because the holes are filled in the order of 
1. the hole declared in the body of *f* (2)
2. the hole declared in the body of *g* (7)
3. the hole declared in the assignment to *x* (17)

# Continuation Objects

The continuation object will capture the body to be executed when it is called and the **context** which the body will be executed in. The context contains all variable bindings used within the continuation. Variables that are not affected by the continuation are not required to be preserved. A continuation may be called multiple times over the context that was captured with its declaration. The effects of multiple calls to the same continuation are the same as if the continuation block contained multiple static repetitions.
For example, given the two continuations,

	a = ( | x = x + 1 )
	b = ( | x = x + 1, x = x + 1)

two consecutive calls to the continuation *a*

	(a)
	(a)

results in the same effect on the program state as a single call to *b*

	(b)

A single continuation may capture multiple contexts if its body contains multiple separate lexical scopes. The context relative to the execution of the continuation contains all bindings and sub-contexts required to execute the continuation body.