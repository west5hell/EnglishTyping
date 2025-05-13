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

### func Lines

```go
func Lines(s []byte) iter.Seq[[]byte]
```

Lines returns an iterator over the newline-terminated lines in the byte slice s. The lines yielded by the iterator include their terminating newlines. If s is empty, the iterator yields no lines at all. If s does not end in a newline, the final yielded line will not end in a newline. It returns a single-user iterator.

### func Map

```go
func Map(mapping func(r rune) rune, s []byte) []byte
```

Map returns a copy of the byte slice s with all its characters modified according to the mapping function. If mapping returns a negative value, the character is dropped from the byte slice with no replacement. The characters in s and the output are interpreted as UTF-8-encoded code points.

###### Example

```go
package main

import (
	"bytes"
	"fmt"
)

func main() {
	rot13 := func(r rune) rune {
		switch {
			case r >= 'A' && r <= 'Z':
				return 'A' + (r-'A'+13)%26
			case r >= 'a' && r <= 'z':
				return 'a' + (r-'a'+13)%26
		}
		return r
	}
	fmt.Printf("%s\n", bytes.Map(rot13, []byte("'Twas brillig and the slithy gopher...")))	// 'Gjnf oevyyvt naq gur fyvgul tbcure...
}
```

### func Repeat

```go
func Repeat(b []byte, count int) []byte
```

Repeat returns a new byte slice consisting of count copies of b.

It panics if count is negative or if the result of (len(b) \* count) overflows.

###### Example

```go
package main

import (
	"bytes"
	"fmt"
)

func main() {
	fmt.Printf("ba%s", bytes.Repeat([]byte("na"), 2))	// banana
}
```

### func Replace

```go
func Replace(s, old, new []byte, n int) []byte
```

Replace returns a copy of the slice s with the first n non-overlapping instances of old replaced by new. If old is empty, it matches at the beginning of the slice and after each UTF-8 sequence, yielding up to k+1 replacements for a k-rune slice. If n < 0, there is no limit on the number of replacements.

###### Example

```go
package main

import (
	"bytes"
	"fmt"
)

func main() {
	fmt.Printf("%s\n", bytes.Replace([]byte("oink oink oink"), []byte("k"), []byte("ky"), 2))		// oinky oinky oink
	fmt.Printf("%s\n", bytes.Replace([]byte("oink oink oink"), []byte("oink"), []byte("moo"), -1))	// moo moo moo
}
```

### func ReplaceAll

```go
func ReplaceAll(s, old, new []byte) []byte
```

ReplaceAll returns a copy of the slice s with all non-overlapping instances of old replaced by new. If old is empty, it matches at the beginning of the slice and after each UTF-8 sequence, yielding up to k+1 replacements for a k-rune slice.

###### Example

```go
package main

import (
	"bytes"
	"fmt"
)

func main() {
	fmt.Printf("%s\n", bytes.ReplaceAll([]byte("oink oink oink"), []byte("oink"), []byte("moo")))	// moo moo moo
}
```

### func Runes

```go
func Runes(s []byte) []rune
```

Runes interprets s as a sequence of UTF-8-encoded code points. It returns a slice of runes (Unicode code points) equivalent to s.

###### Example

```go
package main

import (
	"bytes"
	"fmt"
)

func main() {
	rs := bytes.Runes([]byte("go gopher"))
	for _, r := range rs {
		fmt.Printf("%#U\n", r)
	}
}
```

### func Split

```go
func Split(s, sep []byte) [][]byte
```

Split slice s into all subslices separated by sep and returns a slice of the subslices between those sepatators. If sep is empty, Split splits after each UTF-8 sequence. It is equivalent to SplitN with a count of -1.

To split around the first instance of a separator, see `Cut`.

###### Example

```go
package main

import (
	"bytes"
	"fmt"
)

func main() {
	fmt.Printf("%q\n", bytes.Split([]byte("a,b,c"), []byte(",")))							// ["a" "b" "c"]
	fmt.Printf("%q\n", bytes.Split([]byte("a man a plan a canal panama"), []byte("a ")))	// ["" "man " "plan " "canal panama"]
	fmt.Printf("%q\n", bytes.Split([]byte(" xyz "), []byte("")))							// [" " "x" "y" "z" " "]
	fmt.Printf("%q\n", bytes.Split([]byte(""), []byte("Bernardo O'Higgins")))				// [""]
}
```

