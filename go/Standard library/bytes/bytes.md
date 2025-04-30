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

### func Compare

```go
func Compare(a, b []byte) int
```

Compare returns an integer comparing two byte slices lexicographically. The result will be 0 if a == b, -1 if a < b, and +1 if a > b. A nil argument is equivalent to an empty slice.

###### Example

```go
package main

import "bytes"

func main() {
	// Interpret Compare's result by comparing it to zero.
	var a, b []byte
	if bytes.Compare(a, b) < 0 {
		// a less b
	}
	if bytes.Compare(a, b) <= 0 {
		// a less or equal b
	}
	if bytes.Compare(a, b) > 0 {
		// a greater b
	}
	if bytes.Compare(a, b) >= 0 {
		// a greater or equal b
	}

	// Prefer Equal to Compare for equality comparisons.
	if bytes.Equal(a, b) {
		// a equal b
	}
	if !bytes.Equal(a, b) {
		// a not equal b
	}
}
```

###### Example (Search)

```go
package main

import (
	"bytes"
	"slices"
)

func main() {
	// Binary search to find a matching byte slice.
	var needle []byte
	var haystack [][]byte	// Assume sorted
	_, found := slices.BinarySearchFunc(haystack, needle, bytes.Compare)
	if founc {
		// Fount it!
	}
}
```

### func Contains

```go
func Contains(b, subslice []byte) bool
```

Contains reports whether subslice is within b.

###### Example

```go
package main

import (
	"bytes"
	"fmt"
)

func main() {
	fmt.Println(bytes.Contains([]byte("seafood"), []byte("foo")))
	fmt.Println(bytes.Contains([]byte("seafood"), []byte("bar")))
	fmt.Println(bytes.Contains([]byte("seafood"), []byte("")))
	fmt.Println(bytes.Contains([]byte(""), []byte("")))
}
```

### func ContainsAny

```go
func ContainsAny(b []byte, chars string) bool
```

ContainsAny reports whether any of the UTF-8-encoded code points in chars are within b.

###### Example

```go
package main

import (
	"bytes"
	"fmt"
)

func main() {
	fmt.Println(bytes.ContainsAny([]byte("I like seafood."), "fÄo!"))
	fmt.Println(bytes.ContainsAny([]byte("I like seafood."), "去是伟大的."))
	fmt.Println(bytes.ContainsAny([]byte("I like seafood."), ""))
	fmt.Println(bytes.ContainsAny([]byte(""), ""))
}
```

### func ContainsFunc

```go
func ContainsFunc(b []byte, f func(rune) bool) bool
```

ContainsFunc reports whether any of the UTF-8-encoded code points r within b satify f(r).

###### Example

```go
package main

import (
	"bytes"
	"fmt"
)

func main() {
	f := func(r rune) bool {
		return r >= 'a' && r <= 'z'
	}
	fmt.Println(bytes.ContainsFunc([]byte("HELLO"), f))
	fmt.Println(bytes.ContainsFunc([]byte("World"), f))
}
```

### func ContainsRune

```go
func ContainsRune(b []byte, r rune) bool
```

ContainsRune reports whether the rune is containted in the UTF-8-encoded byte slice b.

###### Example

```go
package main

import (
	"bytes"
	"fmt"
)

func main() {
	fmt.Println(bytes.ContainsRune([]byte("I like seafood."), 'f'))
	fmt.Println(bytes.ContainsRune([]byte("I like seafood."), 'ö'))
	fmt.Println(bytes.ContainsRune([]byte("去是伟大的!"), '大'))
	fmt.Println(bytes.ContainsRune([]byte("去是伟大的!"), '!'))
	fmt.Println(bytes.ContainsRune([]byte(""), '@'))
}
```

### func Count

```go
func Count(s, sep []byte) int
```

Count counts the number of non-overlapping instances of sep in s. If sep is an empty slice, Count returns 1 + the number of UTF-8-encoded code points in s.

###### Example

```go
package main

import (
	"bytes"
	"fmt"
)

func main() {
	fmt.Println(bytes.Count([]byte("cheese"), []byte("e")))
	fmt.Println(bytes.Count([]byte("five"), []byte("")))	// before & after each rune
}
```

