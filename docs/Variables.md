# Variables

## Declaration

Local variables and arguments must start with a __lowercase letter__ followed by zero or more characters `a-zA-Z0-9_`.

Variables are created by specifying a name and a type.

    myVar string

If the variable is going to be assigned immediately, there is no need to declare it first.

    myVar var = "hello"

The type can usually be inferred from the assignment, but if you would like to explicitly declare it, you can do that too.

    myVar string = "hello"

## Immutablility

Variables are immutable by default:

```
str var = "hello"
str = "world" // <- reassigning causes compiler error
```

To make mutable, use the `mut` keyword.

```
str mut = "I'm mutable!"
str = "new value"

// declaration form:
str2 string mut
```