### func SplitAfter

```go
func SplitAfter(s, sep []byte) [][]byte
```

SplitAfter slices s into all subslices after each instance of sep and returns a slice of those subslices. If sep is empty, SplitAfter splits after each UTF-8 sequence. It is equivalent to SplitAfterN with a count of -1.

###### Example

```go
package main

import (
	"bytes"
	"fmt"
)

func main() {
	fmt.Printf("%q\n", bytes.SplitAfter([]byte("a,b,c"), []byte(",")))	// ["a," "b," "c"]
}
```

### SplitAfterN

```go
func SplitAfterN(s, sep []byte, n int) [][]byte
```

SplitAfterN slices s into subslices after each instance of sep and returns a slice of those subslices. If sep is empty, SplitAfterN splits after each UTF-8 sequence. The count determines the number of subslices to return:

- n > 0: at most n subslices; the last subslice will be the unsplit remainder;
- n == 0: the result is nil (zero subslices);
- n < 0: all subslices.

```go
package main

import (
	"bytes"
	"fmt"
)

func main() {
	fmt.Printf("%q\n", bytes.SplitAfterN([]byte("a,b,c"), []byte(","), 2))	// ["a," "b,c"]
}
```

### func SplitAfterSeq

```go
func SplitAfterSeq(s, sep []byte) iter.Seq[[]byte]
```

SplitAfterSeq returns an iterator over subslices of s split after each instance of sep. The iterator yields the same subslices that would be returned by SplitAfter(s, sep), but without constructing a new slcie containing the subslices. It returns a single-use iterator.

### func SplitN

```go
func SplitN(s, sep []byte, n int) [][]byte
```

SplitN slices s into subslices separated by sep and returns a slice of the subslices between those separators. If sep is empty, SplitN splits after each UTF-8 sequence. The count determines the number of subslices to return:

- n > 0: at most n subslices; the last subslice will be the unsplit remainder;
- n == 0: the result is nil (zero subslices);
- n < 0: all subslices.

To split around the first instance of s separator, see Cut.

###### Example

```go
package main

import (
	"bytes"
	"fmt"
)

func main() {
	fmt.Printf("%q\n", bytes.SplitN([]byte("a,b,c"), []byte(","), 2))	// ["a" "b,c"]
	z := bytes.SplitN([]byte("a,b,c"), []byte(","), 0)
	fmt.Printf("%q (nil = %v)\n", z, z == nil)	// [] (nil = true)
}
```

### func SplitSeq

```go
func SplitSeq(s, sep []byte) iter.Seq[[]byte]
```

SplitSeq returns an iterator over all subslices of s separated by sep. The iterator yields the same subslices that would be returned by Split(s, sep), but without constructing a new slice containing the subslices. It returns a single-use iterator.

### func ToLower

```go
func ToLower(s []byte) []byte
```

ToLower returns a copy of the byte slice s with all Unicode letters mapped to their lower case.

###### Example

```go
package main

import (
	"bytes"
	"fmt"
)

func main() {
	fmt.Printf("%s", bytes.ToLower([]byte("Gopher")))	// gopher
}
```

### func ToLowerSpecial

```go
func ToLowerSpecial(c unicode.SpecialCase, s []byte) []byte
```

ToLowerSpecial treats s as UTF-8-encoded bytes and returns a copy with all the Unicode letters mapped to their lower case, giving priority to the special casing rules.

###### Example

```go
package main

import (
	"bytes"
	"fmt"
	"unicode"
)

func main() {
	str := []byte("AHOJ VÝVOJÁRİ GOLANG")
	totitle := bytes.ToLowerSpecial(unicode.AzeriCase, str)
	fmt.Println("Original : " + string(str))	//	Original : AHOJ VÝVOJÁRİ GOLANG
	fmt.Println("ToLower : " + string(totitle))	//	ToLower : ahoj vývojári golang
}
```

### func ToTitle

```go
func ToTitle(s []byte) []byte
```

ToTitle treats s as UTF-8-encoded bytes and returns a copy with all the Unicode letters mapped to their title Case.

###### Example

```go
package main

import (
	"bytes"
	"fmt"
)

func main() {
	fmt.Printf("%s\n", bytes.ToTitle([]byte("loud noises")))	//	LOUD NOISES
	fmt.Printf("%s\n", bytes.ToTitle([]byte("брат")))			//	БРАТ
}
```

