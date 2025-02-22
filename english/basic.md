##  What are some advantages of using Go?
#### Go is an attempt to introduce a new, concurrent, garbage-collected language with fast compilation and the following benefits:
* It is possible to compile a large Go program in a few seconds on a single computer.
* Go provides a model for software construction that makes dependency analysis easy and avoids much of the overhead of C-style include files and libraries.
* Go's type system has no hierarchy, so no time is spent defining the relationships between types. Also, although Go has static types, the language attempts to make types feel lighter weight than in typical OO languages.
* Go is fully garbage-collected and provides fundamental support for concurrent execution and communication.
* By its design, Go proposes an approach for the construction of system software on multicore machines.

---
##  Explain Type Assertions in Go
#### Short answer In one line:

* x.(T) asserts that x is not nil and that the value stored in x is of type T.

#### Why would I use them:
* to check x is nil
* to check what is the dynamic type held by interface x
* to extract the dynamic type from x

#### What exactly they return:
* t := x.(T) => t is of type T; if x is nil, it panics.
* t,ok := x.(T) => if x is nil or not of type T => ok is false otherwise ok is true and t is of type T.

---

##  Difference between any/interface{} as constraint vs. type of argument?
Beside any and interface{} being type aliases — hence, equivalent in usage —, there is a practical difference between any as type parameter and any as regular function argument, as in your example.

The difference is that in printAny[T any](foo T) the type of foo is not any/interface{}, but it's T. And T after instantiation is a concrete type, that may or may not be an interface itself. You can then only pass arguments to an instantiated printAny that can be assigned to that concrete type.

How this impacts your code is most evident with multiple arguments. If we change the function signatures a bit:
```go
func printInterface(foo, bar any) {
    fmt.Println(foo, bar)
}

func printAny[T any](foo, bar T) {
    fmt.Println(foo, bar)
}
```
#### After instantiation:

* the function printAny accepts any two arguments of the same type — whichever is used to instantiate T
* printInterface, which is equivalent to printInterface(foo, bar interface{}) can still accept two arguments of different types, since both would be individually assignable to any/interface{}.
```go
printInterface(12.5, 0.1)    // ok
printInterface(12.5, "blah") // ok
printAny(10, 20)             // ok, T inferred to int, 20 assignable to int
printAny(10, "k")            // not ok, T inferred to int, "k" not assignable to int
printAny[any](10, "k")       // ok, T explicitly instantiated to any, int and string assignable to any

printAny(nil, nil)           // not ok, no way to infer T
printAny[any](nil, nil)      // ok, T explicitly instantiated to any, nil assignable to any
```
---
##  When to use os.Exit() and panic()?
Now, os.Exit and panic are quite different. panic is used when the program, or its part, has reached an unrecoverable state.

When panic is called, including implicitly for run-time errors such as indexing a slice out of bounds or failing a type assertion, it immediately stops execution of the current function and begins unwinding the stack of the goroutine, running any deferred functions along the way. If that unwinding reaches the top of the goroutine's stack, the program dies.

os.Exit is used when you need to abort the program immediately, with no possibility of recovery or running a deferred clean-up statement, and also return an error code (that other programs can use to report what happened). This is useful in tests, when you already know that after this one test fails, the other will fail as well, so you might as well just exit now. This can also be used when your program has done everything it needed to do, and now just needs to exit, i.e. after printing a help message.

Most of the time you won't use panic (you should return an error instead), and you almost never need os.Exit outside of some cases in tests and for quick program termination.

---

