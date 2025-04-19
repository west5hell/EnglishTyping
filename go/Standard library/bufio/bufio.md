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

### func ScanBytes

```go
func ScanBytes(data []byte, atEOF bool) (advance int, token []byte, err error)
```

ScanBytes is a split function for a Scanner that returns each byte as a token.

### func ScanLines

```go
func ScanLines(data []byte, atEOF bool) (advance int, token []byte, err error)
```

ScanLines is a split function for a Scanner that returns each line of text, stripped of any trailing end-of-line marker. The returned line may be empty. The end-of-line marker is one optional carriage return followed by one mandatory newline. In regular expression notation, it is `\r?\n`. The last non-empty line of input will be returned even if it has no newline.

### func ScanRunes

```go
func ScanRunes(data []byte, atEOF bool) (advance int, token []byte, err error)
```

ScanRunes is split function for a Scanner that returns each UTF-8-encoded rune as a token. the sequence of runes returned is equivalent to that from a range loop over the input as a string, which means that erroneous UTF-8 encodings translate to U+FFFD = "\xef\xbf\xbd". Because of the Scan interface, this makes it impossible for the client to distinguish correctly encoded replacement runes from encoding errors.

### func ScanWords

```go
func ScanWords(data []byte, atEOF bool) (advance int, token []byte, err error)
```

ScanWords is a split function for a Scanner that returns each space-separated word of text, with surrounding spaces deleted. It will never return an empty string. The definition of space is set by unicode.IsSpace.

## Types

### type ReadWriter

```go
type ReadWriter struct {
    *Reader
    *Writer
}
```

ReadWriter stores pointers to a Reader and a Writer. It implements io.ReadWriter.

### func NewReadWriter

```go
func NewReadWriter(r *Reader, w *Writer) *ReadWriter
```

NewReadWriter allocates a new ReadWriter that dispatches to r and w.

### type Reader

```go
type Reader struct {
    // contains filtered or unexported fields.
}
```

Reader implements buffering for an io.Reader object. A new Reader is created by calling NewReader or NewReaderSize; alternatively the zero value of a Reader may be used after calling [Reset] on it.

### func NewReader

```go
func NewReader(rd  io.Reader) *Reader
```

NewReader returns a new Reader whose buffer has the default size.

### func NewReaderSize

```go
func NewReaderSize(rd io.Reader, size int) *Reader
```

NewReaderSize returns a new Reader whose buffer has at least the specified size. If the argument io.Reader is already a Reader with large enough size, it returns the underlying Reader.

### func (\*Reader) Buffered

```go
func (b *Reader) Buffered() int
```

Buffered returns the number of bytes that can be read from the current buffer.

### func (\*Reader) Discard

```go
func (b *Reader) Discard(n int) (discard int, err error)
```

Discard skips the next n bytesm returning the number of bytes discarded.

If Discard skips fewer than n bytes, it also returns an error. If 0 <= n <= b.Buffered(), Discard is guaranteed to succeed without reading from the underlying io.Reader.

### func (\*Reader) Peek

```go
func (b *Reader) Peek(n int) ([]bytes, error)
```

Peek returns the next n bytes without advancing the reader. The bytes stop being calid at the next read call. If necessary, Peek will read more bytes into the buffer in order to make n bytes available. If Peek returns fewer than n bytes, it also returns an error explaining why the read is short. The error is ErrBufferFull if n is larger than b's buffer size.

Calling Peek prevents a Reader.UnreadByte or Reader.UnreadRune call from succeeding until the next read operation.

### func (\*Reader) Read

```go
func (b *Reader) Read(p []byte) (n int, err error)
```

Read reads data into p. It returns the number of bytes read into p. The bytes are taken from at most one Read on the underlying Reader, hence n may be less than len(p). To read exactly len(p) bytes, use io.ReadFull(b, p). If the underlying Reader can return a non-zero count with io.EOF, then this Read method can do so as well; see the io.Reader docs.

### func (\*Reader) ReadByte

```go
func (b *Reader) ReadByte() (byte, error)
```