### func ToTitleSpecial

```go
func ToTitleSpecial(c unicode.SpecialCase, s []byte) []byte
```

ToTitleSpecial treats s as UTF-8-encoded bytes and returns a copy with the Unicode letters mapped to their title case, giving priority to the special casing rules.

```go
package main

import (
	"bytes"
	"fmt"
	"unicode"
)

func main() {
	str := []byte("ahoj vývojári golang")
	totitle := bytes.ToTitleSpecial(unicode.AzeriCase, str)
	fmt.Println("Original : " + string(str))	//	Original : ahoj vývojári golang
	fmt.Println("ToTitle : " + string(totitle))	//	ToTitle : AHOJ VÝVOJÁRİ GOLANG
}
```

### func ToUpper

```go
func ToUpper(s []byte) []byte
```

ToUpper returns a copy of the byte slice s with all Unicode letters mapped to their upper case.

###### Example

```go
package main

import (
	"bytes"
	"fmt"
)

func main() {
	fmt.Printf("%s", bytes.ToUpper([]byte("Gopher")))	//	GOPHER
}
```

### func ToUpperSpecial

```go
func ToUpperSpecial(c unicode.SpecialCase, s []byte) []byte
```

ToUpperSpecial treats s as UTF-8-encoded bytes and returns a copy with all the Unicode letters mapped to their upper case, giving priority to the special casing rules.

###### Example

```go
package main

import (
	"bytes"
	"fmt"
	"unicode"
)

func main() {
	str := []byte("ahoj vývojári golang")
	totitle := bytes.ToUpperSpecial(unicode.AzeriCase, str)
	fmt.Println("Original : " + string(str))	//	Original : ahoj vývojári golang
	fmt.Println("ToUpper : " + string(totitle))	//	ToUpper : AHOJ VÝVOJÁRİ GOLANG
}
```

### func ToValidUTF8

```go
func ToValidUTF8(s, replacement []byte) []byte
```

ToValidUTF8 treats s as UTF-8-encoded bytes and returns a copy with each run of bytes representing invalid UTF-8 replaced with bytes in replacement, which may be empty.

###### Example

```go
package main

import (
	"bytes"
	"fmt"
)

func main() {
	fmt.Printf("%s\n", bytes.ToValidUTF8([]byte("abc"), []byte("\uFFFD")))				//	abc
	fmt.Printf("%s\n", bytes.ToValidUTF8([]byte("a\xffb\xC0\xAFc\xff"), []byte("")))	//	abc
	fmt.Printf("%s\n", bytes.ToValidUTF8([]byte("\xed\xa0\x80"), []byte("abc")))		//abc
}
```

### func Trim

```go
func Trim(s []byte, cutset string) []byte
```

Trim returns a subslice of s by slicing off all leading and trailing UTF-8-encoded code points contained in cutset.

###### Example

```go
package main

import (
	"bytes"
	"fmt"
)

func main() {
	fmt.Printf("[%q]", bytes.Trim([]byte(" !!! Achtung! Achtung! !!! "), "! "))	// ["Achtung! Achtung]
}
```

### func TrimFunc

```go
func TrimFunc(s []byte, f func(r rune) bool) []byte
```

TrimFunc returns a subslice of s by slicing off all leading and trailing UTF-8-encoded code points c that satisfy f(c).

###### Example

```go
package main

import (
	"bytes"
	"fmt"
	"unicode"
)

func main() {
	fmt.Println(string(bytes.TrimFunc([]byte("go-gopher!"), unicode.IsLetter)))			// -gopher!
	fmt.Println(string(bytes.TrimFunc([]byte("\"go-gopher!\""), unicode.IsLetter)))		// "go-gopher!"
	fmt.Println(string(bytes.TrimFunc([]byte("go-gopher!"), unicode.IsPunct)))			// go-gopher
	fmt.Println(string(bytes.TrimFunc([]byte("1234go-gopher!567"), unicode.IsNumber)))	// go-gopher!
}
```

### func TrimLeft

```go
func TrimLeft(s []byte, cutset string) []byte
```

TrimLeft returns a subslice of s by slicing off all leading UTF-8-encoded code points contained in cutset.

###### Example

```go
package main

import (
	"bytes"
	"fmt"
)

func main() {
	fmt.Print(string(bytes.TrimLeft([]byte("453gopher8257"), "0123456789")))	// gopher8257
}
```

