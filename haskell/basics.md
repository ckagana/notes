# Basics

- [The interactive shell](#the-interactive-shell)
- [Using functions](#using-functions)
- [Conditionals](#conditionals)


## The interactive shell

The *Glasgow Haskell Compiler (GHC)* provides an interactive shell, invoked issuing the `ghci` command:

```
$ ghci
Prelude>
```

The default prompt shows the name of the currently loaded modules. The `prelude` module is always loaded by default.

It's possible to customize the prompt look, for example with `:set prompt "ghci> "`. This comes in handy in particular when loading several modules, because all their names will be displayed inside the prompt, thus making it rather long.

All commands starting with `:` are meta-commands rather than proper Haskell function names, and are used to give instructions to the interactive shell.


## Using functions

In Haskell all operations are performed with functions: even basic arithmetic operators, like `+` and `*` are actually defined as functions. Thus, all functions can be *prefix* or *infix*. Operators are functions whose names are comprised only of special characters, like `==`, `+`, `*`, etc. Operators working with two arguments are infix by default:

```
ghci> 1 * 2
2
```

meaning that the operator is places between its two arguments. On the other hand, functions are prefix by default, like:

```
ghci> min 9 10
9
```

where first comes the name of the function, and then the list of its arguments, separated by spaces.

Functions have higher precedence than operators, for example:

```
ghci> max 5 4 + 1
6
```

which is just like:

```
ghci> (max 5 4) + 1
6
```

Functions that take two parameters can be written with infix notation as well, as long as they are wrapped in backticks:

```
ghci> 92 `div` 10
9.2
```

Similarly, operators can be written with prefix notation, as long as they are surrounded by parenthesis:

```
ghci> (==) 3 2
False
```

To define a new identifier, and thus also a new function, in the interactive shell, we must write a declaration, using the `let` keyword:

```
ghci> let doubleUs x y = x * 2 + y * 2
ghci> doubleUs 2.3 34.2
73.0
```

Inside source files, instead, the `let` declaration can be omitted. Source files are loaded into the interactive shell using the `:l` command:

```haskell
-- functions.hs
doubleUs x y = x * 2 + y * 2
```

```
ghci> :l functions
ghci> doubleUs 2.3 34.2
73.0
```

Since `'` is a valid character to use in function names, we can write functions named `conanO'Brien`, or `myFunction'`. In fact, using an apostrophe as the last character of a function name is a common practice to identify a slightly different version of a previously defined function.

Functions' names can't begin with an uppercase letter. Furthermore, functions that don't take any argument are called *definitions*, and are actually interchangeable with their body, since they don't depend on any parameter, like the idea of constants in other languages.


## Conditionals

Conditional structures in Haskell are quite similar to those in other languages:

```haskell
doubleSmallNumber x = if x > 100
    then x
    else x * 2

doubleSmallNumber x = (if x > 100 then x else x * 2) + 1
```

However, in Haskell the `else` branch is mandatory, and this is because, like everything else, conditionals in Haskell are expressions, so they should always produce a value, and if the condition isn't met, we still have to tell the program what the value of the expression is then.
