# Testing SQLJson

In order to test our new custom class, let's update our tutorial example:

## Add the json package

```pony
use "json"
```

## Create a JsonDoc to store

We'll use the simple example from the package documentation, and create two functions. One to write, one to read - and we'll display the serialized output directly from JsonDoc object our SQLJson created!

```pony
  var json: JsonDoc = JsonDoc
  var obj: JsonObject = JsonObject

  obj.data("key") = "value"
  obj.data("property") = true
  obj.data("array") = JsonArray.from_array([ as JsonType: I64(1); F64(2.5); false])
  json.data = obj

  write_json_record(sth, json)?
  Debug.out(read_json_record(sth)?.string())
```

## Write the JsonDoc (write\_json\_record)

In this function, we'll just insert our JsonDoc into a row with the name "Json Record".  Note that this function takes a pony class `JsonDoc`.

```pony
  fun write_json_record(sth: ODBCStmt, json: JsonDoc)? =>
    var name: SQLVarchar = SQLVarchar(254)
    var jsonfrag: SQLJson = SQLJson(1023)
    sth
      .> prepare("insert into psqldemo (name,jsonfragment) values (?,?)")?
      .> bind_parameter(name)?
      .> bind_parameter(jsonfrag)?

    name.write("Json Record")
    jsonfrag.write(json)

    sth
      .> execute()?
      .> finish()?
```

## Read our JsonDoc (read\_json\_record)

In this function we'll read the row and return a `JsonDoc`.

```pony
  fun read_json_record(sth: ODBCStmt): JsonDoc ? =>
    var name: SQLVarchar = SQLVarchar(254)
    var jsonfrag: SQLJson = SQLJson(1023)
    sth
      .> prepare("select jsonfragment from psqldemo where name = ?")?
      .> bind_parameter(name)?
      .> bind_column(jsonfrag)?

    name.write("Json Record")
    sth
      .> execute()?
      .> fetch()?
      .> finish()?
    jsonfrag.read()?
```
