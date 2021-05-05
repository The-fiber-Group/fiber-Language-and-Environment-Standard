Proposal to Implement all Control Flow Using Delimited Continuations
===

This proposal putlines a control flow primitive system based around the concept of continuations which can be used to compose more complex control flow structures.
---

# Data Types

This proposal introduces the data type "Context"
The context data type supports equality comparisons.
Two contexts are equal if they refer to the same location within a program.

# Operators

* Continuation declaration \
`(` `=` [continuation name] [expressions]* `)`

* Continuation call syntax:\
`(` [continuation Name] [values]* `)`

* Continuation "Hole" declaration:\
 `(?` [Type] `)`

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

# Using Delimited Continuations

Normally parentheses are used to specify the order of evaluation in an expression.

	( 2 + 10 ) / 2

This proposal extends the "precedence" action of parentheses with labelling delimited continuations.

If the first token following an opening parenthese is the equals sign, `=`, then the expressions inside the closed set of parentheses are established in the environment as a delimited continuation object bound to the symbol following the equals sign.

	( = x 2 + 10 ) / 2

The direction of assignment is backwards from normal syntax. It flows inward and to the right. This does two things, it emphasizes that the expression outside the parentheses still depends on the evaluation of what's in the parentheses. It also emphasizes that the object being named is the body of the parentheses, rather than its result.

A continuation can contain a number of "holes" where values are provided when the continuation is entered. The values that can be given to holes are constrained by what values are accepted by the operations that depend on the value provided by a hole. A "hole" can optionally be given a data type annotation which will limit aceptable values to those of the data type.

	( = x 2 + (?) )
	( = y (array-append (?Type-Array) 5 ) )

Continuations are entered using the same syntax as calling a function, but instead of the name of a function, the name of a continuation is given. Instead of function parameters, values for the continuation holes can be given. 

	( = x 2 + (?) (x 5) )

If a continuation body contains multiple expressions, the result of the last expression is used.

	1 + ( = x 2 + (?) , 2 + 2 (x 1) ) => 1 + (4) => 5

# Example

## The example from the wikipedia page on delimite continuations

	2 * ( = r 1 + (?Nat) (r 5) )

## The Fibonacci Sequence
	{fib n : Nat -> r : Nat =
		a = 0, 
		r = 1,
		( = c, ``capture (reset)
			? n > 0
			{
				n = n - 1,
				t = a + r,
				a = r,
				r = t,
				(c) ``call (shift)
			}
		)
	}
