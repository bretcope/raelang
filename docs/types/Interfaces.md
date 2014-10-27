# Interfaces

RaeLang does not have an interface type. However, all structs can be used as interfaces, both implicitly and explicitly. Struct definitions without a constructor can _only_ be used as an interface, and therefore their methods do not require a body.

If struct B implements all of the same public members (properties and methods) as stuct A, then instances of struct B are castable into instances of type A. Keep in mind that the signature of all methods and properties must match exactly, including mutability.

```
type IStringReader struct
{
	func Read (length uint32) string
}

type MyReader struct
{
	func Read (length uint32) string
	{
		return "reading from struct B".SubString(0, length)
	}
}
```

```
func ReaderToMyReader (rdr IStringReader) MyReader
{
	return MyReader<rdr>
}
```

If we know that MyReader specifically intends to implement the interface of IStringReader, the we could have explicitly declared this. The decision of whether to explicitly declare intent is a human-factors decision. It may make your code more understandable and produce earlier compile-time errors, but it is not required.

```
type MyReader struct : IStringReader
{
}

type Multiple struct : Interface1, Interface2, Interface3
{
}
```

It is important to note a few things here which may be different from other languages you're used to:

* Even if `IStringReader` had been a instantiable with a constructor, it could still be used as an interface. There is no need to create a separate interface.
* __This is not inheritance.__ A struct which implements the interface of another struct _does not_ inherit any behavior. Struct inheritance is not supported in RaeLang.