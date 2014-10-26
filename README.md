# RaeLang

This isn't a real language. It will likely never be a real language. It's more of a thought experiment on how I might design a language if I were designing one.
  
## Big Picture

The primary guiding principle is "opt into complexity." It borrows from functional styles in that it strongly encourages immutablility and discourages side-effects, but still allows you to opt-in to that complexity if you really need it. Most of these ideas come from other languages, and I'm sure even the stuff that I came up with on my own has probably already been implemented in some language (possibly better than the way I describe it here).

Again, this isn't a real language or spec. It's incomplete, and all my opinions are subject to change. There are tons more things I haven't even touched on yet and/or haven't thought through well.

## Variables

### Declaration

Local variables and arguments must start with a __lowercase letter__ followed by zero or more characters `a-zA-Z0-9_`.

Variables are created by specifying a name and a type.

    myVar string

If the variable is going to be assigned immediately, there is no need to declare it first.

    myVar var = "hello"

The type can usually be inferred from the assignment, but if you would like to explicitly declare it, you can do that too.

    myVar string = "hello"

### Immutablility

Variables are immutable by default:

```
str var = "hello"
str = "world" // <- throws exception
```

To make mutable, use the mut keyword.

```
str mut = "I'm mutable!"
str = "new value"

// declaration form:
str2 string mut
```

### Nullables

No type is implicitly nullable. To make it nullable, append the type with `?`.

    myVar MyType? mut = null

The compiler will enforce that null checks are performed in code which uses nullable types before attempting to run any operations on it. You must check for null using:

```
if (myVar != null)
	myVar.MethodCall()
```

Attempting to use the value of a nullable type without checking for null first will cause a compile-time error.

## Functions

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

### Scope

Inside a function, all local variables are block-scoped. Top level functions are package-scoped. If the function definition is prefaced with `export` then it can be called by other packages. 

### Closures

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

### Function Parameter Shorthand

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

## Types

All type definitions in RaeLang are backed by class, struct, a primitive, or derived type.

### Primitive Types

Primitive types are built into the language. They are the only types which are allowed to begin with a lowercase letter. All derived types must begin with a capital letter.

The following primitives exist in the language:

| Type Group | Primitive Names |
| :--------- | :-------------- |
| Integers | int8, int16, int32, int64, uint8, uint16, uint32, uint64 |
| Floating Point | float32, float64 |
| Binary | byte |
| Boolean | bool |
| Character | char, string |

### Derived Types

Primitive types can be aliased using a type definition.

```
type Double float64
```

However, there is no implicit casting of variables, so if you would like to convert between the `Double` type we just created, and a `float64`, you'll need to do it explicitly. To cast something, use `TypeName<value>`.

```
myDouble var = Double<2.73> 
```

These derived types can be restricted to only a subset of valid values using a validator.

```
type OneToTen uint8 validated
{
	if (value < 1 || value > 10)
		throw new Exception()
}
```

See the section on Validators later in this document for more information.

### Class and Struct Types

Class and struct types are similar to classes and structs in other languages. The only difference between a class and a struct is that a class can have mutable properties, and a struct cannot (making a struct entirely immutable).

```
type MyClass class
{
}
```

#### Properties

* All properties must be declared at the top of the class or struct block.
* Struct properties cannot be mutable. Class properties can be mutable, but only when explicitly labeled as such using the `mut` keyword.

```
type MyClass class
{
	// private properties begin with an underscore
	_i int32
	_s string mut // <- this property will be mutable
	
	// public properties begin with an uppercase letter
	SomethingPublic string
}
```

#### Constructors

* The constructor must be declared _after_ all properties.
* There are no implicit constructors. If there is no constructor, then there is no way to construct an instance of that class/struct (such a type might still be useful as an interface).

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

#### Methods

* Methods must be declared inside the type definition block _after_ any properties or constructor.
* The first character (underscore or uppercase letter) determines visibility, just like with properties.
* Methods follow the exact same syntax as top-level functions. The only difference is that they are placed inside the class/struct definition block.

```
type MyType class
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

#### Accessors

RaeLang does not support getters and setters in the traditional sense. However, there are field validators and composed properties.

##### Validators

Validated fields are declared using the `validated` keyword, followed by a code block. This code block has read access to the current object, and it can call any non-mutating methods, but it may not modify the state of any scope outside this block (you can't even change other fields on the current object). The `value` keyword contains the value which is attempting to be assigned to the property. The only two possible outcomes are, 1. the value is assigned to the property, or 2. an exception is thrown (preventing assignment). You cannot use a validator to modify the value being assigned. If you need to modify the input, write a method - it's what they're for.

```
type MyClass class
{
	EmailAddress string validated
	{
		if (!value.Contains('@'))
			throw new Exception('Email addresses must contain an @ symbol.')
	}
	
	Constructor (email string)
	{
		EmailAddress = email
	}
}

// example usage
c1 var = new MyClass("test@example.com")
c2 var = new MyClass("test") // throws an exception
```

Think of validators as an extension of the type system, rather than a setter.

##### Composed Properties

Composed properties are similar to traditional getters in that they are calculated at runtime. However, they are evaluated when the object is created, immediately after the constructor finishes. Also unlike a getter, this value is stored with the object and is never re-evaluated. The composition code block is essentially a non-mutating member function.

```
type MyType struct
{
	FirstName string
	LastName string
	FullName string composed
	{
		return FirstName + " " + LastName
	}
	
	// constructor ...
}
```

The best way to think of composed properties is as a distributed constructor. It's nice syntax sugar, but it also helps keep constructor sizes small by removing one more tedious task you might have to otherwise put in there. If you need a traditional getter which can potentially produce a different value at different times in an object's lifetime, you should use a function.

### Interfaces

RaeLang does not have an interface type. However, all classes and structs can be used as interfaces, both implicitly and explicitly. Struct and class definitions without a constructor can _only_ be used as an interface, and therefore their methods do not require a body.

If struct B implements all of the same public members (properties and methods) as stuct A, then instances of struct B are castable into instances of type A. Keep in mind that the signature of all methods and properties must match exactly, including mutability.

```
type IStringReader class
{
	func Read (length uint32) string
}

type MyReader class
{
	func Read (length uint32) string
	{
		return "reading from class B".SubString(0, length)
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
type MyReader class : IStringReader
{
}

type Multiple class : Interface1, Interface2, Interface3
{
}
```

It is important to note a few things here which may be different from other languages you're used to:

* Even if `IStringReader` had been a full class with a constructor, it could still be used as an interface. There is no need to create a separate interface.
* __This is not inheritance.__ A class which implements the interface of another class _does not_ inherit any behavior. Class inheritance is not supported in RaeLang.
* Struct types can only implement other struct types, and class types can only implement other class types. When designing an interface, it's important to consider whether mutable properties will be required in types which implement the interface.

## Exceptions

Error handling is a controversial topic in many, if not most, languages. Should you throw an exception, return an error, have special check-error methods, return null, or... ? Languages which return an error or error code, often suffer from programmers simply ignoring the error. Languages which implement exceptions suffer from the line noise of try/catch blocks, and, in the end, they often just ignore the error as well.

RaeLang uses an approach which is in-between the two traditional approaches, but tries to encourage better practices than either.

### When to use Exceptions

There is exactly one reason to ever use an exception: when there is not a clear and definitively correct code path to continue on. Exceptions are not really "exceptional" because, in many cases, they are actually expected behavior. So let's not even call them that. RaeLang calls exceptions "BailReports" because all it really tells you, for sure, is that the function bailed out of execution.

### BailReport Struct

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

### How to use Exceptions (bailing) in RaeLang

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