ReadByte reads and returns a single byte. If no byte is available, returns an error.

### func (\*Reader) ReadBytes

```go
func (b *Reader) ReadBytes(delim byte) ([]byte, error)
```

ReadBytes reads until the first occurrence of delim in the input, returning a slice containing the data up to and including the delimiter. If ReadBytes encounters an error before finding a delimiter, it returns the data read before the error and the error itself (often io.EOF). ReadBytes returns err != nil if and only if the returned data does not end in delim. For simple uses, a Scanner may be more convenient.

### func (\*Reader) ReadLine

```go
func (b *Reader) ReadLine() (line []byte, isPrefix bool, err error)
```

ReadLine is a low-level line-reading primitive. Most callers should use Reader.ReadBytes('\n') or Reader.ReadString('\n') instead or use a Scanner.

ReadLine tries to return a single line, not including the end-of-line bytes. If the line was too long for the buffer then isPrefix is set and the beginning of the line is returned. The rest of the line will be returned from future calls. isPrefix will be false when returning the last fragment of the line. The returned buffer is only valid until the next call to ReadLine. ReadLine either returns a non-nil line or it returns an error, never both.

The next returned from ReadLine does not include the line end ("\r\n" or "\n"). No indication or error is given if the input ends without a final line end. Calling Reader.UnreadByte after ReadLine will always unread the last byte read (possibly a character belonging to the line end) even if that byte is not part of the line returned by ReadLine.

### func (\*Reader) ReadRune

```go
func (b *Reader) ReadRune() (r rune, size int, err error)
```

ReadRune reads a single UTF-8 encoded Unicode character and returns the rune and its size in bytes. If the encoded rune is invalid, it consumes one byte and returns unicode.ReplacementChar (U+FFFD) with a size of 1.

### func (\*Reader) ReadSlice

```go
func (b *Reader) ReadSlice(delim byte) (line []byte, err error)
```

ReadSlice reads until the first occurrence of delim in the input, returning a slice pointing at the bytes in the buffer. The bytes stop being valid at the next read. If ReadSlice encounters an error before finding a delimiter, it returns all the data in the buffer and the error itself (often io.EOF). ReadSlice fails with error ErrBufferFull if the buffer fills without a delim. Because the data returned from ReadSlice will be overwrittern by the next I/O operation, most clients should use Reader.ReadBytes or ReadString instead. ReadSlice returns err != nil if and only if line does not end in delim.

### func (\*Reader) ReadString

```go
func (b *Reader) ReadString(delim byte) (string, error)
```

ReadString reads until the first occurrence of delim in the input, returning a string containing the data up to and including the delimiter. If ReadString enveounters an error before finding a delimiter, it returns the data read before the error and the error itself (often io.EOF). ReadString returns err != nil if and only if the returned data does not end in delim. For simple uses, a Scanner may be more conveninent.

### func (\*Reader) Reset

```go
func (b *Reader) Reset(r io.Reader)
```

Reset discards any buffered data, resets all state, and switches the buffered reader to read from r. Calling Reset on the zero valur of Reader initializes the internal buffer to the default size. Calling b.Reset(b) (that is, resetting a Reader to itself) does nothing.

### func (\*Reader) Size

```go
func (b *Reader) Size() int
```

Size returns the size of the underlying buffer in bytes.

### func (\*Reader) UnreadByte

```go
func (b *Reader) UnreadByte() error
```

UnreadByte unreads the last byte. Only the most recently read byte can be unread.

UnreadByte returns an error if the most recent method called on the Reader was not a read operation. Notably, Reader.Peek, Reader.Discard, and Reader.WriteTo are not considered read operatins.

### func (\*Reader) UnreadRune

```go
func (b *Reader) UnreadRune() error
```

UnreadRune unreads the last rune. If the most recent method called on the Reader was not a Reader.ReadRune, Reader.UnreadRune returns an error. (In this regard it is sticter than Reader.UnreadByte, which will unread the last byte from any read operation.)

### func (\*Reader) WriteTo

```go
func (b *Reader) WriteTo(w io.Writer) (n int64, err error)
```

