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
