Proposal to Implement all Control Flow Using Delimited Continuations
===

This proposal putlines a control flow primitive system based around the concept of continuations which can be used to compose more complex control flow structures.
---

# Concepts
A continuation is a data structure that stores program state. Continuations can be used to execute portions of a program with dynamic and non-deterministic order rather than the lexical order of appearance of expressions.

This proposal implements continuations as objects in the environment that contain information about the expressions to be evaluated and the state of the environment at the time when the continuation was created.

Continuations may be "called" similarly to functions, but instead of parameters, the continuation call will specify the next continuation that execution will kove to after the continuation being called is evaluated. Continuations may also be dynamically used as an environment for the evaluation of new expressions.

# Extensions

This proposal extends the function of the closed pair of patentheses. Besides their algebraic function of forcing precedence for order of evaluation in expressions, parentheses will now support dynamic variable shadowing and the evaluation of multiple expressions within one closed pair of parens. Variables may be declared inside a closed pair of parens which will shadow variable names used outside the parens.

Closed pairs of parentheses may now be assigned an identifier. Named parens will instantiate an evaluation object in the environment containing the expressions inside the pair as well as the variable names used in the environment at the time of evaluation. This object will be instantiated in the environment when the parentheses are evaluated. The expressions inside the continuation will not be evaluated.

# Operators

* Continuation Naming Operator: `:`

* Continuation Calling Operator: `(^` , `)`

* Evaluate Using Continuation Operator: `(>`, `)`
