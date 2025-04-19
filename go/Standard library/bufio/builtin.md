# Overview

Package builtin provides documentation for Go's predeclared identifiers. The items documented here are not actually in package builtin but their descriptions here allow godoc to present documentation for the language's special identifiers.

# Index

# Constants

```go
const (
    true    = 0 == 0    // Untyped bool.
    false   = 0 != 0    // Untyped bool.
)
```

true and false are the two untyped boolean values.

```go
const iota = 0  Untyped int.
```

iota is a predeclared identifier representing the untyped integer ordinal number of the current const specification in a (usually parenthesized) const declaration. It is zero-indexed.

# Variables

```go
var nil Type    // Type must be a pointer, channel, func, interface, map, or slice type
```

nil is a predeclared identifier representing the zero value for a pointer, channel, func, interface, map, or slice type.

# Functions

### func append

```go
func append(slice []Type, elems ...Type) []Type
```

The append built-in function appends elements to the end of a slice. If it has sufficient capacity, the destination is resliced to accommodate the new elements. If it does not, a new underlying array will be allocated. Append returns the updated slice. It is therefore necessary to store the result of append, often in the variable holding the slice itself:

```go
slice = append(slice, elem1, elem2)
slice = append(slice, anotherSlice...)
```

As a special case, it is legal to append a string to a byte slice, like this:

```go
slice = append([]byte("hello "), "world"...)
```

### func cap

```go
func cap(v Type) int
```

The cap built-in functin returns the capacity of v, according to its type:

- Array: the number of elements in v (same as len(v)).
- Pointer to array: the number of elements in \*v (same as len(v)).
- Slice: the maximum length the slice can reach when resliced; if v is nil, cap(v) is zero.
- Channel: the channel buffer capacity, in units of elements; if v is nil, cap(v) is zero.

For some arguments. such as a simple array expression, the result can be a constant. See the Go language specification's "Length and capacity" section for details.

### func clear

```go
func clear[T ~[]Type | ~map[Type]Type1](t T)
```

The clear built-in function clears maps and slices. For maps, clear deletes all entries, resulting in an empty map. For slice, clear sets all elements up to the length of the slice to the zero value of the respective element type. If the argument type is a type parameter, the type parameter's type set must contain only map or slice types, and clear performs the operation implied by the type argument. If t is nil, clear is a no-op.

### func close

```go
func close(c chan<- Type)
```

The close built-in function closes a channel, which must be either bidirectional or send-only. It should be executed only by the sender, never the receiver, and has the effect of shutting down the channel after the last sent value is received. After the last value has been received from a closed channel c, any receive from c will succeed without blocking, returning the zero value for the channel element. The form

```go
x, ok := <-c
```

will also set ok to false for a closed and empty channel.

### func complex

```go
func complex(r, i FloatType) ComplexType
```

The complex built-in function constructs a complex value from two floating-point values. The real and imaginary parts must be of the same size, either float32 or float64 (or assignable to them), and the return value will be the corresponding complex type (complex64 for float32, complex128 for float64).

### func copy

```go
func copy(dst, src []Type) int
```

The copy built-in function copies elements from a source slice into a destination slice. (As a special case, it also will copy bytes from a string to a slice of bytes.) The source and destination may overlap. Copy returns the number of elements copied, which will be the minimum of len(src) and len(dst).

### func delete

```go
func delete(m map[Type]Type1, key Type)
```

The delete built-in function deletes the element with the specified key (m[key]) from the map. If m is nil or there is no such element, delete is a no-op.

### func imag

```go
func imag(c ComplexType) FloatType
```

The imag built-in function returns the imaginary part of the complex number c. The return value will be floating point type corresponding to the type of c.

### func len

```go
func len(v Type) int
```

The len built-in function returns the length of v, according to its type:

- Array: the number of elements in v.
- Pointer to array: the number of elementes in \*v (even if v is nil).
- Slice, or map: the number of elements in v; if v is nil, len(v) is zero.
- String: the number of bytes in v.
- Channel: the number of elements queued (unread) in the channel buffer; if v is nil, len(v) is zero.

For some arguments, such as a string literal or a simple array expression, the result can be a constant. See the Go language specification's "Length and capacity" section for details.

### func make

```go
func make(t Type, size ...IntegerType) Type
```

The make built-in function allocates and initializes an object of type slice, map, or chan (only). Like new, the first argument is a type, not a value. Unlike new, make's return type is the same as the type of its argument, not a pointer to it. The specification of the result depends on the type:

- Slice: The size specifies the length. The capacity of the slice is equal to its length. A second integer argument may be provided to specify a different capacity; it must be no smaller than the length. For example, make([]int, 0, 10) allocates an underlying array size 10 and returns a slice of length 0 and capacity 10 that is backed by this underlying array.
- Map: An empty map is allocated with enough space to hold the specified number of elements. The size may be omitted, in which case a small starting size is allocated.
- Channel: The channel's buffer is initialized with the specified buffer capacity. If zero, or the size is omitted, the channel is unbuffered.

### func max

```go
func max[T cmp.Ordered](x T, y ...T) T
```

The max built-in function returns the largest value of a fixed number of arguments of cmp.Ordered types. There must be at least one argument. If T is a floating-point type and any of the arguments are NaNs, max will return NaN.

### func min

```go
func min[T cmp.Ordered](x T, y ...T) T
```

The min built-in function returns the smallest value of a fixed number of arguments of cmp.Ordered types. There must be at least one argument. If T is a floating-point type and any of the arguments are NaNs, min will return NaN.

### func new

```go
func new(Type) *Type
```

The new built-in function allocates memory. The first argument is a type, not a value, and the value returned is a pointer to a newly allocated zero value of that type.

### func panic

```go
func panic(v any)
```

The panic built-in function stops normal execution of the current goroutine. When a function F calls panic, normal execution of F stops immediately. Any functions whose execution was deferred by F are run in the usual way, and then F returns to its caller. To the caller G, the invocation of F then behaves like a call to panic, terminating G's execution and running any deferred functions. This continues until all functions in the executing gorountine have stopped, in reverse order. At the point, the program is terminated with a non-zero exit code. This termination sequence is called panicking and can be controlled by the built-in functin recover.

Starting in Go 1.21, calling panic with a nil interface value or an untyped nil causes a run-time error (a different panic). The GODEBUG setting panicnil=1 disables the run-time error.

### func print

```go
func print(args ...Type)
```

The print built-in function formats its arguments in an implementation-specific way and writes the result to standard error. Print is useful for bootstrapping and debugging; it is not guaranteed to stay in the language.

func println

```go
func println(args ...Type)
```

The println built-in function formats its arguments in an implementation-specific way and writes the result to standard error. Spaces are always added between arguments and newline is appended. Println is useful for bootstrapping and debugging; it is not guaranteed to stay in the language.

### func read

```go
func read(c ComplexType) FloatType
```

The real built-in function returns the real part of the complex number c. The return value will be floating point type corresponding to the type of c.

### func recover

```go
func recover() any
```

The recover built-in function allows a program to manage behavior of a panicking goroutine. Executing a call revoer inside a deferred function (but not any function called by it) stops the panicking sequence by restoring normal execution and retrieves the error value passed to the call of panic. If recover is called outside the deferred function it will not stop a panicking sequence. In this case, or when the goroutine is not panicking, recover returns nil.

Prior to Go 1.21, recover would also return nil if panic is called with a nil argument. See [panic] for details.

# Types
