# Sub Types

Primitive types can be aliased using a type definition.

```
type Double float64
```

However, there is no implicit casting of variables, so if you would like to convert between the `Double` type we just created, and a `float64`, you'll need to do it explicitly. To cast something, use `TypeName<value>`.

```
myDouble var = Double<2.73> 
```

These sub-types can be restricted to only a subset of valid values using a [validator](./Validators.md).

```
type OneToTen uint8 validated
{
	if (value < 1 || value > 10)
		bail
}
```
