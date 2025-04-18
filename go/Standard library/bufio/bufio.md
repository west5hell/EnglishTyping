# Overview

Package bufio implements buffered I/O. It wraps an io.Reader or io.Writer object, creating another object (Reader or Writer) that also implements the interface but provides buffering and some help for textual I/O.

## Index

## Examples

## Constants

```go
const (
    // MaxScanTokenSize is the maximum size used to buffer a token
    // unless the user provides an explicit buffer with [Scanner.Buffer].
    // The actual maximum token size may be smaller as the buffer
    // may need to include, for instance, a newline.
    MaxScanTokenSize = 64 * 1024
)
```

## Variables

```go
var (
    ErrInvalidUnreadByte    = errors.New("bufio: invalid use of UnreadByte")
    ErrInvalidUnreadRune    = errors.New("bufio: invalid use of UnreadRune")
    ErrBufferFull           = errors.New("bufio: buffer full")
    ErrNegativeCount        = errors.New("bufio: negative count")
)
```

```go
var (
    ErrTooLong          = errors.New("bufio.Scanner: token too long")
    ErrNegativeAdvance  = errors.New("bufio.Scanner: SplitFunc returns negative advance count")
    ErrAdvanceTooFar    = errors.New("bufio.Scanner: SplitFunc returns advance count beyond input")
    ErrBadReadCount     = errors.New("bufio.Scanner: Read returned impossible count")
)
```

Errors returned by Scanner.

```go
var ErrFinalToken = errors.New("final token")
```

ErrFinalToken is a special sentinel error value. It is intended to be returned by a Split function to indicate that the scanning should stop with no error. If the token being delivered with this error is not nil, the token is the last token.

The value is useful to stop processing early or when it is necessary to deliver a final empty token (which is different from a nil token). One could achieve the same behavior with a custom error value but providing one here is tidier. See the emptyFinalToken example for a use of this value.

## Functions

**func ScanBytes**

```go
func ScanBytes(data []byte, atEOF bool) (advance int, token []byte, err error)
```

ScanBytes is a split function for a Scanner that returns each byte as a token.

**func ScanLines**

```go
func ScanLines(data []byte, atEOF bool) (advance int, token []byte, err error)
```

ScanLines is a split function for a Scanner that returns each line of text, stripped of any trailing end-of-line marker. The returned line may be empty. The end-of-line marker is one optional carriage return followed by one mandatory newline. In regular expression notation, it is `\r?\n`. The last non-empty line of input will be returned even if it has no newline.

**func ScanRunes**

```go
func ScanRunes(data []byte, atEOF bool) (advance int, token []byte, err error)
```

ScanRunes is split function for a Scanner that returns each UTF-8-encoded rune as a token. the sequence of runes returned is equivalent to that from a range loop over the input as a string, which means that erroneous UTF-8 encodings translate to U+FFFD = "\xef\xbf\xbd". Because of the Scan interface, this makes it impossible for the client to distinguish correctly encoded replacement runes from encoding errors.

**func ScanWords**

```go
func ScanWords(data []byte, atEOF bool) (advance int, token []byte, err error)
```

ScanWords is a split function for a Scanner that returns each space-separated word of text, with surrounding spaces deleted. It will never return an empty string. The definition of space is set by unicode.IsSpace.

## Types

**type ReadWriter**

```go
type ReadWriter struct {
    *Reader
    *Writer
}
```

ReadWriter stores pointers to a Reader and a Writer. It implements io.ReadWriter.

**func NewReadWriter**

```go
func NewReadWriter(r *Reader, w *Writer) *ReadWriter
```

NewReadWriter allocates a new ReadWriter that dispatches to r and w.

**type Reader**

```go
type Reader struct {
    // contains filtered or unexported fields.
}
```

Reader implements buffering for an io.Reader object. A new Reader is created by calling NewReader or NewReaderSize; alternatively the zero value of a Reader may be used after calling [Reset] on it.

**func NewReader**

```go
func NewReader(rd  io.Reader) *Reader
```

NewReader returns a new Reader whose buffer has the default size.

**func NewReaderSize**

```go
func NewReaderSize(rd io.Reader, size int) *Reader
```

NewReaderSize returns a new Reader whose buffer has at least the specified size. If the argument io.Reader is already a Reader with large enough size, it returns the underlying Reader.

**func (\*Reader) Buffered**

```go
func (b *Reader) Buffered() int
```

Buffered returns the number of bytes that can be read from the current buffer.

**func (\*Reader) Discard**

```go
func (b *Reader) Discard(n int) (discard int, err error)
```

Discard skips the next n bytesm returning the number of bytes discarded.

If Discard skips fewer than n bytes, it also returns an error. If 0 <= n <= b.Buffered(), Discard is guaranteed to succeed without reading from the underlying io.Reader.

**func (\*Reader) Peek**

```go
func (b *Reader) Peek(n int) ([]bytes, error)
```

Peek returns the next n bytes without advancing the reader. The bytes stop being calid at the next read call. If necessary, Peek will read more bytes into the buffer in order to make n bytes available. If Peek returns fewer than n bytes, it also returns an error explaining why the read is short. The error is ErrBufferFull if n is larger than b's buffer size.

Calling Peek prevents a Reader.UnreadByte or Reader.UnreadRune call from succeeding until the next read operation.