##  []byte(string) vs []byte(*string)
#### []byte("something") is not a function (or method) call, it's a type conversion.
#### The type conversion "itself" does not copy the value. Converting a string to a []byte however does, and it needs to, because the result byte slice is mutable, and if a copy would not be made, you could modify / alter the string value (the content of the string) which is immutable, it must be as the Spec: String types section dictates:
#### Strings are immutable: once created, it is impossible to change the contents of a string.
#### Note that there are few cases when string <=> []byte conversion does not make a copy as it is optimized "away" by the compiler. These are rare and "hard coded" cases when there is proof an immutable string cannot / will not end up modified.
#### Such an example is looking up a value from a map where the key type is string, and you index the map with a []byte, converted to string 
```go
key := []byte("some key")

var m map[string]T
// ...
v, ok := m[string(key)]  // Copying key here is optimized away
```
Another optimization is when ranging over the bytes of a string that is explicitly converted to a byte slice:
```go
s := "something"
for i, v := range []byte(s) { // Copying s is optimized away
    // ...
}
```
(Note that without the conversion the for range would iterate over the runes of the string and not over its UTF8-encoded bytes.)

---

##  What is the difference between the string and []byte in Go?
string and []byte are different types, but they can be converted to one another:

* Converting a slice of bytes to a string type yields a string whose successive bytes are the elements of the slice.
* Converting a value of a string type to a slice of bytes type yields a slice whose successive elements are the bytes of the string.

#### Strings are actually very simple: they are just read-only slices of bytes with a bit of extra syntactic support from the language

#### When to use one over the other?

Depends on what you need. Strings are immutable, so they can be shared and you have guarantee they won't get modified.

Byte slices can be modified (meaning the content of the backing array).

Also if you need to frequently convert a string to a []byte (e.g. because you need to write it into an io.Writer()), you should consider storing it as a []byte in the first place.

Also note that you can have string constants but there are no slice constants. This may be a small optimization. Also note that:

The expression len(s) is constant if s is a string constant.

Also if you are using code already written (either standard library, 3rd party packages or your own), in most of the cases it is given what parameters and values you have to pass or are returned. E.g. if you read data from an io.Reader, you need to have a []byte which you have to pass to receive the read bytes, you can't use a string for that.

This example:
```go
bb := []byte{'h','e','l','l','o',127}
```
What happens here is that you used a composite literal (slice literal) to create and initialize a new slice of type []byte (using Short variable declaration). You specified constants to list the initial elements of the slice. You also used a byte value 127 which - depending on the platform / console - may or may not have a visual representation.

---
##  When to use []byte or string in Go?
My advice would be to use string by default when you're working with text. But use []byte instead if one of the following conditions applies:

* The mutability of a []byte will significantly reduce the number of allocations needed.
* You are dealing with an API that uses []byte, and avoiding a conversion to string will simplify your code.
---
##  Why character switching in string is not allowed in Golang?
I understand that Go string is basically an array of bytes.

#### Not exactly. A string is made up of
* a pointer to an array of bytes, and
* an integer that corresponds to the length of that array.

#### notes:
* Strings are immutable: once created, it is impossible to change the contents of a string.
* In Go, a string is in effect a read-only slice of bytes.

Immutability has many advantages—for one thing, it's easy to reason about—but it can be perceived as a nuisance. Of course, overwriting a string variable is legal:
```go
str := "hello"
str = "eello"
```

Moreover, you can always convert the string to a data structure that is mutable (i.e. a []byte or a []rune), make the required changes, and then convert the result back to a string.

```go
str := "hello"
fmt.Println(str)
bs := []byte(str)
bs[0] = bs[1]
str = string(bs)
fmt.Println(str)
```
##  Is Go a new language, framework or library?
#### Go isn't a library and not a framework, it's a new language.
#### Go is mostly in the C family (basic syntax), with significant input from the Pascal/Modula/Oberon family (declarations, packages). Go does have an extensive library, called the runtime, that is part of every Go program. Although it is more central to the language, Go's runtime is analogous to libc, the C library. It is important to understand, however, that Go's runtime does not include a virtual machine, such as is provided by the Java runtime. Go programs are compiled ahead of time to native machine code.

##  What is Go?
#### Go is a general-purpose language designed with systems programming in mind. It was initially developed at Google in year 2007 by Robert Griesemer, Rob Pike, and Ken Thompson. It is strongly and statically typed, provides inbuilt support for garbage collection and supports concurrent programming. Programs are constructed using packages, for efficient management of dependencies. Go programming implementations use a traditional compile and link model to generate executable binaries.

