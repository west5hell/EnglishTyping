# Overview

Package bytes implements functions for the manipulation of byte slices. It is analogous to the facilities of the strings package.

# Index

# Constants

```go
const MinRead = 512
```

MinRead is the minium slice size passed to a Buffer.Read call by Buffer.ReadFrom. As long as the Buffer has at least MinRead bytes beyond what is required to hold the contents of r, Buffer.ReadFrom will not grow the underlying buffer.

# Variables

```go
var ErrTooLarge = errors.New("bytes.Buffer: too large")
```

ErrTooLarge is passed to panic if memory cannot be allocated to store data in a buffer.

# Functions

### func Clone

```go
func Clone(b []byte) []byte
```

Clone Returns a copy of b[:len(b)]. The result may have additional unused capacity. Clone(nil) returns nil.

###### Example

```go
package main

import (
	"bytes"
	"fmt"
)

func main() {
	b := []byte("abc")
	clone := bytes.Clone(b)
	fmt.Printf("%s\n", clone)
	clone[0] = 'd'
	fmt.Printf("%s\n", b)
	fmt.Printf("%s\n", clone)
}
```
