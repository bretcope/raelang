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

## Type Definitions

RaeLang doesn't make any distinction between classes, structs, and interfaces. RaeLang only has type definitions. Reference (declared with `reftype`) types are passed via pointers in assignments, function arguments, and return statements. They can be composed of properties of any types and instances may survive beyond the scope they were created in.
 
 Value types (declared with `valtype`) are always passed by value (copied, as necessary). Their properties can only be other value types. This means that value types _always_ occupy the same total memory space. Instances of value types don't survive beyond the scope they were created in.

```
reftype MyType
{
}
```

All types are internal to a package unless explicitly exported using the `export` keyword.

```
export reftype MyPublicType
{
}
```

### Constructors

There are no implicit constructors. If there is no constructor, then there is no way to construct an instance of that type (such a type might still be useful as an interface).

```
reftype MyType
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

### Members

Visibility is determined by the first character. Public members begin with an uppercase letter. Private members begin with an underscore. Like local variables, members are immutable by default unless .

#### Properties

Properties must be declared inside the type definition block _before_ any method or constructor.

```
reftype MyType
{
	_i int32
	_s string
	MutableProp string mut
	
	Constructor ()
	{
	}
}
```

#### Methods

Methods must be declared inside the type definition block _after_ any properties or constructor.

func `methodName [mut] ( argumentList ) returnType`

```
reftype MyType
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

### Accessors

RaeLang does not support getters and setters in the traditional sense. However, there are field validators and composed properties.

#### Validators

Validated fields are declared using the `validated` keyword, followed by a code block. This code block has read access to the current object, and it can call any non-mutating methods, but it may not modify the state of any scope outside this block (you can't even change other fields on the current object). The `value` keyword contains the value which is attempting to be assigned to the property. The only two possible outcomes are, 1. the value is assigned to the property, or 2. an exception is thrown (preventing assignment). You cannot use a validator to modify the value being assigned. If you need to modify the input, write a method - it's what they're for.

```
reftype MyClass
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

### Composed Properties

Composed properties are similar to traditional getters in that they are calculated at runtime. However, they are evaluated when the object is created, immediately after the constructor finishes. Also unlike a getter, this value is stored with the object and is never re-evaluated. The composition code block is essentially a non-mutating member function.

```
reftype MyType
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