##  What is static type declaration of a variable in Go?
#### Static type variable declaration provides assurance to the compiler that there is one variable existing with the given type and name so that compiler proceed for further compilation without needing complete detail about the variable. A variable declaration has its meaning at the time of compilation only, compiler needs actual variable declaration at the time of linking of the program.

##  What kind of type conversion is supported by Go?
#### Go is very strict about explicit typing. There is no automatic type promotion or conversion. Explicit type conversion is required to assign a variable of one type to another.
#### Consider:
```go
i := 55      //int
j := 67.8    //float64
sum := i + int(j) //j is converted to int
```
##  Why the Go language was created?
#### Go was born out of frustration with existing languages and environments for systems programming.
#### Go is an attempt to have:
* an interpreted, dynamically typed language with
* the efficiency and safety of a statically typed, compiled language
* support for networked and multicore computing
* be fast in compilation
#### To meet these goals required addressing a number of linguistic issues: an expressive but lightweight type system; concurrency and garbage collection; rigid dependency specification; and so on. These cannot be addressed well by libraries or tools so a new language was born.

##  Decoding JSON using json.Unmarshal vs json.NewDecoder.Decode
#### It really depends on what your input is. If you look at the implementation of the Decode method of json.Decoder, it buffers the entire JSON value in memory before unmarshalling it into a Go value. So in most cases it won't be any more memory efficient (although this could easily change in a future version of the language).
#### So a better rule of thumb is this:
* Use json.Decoder if your data is coming from an io.Reader stream, or you need to decode multiple values from a stream of data.
* Use json.Unmarshal if you already have the JSON data in memory.
* Marshal => String
* Encode => Stream
#### For the case of reading from an HTTP request, I'd pick json.Decoder since you're obviously reading from a stream.

##  Concept: Dynamic Type and Dynamic Value of an Interface Value
#### Interface values are values whose types are interface types.

#### Each interface value can box a non-interface value in it. The value boxed in an interface value is called the dynamic value of the interface value. The type of the dynamic value is called the dynamic type of the interface value. An interface value boxing nothing is a zero interface value. A zero interface value has neither a dynamic value nor a dynamic type.

#### An interface type can specify zero or several methods, which form the method set of the interface type.

##  Concept: Concrete Value and Concrete Type of a Value
#### For a (typed) non-interface value, its concrete value is itself and its concrete type is the type of the value.

#### A zero interface value has neither concrete type nor concrete value. For a non-zero interface value, its concrete value is its dynamic value and its concrete type is its dynamic type.

##  Fact: Types Which Support or Don't Support Comparisons
#### Currently (Go 1.19), Go doesn't support comparisons (with the == and != operators) for values of the following types:
* slice types
* map types
* function types
#### any struct type with a field whose type is incomparable and any array type which element type is incomparable.
#### Above listed types are called incomparable types. All other types are called comparable types. Compilers forbid comparing two values of incomparable types.

##  Difference of Json Encoding vs Marshaling and json decoding vs UnMarshaling
#### Don’t let the Golang json.NewEncoder / (json.Encoder).Encode and json.Marshal methods confuse you.

#### They both marshal objects into JSON encoded strings.

#### The difference being the Encoder, first marshals the object to a JSON encoded string, then writes that data to a buffer stream .The Encoder therefore, uses more code and memory overhead than the simpler json.Marshal.

#### Encoding/decoding JSON refers to the process of actually reading/writing the character data to a string or binary form.

#### Marshaling/Un marshaling refers to the process of mapping JSON types from and to Go data types and primitives.

#### In Golang, struct data is converted into JSON using Marshal() and JSON data to string using Un marshal() method.

Encoder and decoder write struct to slice of a stream or read data from a slice of a stream and convert it into a struct. Internally, it also implements the marshal method.

The only difference is if you want to play with string or bytes use marshal, and if any data you want to read or write to some writer interface, use encodes and decode.

##  what are primitive data types?
#### Think of primitive data types as atoms. Atoms are the smallest particles in the universe which cannot be further broken. Like atoms, primitive data types also cannot be further broken down. And like atoms, on combining them, we can form complex data types but more on that later.