### func Cut

```go
func Cut(s, sep []byte) (before, after []byte, found bool)
```

Cut slices a around the first instance of sep, returning the next before and after sep. The found result reports whether sep appears in s. If sep does not appear in s, cut returns s, nil, false.

Cut returns slices of the original slice s, not copies.

###### Example

```go
package main

import (
	"bytes"
	"fmt"
)

func main() {
	show := func(s, sep string) {
		before, after found := bytes.Cut([]byte(s), []byte(sep))
		fmt.Println("Cut(%q, %q) = %q, %q, %v\n", s, sep, before, after, found)
	}
	show("Gopher", "Go")		//	Cut("Gopher", "Go") = "", "pher", true
	show("Gopher", "ph")		//	Cut("Gopher", "ph") = "Go", "er", true
	show("Gopher", "er")		//	Cut("Gopher", "er") = "Goph", "", true
	show("Gopher", "Badger")	//	Cut("Gopher", "Badger") = "Gopher", "", false
}
```

### func CutPrefix

```go
func CutPrefix(s, prefix []byte) (after []byte, found bool)
```

CutPrefix returns s without the provided leading prefix byte slice and reports whether it found the prefix. If s doesn't start with prefix, CutPrefix returns s, false. If prefix is the empty byte slice, CutPrefix returns s, true.

CutPrefix returns slices of the original slice s, not copies.

###### Example

```go
package main

import (
	"bytes"
	"fmt"
)

func main() {
	show := func(s, sep string) {
		after, found := bytes.CutPrefix([]byte(s), []byte(sep))
		fmt.Printf("CutPrefix(%q, %q) = %q, %v\n", s, sep, after, found)
	}
	show("Gopher", "Go")	//	CutPrefix("Gopher", "Go") = "pher", true
	show("Gopher", "ph")	//	CutPrefix("Gopher", "ph") = "Gopher", false
}
```

### func CutSuffix

```go
func CutSuffix(s, suffix []byte) (before []byte, found bool)
```

CutSuffix returns a without the provided ending suffix byte slice and reports whether it found the suffix. If s doesn't end with suffix, CutSuffix returns s, false. If suffix is the empty byte slice, CutSuffix returns s, true.

CutSuffix returns slices of the original slice s, not copies.

###### Example

```go
package main

import (
	"bytes"
	"fmt"
)

func main() {
	show := func(s, sep string) {
		before, found := bytes.CutSuffix([]byte(s), []byte(sep))
		fmt.Printf("CutSuffix(%q, %q) = %q, %v\n", s, sep, before, found)
	}
	show("Gopher", "Go")	//	CutSuffix("Gopher", "Go") = "Gopher", false
	show("Gopher", "er")	//	CutSuffix("Gopher", "er") = "Goph", true
}
```

### func Equal

```go
func Equal(a, b []byte) bool
```

Equal reports whether a and b are the same length and contain the same bytes. A nil argument is equivalent to an empty slice.

###### Example

```go
package main

import (
	"bytes"
	"fmt"
)

func main() {
	fmt.Println(bytes.Equal([]byte("Go"), []byte("Go")))	//	true
	fmt.Println(bytes.Equal([]byte("Go"), []byte("C++")))	// false
}
```

### func EqualFold

```go
func EqualFold(s, t []byte) bool
```

EqualFold reports whether s and t, interpreted as UTF-8 strings, are equal under simple Unicode case-folding, which is a more general form of case-insensitivity.

###### Example

```go
package main

import (
	"bytes"
	"fmt"
)

func main() {
	fmt.Println(bytes.EqualFold([]byte("Go"), []byte("go")))	//	true
}
```

### func Fields

```go
func Fields(s []byte) [][]byte
```

Fields interprets s as a sequence of UTF-8-encoded code points. It splits the slice a around each instance of one or more consecutive white space characters, as defined by unicode.IsSpace, returning s slice of subslices of s or an empty slice is s contains only white space.

###### Example

