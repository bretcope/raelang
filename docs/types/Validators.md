# Validators

Validators are an extension of the type system which allows input to be validated before assignment. They are similar to setters in the sense that they can execute code prior to an assignment, but they are fundamentally different in many other ways.

* They can be applied to [Sub Types](./SubTypes.md) as well as struct fields.
* They cannot mutate any state.
* They can only call non-mutating functions.
* They can _prevent_ an assignment by [bailing](../Bailing.md), but they cannot mutate the value attempting to be assigned or assign a different value.

## Validated Sub Types

```
type OneToTen uint8 validated
{
	if (value < 1 || value > 10)
		bail
}
```

## Validated Fields

```
type MyStruct struct
{
	ShortString string validated
	{
		if (value.Length > 10)
			bail
	}
	
	Constructor (s string)
	{
		ShortString = s
	}
}
```
