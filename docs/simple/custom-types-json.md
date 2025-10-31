# Implementing SQLJson

Here is a simple example for how to implement a custom SQL Type for SQLJson

## Dependencies

Firstly, we need to ensure that `ponylang/json` is included in our corral.json, like so:

```quote
red@panic:~/project/psql-demo$ corral add github.com/ponylang/json.git --version 0.2.0
red@panic:~/project/psql-demo$ corral fetch
```

## Starting our class

We need to include the `ponylang/json` dependency, ensure that the class uses the `SQLType` trait, and create the boilerplate code.

Unfortunately, since Json can be any arbitrary size, we will need our end-user to declare the size of the buffer they wish to use.

Here's what we start with:

```pony
use "pony-odbc"
use "json"

class SQLJson is SQLType
  """
  An example class that represents a PostgreSQL Json type
  """
  var _v: CBoxedArray = CBoxedArray
  var _err: SQLReturn val = SQLSuccess

  new create(size: USize) => _v.alloc(size)
    """
    Creates a SQLJson object.  You must specify the textual size at creation.
    """

  fun \nodoc\ ref get_boxed_array(): CBoxedArray => _v
  fun \nodoc\ ref set_boxed_array(v': CBoxedArray) => _v = v'

  fun \nodoc\ ref get_err(): SQLReturn val => _err
  fun \nodoc\ ref set_err(err': SQLReturn val) => _err = err'
```

Now we simply refer to the [ponylang/json documentation](https://ponylang.github.io/json/json--index/) to work out how to serialize / deserialize the data:

To serialize, we simply call `JsonDoc.string()`.

```pony
  fun ref write(json: JsonDoc): Bool =>
    """
    Write the serialized JsonDoc to the buffer.

    Will return true if written and verification succeeds.

    Will return false if the string is too long for the buffer or
    the readback doesn't match for some other reason.
    """
    _write(json.string())
```

To deserialize, we simply call `JsonDoc.parse()?` on the string returned from the database:

```pony
  fun ref read(): JsonDoc ref ? =>
    """
    Once we have confirmed that the data is NOT NULL, we create a new JsonDoc instance and call `JsonDoc.parse()?` on it with the string to populate it.
    """
    if (get_boxed_array().is_null()) then
      error
    else
      var json: JsonDoc = JsonDoc
      json.parse(_v.string())?
      json
    end
```

Hopefully you can see from this example, that we have gone out of our way to make the creation of custom SQL Types as painless as possible.