```go
package main

import (
	"bytes"
	"fmt"
)

func main() {
	fmt.Printf("Fields are: %q", bytes.Fields([]byte("  foo bar  baz   ")))	// Fields are: ["foo" "bar" "baz"]
}
```

### func FieldsFunc

```go
func FieldsFunc(s []byte, f func(rune) bool) [][]byte
```

FieldsFunc interprets s as s sequence of UTF-8-encoded code points. It splits the slice s at each run of code points c satisfying f(c) and returns a slice of subslices of s. If all code points in s satisfy f(c), or len(s) == 0, an empty slice is returned.

FieldsFunc makes no guarantees about the order in which it calls f(c) and assumes that f always returns the same value for a given c.

###### Example

```go
package main

import (
	"bytes"
	"fmt"
	"unicode"
)

func main() {
	f := func(c rune) bool {
		return !unicode.IsLetter(c) && !unicode.IsNumber(c)
	}
	fmt.Printf("Fields are: %q", bytes.FieldsFunc([]byte("  foo1;bar2,baz3..."), f))	//	Fields are: ["foo1" "bar2" "baz3"]
}
```

### func FieldsFuncSeq

```go
func FieldsFuncSeq(s []byte, f func(rune) bool) iter.Seq[[]byte]
```

FieldsFuncSeq returns an iterator over subslices of s split around runs of Unicode code points satisfying f(c). The iterator yields the same subslices that would be returned by FieldsFunc(s), but without constructing a new slice containing the subslices.

### func FieldsSeq

```go
func FieldsSeq(s []byte) iter.Seq[[]byte]
```

FieldsSeq returns an iterator over subslices of s split around runs of whitespace characters, as defined by unicode.IsSpace. The iterator yields the same subslices that would be returned by Fields(s), but without constructing a new slice containint the subslices.

### func HasPrefix

```go
func HasPrefix(s, prefix []byte) bool
```

HasPrefix reports whether the byte slice s begins with prefix.

```go
package main

import (
	"bytes"
	"fmt"
)

func main() {
	fmt.Println(bytes.HasPrefix([]byte("Gopher"), []byte("Go")))	// true
	fmt.Println(bytes.HasPrefix([]byte("Gopher"), []byte("C")))		// false
	fmt.Println(bytes.HasPrefix([]byte("Gopher"), []byte("")))		// true
}
```

### func Index

```go
func Index(s, sep []byte) int
```

Index returns the idnex of the first instance of sep in s, or -1 if sep is not present in s.

```go
package main

import (
	"bytes"
	"fmt"
)

func main() {
	fmt.Println(bytes.Index([]byte("chicken"), []byte("ken")))	// 4
	fmt.Println(bytes.Index([]byte("chicken"), []byte("dmr")))	// -1
}
```

### func IndexAny

```go
func IndexAny(s []byte, chars string) int
```

IndexAny interprets s as a sequence of UTF-8-encoded Unicode code points. It returns the byte index of the first occurrence in s of any of the Unicode code points in chars. It returns -1 if chars is empty or if there is no code point in common.

###### Example

```go
package main

import (
	"bytes"
	"fmt"
)

func main() {
	fmt.Println(bytes.IndexAny([]byte("chicken"), "aeiouy"))	// 2
	fmt.Println(bytes.IndexAny([]byte("crwth"), "aeiouy"))		// -1
}
```

### func IndexByte

```go
func IndexByte(b []byte, c byte) int
```

IndexByte returns the index of the first instance of c in b, or -1 if c is not present in b.

###### Example

```go
package main

import (
	"bytes"
	"fmt"
)

func main() {
	fmt.Println(bytes.IndexByte([]byte("chicken"), byte('k')))	// 4
	fmt.Println(bytes.IndexByte([]byte("chicken"), byte('g')))	// -1
}
```

### func IndexFunc

```go
func IndexFunc(s []byte, f func(r rune) bool) int
```

IndexFunc interprets s as a sequence of UTF-8-encoded code points. It returns the byte index in s of the first Unicode code point satisfying f(c), or -1 if none do.

###### Example