WriteTo implements io.WriterTo. This may make multiple calls to the Reader.Read method of the underlying Reader. If the underlying reader supports the Reader.WriteTo method, this calls the underlying Reader.WriteTo without buffering.

### type Scanner

```go
type Scanner struct {
    // Contains filtered or unexported fields
}
```

Scanner provides a convenient interface for reading data such as a file of newline-delimited lines of text. Successive calss to the Scanner.Scan method will step through the 'tokens' of a file, skipping the bytes between the tokens. The specification of a token is defined by a split function of type SplitFunc; the default split function breaks the input into lines with line termination stripped. Scanner.Split functions are defined in this package for scanning a file into lines, bytes, UTF-8-encoded runes, and space-delimited words. The client may instead provide a custom split function.

Scanning stops unrecoverably at EOF, the first I/O error, or a token too large to fit in the Scanner.Buffer. When a scan stops, the reader may have advanced arbitrarily far past the last token. Programs that need more control over error handling or large tokens, or must run sequential scans on a reader, should use bufio.Reader instead.

###### Example(Custom)

Use a Scanner with a custom split function (built by wrapping ScanWords) to validate 32-bit decimal input.

```go
package main

import (
	"bufio"
	"fmt"
	"strconv"
	"strings"
)

func main() {
	// An artificial input source.
	const input = "1234 5678 1234567901234567890"
	scanner := bufio.NewScanner(strings.NewReader(input))
	// Create a custom split function by wrapping the existing ScanWords function.
	split := func(data []byte, atEOF bool) (advance int, token []byte, err error) {
		advance, token, err = bufio.ScanWords(data, atEOF)
		if err == nil && token != nil {
			_, err = strconv.ParseInt(string(token), 10, 32)
		}
		return
	}
	// Set the split function for the scanning operation.
	scanner.Split(split)
	// Validate the input
	for scanner.Scan() {
		fmt.Printf("%s\n", scanner.Text())
	}

	if err := scanner.Err(); err != nil {
		fmt.Printf("Invalid input: %s", err)
	}
}
```

###### Example(EarlyStop)

Use a Scanner with a custom split function to parse a comma-separated list with an empty final value but stops at the token "STOP".

```go
package main

import (
	"bufio"
	"bytes"
	"fmt"
	"os"
	"strings"
)

func main() {
	onComma := func(data []byte, atEOF bool) (advance int, token []byte, err error) {
		i := bytes.IndexByte(data, ',')
		if i == -1 {
			if !atEOF {
				return 0, nil, nil
			}
			// If we have reached the end, return the last token.
			return 0, data, bufio.ErrFinalToken
		}
		// If the token is "STOP", stop the scanning and ignore the rest.
		if string(data[:i]) == "STOP" {
			return i + 1, nil, bufio.ErrFinalToken
		}
		// Otherwise, return the token before the comma.
		return i + 1, data[:i], nil
	}
	const input = "1,2,STOP,4,"
	scanner := bufio.NewScanner(strings.NewReader(input))
	scanner.Split(onComma)
	for scanner.Scan() {
		fmt.Printf("Got a token %q\n", scanner.Text())
	}
	if err := scanner.Err(); err != nil {
		fmt.Fprintln(os.Stderr, "reading input:", err)
	}
}
```

###### Example (EmptyFinalToken)

Use a Scanner with a custom split function to parse a comma-separated list with an empty final value.

```go
package main

import (
	"bufio"
	"fmt"
	"os"
	"strings"
)

func main() {
	// Comma-separated list; last entry is empty.
	const input = "1,2,3,4,"
	scanner := bufio.NewScanner(strings.NewReader(input))
	// Define a split function that separates on commas.
	onComma := func(data []byte, atEOF bool) (advance int, token []byte, err error) {
		for i := 0; i < len(data); i++ {
			if data[i] == ',' {
				return i + 1, data[:i], nil
			}
		}
		if !atEOF {
			return 0, nil, nil
		}
		// There is one final token to be delivered, which may be the empty string.
		// Returning bufio.ErrFinalToken here tells Scan there are no more tokens after this
		// but does not trigger an error to be returned from Scan itself.
		return 0, data, bufio.ErrFinalToken
	}
	scanner.Split(onComma)
	// Scan.
	for scanner.Scan() {
		fmt.Printf("%q ", scanner.Text())
	}
	if err := scanner.Err(); err != nil {
		fmt.Fprintln(os.Stderr, "reading input:", err)
	}
}
```