### func TrimLeftFunc

```go
func TrimLeftFunc(s []byte, f func(r rune) bool) []byte
```

TrimLeftFunc treats s as UTF-8-encoded bytes and returns a subslice of s by slicing off all leading UTF-8-encoded code points c that satisfy f(c).

###### Example

```go
package main

import (
	"bytes"
	"fmt"
	"unicode"
)

func main() {
	fmt.Println(string(bytes.TrimLeftFunc([]byte("go-gopher"), unicode.IsLetter)))			// -gopher
	fmt.Println(string(bytes.TrimLeftFunc([]byte("go-gopher!"), unicode.IsPunct)))			// go-gopher!
	fmt.Println(string(bytes.TrimLeftFunc([]byte("1234go-gopher!567"), unicode.IsNumber)))	// go-gopher!567
}
```

### func TrimPrefix

```go
func TrimPrefix(s, prefix []byte) []byte
```

TrimPrefix returns s without the provided leading prefix string. If s doesn't start with prefix, s is returned unchanged.

###### Example

```go
package main

import (
	"bytes"
	"fmt"
)

func main() {
	var b = []byte("Goodbye,, world!")
	b = bytes.TrimPrefix(b, []byte("Goodbye,"))
	b = bytes.TrimPrefix(b, []byte("See ya,"))
	fmt.Printf("Hello%s", b)	// Hello, world!
}
```

### func TrimRight

```go
func TrimRight(s []byte, cutset string) []byte
```

TrimRight returns a subslice of s by slicing off all trailing UTF-8-encoded code points that are contained in cutset.

###### Example

```go
package main

import (
	"bytes"
	"fmt"
)

func main() {
	fmt.Print(string(bytes.TrimRight([]byte("453gopher8257"), "0123456789")))	// 453gopher
}
```

### func TrimRightFunc

```go
func TrimRightFunc(s []byte, f func(r rune) bool) []byte
```

TrimRightFunc returns a subslice of s by slicing off all trailing UTF-8-encoded code points c that satisfy f(c).

###### Example

```go
package main

import (
	"bytes"
	"fmt"
	"unicode"
)

func main() {
	fmt.Println(string(bytes.TrimRightFunc([]byte("go-gopher"), unicode.IsLetter)))			//	go-
	fmt.Println(string(bytes.TrimRightFunc([]byte("go-gopher!"), unicode.IsPunct)))			//	go-gopher
	fmt.Println(string(bytes.TrimRightFunc([]byte("1234go-gopher!567"), unicode.IsNumber)))	//	1234go-gopher!
}
```

### func TrimSpace

```go
func TrimSpace(s []byte) []byte
```

TrimSpace returns a subslice of s by slicing off all leading and trailing white space, as defined by Unicode.

###### Example

```go
package main

import (
	"bytes"
	"fmt"
)

func main() {
	fmt.Printf("%s", bytes.TrimSpace([]byte(" \t\n a lone gopher \n\t\r\n")))	//	a lone gopher
}
```

### func TrimSuffix

```go
func TrimSuffix(s, suffix []byte) []byte
```

TrimSuffix returns s without the provided trailing suffix string. If s doesn't end with suffix, s is returned unchanged.

###### Example

```go
package main

import (
	"bytes"
	"os"
)

func main() {
	var b = []byte("Hello, goodbye, etc!")
	b = bytes.TrimSuffix(b, []byte("goodbye, etc!"))
	b = bytes.TrimSuffix(b, []byte("gopher"))
	b = append(b, bytes.TrimSuffix([]byte("world!"), []byte("x!"))...)
	os.Stdout.Write(b)

	// Hello, world!
}
```

## Type

### type Buffer

```go
type Buffer struct {
	// contains filtered or unexported fields
}
```

A Buffer is a variable-sized buffer of bytes with Buffer.Read and Buffer.Write methods. The zero value for Buffer is an empty buffer ready to use.

###### Example

```go
package main

import (
	"bytes"
	"fmt"
	"os"
)

func main() {
	var b bytes.Buffer	// A Buffer needs no initialization
	b.Write([]byte("Hello "))
	fmt.Fprintf(&b, "world!")
	b.WriteTo(os.Stdout)
}

// Hello world!
```

###### Example(Reader)

