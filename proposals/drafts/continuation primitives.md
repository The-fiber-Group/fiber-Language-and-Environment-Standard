Proposal to Implement all Control Flow Using Delimited Continuations
===

This proposal putlines a control flow primitive system based around the concept of continuations which can be used to compose more complex control flow structures.
---

# Data Types

This proposal introduces the data type "Context"
The context data type supports equality comparisons.
Two contexts are equal if they refer to the same location within a program.

# Operators

* Continuation delimiters: \
`{:` [Expressions]* `}`

* Continuation assignment:\
 [Symbol] `=` [Continuation]

* Continuation call syntax:\
`(` [Continuation Name] [Values]* `)`

* Continuation "Hole" declaration:\
 `(/` [Type] [Value]? `)`

# Concepts
A continuation represents "the rest of the program". A delimited continuation allows a select portion of the "rest" of the program to be re-used or reordered.

For example, there are two continuations in the mathematical expression `1 + 2 + 3`. They can be separated by parentheses as(`1 + `(`2 + 3`).

These can be separated into the following individual continuations where `[]` represents a "hole" or a dependency on the result of a previous computation...

`1 + []`\
`2 + 3`

Delimited continuations allow you to use these two continuations as objects where they may be repeated or reordered.

Let the continuation `1 + []` be called "A", the continuation `2 + 3` be called "B", and the new continuation, `2 * []` be called "C".

A continuation "hole" can be filled by performing a "call" to it and providing the hole with a value such as, 


`A (1)` => `1 + 1` => 2

Continuations can be functionally composed in a new order that's different from the lexical order in which the continuation bodies were written.

`A (C (B ()))` => `(1 + (2 * (2 + 3) ) )` => 11

or 

`C (C (C (2)))` => `2 * 2 * 2 * 2` => 16

# Example

	{fib n:nat -> r:nat = 
		a = 0, r = 1,
		c = {:
			? n > 0
			{
				n = n - 1,
				t = a + r,
				a = r,
				r = t,
				(c)
			}}}