###### Example (Lines)

The simplest use of a Scanner, to read standard input as a set of lines.

```go
package main

import (
	"bufio"
	"fmt"
	"os"
)

func main() {
	scanner := bufio.NewScanner(os.Stdin)
	for scanner.Scan() {
		fmt.Println(scanner.Text()) // Println will add back the final '\n'
	}
	if err := scanner.Err(); err != nil {
		fmt.Fprintln(os.Stderr, "reading standard input:", err)
	}
}
```

###### Example (Words)

Use a Scanner to implement a simple word-count utility by scanning the input as a sequence of space-delimited tokens.

```go
package main

import (
	"bufio"
	"fmt"
	"os"
	"strings"
)

func main() {
	// An artificial input source.
	const input = "Now is the winter of our discontent,\nMade glorious summer by this sun of York.\n"
	scanner := bufio.NewScanner(strings.NewReader(input))
	// Set the split function for the scanning operation.
	scanner.Split(bufio.ScanWords)
	// Count the words.
	count := 0
	for scanner.Scan() {
		count++
	}
	if err := scanner.Err(); err != nil {
		fmt.Fprintln(os.Stderr, "reading input:", err)
	}
	fmt.Printf("%d\n", count)
}
```

### func NewScanner

```go
func NewScanner(r io.Reader) *Scanner
```

NewScanner returns a new Scanner to read from r. The split function defaults to ScanLines.

### func (\*Scanner) Buffer

```go
func (s *Scanner) Buffer(buf []byte, max int)
```

Buffer sets the initial buffer to use when scanning and the maximum size of buffer that may be allocated during scanning. The maximum token size must be less than the larger of max and cap(buf). If max <= cap(buf), Scanner.Scan will use this buffer only and do no allocation.

By default, Scanner.Scan uses an internal buffer and sets the maximum token size to MaxScanTokenSize.

Buffer panics if it is called after scanning has started.

### func (\*Scanner) Bytes

```go
func (s *Scanner) Bytes() []byte
```

Bytes returns the most recent token generated by a call to Scanner.Scan. The underlying array may point to data that will be overwritten by a subsequent call to Scan. It does no allocation.

###### Example

Return the most recent call to Scan as a []byte.

```go
package main

import (
	"bufio"
	"fmt"
	"os"
	"strings"
)

func main() {
	scanner := bufio.NewScanner(strings.NewReader("gopher"))
	for scanner.Scan() {
		fmt.Println(len(scanner.Bytes()) == 6)
	}
	if err := scanner.Err(); err != nil {
		fmt.Fprintln(os.Stderr, "shouldn't see an error scanning a string")
	}
}
```

### func (\*Scanner) Err

```go
func (s *Scanner) Err() error
```

Err returns the first non-EOF error that was encountered by the Scanner.

### func (\*Scanner) Scan

```go
func (s *Scanner) Scan() bool
```

Scan advances the Scanner to the next token, which will then be available through the Scanner.Bytes or Scanner.Text method. It returns false when there are no more tokens, either by reaching the end of the input or an error. After Scan returns false, the Scanner.Err method will return any error that occurred during scanning, except that if it was io.EOF, Scanner.Err will return nil. Scan panics if the split function returns too many empty tokens without advancing the input. This is a common error mode for scanners.

### func (\*Scanner) Split

```go
func (s *Scanner) Split(split SplitFunc)
```

Split sets the split function for the Scanner. The default split function is ScanLines.

Split panics if it is called after scanning has started.

### func (\*Scanner) Text

```go
func (s *Scanner) Text() string
```

Text returns the most recent token generated by a call to Scanner.Scan as a newly allocated string holding its bytes.