##  Primitive data types in Golang
#### int, float, byte, string, rune & bool

## What is an enum in Golang?
#### An enum, or enumerator, is a data type consisting of a set of named constant values. Enums are a powerful feature with a wide range of uses. However, in Golang, they’re implemented quite differently than most other programming languages. In Golang, we use a predeclared identifier, ​iota, and the enums are not strictly typed.
#### Syntax
#### A typical enum Direction with four possible values can be defined as:
```
type Direction int
const (
North Direction = iota
South
East
West
)
```
#### The iota is a built-in, predeclared identifier that represents successive untyped integer constants. Its value is the index of the respective ConstSpec in that constant declaration – it starts at zero.

##  Mutable Data Type
#### Mutable data type is a data type which can be modified without reallocating any chunk of the memory assigned at the time of initialization. In simple words, a variable is mutable if its value can be altered without reallocating itself to a new memory space.
#### In mutable data type the value located in a memory address can be modified. This means we do not have to reallocate any memory or change the pointer of a variable to point to other address in order to change the value of the variable.

##  Immutable Data Types
#### Immutable data type is a data type which cannot be modified without allocating a new memory. So, the immutable data type has to reallocate memory for making changes to the value of a variable. This might be a downside if the variable is holding a large sets of values, it will require a lot of memory re-allocation for a slight change in the value.
#### Immutable data types also mean that you cannot change the value in the memory address which the variable is pointing to, but you can make the variable point to a different memory location under the hood to change or modify the content of a variable.

## In golang there are a few mutable data types
* #### Slice
#### Slice and Arrays are mutable data types in golang, this means the value of the elements in slice or array can be changed after initialization without re-allocations of memory.
* #### Array
#### You won't be able to see the concept of mutable data types with arrays as they are not referenced to any memory address, it is a collection of a single type of value and it is thus static. Since it is not a reference to any memory address, the value of the elements doesn't change if we change the value of an element in the copy of the array.
* #### Map
#### Map is similar to slices in a way they are references to the memory address. A map as we have explored in the seventh part of the series], they are a pair of key and value pairs. The map is internally a reference to a hash map, a hash map is an abstract data type or a structure in Golang, it basically is an array of buckets. Buckets contain high-order bits with a hash(random value) to make the keys distinct in the map. The number of buckets is initially 8, but it expands as required so it doubles the number of buckets and assigns the value to the map elements
* #### Channels

## In golang there are a few immutable data types
* #### Boolean, Int, Float
#### The boolean data type on golang is an immutable data type which means it re-allocates the memory for any change in the value of the boolean variable. Boolean variables are simple as they can have two values either true or false . If we declare a boolean variable initialize it with a value, if we further wanted to change the value of the variable, it is done by reallocating the memory address which was initially holding the value.
* #### Pointers
#### Pointer as well is an immutable data type in golang, we cannot change the value in this case the memory address of a variable which we are pointing to, directly but we need to re-allocate the memory for changing the value of the variable.
* #### String
#### Strings are the classical example of immutable data types in golang, this data type is quite commonly used and is quite important for creating a wide variety of applications. The value of the string variable can be changed but the process happens with/without changing the value of the memory address of the initial value, we have to change the memory address of the variable in order to change the value. This might not be evident by even using &variable_name as it doesn't give the actual insight about the internal working of how the memory address might be processed at run time.
* #### Interfaces
## What Is the Meaningfulness of Type Embedding?
#### The main purpose of type embedding is to extend the functionalities of the embedded types into the embedding type, so that we don't need to re-implement the functionalities of the embedded types for the embedding type.

#### Many other object-oriented programming languages use inheritance to achieve the same goal of type embedding. Both mechanisms have their own benefits and drawbacks.

#### If a type T inherits another type, then type T obtains the abilities of the other type. At the same time, each value of type T can also be viewed as a value of the other type.

#### If a type T embeds another type, then type other type becomes a part of type T, and type T obtains the abilities of the other type, but none values of type T can be viewed as values of the other type.


