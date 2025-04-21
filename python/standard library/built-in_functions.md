# Built-in Functions

The Python interpreter has a number of functions and types built into it that are always available.

### abs(_x_)

Return the absolute value of a number. The argument may be an integer, a floating-point number, or an object implementing `__abs__()`. If the argument is a complex number, its magnitude is returned.

### aiter(_async_iterable_)

Return an asynchronous iterator for an asynchronous iterable. Equivalent to calling `x.__aiter__()`.

Note: Unlike `iter()`, `aiter()` has no 2-argument variant.

> Added in version 3.10.

### all(_iterable_)

Return `True` if all elements of the _iterable_ are true (or if the iterable is empty). Equivalent to:

```python
def all(iterable):
    for element in iterable:
        if not element:
            return False
    return True
```

### _awaitable_ anext(_async_iterator_)

### _awaitable_ anext(_async_iterator_, _default_)

When awaited, return the next item from the given asynchronous iterator, or _default_ if given and the iterator is exhausted.

This is the async variant fo the `next()` builtin, and behaves similarly.

This calls the `__anext__()` method of _async_iterator_, returning an awaitable. Awaiting this returns the next value of the iterator. If _default_ is given, it is returned if the iterator is exhausted, otherwise `StopAsyncIteration` is raised.

> Added in version 3.10.

### any(_iterable_)

Return `True` if any element of the _iterable_ is true. If the iterable is empty, return `False`. Equivalent to:

```python
def any(iterable):
    for element in iterable:
        if element:
            return True
    return False
```

### ascii(_object_)

As `repr()`, return a string containing a printable representation fo an object, but escape the non-ASCII characters in the string returned by `repr()` using `\x`, `\u`, or `\U` escape. This generates a string similar to that returned by `repr()` in Python 2.

### bin(_x_)

Convert an integer number to a binary string prefixed with "0b". The result is a valid Python expression. If _x_ is not a Python `int` object, it has to define an `__index()__` method that returns an integer. Some examples:

```shell
>>> bin(3)
'0b11'
>>> bin(-10)
'-0b1010'
```

If the prefix "0b" is desired or not, you can use either of the following ways.

```shell
>>> format(14, '#b'), format(14, 'b')
('0b1110', '1110')
>>> f'{14:#b}', f'{14:b}'
('0b1110', '1110')
```

See also `format()` for more information.

### class bool(_object=False_, _/_)

Return a Boolean value, i.e. one of `True` or `False`. The argument is converted using the standard truth testing procedure. If the argument is false or omitted, this returns `False`; otherwise, it returns `True`. The bool class is a subclass of int (see Numeric Types - int, float, complex). It cannot be subclassed further. Its only instances are `False` and `True` (see Boolean Type - bool).

> Changed in version 3.7: The parameter is now positional-only.

### breakpoint(_\*args, \*\*kws_)

This function drops you into the debugger at the call site. Specifically, it calls `sys.breakpointhook()`, passing `args` and `kws` straight through. By default, `sys.breakpointhook()` calls `pdb.set_trace()` expecting no arguments. In this case, it is purely a convenience function so you don't have to explicitly import `pdb` or type as much code to enter the debugger. However, `sys.breakpointhook()` can be set to some other function and `breakpoint()` will automatically call that, allowing you to drop into the debugger of choice. If `sys.breakpointhook()` is not accessible, this function will raise `RuntimeError`.

By default, the behavior of `breakpoint()` can be changed with the `PYTHONBREAKPOINT` environment variable. See `sys.breakpointhook()` for usage details.

Note that this is not guaranteed if `sys.breakpointhook()` has been replaced.

Raises an auditing event `builtins.breakpoint` with argument `breakpointhook`.

> Added in version 3.7.

### class bytearray(_source=b''_)

### class bytearray(_source, encoding_)

### class bytearray(_source, encoding, errors_)

Return a new array of bytes. The bytearray class is a mutable sequence of integers in the range 0 <= x < 256. It has most of the usual methods of mutable sequences, described in Mutable Sequence Types, as well as most methods that that bytes type has, see Bytes and Bytearray Operations.

The optional _source_ parameter can be used to initialize the array in a few different ways:

- If it is a _string_, you must also give the _encoding_ (and optionally, _errors_) parameters; `bytearray()` then converts the string to bytes using `str.encode()`.
- If it is an _integer_, the array will have that size and will be initialized with null bytes.
- If it is an object conforming to the buffer interface, a read-only buffer to the object will be used to initilize the bytes array.
- If it is an _iterable_, it must be an iterable of integers in the range `0 <= x < 256`, which are used as the initial contents of the array.

Without an argument, an array of size 0 is created.