```go
package main

import (
	"bytes"
	"encoding/base64"
	"io"
	"os"
)

func main() {
	// A Buffer can turn a string or a []byte into an io.Reader.
	buf := bytes.NewBufferString("R29waGVycyBydWxlIQ==")
	dec := base64.NewDecoder(base64.StdEncoding, buf)
	io.Copy(os.Stdout, dec)
}

Gophers rule!
```

### func NewBuffer

```go
func NewBuffer(buf []byte) *Buffer
```

NewBuffer creates and initializes a new Buffer using buf as its initial contents. The new Buffer takes ownership of buf, and the caller should not use buf after this call. NewBuffer is intended to prepare a Buffer to read existing data. It can also be used to set the intial size of the internal buffer for writing. To do that, buf should have the desired capacity but a length of zero.

In most cases, new(Buffer) (or just declaring a Buffer variable) is sufficient to intialize a Buffer.

### func NewBufferString

```go
func NewBufferString(s string) *Buffer
```

NewBufferString creates and initializes a new Buffer using string s as its initial contents. It is intended to prepare a buffer to read an existing string.

In most cases, new(Buffer) (or just declaring a Buffer variable) is sufficient to initialize a Buffer.

### func (\*Buffer) Available

```go
func (b *Buffer) Available() int
```

Available returns how many bytes are unused in the buffer.

### func (\*Buffer) AvailableBuffer() []byte

```go
func (b *Buffer) AvailableBuffer() []byte
```

AvailableBuffer returns an empty buffer with b.Available() capacity. This buffer is intended to be appended to and passed to an immediately succeeding Buffer.Write call. The buffer is only valid until the next write operation on b.

###### Example

```go
package main

import (
	"bytes"
	"os"
	"strconv"
)

func main() {
	var buf bytes.Buffer
	for i := 0; i < 4; i++ {
		b := buf.AvailableBuffer()
		b = strconv.AppendInt(b, int64(i), 10)
		b = append(b, ' ')
		buf.Write(b)
	}
	os.Stdout.Write(buf.Bytes())
}
// 0 1 2 3
```

### func (\*Buffer) Bytes

```go
func (b *Buffer) Bytes() []byte
```

Bytes returns a slice of length b.Len() holding the unread portion of the buffer. The slice is valid for use only until the next buffer modification (that is, only until the next call to a method like Buffer.Read, Buffer.Write, Buffer.Reset, or Buffer.Truncate). The slice aliases the buffer content at least until the next buffer modification, so immediate changes to the slice will affect the result of future reads.

###### Example

```go
package main

import (
	"bytes"
	"os"
)

func main() {
	buf := bytes.Buffer{}
	buf.Write([]byte{'h', 'e', 'l', 'l', 'o', ' ', 'w', 'o', 'r', 'l', 'd'})
	os.Stdout.Write(buf.Bytes())
}

// hello world
```

### func (\*Buffer) Cap

```go
func (b *Buffer) Cap() int
```

Cap returns the capacity of the buffer's underlying byte slice, that is, the total space allocated for the buffer's data.

###### Example

```go
package main

import (
	"bytes"
	"fmt"
)

func main() {
	buf1 := bytes.NewBuffer(make([]byte, 10))
	buf2 := bytes.NewBuffer(make([]byte, 0, 10))
	fmt.Println(buf1.Cap())	// 10
	fmt.Println(buf2.Cap())	// 10
}
```

### func (\*Buffer) Grow

```go
func (b *Buffer) Grow(n int)
```

Grow grows the buffer's capacity, if necessary, to guarantee space for another n bytes. After Grow(n), at least n bytes can be written to the buffer without another allocation. If n is negative, Grow will panic. If the buffer can't grow it will panic with ErrTooLarge.

###### Example

```go
package main

import (
	"bytes"
	"fmt"
)

func main() {
	var b bytes.Buffer
	b.Grow(64)
	bb := b.Bytes()
	b.Write([]byte("64 bytes or fewer"))
	fmt.Printf("%q", bb[:b.Len()])	//"61 bytes or fewer"
}
```

### func (\*Buffer) Len

```go
func (b *Buffer) Len() int
```

Len returns the number of bytes of the unread portion of the buffer; b.Len() == len(b.Bytes()).

###### Example

```go
package main

import (
	"bytes"
	"fmt"
)

func main() {
	var b bytes.Buffer
	b.Grow(64)
	b.Write([]byte("abcde"))
	fmt.Printf("%d", b.Len())	// 5
}
```
