Syntax Test
===


# Initializing Arrays

	arr = [x:Nat_4]
	y:arr 
		..0 = 1 
		..1 = 2 
		..2 = 3 
		..3 = 4
	another-a = [
		h:Char = "h"
		e:char = "e"
		l:char = "l"
		ll:char = "l"
		o:char = "o"
		]
	y:another-a.h ``=> "h"

# Quotation Macro Definition

	{ ; '" str:[Uncd_* ] '" -> str}
	{ ; '" ... '\" -> c:Char | c = "\""}
	{ ; '" ... '\ 'U c[Char_4] -> h:Number}
The strings and characters are returned directly because they don't need to be modified first, only captured. The backslash macro consumes extra quotations that might cause the quotation macro to terminate early.

# Defining Type Interfaces

	new-bool = [byte 
		and bool bool -> bool
		or bool bool -> bool
		not bool -> bool
		is bool -> bool
		]
	{and x y -> z | z = x&&y}
	{or x y -> z | z = x||y}
	{not x -> y | y = !!x}
	{is x -> y | y = x}

# Defining Modules

Modules use braces `{}` like functions do because they're both about separating code into reuseable components. Modules do not include the function operator, `->` though. Instead Modules use two bars to separate the module into three sections. The first is the name and dependencies segment, the second is the external symbols segment, the third is the internal symbols segment. For example.

	{mod-A "mod filename" | x : Nat = 1 | y : Nat = 1}
	{mod-B mod-A | x : Nat = mod-A x | y : Nat = mod-A x}
	{mod-C mod-B | x : Nat = mod-B x | y : Nat = mod-A x}
	{mod-D mod-A | | k : Int = 1}
	{mod-D | j : Int = 1}

If you're wondering about functions with no parameters, they're expressed as 

	{f -> | ... }

where the function specifier `->` is still used, but no parameter names are given.

# Defining Abstract Primitive Types

	J = [ [m] [n] [o] | 
		{ . J '@ J -> J} 
		{ . '# J -> J} 
	]
	{ . x '@ y -> z |
		? x == m
		(
			? y == m (z = m)
			? y == n (z = o)
			? y == o (z = n)
		)
		? x == n
		(
			? y == m (z = o)
			? y == n (z = n)
			? y == o (z = m)
		)
		? x == o
		(
			? y == m (z = n)
			? y == n (z = m)
			? y == o (z = o)
		)
	}
	{ . '# x -> y |
	}

# Continuations

Continuations extend parentheses so that they can be called as functions
and they capture a context.

Each continuation must have a return type declared and the parameters are 
declared by interleaving type declarations within the continuation body.
The returned data is declared as the result of the evaluation of one of the 
expressions within the continuation. The return type is declared by the return 
operator `->`. A data type goes on the left and an expression which will be 
returned goes on the right. 

	add-a-bunch = ( . Nat -> 1 + 1 + Nat + 1 + 1 )

	(
		x:Nat = 1 
		y:Nat = 1
		z:Nat = add-a-bunch x + add-a-bunch y
	)

# basic data types

Function type: `{}`
Type Type: `[]`
Continuation/Expression type: `()`

# Macros

Macros are always inlined, if you want a macro to behave like a function call, 
then have it wrap a lambda. There's two distinct kinds of macros, syntax macros
and reader macros. Syntax macros match token patterns, reader macros match 
individual character patterns. Decimal notation is a character pattern for 
example. Reader macros get passed the stream as a finite array of characters.
they can iterate through those characters and produce syntax from them.

reader macros are signified by the first terminal in a macro being a quoted string.

reader macro which is triggered by quotes

	{ . "\"" x : ["] -> y : [char_nat] |
		count:nat = 0
		pos:Nat = 0
		( find-count = 
			( . 
				? x(pos) != "\"" 
				( ? x(pos) == "\\" 
					(pos = pos + 2)
					(pos = pos + 1) 
				) 
				( )
			)
		)
	}

# digraph unicode equivalents

`-` is equivalent to &minus;

`/` is equivalent to &#x2215;

`*` is equivalent to &lowast;

type composition `.` is equivalent to &#x2218;

`&&` is equivalent to &and;

`||` is equivalent to &or;

`<=` is equivalent to &le;

`>=` is equivalent to &ge;

`<<` is equivalent to &lt;

`>>` is equivalent to &gt;

`==` is equivalent to &#x2261;

type equality `=:=` is equivalent to &#x2263;

reference-of operator `~/` is equivalent to &#x2240;

dereference operator `_\` is equivalent to &sim;

return operator `->` is equivalent to &rarr;

comments <code>``</code> are equivalent to &#x220e;

the Any type is equivalent to &#x25a3;

the none type is equivalent to &#x25a1;

function type `{}` is equivalent to &rArr;

continuation type `()` is equivalent to &#x25b7;

Reference type `~~` is equivalent to &#x25dc;

Type type `[]` is equivalent to &#x25a9;

`Nat` is equivalent to &#x2115;

`Int` is equivalent to &#x2124;

`Float` is equivalent to &#x211a;

