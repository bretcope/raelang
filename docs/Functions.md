# Functions

Functions are declared with the `func` keyword, a name (optional in some cases), a parameter list, and a return list.

```
func Add (a int32, b int32) int32
{
	return a + b
}
```

Functions are first-class citizens. They can be assigned to variables, and passed between functions.

```
func DoSomething (callback func (int32) void) void
{
	callback(23)
}
```

Functions can return more than one value.

```
func TwoThings () int, string
{
	return 42, "is the meaning"
}
```

    num, str var = TwoThings()

## Scope

Inside a function, all local variables are block-scoped. Top level functions are package-scoped. If the function definition is prefaced with `export` then it can be called by other packages. 

## Closures

Closures must be created explicitly, as desired.

```
func Stuff (a int32, b int32, c string) void
{
	DoSomething(func [a, b] (val int32) void
	{
		// this can access a and b, but not c
	})
}
```

If you need access to everything in the outer scope, then you can use a wildcard.

```
func [*] (a int32) void
{
	// this has access to everything in the outer scope
}
```

## Function Parameter Shorthand

If you are defining a function as a parameter inside a function call, then you can omit the parameter and return types because they can be inferred from the function you are calling. In the closure example, we could have written that as:

```
DoSomething(func [a, b] (val)
{
})
```

Or even more cleanly if we don't need a closure:

```
DoSomething(func (val)
{
})
```