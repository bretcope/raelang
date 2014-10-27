# Nullables

No type is implicitly nullable. To make it nullable, append the type with `?`.

    myVar MyType? mut = null

The compiler will enforce that null checks are performed in code which uses nullable types before attempting to run any operations on it. You must check for null using:

```
if (myVar != null)
	myVar.MethodCall()
```

Attempting to use the value of a nullable type without checking for null first will cause a compile-time error.