See also [Binary Sequence Type -- bytes, bytearray, memoryview](https://docs.python.org/3/library/stdtypes.html#binaryseq) and [Betearray Objects](https://docs.python.org/3/library/stdtypes.html#typebytearray).

### class bytes(_source=b''_)

### class bytes(_source, encoding_)

### class bytes(_source, encoding, errors_)

Return a new "bytes" object which is an immutable sequence of integers in the range `0 <= x < 256`. `bytes` is an immutable version of `bytearray` - it has the same non-mutating methods and the same indexing and slicing behavior.

Accordingly, constructor arguments are interpreted as for `bytearray()`.

Bytes objects can also be created with literals, see [String and Bytes literals](https://docs.python.org/3/reference/lexical_analysis.html#strings).

See also [Binary Sequence Types -- bytes, bytearray, memoryview](https://docs.python.org/3/library/stdtypes.html#binaryseq), [Bytes Objects](https://docs.python.org/3/library/stdtypes.html#typebytes), and [Bytes and Bytearray Operations](https://docs.python.org/3/library/stdtypes.html#bytes-methods).

### callable(_object_)

Return `True` if the _object_ argument appears callable, `False` if not. If this returns `True`, it is still possible that a call fails, but if it is `False`, calling _object_ will never succeed. Note that classes are callable (calling a class returns a new instance); instances are callable if their class has a `__call__()` method.

> Added in version 3.2: This function was first removed in Python 3.0 and then brought back in Python 3.2

### chr(_i_)

Return the string representing a character whose Unicode code point is the integer _i_. For example, `chr(97)` returns the string `'a'`, while `chr(8364)` returns the string `€`. This is the inverse of `ord()`.

The valid range for the argument is from 0 through 1,114,111 (0x10FFFF in base 16). `ValueError` will be raised if _i_ is outside that range.

### @classmethod

Transform a method into a class method.

A class method receives the class as an implicit first argument, just like an instance method receives the instance. To declare a class method, use this idiom:

```python
class C:
    @classmethod
    def f(cls, arg1, arg2): ...
```

The `@classmethod` form is a functin [decorator](https://docs.python.org/3/glossary.html#term-decorator) - see [Function definitions](https://docs.python.org/3/reference/compound_stmts.html#function) for details.

A class method can be called either on the class (such as `C.f()`) or on an instance (such as `C().f()`). The instance is ignored except for its class. If a class method is called for a derived class, the derived class object is passed as the implied first argument.

Class methods are different than C++ or Java static methods. If you want those, see `staticmethod()` in this section. For more information on class methods, see [The standard type hierarchy](https://docs.python.org/3/reference/datamodel.html#types).

> Changed in version 3.9: Class methods can now wrap other descriptors such as `property()`.
> Changed in version 3.10: Class methods now inherit the method attributes (`__module__`, `__name__`, `__qualname__`, `__doc__` and `__annotations__`) and have a new `__wrapped__` attribute.
> Deprecated since version 3.11, removed in version 3.13: Class methods can no longer wrap other descriptors such as `property()`.

### compile(_source, filename, mode, flags=0, dont_inherit=False, optimize=-1_)

Compile the _source_ into a code or AST object. Code objects can be executed by `exec()` or `eval()`. _source_ can either be a normal string, a byte string, or an AST object. Refer to the ast module documentation for information on how to work with AST object.

The _filename_ argument should give the file from which the code was read; pass some recognizable value if it wasn't read from a file (`'<string>'` is commonly used).

The _mode_ argument specifies what kind of code must be compiled; it can be `exec` if _source_ consists of a sequence of statements, `eval` if it consists of a single expression, or `single` if it consists of a single interactive statement (in the latter case, expression statements that evaluate to something other than `None` will be printed).

The optional arguments _flags_ and _dont_inherit_ control which compiler options should be activated and which future features should be allowed. If neither if present (or both are zero) the code is compiled with the same flags that affect the code that is calling `compile()`. If the _flags_ argument is given and _dont_inherit_ is not (or is zero) then the compiler options and the future statements specified by the _flags_ argument are used in addition to those that would be used anyway. If _dont_inherit_ is a non-zero integer then the _flags_ argument is it - the flags (future features and compiler options) in the surrounding code are ignored.

Compiler options and future statemens are specified by bits which can be bitwise ORed together to specify multiple options. The bitfield required to specify a given future feature can be found as the compiler_flag attribute on the `_Feature` instance in the `__future__` module. Compiler flags can be found in ast module, with `PyCF*` prefix.

The argument _optimize_ specifies the optimization level of the compiler; the default value of `-1` selects the optimization level of the interpreter as given by `-0` options. Explicit levels are `0` (no optimization; `__debug__` is true), `1` (asserts are removed, `__debug__` is false) or `2` (docstrings are removed too).

The fucntion raise `SyntaxError` if the compiled source is invalid, and `ValueError` if the source contains null bytes.

If you want to parse Python code into its AST representation, see [ast.parse()](https://docs.python.org/3/library/ast.html#ast.parse).

Raises an [auditing event](https://docs.python.org/3/library/sys.html#auditing) `compile` with arguments `source` and `filename`. This event may also be raised by implicit compilation.

> Note: When compiling a string with multi-line code in `single` or `eval` mode, input must be terminated by at least one newline character. This is to faciliate detection of incomplete and complete statements in the `code` module.

> Warning: It is possible to crash the Python interpreter with a sufficiently large/complex string when compiling to an AST object due to stack depth limitations in Python's AST compiler.

> Changed in version 3.2: Allowed use of Windows and Mac newlines. Also, input in `exec` mode does not have to end in a newline anymore. Added the _optimize_ parameter.

> Changed in version 3.5: Previously, `TypeError` was raised when null bytes were encountered in _source_.

> Added in version 3.8: `ast.PyCF_ALLOW_TOP_LEVEL_AWAIT` can now be passed in flags to enable support for top-level `await`, `async for`, and `async with`.
