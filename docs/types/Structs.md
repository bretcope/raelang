# Structs

Struct types are similar to classes and structs in other languages.

```
type MyStruct struct
{
}
```

## Fields

* All fields must be declared at the top of the struct block.
* By default, fields are immutable. To make them mutable, use the `mut` keyword.

```
type MyStruct struct
{
	// private properties begin with an underscore
	_i int32
	_s string mut // <- this property will be mutable
	
	// public properties begin with an uppercase letter
	SomethingPublic string
}
```

## Constructors

* The constructor must be declared _after_ all properties.
* There are no implicit constructors. If there is no constructor, then there is no way to construct an instance of that struct (such a type might still be useful as an interface).

```
type MyStruct struct
{
	_i int32
	_s string
	
	Constructor (i int32, s string)
	{
		_i = i
		_s = s
	}
}
```

## Methods

* Methods must be declared inside the type definition block _after_ any properties or constructor.
* The first character (underscore or uppercase letter) determines visibility, just like with properties.
* Methods follow the exact same syntax as top-level [functions](../Functions.md). The only difference is that they are placed inside the struct definition block.

```
type MyType struct
{
	_i int32
	_s string
	
	Constructor ()
	{
	}
	
	func MyPublicMethod (s string) string
	{
		return _s + s
	}
	
	func _MyPrivateMethod (i int32) int32
	{
		return _i + i
	}
}
```

Method signatures cannot be overloaded. However, defaults may be provided.

```
func MyMethod (s string = "hello") string
```

Just like properties, methods are non-mutating by default. You must opt-in to the complexity of state mutation as you need it. If you intend to mutate properties of the "this" object, then declare the method as self-mutating:

```
func MyMethod mut (s string) void
{
	MyProp = s
}
```

If you intend to mutate an input parameter, you must declare it as mutable:

```
func MyMethod (x SomeType mut) void
{
	x.SomeProp += 7
}
```

It is important to understand that, for obvious reasons, non-mutating methods cannot call any mutating methods.

## Accessors

RaeLang does not support getters and setters in the traditional sense. However, there are validated fields and lazy fields which provide some of the benefits of getters and setters without the side-effects.

### Validated Fields

See [Validators](./Validators.md).

### Lazy Fields

Lazy fields are similar to traditional getters in that they are calculated at runtime. However, they are lazy-evaluated when accessed the first time, then cached and never re-evaluated.

```
type MyType struct
{
	FirstName string
	LastName string
	FullName string lazy
	{
		value = FirstName + " " + LastName
	}
	
	// constructor ...
}
```

Lazy fields can only access immutable fields and methods, and lazy fields cannot be used inside constructors. If you need a traditional getter which can potentially produce different values at different points in an object's lifetime, then just use a function... that's what they're for.
