# Bailing (Exceptions)

Error handling is a controversial topic in many, if not most, languages. Should you throw an exception, return an error, have special check-error methods, return null, or... ? Languages which return an error or error code, often suffer from programmers simply ignoring the error. Languages which implement exceptions suffer from the line noise of try/catch blocks, and, in the end, they often just ignore the error as well.

RaeLang uses an approach which is in-between the two traditional approaches, but tries to encourage better practices than either.

## When to use Exceptions

There is exactly one reason to ever use an exception: when there is not a clear and definitively correct code path to continue on. Exceptions are not really "exceptional" because, in many cases, they are actually expected behavior. So let's not even call them that. RaeLang calls exceptions "BailReports" because all it really tells you, for sure, is that the function bailed out of execution without returning what it promised to return.

## BailReport Struct

The BailReport struct has the following interface:

```
type BailReport struct
{
	Message string
	Code int
	Data struct?
	StackTrace StackItem[]
	
	Constructor (message string = "", code int = 0, data struct? = null)
	
	func ToString () string
}
```

Any struct which implements the BailReport interface can be used in a bail statement.

> StackItem interface hasn't been specified yet... probably file, line, character, and function name.

## How to use Exceptions (bailing) in RaeLang

RaeLang does not have try/catch blocks, and therefore we don't use the word `throw` either. Instead, we use `bail`. Imagine we have a function which bails when the lookup value isn't available:

```
func Lookup (i int) string
{
	switch (i)
	{
		0:
			return "none"
		1, 2, 3:
			return "a few"
		default:
			bail new BailReport("I don't know what you're talking about")
	}
}
```

When we call this function, we can choose to either capture this bail report or not. To capture it, use a special variable (prefixed with a dollar sign) as the first variable in the return list.

```
$br, str var = Lookup(7)
```

Just capturing the bail report is not enough to prevent it from bubbling up the call stack. We have to explicitly handle the bail report using an `unbail` block. However, __the unbail block is not a catch__ block. It gives you the _opportunity_ to recover by providing alternate values for variables which were supposed to have been assigned by the function call.

In the above example, we are attempting to assign the `str` variable, but if the function bails, then `str` will be undefined, which is not a valid value. If, by the end of the unbail block, `str` is assigned a value, then execution will continue. Otherwise the current function will bail, propagating the original bail report up the call stack.

```
$br, str var = Lookup(7)
unbail ($br)
{
	if ($br.Message == "I don't know what you're talking about")
	{
		str = "lots"
	}
}
```

Alternatively, you could execute a return statement inside the unbail block which would also prevent an implicit bail.

You can perform any code you want inside the unbail block, including assigning other variables or performing function calls. It is the only block where you can access the special `$br` variable.

Any code which is in-between the bail point and the unbail block will not execute in the case of a bail. However, the unbail must be in the same block as the bail point. For example, the following code is __not__ valid:

```
if (x == y)
{
	$br, str var = Lookup(7)
}
else
{
	$br, str var = Lookup(8)
}

unbail ($br)
{
	// ...
}
```

However, this code would be valid:

```
if (x == y)
{
	$br, str = Lookup(7)
	
	// ... do something which will be skipped if Lookup bails
	
	unbail ($br)
	{
		return false
	}
}
```

because the bail and unbail are in the same block.

### Bail Shorthand

```
bail "message"
```

is shorthand for

```
bail new BailReport("message")
```
