Proposal for Function Syntax in the fiber Programming Language
===

This proposal describes a syntax for declaring function signatures and defining function behavior.
---

# Keywords
None.

# Operators

* Function Definition Opening Delimiter: `{`

	This operator denotes the beginning of a function signature or definition.

* Function Definition Closing Delimiter: `}`

	This operator denotes the end of a function signature or definition.

* Function Body Separator: `|`

	This operator associates a function signature on the left with a function body on the right.

* Function Return Parameter Prefix: `=`

	This operator denotes the beginning of a list of function return parameters.

* Transparent Function Opening Delimiter: `{.`

	This operator denotes the beginning of specifically a transparent function.

* Inline Function Opening Delimiter: `{-` 

	This operator denotes the beginning of specifically an inline function.

* Anonymous Function (Lambda): `'`

	This operator is written in place of a function name when an anonymous function (lambda) is declared.

# Function Notation

## Signature Notation

A function signature is enclosed between a closed pair of Function Definition Delimiter operators. The signature declaration is composed of a series of symbol tokens. The first token following the opening delimiter is the name of the function. This may be replaced by the Anonymous Function operator. Following the function name, a number of program variable declarations may be written. These are used to declare the name and data type of the function's input parameters. The input parameters may be followed directly by the Function Body Separator operator, which denotes a function that does not return a value, or by the Function Return Parameter Prefix operator, which will then be followed by an additional number of program variable declarations which will be used to declare the name and data type of the function's return parameters.

	{f x y | ... }
A function signature with two untyped parameters that does not return a value.

	{f x:Nat y:Int | ... }

A function signature with two typed parameters that does not return a value.

	{f x y = z | ... }
A function signature that returns a value, *z*.

	{f x y = a b | ... }
A function signature that returns two values

	{f = a b | ... }
A function with no parameters that returns two values

## Body Notation

The function body is a series of expressions which are evaluated in the order they are written. The expressions are written after the Function Body Separator.

	{f x y = z | z = x + y}

# Function Shorthand Notation

A function may be declared using a shorthand notation with some limitations.
Shorthand notation does not include parameter declarations and the body can only
include a single expression. The function name is followed immediately by the Return Parameter Prefix operator which is followed by a single expression. The variables used in that expression are treated as parameters.

the shorthand version of 
	
	{f x y = z | z = x + y} 

would be

	{f = x + y}