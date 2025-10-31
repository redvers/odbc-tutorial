# Why Custom Types?

As all data going in and out of the ODBC API goes in and out in a textual format by default for portability, it's very rare that it is something you need to do at all.

You can just use SQLVarchars, or create your own type aliases for code readability like this:

```pony
type SQLXml is SQLVarchar
type SQLJson is SQLVarchar
type SQLTimestampWTz is SQLVarchar
```

… and in our example:

```pony
  var name: SQLVarchar = SQLVarchar(254)
  var xmlfrag: SQLXml = SQLXml(1024)
  var jsonfrag: SQLJson = SQLJson(1024)

  sth
    .> prepare("insert into psqldemo (name, xmlfragment, jsonfragment) values (?,?,?)")?
    .> bind_parameter(name)?
    .> bind_parameter(xmlfrag)?
    .> bind_parameter(jsonfrag)?

  name.write("Some Name")
  xmlfrag.write(
    """
    <foo>bar</foo>
    """)

  jsonfrag.write(
    """
    {
      "packages": [],
      "deps": [
        {
          "locator": "github.com/redvers/pony-odbc.git"
        }
      ]
    }
    """)

  sth.execute()?
```

But, wouldn't it be better if we could create SQL Types that give us legitimate pony classes for our inputs and outputs like this:

```pony
  // SQLXml
  fun write(xml: Xml2Doc): Bool
  fun read(): Xml2Doc ?

  // SQLJson
  fun write(json: JsonDoc): Bool
  fun read(): JsonDoc ?

  // SQLTimestampWTz
  fun write(ts: PosixDate, tz: String): Bool
  fun read(): (PosixDate, String iso^) ?
```

In the following pages, we will fully implement SQLJson, compatible with `ponylang/json`.
