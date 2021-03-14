fiber Syntax
===

This is a rough overview of the language syntax decisions that have been made so far
---

# Tokens
Tokens are grouped into two classes: **Symbols** and **Operators**.
All tokens are delimited by blank space and control characters.
`[\x00-\x20\x7f]`

## Symbols
A symbol is a contiguous series of alphanumeric characters: `[0-9A-Za-z]`.
Symbol tokens are also delimited by Operators and non-alphanumeric characters.

## Operators
An operator is a contiguous series of non-alphanumeric characters: <code>[!-\/:-@\[-`\{-~]</code>.
Operator tokens are also delimited by Symbols and alphanumeric characters.

## Parsing Rules

It's important to note that operator meanings are heavily affected by whitespace. `---`, `- --`, `-- -`, are interpreted as three different operators: `-`, `--`, and `---`.

*I think this also helps with readability in nested expressions/declarations because you can't squish everything together. i.e. `{( )}`, `{ ( ) }` are different. The first is a single closed delimiter set, the latter is two nested delimiter sets. The nesting is much more obvious here (at least to me).*

# Declarations
---

## Comments
Comments are delimited on either side by double backquotes

	``a comment``

Comments can include any number of lines. The body of a comment cannot include a double backquote

	``
	a
	multiline
	comment
	``

## Numerical Constants
A symbol may represent a numerical constant if...

*	contains only numerical characters
*	is prefixed with `0x`.

A symbol is a decimal number if it contains only `[0-9]`.

A symbol is a hexadecimal number if it contains `0x[0-9A-F]`.
hexadecimal numbers cannot use a radix point.

Two operators have special behavior when used with numerical constants: `.` and `-`.

The period dot is a binary operator that accepts a numerical constant on either side and produces a floating point decimal value.

EX:

	100.001

The minus sign negates the value of the constant.

EX:

	-1
	-100.001

## Character Constants
A single character code point from ASCII or Unicode can be declared as a numerical constant. Printable ASCII characters are surrounded by double quotes.

	"H"
	"+"
	"\""
	"\\"

Non-printable characters are declared using a forward slash prefixing a two digit hexadecimal value.

	"\20"

Unicode code points are represented either by writing them literally in a UTF-8 encoded text file or by prefixing a hexadecimal value with `\U`.

	"😊"
	"\U1F60A"

# String Data
A string can be declared that is ASCII, UTF-8, or Unformatted.

An ASCII string contains only ASCII encoded characters surrounded by double quotes. The double quote character is included in a string by prefixing it with a forward slash. The forward slash character is included in a string by prefixing it with another forward slash. Non-printable ASCII values are included by prefixing a two-digit hexadecimal value with a forward slash.

	"Hello" => Hello
	"\"Hello\"" => "Hello"
	"Hello World" => Hello World
	"Hello\20World" => Hello World

A Unicode string is represented in the same way as an ASCCI string except it contains Unicode characters encoded into a UTF-8 source text or as a forward slash-escaped hexadecimal number delimited by a period dot.

	"Hello😊"
	"Hello\U1F60.A"

A byte string is a series of unformatted 8 bit byte values.
A byte string is delimited by sets of two double quotes.
Each byte in a byte string is represented by a two digit hexadecimal value.
White space is ignored and can be used anywhere in the byte string.

"" 20 EF A1 24""
""20EFA124""

## Variable Declaration
A variable is declared using the `=` binary operator.
The left operand is a Symbol which is used as a variable name.
The right operand is a constant or an expression which returns a data object.

	var = 1
	var = "Hello"
	var = 3.14159
	var = 1 + 4 - 5 / 6

Variables are untyped (the type is inferenced from the type of the data object returned by the right expression).
Variables can be optinally declared with a typename. Variables declared with a typename are not required to be assigned a constant or data object. A variable is assigned a type using the `;` operator.

	var;Nat
	var;Char
	var;Mem Byte

Multiple variable names can have the same assignment applied to them by using the pipe character. The pipe is used to prefix either side of the `=` operator. When multiple variables are use on the left side, types are assigned to the variables in postfix. All variables preceding the most recent typename are assigned that type.

	|var1 var2 var3 var4 ;Nat ``all vars given type "Nat"```
	|var1 var2 var3 var4 = 5 ``all vars assigned 5``
	|var1 var2 var3 var4 = | 5 6 7 8 ``four vars assigned four values``
	|var1;Nat var2;Car var3;Int = 20
	|var1;Nat var2 var3 var4 ;Int = | 1 -2 -3 -4 ``four differently typed vars assigned four different constant values``


### Multiple declaration test 2

	var1 = "string"
	\ var1 var2 var3 = "string"
	\ var1 var2 var3 = \ "one" "two" "three"
	\ var1 var2 var3 = \ 1 2 3

# Types
fiber has the built in types...

*	Nat

	an unsigned 32 bit integer
*	Int

	a signed 32 bit integer
*	Float

	a 32 bit IEEE binary floating point value
*	Real

	a 64 bit IEEE binary floating point value
*	True

	a true boolean value
*	False

	a false boolean value
*	Byte

	an unsigned 8 bit value
*	Char

	a signed 8 bit value
*	Uncd

	a unicode code point
*	Nat16

	a 16 bit unsigned value
*	Nat 64

	a 64 bit unsigned value
*	Int16

	a signed 16 bit value
*	Int64

	a signed 64 bit value
*	Mem ?

	a memory address. Mem is also a dependent type (used as `Mem Nat`)

## Type hierarchy
fiber types are organized in a hierarchy using supertyps.
The supertypes are:
*	Number
*	Bool
*	Graph
*	Bin
*	Any

The hierarchy organization is as listed below...

*	Number
	-	Nat
	-	Int
	-	Nat16
	-	Nat64
	-	Int16
	-	Int64
	-	Float
	-	Real
	-	Mem
	-	Byte
	-	Char
	-	Uncd
*	Bool
	-	True
	-	False
*	Graph
	-	Char
	-	Byte
	-	Uncd
*	Bin
*	Any
	-	Number
	-	Bool
	-	Graph
	-	Bin

## Bin
fiber allows for declaring compound data structures using the "Bin". A bin can be used to represent record or array types. A bin resembles the mathematical data structure Sequence. All fields are indexed starting with one. Any field may be optionally named. A bin declaration is delimited by a closed pair of square braces. A bin declaration requires a symbol be the first token after the opening square brace. The symbol will be used as the name for that bin declaration.

	B = [x y z] ``untyped named fields``
	B = [Nat Nat Char] ``typed unnamed fields``
	B = [x;Nat y;Nat z;Char] ``named and typed fields``

A Bin can be declared with unions of different fields.
Groups of fields can be unioned using nested Bin declaration.
Field indicies are not affected by unions. The Bin "J" below has six fields indexed 1-6.

	H = [x | y z] ``union of fields x and y
	I = [Nat Nat | Char] ``union of fields 2 and 3``
	J = [J [x y z] | [a b c] ] ``union of x,y,z and a,b,c``

nested bin declarations are unnamed by default but can be optionally named using the `:` operator after the first symbol in the nested declaration.

	E = [ [x: a b c] [a: x y z] ] ``two nested bins``

### Bin Arrays
Arrays are a series of the same type.
Arrays are required to have a type.
Arrays are declared using the `_` binary operator. It accepts a typename as the left operand and a whole number of indicies as the right operand.

	X = [Nat_ 10] ``an array of twenty Nats``
	Y = [Char _ 40] ``an array of 40 Chars``
	Z = [Char _ 40 x y Int] ``A Bin containing an array and some individual fields``

## Initializign Bins

Bin field are initialized using a similar syntax to the function call (described later). The initialization operator `:` takes a bin declaration name as the left operand and a series of initial values as the right operands.
The right operands are delimited by a period.

	foo = [x y]
	var = foo: 1 2 .
the fields populated with each initial value follow the order the fields were declared in. Not all fields are required to be initialized in one expression.

	foo = [x y z]
	var = foo: 1 .
	``only the first field is initialized``
fields can be populated in the beginning and end while skipping some middle fields using the elipses operator `...`.

	foo = [a b c d e f]
	var = foo: 1 2 ... 9 10 .
	``only fields "a", "b", "e", and "f" are populated``

### Accessing Bin fields
Bin fields are accessed using the `.` operator. The operator accepts a Bin object as its left operand and a field name or index as its right operand.
Multiple accesses can be concatenated in the case of nested declarations.

	foo = [x y z]
	bar = [ [x: a b c] [a: x y z] ]
	X = foo
	Y = bar
	X.x ``access at field x``
	X.3 ``access at field 'z'``

	B.x ``produces the data at field 'x'``
	B.3 ``produces the data at field 3``
	H.x ``produces the data at field 'x'``
	H.y ``produces the data at field 'y'``
	J.x ``produces the first field``
	J.1 ``produces same field as above``
	E.x.a ``access a nested field``
	E.a.z ``access another field

Bin Arrays are accessed using index numbers.

	Z.25 ``produces the 25th Char in the array``
	Z.41 ``produces the named field, "x"``
	Z.y ``produces the field "y"``

## Declaring New Types
A type declaration begins with the operator, `(>` and is delimited with a closing paren, `)`. A symbol representing the name of the type will follow the type declaration operator.

	(>T)

Additional types following the type name are declared as subtypes of the new type.
		
	(>T a b c)

The types following the typename can be separated by a semicolon. The names before the colon are declared to be supertypes of the new type. The names after the semicolon are declared to be subtypes of the new type.

	(>T x y z ; a b c) ``supertypes: x y z subtypes: a b c``
	(>T x y z ; ) ``only supertypes: x y z``
	(>T ; a b c) ``only subtypes (same as previous code example)``

A type can recursively include its own typename which allows for additional declaration syntax.

	(>T T) ``meaningless, same as (>T). see below examples for real use``

A type of a Bin is declared using the square bracket operators without a declared bin name. The name of an already declared bin can also be used similarly to the subtype declaration.

	(>T [ Nat Char Int ] )
	(>T Z ) ``the name of a bin from the Bin examples``

A bin type can also include wildcards

	(>T [Char ? ] )

fiber also supports substructural types using different type prefix operators to denote usage patterns. The `.` operator denotes one required use. The `,` operator denotes one optional use. The `...` operator denotes an undeterminable number of used of the preceding type.
Use patterns than can be declared using a series of these operators.

	(>T .T) ``a linear type``
	(>T .T .T .T) ``a linear type with three required uses``
	(>T ,T) ``an affine type``
	(>T .T ,T) ``a type with on required and one optional use
	(>T .T ... ) ``a type requiring at least one use.
substructural types can also use a pattern of subtypes.

	(>T .Int .Int ,Nat) ``Two uses as an Int, one optional use as a Nat``
	(>T ,Char ... .Byte) ``Unlimited optional uses as a Char, once used, the last use is required to be a Byte``

Function types are declared with a parameter notation

	(>T x-y-z) ``a function with three parameters that doesn't return``
	(>T x-y=z) ``a function with two parameters that returns``

Types can also pattern match arbitrary expressions.

	(>T {Number + Number} ) ``The type of two numbers being added``

Expression types can include wildcards in the place of typed operands.

	(>T {Number + ? } )

Dependent Types can be declared which are Types that are influenced by a secondary type or constant. The **Mem** and **Bin** types are forms of Dependent Types. The `_` operator is used to denote a dependency. Multiple dependencies can be declared with a type.

	(>T _ ) ``a singularly dependent type``
	(>T _ _ ) ``a type that depends on two values``
dependent types are used by writing the name of the secondary types after the dependent typename. Dependent types cannot be used without also using the secondary types.

	T x ``T from the first example``
	T k j ``T from the second example``

Constant values are also types.

	(>ThreePrimes 5 7 13)
	(>ThisString "ThisString" )

# Function Declaration
Functions are declared using closed pairs of curly braces. The opening curly brace requires a symbol to follow it as a function name. After the function name, a function signature is required which follows the syntax pattern of the function type, but allows for named parameters.

Functions can be optionally declared separately or defined in the same declaration expression.

	{f x y = z} ``a function declaration that returns``

## Parameter Declaration
Parameters can be untyped, typed, named, or unnamed, or any combination of types and names.

	``parameter declaration subexpression examples``
	x `` an untyped parameter``
	x;Nat ``a typed parameter``
	_ ;Nat ``a typed parameter (note the whitespace between _ and ;)``

The `->` operator denotes return parameters. All parameters declared after this operator are declared as return parameters.

	x y z ``three parameters, no return``
	x y z -> a b c ``three parameters, three returns``

Parameters can be declared with default values using the identifier syntax.

### Unnamed Parameter Warning
If only one function signature/declaration exists, unnamed parameters cannot be used. All parameters must be named. Unnamed parameters are only used when overloading functions with alternate signatures.

## Function Definition
Functions are defined by appending a number of expressions after the function signature. A function signature delimits the signature expression with the `:` operator.

	{Addition x y = z : z = x + y}

A function with return parameters will return those parameters with the values they contain at the end of the function body.

	{MultiRet x y = a b : b = x a = y}

## Calling Functions

Functions are called using the `:` binary operator. The left operand accepts a function name. The right side accepts a number of expressions or constants as parameters.

	Addition: 1 2

Multiple variable assignment can be used to accept functions with multiple return parameters.

	| one two = MultiRet: "a" "b"

The variables are assigned the return parameters in the order the return parameters were declared. *one* would be assigned the value in *a*, *two* would be assigned the value in *b*.

## Overloaing functions
The previous examples have been "generic" functions in that the parameters are only implicitly typed by the restrictions of how they're used in the function body (Addition can't accept any parameters that the `+` operator can't accept). Functions can be overloaded with any number of separate signatures. The only restriction is that a function with a number of untyped parameters cannot be overloaded by a function with the same number of untyped parameters.

	``this does not work``
	{f x y}
	{f  y x} ``nonsensical/ambiguous only the parameter names were changed``
	{f  x y = z} ``ambiguous where this is used and the first is not``
return parameters cannot overload a function. two overloading functions can use different numbers/types of return parameters only if they have different input parameters as well.

	``these two do not work``
	{x x = z}
	{x x = a b} 
	``these two do work``
	{x x = z}
	{x x y = a b}

functions can be overloaded using different numbers of parameters, or by using differently typed parameters in each signature.

	``this works``
	{f x y}
	{f x;Nat y}
	{f _ ;Int y;Nat}
	{f x y z}
	{f x y _ ;Nat}
	{f x;Int y;Int = z;Nat}

All parameters across all signatures in an overloaded function are required to be named. For example, one of the overloading signatures introduces a third parameter, *z*, which is then overloaded with a specific *Nat* type in another signature. The only reason that the second use of *z* is unnamed is because *z* was originally named in another signature and thus the name of the unnamed parameter can be inferred to be "z" which allows that unnamed parameter to be used in the overloaded definition.

## Pass By Reference
By default, parameters are copied to the function. The `&` operator is used to prefix parameters that will be passed by reference. This operator cannot be used to overload functions.

	``a pass-by-reference function``
	{InPlaceAdd &x;Number y : x = x + y }
	``modifies whatever is passed in to "x" in-place``

## Transparent Functions
A transparent function is declared with the `{{` operator instead of the `{` operator. A transparent function does one thing differently than other functions. A transparent function will return the object that the function it calls returns instead of its own return parameter. A transparent function will also overwrite its own stack if it calls itself. This allows for function recursion without stack overflow. A transparent function that calls itself will always occupy a static area in the program stack until it exits. Any function called by a transparent function is required to have an identical return parameter type to the transparent function.

	``a transparent function that calculates the fibbonanci sequence recursively``
	{{fib a b c = d : 
		c = c - 1
		? c > 0
		d = fib: b a + b n,
		d = a
	}

## Variadic Functions

Variadic functions are declared using a two-parameter system.
The first variadic parameter is a Memory pointer which locates the beginning of the variadic arguments. The second parameter is a *Nat* number that holds the size in bytes of the variadic argument section.

Variadic functions are declared using an elipses followed by two parameters.

	{varFunc x y ... a b}
Variadic functions can accept required parameters after the variadic parameteers.

	``here "j" and "k" are normal parameters following the variadic parameters``
	{varFunc x y ... a b j k}
Variadic parameters are accessed by dereferencing and casting the pointer parameter. (This is pretty much identical to C except you don't need a special header kludge and the variadic parameter doesn't need to be the last thing.)

## Polymorphic Types
Polymorphic types are similar to dependent types, but use named depnedencies and include special interface definitions. (Polymorphic Types are styleized as "interfaces" in the Haskell and Go languages)

A polymorphic type is declared using underscores in the same manner as a dependent type, however the underscores are named and are used in a number of generic function declarations after the underscores. (These underscores are referred to as "type parameters".)

	(>T _x _y {Tfunc1 x y} {Tfunc2 x y z} )
The function parameters sharing the names of the type parameters cannot be declared with types.

	``this doesn't work``
	(>T _x _y {WrongFunc x;Nat y;Nat} )

The generic functions declared with the ploymorphic type are required to be overloaded with type-specific functions. 

	(>T _x {Tf x = z} )
	{Tf x;Nat = z;Char} ``function definition not necessary for the example. you get the idea``

---

# Expressions
Expressions in fiber are evaluated and will always return a data object.
Expressions consist of operators, variables, constants, and function calls.

## Math Operators

+, -, *, /, %

Math operators are binary. The subtrahend/divisor are the right operand.
Math operators do not follow precedence rules. They are left-assoiative.
(The operators' math functions are self evident.)

## Logic Operators

-	`&` AND
-	`|` OR
-	`^` XOR
-	`!` NOT

Logic operators perform boolean logic operations on boolean data types and binary logic operations on numbers. The difference is performing a logical AND just flips the first bit around in Boolean data types where a binary AND performs a bitwise AND on the entire number data type.

## Comparison

-	`>` Greater Than
-	`<` Less Than
-	`>=` Greater Than or Equal
-	`<=` Less Than or Equal
-	`==` Equal
-	`!=` Not Equal

Comparison operators are binary and evaluate to a boolean True or False value. Comparison operators can be combined with logical operators and parentheses for complex boolean comparision expressions.

## Type Comparisons

Comparison expressions can also be used with typenames.
Type comparisons can only use the Equal and Not Equal comparison operators.

	(>J)
	(>K)
	J == K
	J != K

Type Comparison includes a "type of" prefix operator, `|(` which produces a typename that can be used in a type comparison.

	var = 1
	Nat == |(var ``evaluates to true because 1 is a Nat``

## Type Size
The "size of" prefix operator, `:[` takes a typename and produces the byte size of that type.

	var = :[Nat
	var == 4 ``evaluates to true because the type Nat is 32 bits``

## Parentheses

Parentheses are used to force the evaluation of a subexpression before the surrounding expression.

	48 / 8 / 2 ``evaluates to 3``
	48 / (8 / 2) ``evaluates to 12``

## Pointers

-	`~` Produce the memory address of a data object.
-	`#` Dereference an address to produce a data object.

These operators deal with the *Mem* dependent type. The seconary type of *Mem* is decided by the type of the data object being referenced.

	x = 1 ``a "Nat" datatype``
	y = ~1 ``a "Mem Nat" datatype
	z = #y ``a "Nat" datatype

Pointers only support addition and subtraction. They do not support any logical operations.

## Bitwise Operations

-	`#>` Shift Right
-	`<#` Shift Left
-	`#>>` Arithmetic Shift Left (preserves sign)
-	`#~` Rotate Right
-	`~#` Rotate Left

The bit shifting operations are self explainatory. They can be combined with all other expressions.

## Type Casting

-	`$:` 

the type cast binary operator will enforce the use of the data object on the right as the type given by the typename on the left

---

# Branching
Branching and Conditionals are very simple in fiber. There's no structured programming constructs. There's an if-else conditional branch and a label and goto statement.

## If-Else

The if-else construct begins with the `?` operator. It is followed by a conditional expression. A conditional expression is a boolean expression using logical and comparison operators. Two code blocks follow the conditional. The first block is executed if the conditional is true. The second block is executed if the conditional is false. Each block is deliminated by a comma.

	? 2 > 1
	func1: x y,
	func2: x y,
	``end of conditional``

## Labels and Goto

A label is declared by prefixing a Symbol with the `@` operator. The label denotes a specific location in a program where the scope of execution can be relocated dynamically. A label can be read as a *Mem Any* data type, but labels cannot be modified by any operation. (read only)

A Goto is declared with the `_^` operator. It is a prefix operator that accepts a label name as its operand.

Lable and Goto statements are limited to the scope of a function body. This means that a Goto statement cannot move execution to a labe outside of the function the Goto statement is located in.

	``a function with a loop``
	{loop x:
	@LoopTop 
	? x > 1
	x = x - 1
	_^ LoopTop,
	_^ End
	@End
	}

### Special Goto
The operator `_^}` will jump directly to the exit of the function body.

The operator `!^` will jump outside of the local function body to a label in the body of the calling function.

---
# Syntax Extension

Function signatures can optionally be declared as a syntax pattern. This allows for operator overloading, macro behavior, and "compile-time" execution. Syntax extensions are different from regular functions because they support declarations as well as normal expressions.

Syntax signatures begin with the `{;` operator. The first token is a Symbol which the extension is identified with for overloading. it shares the same namespace as the normal functions. After the symbol, a syntax pattern follows.
Syntax extensions are always declared as a signature first. There is no default body, any definition must also be an overload.

Syntax extensions are defined as a series of inputs and operators.
Operators are used to define the syntax pattern used. They are declared with "literal strings". Inputs are similar to function parameters as they allow variables and other expressions in their place.

## Literal Strings
Literal strings declare the syntax pattern required to engage the function.
Literal strings consist of printable characters surrounded by single or double quote pairs.

	'++'
	"-'-"
	'syn'
Literal strings must be either a Symbol or an Operator. Strings such as "+a-b" are not accepted.

Literal strings that are symbols are interpreted as keywords. Reserved keywords are entirely possible in fiber, the base language just doesn't use them.

## Inputs
Inputs are declared in the same way as untyped function parameters. That's all.

## Pattern Operators

There are two pattern operators used. 
The "alternate" operator, `|`, and the "multi" operator, `...`.
These two operators are synonymous with the `[ ]` and `*` operators in regular expressions, except these operators are used to match patterns at the token level.

### Alternate Operator
The alternate operator is a binary operator that takes two literals on either side. The alternate operator declares that either of those literals may appear at the position of the operator an a syntax pattern.

	``given`` {;foo 'hello' | 'Hello'}
	``the syntax pattern will accept`` hello ``or`` Hello
Multiple Alternate operators can be used serially to specify more than two alternates.

	{;foo 'a' | 'b' | 'c' | 'd' 'e' | 'f' | 'g'}
	``exactly the same as the regular expression [abcd]\x20[efg]``

### Multi Operator
The multi operator functions similarly to the variadic function declaration, but this operator produces a Bin. The first field of the bin is always the number of fields in the bin. The bin contains a number of tokens, either operators or symbols.

The name of the Bin is the Symbol directly following the multi operator.

	{;foo '***' ... B}
	``the name of the Bin here is "B"``

The bin is accessed normally, but may contain non-evaluatable declarations.

The multi operator can also be used to declare a repeating pattern using parentheses.

	{;foo ... ( 'a' _ 'b' ) X}
	``anything between the keyletters "a" and "b" is given its own field in the bin "X"``
	``so given`` a x b a f b a j b
	``the Bin X would contain [ 'x' 'f' 'j' ]``

## Handling Syntax Extension Inputs

No inputs are automatically evaluated when a Syntax Extension is used. (Think quoted lists in Lisp.)
A special evaluation prefix operator, `<~>` is used. When an input parameter is prefixed with this operator, the input is evaluated.
The quote prefix operator, <code>`</code>, is used to expand the syntax held by an input parameter without evaluating it. This is useful for expanding input syntax into a declaration.

## Syntax Extension Overloading

Syntax Extensions require at least one definition.
A definition does not need named parameters as all parameters have already been named. 
Syntax extension overloads are written exactly like normal functions, except the parameter declarations are typenames.

## Syntax Expansion Example
Here's a syntax extension to create a Bin using basic C++ struct syntax.

	{;Cstruct 'struct' X '{' ... (_ _ ';' ) I '}'}
	{Cstruct Any Any : 
		[`X `I.2; `I.1 `I.4; `I.3]
	}
	``this takes the struct declaration 
	"struct S { Int a; Int b; }"
	and turns it into
	"[S a;Int b;Int]"

In a real extension, there would be some logic to arrange and expand an unlimited number of field declarations but the process would be similar to this.

**There's more to figure out here with Syntax Extensions, but I'll get to it later. You can see the general idea.**

---
# Proof Syntax
*I have the intuition that this will be useful for taking what the type system implicitly proves and going the last 10% to full verification. Still not sure if this is the easiest form for it to take.*

Proof syntax is used alongside the type system to verify a program.
Types are used to constrain the flow of program state.
Proofs are used to constrain arbitrary program behavior to reasonable behavior out of all possible valid behavior.

Proof syntax supplies three forms of statements.
-	the Clause
-	the Dependency
-	the Directive

All proof statements begin with the proof operator `#!`, followed by a Symbol used to identify the statement.
The type of proof statement being declared is determined by the logical connective used.

## Scope
Proof statements are used inside a function body to prove things about that function. Local variables can be used in proof statements.

An exception is proof statements in the body of a function called inside another function. If the body of function **f** calls function **g**, the names of proof statements in the body of **g** become visible to the statements in **f**. If a proof statement name is used in both **f** and **g**, the name will always refer to the statement made in the body of **f**.

## Connectives
*	`=:` (binary operator)

	Asserts that a comparison expression holds true when the expression on the right is evaluated. Used in Clauses
*	`-|` (binary operator)

	Asserts that the expression on the left is true if and only if the statements on the right are all true. Used in Dependencies
*	`:>` (unary prefix operator)

	Asserts that the expression to the right must be true when the function exits. Used in Directives
*	`-->` (unary prefix operator)
	
	Asserts the possibility that the expression to the right will be true when the function exits. Used in Directives
*	`#_` (unary prefix operator)

	Used in Clauses. Prefixed a variable to refer to its state before the expression is evaluated.

## Clauses
A Clause is a comparison expression connected to another expression.
A clause asserts that the comparison is true after the expression is evaluated.

	``asserts that x is a certain type``
	#! foo |( x == Int =: x = x - 1
	``true only if x is less than or equal to zero``
	``useful for constraining x values without needing conditional branches``

## Dependency
A dependency is a comparison expression that is true if and only if other proof statements are also true. If there are multiple statements that are required, they are separated by commas

	`` "foo" is true if "bar" and "baz" are also true``
	#! bar x <= 0 =:
		#! |( x == Number =:
			x = x - 1
	#! foo |( x == Int -| bar, baz
Another thing that's shown in this example is that the Clause operator is concatenative. Multiple Clauses can be nested around a single expression evaluation.

## Directive
There are two directive operators, `:>` and `-->`. Directives are different than Clauses and Dependencies because Directives can be used to prove assertions about program state over time. A good example is proving termination.

	``a recursive function decrements a counter``
	{{RecDec x;Number : 
		? x > 0
		x = x - 1
		RecDec: x,
		,``there's no else here``
	}
now here's the proof that the function terminates

	{{RecDec x;Number :
		? x > 0
		#! foo x < #_x  =: 
			x = x - 1
		``asserts x after the expression is less than x before``
		RecDec: x,
		,
		#! bar --> x == 0 -| foo
		``x will reach zero at some point because foo proves x gets decremented``
	}

*There's probably a better way to do this with fewer operators, I'll figure it out as I play with use patterns more. Like I said, verification is heuristic at best and now I'm trying to invent a different way to do a task that noboy's figured out how to do properly yet in the first place.*

