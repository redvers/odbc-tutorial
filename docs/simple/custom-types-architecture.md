# SQL Type Architecture

Each SQL Type implementation is a type-aware wrapper around a C buffer.  This buffer is passed to the ODBC driver via `bind_parameter` (for inputs), or `bind_column` (for results).  At a high level, we are responsible for:

- Sizing / Allocating / Deallocating the C-buffer.
- Marshalling / Unmarshalling from the C-buffer to the native pony types.

## The SQLType trait

Almost all of the common-code is already implemented for you in SQLType.  You should only need to implement the class fields, accessors, and the read()? / write functions.

### Example: SQLInteger

Let's look at SQLInteger line-by-line.

```pony
class SQLInteger is SQLType
  """
  The internal class which represents an Integer (I32)
  """
  var _v: CBoxedArray = CBoxedArray
  var _err: SQLReturn val = SQLSuccess
```

When you create your class, you must mark your new SQL Type as a `SQLType`.  This is the type that all the associated functions (such as: `bind_parameter(SQLType): Bool ?` will expect.

Inside you need a (minimum) of two fields, one for the C Buffer (CBoxedArray), and one for SQLReturn.

Next, we need a constructor - and as we know that a textual representation of a SQL Integer (I32) will never exceed 15 characters, we allocate 15 bytes:

```pony
  new create() => _v.alloc(15)
```

Next we need to provide accessors for both of these field variables, so that they can be manipulated by the API:

```pony
  fun \nodoc\ ref get_boxed_array(): CBoxedArray => _v
  fun \nodoc\ ref set_boxed_array(v': CBoxedArray) => _v = v'

  fun \nodoc\ ref get_err(): SQLReturn val => _err
  fun \nodoc\ ref set_err(err': SQLReturn val) => _err = err'
```

Lastly, we implement marshallers and demarshallers:

```pony
  fun ref write(i32: I32): Bool =>
    """
    Write an I32 to this buffer as a string. The string will fit
    as we defined the buffer to be characters on initialization.

    Will return true if written and verification succeeds.

    Will return false if the string is too long for the buffer or
    the readback doesn't match for some other reason.
    """
    _write(i32.string())

  fun ref read(): I32 ? =>
    """
    Read the value of the buffer and convert it into an I32.

    This function is partial just in case the string version
    that comes from the database is not within the I32 range.

    NOTE: SQLite does not enforce type limits, so if you use
    that specific ODBC driver, this is something that must be
    verified.
    """
    if (get_boxed_array().is_null()) then
      error
    else
      _v.string().i32()?
    end
```

## Useful trait functions

There are a few functions in `SQLType` that are automatically available to either assist in debugging, or provide needed functionality.  You do not need to do anything to use these:

| Function Signature      | Purpose                                                        |
|-------------------------|----------------------------------------------------------------|
| reset(): Bool           | Resets and zeros the buffer to its initial state.              |
| string(): String iso^   | Returns the data as delivered by the database as a String.     |
| array(): Array[U8] iso^ | Returns the data as delieverd by the database as an Array[U8]. |
| is\_null(): Bool        | Returns if the buffer was populated by a SQL Null              |
| null()                  | Sets the buffer to represent a SQL Null                        |

## Buffer Behaviour

There is a difference in the way that buffers behave, depending on if they are used as a parameter or as a part of a result set:

### Parameter

You are responsible for ensuring that the buffer is of a sufficient size to hold the value you choose to write to it, and that the value is a valid String.  If you do not, the trait's `_write()` will return false.

THE BUFFER WILL NOT RESIZE IF YOU TRY AND STUFF SOMETHING TOO LARGE INTO IT.  IT CAN'T, BECAUSE THE BUFFER IS ALREADY BOUND TO YOUR PREPARE STATEMENT.

### Column (part of a result set)

Even though you are responsible for ensuring that the buffer is of a sufficient size to hold the resultant value, the API *will* resize, rebind, and re-read the data for you.  The newly created larger buffer replaces the previous one in the API, but as long as you don't poke into the `CBoxedArray` in the implementation, it should be seamless to you.
