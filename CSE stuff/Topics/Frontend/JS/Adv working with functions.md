
----

https://javascript.info/closure

EXPLANATION OF LEXICAL ENVIRONMENT (CLOSURES)

note: lexical env is a theoretical thing. only exists in specifications. js engines implement it differently. 
### [Step 1. Variables](https://javascript.info/closure#step-1-variables)

In JavaScript, every running function, code block `{...}`, and the script as a whole have an internal (hidden) associated object known as the _Lexical Environment_.

The Lexical Environment object consists of two parts:

1. _Environment Record_ – an object that stores all local variables as its properties (and some other information like the value of `this`).
2. A reference to the _outer lexical environment_, the one associated with the outer code.

**A “variable” is just a property of the special internal object, `Environment Record`. “To get or change a variable” means “to get or change a property of that object”.**

In this simple code without functions, there is only one Lexical Environment:

This is the so-called _global_ Lexical Environment, associated with the whole script.
![[Pasted image 20250516220454.png]]
On the picture above, the rectangle means Environment Record (variable store) and the arrow means the outer reference. The global Lexical Environment has no outer reference, that’s why the arrow points to `null`.

As the code starts executing and goes on, the Lexical Environment changes.

Here’s a little bit longer code:

Rectangles on the right-hand side demonstrate how the global Lexical Environment changes during the execution:

1. When the script starts, the Lexical Environment is pre-populated with all declared variables.
    - Initially, they are in the “Uninitialized” state. That’s a special internal state, it means that the engine knows about the variable, but it cannot be referenced until it has been declared with `let`. It’s almost the same as if the variable didn’t exist.
2. Then `let phrase` definition appears. There’s no assignment yet, so its value is `undefined`. We can use the variable from this point forward.
3. `phrase` is assigned a value.
4. `phrase` changes the value.
### [Step 2. Function Declarations](https://javascript.info/closure#step-2-function-declarations)

A function is also a value, like a variable.

**The difference is that a Function Declaration is instantly fully initialized.**

When a Lexical Environment is created, a Function Declaration immediately becomes a ready-to-use function (unlike `let`, that is unusable till the declaration).

That’s why we can use a function, declared as Function Declaration, even before the declaration itself.

For example, here’s the initial state of the global Lexical Environment when we add a function:
![[Pasted image 20250516220648.png]]

Naturally, this behavior only applies to Function Declarations, not Function Expressions where we assign a function to a variable, such as `let say = function(name)...`.

READ STUFF FROM THE LINK

----