```go
package main

import (
	"bytes"
	"fmt"
	"unicode"
)

func main() {
	f := func(c rune) bool {
		return unicode.Is(unicode.Han, c)
	}
	fmt.Println(bytes.IndexFunc([]byte("Hello, 世界"), f))		// 7
	fmt.Println(bytes.IndexFunc([]byte("Hello, world"), f))		// -1
}
```

### func IndexRune

```go
func IndexRune(s []byte, r rune) int
```

IndexRune interprets s as a sequence of UTF-8-encoded code points. It returns the byte index of the first occurrence in s of the given rune. It returns -1 if rune is not present in s. If r is `utf8.RuneError`, it returns the first instance of any invalid UTF-8 byte sequence.

```go
package main

import (
	"bytes"
	"fmt"
)

func main() {
	fmt.Println(bytes.IndexRune([]byte("chicken"), 'k'))	// 4
	fmt.Println(bytes.IndexRune([]byte("chicken"), 'd'))	// -1
}
```

### func Join

```go
func Join(s [][]byte, sep []byte) []byte
```

Join concatenates the elements of s to create a new byte slice. The separator sep is placed between elements in the resulting slice.

###### Example

```go
package main

import (
	"bytes"
	"fmt"
)

func main() {
	s := [][]byte{[]byte("foo"), []byte("bar"), []byte("baz")}
	fmt.Printf("%s", bytes.Join(s, []byte(", ")))	// foo, bar, baz
}
```

### func LastIndex

```go
func LastIndex(s, sep []byte) int
```

LastIndex returns the index of the last instance of sep in s, or -1 if sep is not present in s.

###### Example

```go
package main

import (
	"bytes"
	"fmt"
)

func main() {
	fmt.Println(bytes.Index([]byte("go gopher"), []byte("go")))			// 0
	fmt.Println(bytes.LastIndex([]byte("go gopher"), []byte("go")))		// 3
	fmt.Println(bytes.LastIndex([]byte("go gopher"), []byte("rodent")))	// -1
}
```

### func LastIndexAny

```go
func LastIndexAny(s []byte, chars string) int
```

LastIndexAny interprets s as a sequence of UTF-8-encoded Unicode code points. It returns the byte index of the last occurrence in s of any of the Unicode code points in chars. It returns -1 if chars is empty or if there is no code point in common.

###### Example

```go
package main

import (
	"bytes"
	"fmt"
)

func main() {
	fmt.Println(bytes.LastIndexAny([]byte("go gopher"), "MüQp"))	// 5
	fmt.Println(bytes.LastIndexAny([]byte("go 地鼠"), "地大"))		// 3
	fmt.Println(bytes.LastIndexAny([]byte("go gopher"), "z,!."))	// -1
}
```

### func LastIndexByte

```go
func LastIndexByte(s []byte, c byte) int
```

LastIndexByte returns the index of the last instance of c in s, or -1 if c is not present in s.

###### Example

```go
package main

import (
	"bytes"
	"fmt"
)

func main() {
	fmt.Println(bytes.LastIndexByte([]byte("go gopher"), byte('g')))	// 3
	fmt.Println(bytes.LastIndexByte([]byte("go gopher"), byte('r')))	// 8
	fmt.Println(bytes.LastIndexByte([]byte("go gopher"), byte('z')))	// -1
}
```

### func LastIndexFunc

```go
func LastIndexFunc(s []byte, f func(r rune) bool) int
```

LastIndexFunc interprets s as a sequence of UTF-8-encoded code points. It returns the byte index in s of the last Unicode code point satisfying f(c), or -1 if none do.

###### Example

```go
package main

import (
	"bytes"
	"fmt"
	"unicode"
)

func main() {
	fmt.Println(bytes.LastIndexFunc([]byte("go gopher!"), unicode.IsLetter))	// 8
	fmt.Println(bytes.LastIndexFunc([]byte("go gopher!"), unicode.IsPunct))		// 9
	fmt.Println(bytes.LastIndexFunc([]byte("go gopher!"), unicode.IsNumber))	// -1
}
```
