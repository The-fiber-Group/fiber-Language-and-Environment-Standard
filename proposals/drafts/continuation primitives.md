Proposal to Implement all Control Flow Using Delimited Continuations
===

This proposal putlines a control flow primitive system based around the concept of continuations which can be used to compose more complex control flow structures.
---

# Concepts
A continuation is a data structure that stores program state. Continuations can be used to execute portions of a program with dynamic and non-deterministic order rather than the lexical order of appearance of expressions.

This proposal implements continuations as objects in the environment that contain the expressions to be evaluated and the state of the environment at the time when the continuation was created.

While the expression body of the continuation and the environment are stored together, they may be utilized together or separately. A continuation may be "called", where the stored expressions are evaluted in its environment. The body of the continuation may be evaluated within an arbitrary environment, or other arbitrary expressions may be evaluated in that continuation's environment.

# Extensions

This proposal extends the capabilities of the closed parentheses pair of operators. In addition to their normal function of forcing precedence in the order of evaluation of expressions, parentheses will now support dynamic variable shadowing via variable declaration statements and the evaluation of multiple expressions provided within a single closed pair of parens.

Closed parentheses pairs will contain their own evaluation environment nested in the outer environment which inherits all objects instantiated in the outer environment and allows for the instantiation of new objects within the scope of the parentheses.

Closed pairs of parentheses may now be assigned a label. The body of labelled parens will not be evaluated, but instead will instantiate a continuation object in the environment at the point where the named parens are evaluated.

# Operation

A labelled parens will instantiate a continuation object in the environment when evaluated.

Each expression enclosed in a pair of parens will have an implicit continuation to the next lexically occurring expression within that pair of parens. This does not apply to solitary expressions because after evaluation completes, execution moves outside the parens.

# Operators

* Continuation Naming Operator: `:`

* Set Continuation Operator: `\^`

	This is a suffix operator which accepts the name of a continuation or an unlabelled expression which execution will move to after the current body has been evaluated.

* Call Continuation Delimiters: `(*` ... `)`

	The first parameter in the delimiters is the name of a continuation or an unlabelled expression surrounded by parens. The second parameter is the name of the next continuation the execution will go to after evaluating the body. If an expression is used in stead of a continuation, the current evaluation environment is used when evaluating the expression.

* Evaluate Using Environment Delimiters: `(#` ... `)`

	The first parameter is the name of a continuation or an expression surrounded by parens. The second parameter may be the name of a continuation or a series of expressions. The expressions or the continuation body will be evaluated in the environment specified by the first paramter.

* Current Continuation/Environment Reference: `.`

	This operator represents a reference to the current evaluation environment when used in an Evaluate Using Environment expression, or a reference to the current next continuation when used in a Call Continuation expression.

* Current Continuation Reference: `.`

	This operator represents a reference to the continuation to be evaluated after evaluation of the current body is complete.

* Previous Evaluation Reference: `<-`

	This operator represents a reference to the body whose evaluation was completed immediately prior to the evaluation of the body currently being evaluated.





# Examples

### Evaluating in Arbitrary Environments

	env1 : (
	  x:Int = 1 y:Int = 2
	)
	env2 : (
	  x:Int = 3 y:Int = 4
	)
	(
	  x:Int = 5 y:Int = 6
	  x + y ``returns 11
	  (# env1 x + y) ``returns 3
	  (# env2 x + y) ``returns 7
	)

### Looping

	(
	  i:Int = 0
	  loop : (
	    ? i < 10
	    {
	      i = i + 1
	      (*loop . ) ``evaluates this body in the current environment
	    }
	    ``if i is not less than 10, move on to the next body
	  )
	)

### Deferring Execution

	(
		X:Int = 4
		``defer first statement
		\^ (* (x = 0) . )
		y:Bool = true
		``defer second statement
		\^ (* (y = false) . )
		``defer third statement
		z = "hello"
		\^ (* (z = "nil") . )
	)
	``once execution leaves this body 
	``	z is set, 
	``	then y is set, 
	``	then x is set.
	``this works because the continuations for each deferred expression are chained so that
	``	body -> z -> y -> x