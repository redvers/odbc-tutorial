# Simple Queries

Let's write a simple function to query the database for the id (I64) for a row with a specific name:

## Preparing the Query

A reminder that the part of the table we're interested in looks like this:

```sql
CREATE TABLE psqldemo (
  id BIGSERIAL,
  name VARCHAR(254) UNIQUE NOT NULL,
  ⋮
```

In order to fulfil our function, we will need to provide a SQLVarchar _in_, and a SQLBigInteger _out_.

```pony
fun id_from_name(sth: ODBCStmt, qname: String): I64 ? -=>
  var id: SQLBigInteger = SQLBigInteger
  var name: SQLVarchar = SQLVarchar(254)
```

Like before, we need to bind our name _parameter_ to our query using `bind_parameter()?`. In addition, we need to bind a _column_ for every column that will be in the query's result set.  We do this using the somewhat intuitive `bind_column()?` function:

```pony
  sth
    .> prepare("select id from psqldemo where name = ?")?
    .> bind_parameter(name)?
    .> bind_column(id)?
```

Then we can write our value to the name _parameter_, execute the query and fetch the (singular, due to name being UNIQUE) result back.

```pony
  name.write(qname)
  sth.execute()?

  if (sth.fetch()?) then
    sth.finish()?
    id.read()?
  else
    error // No row returned
  end
```

NOTE: There is a trap here. You *must* check the return value of `fetch()?`. If you do not, you are going to end up with the previous value of `id`, not the one that would be returned.  (Arguably in this case it doesn't matter as the value of `id` defaults to SQLNull so an `id.read()?` would fail regardless). But let's try to set a good example eh?

Let's add some example calls to our Main.create function to test this:

```pony
  let sth: ODBCStmt = dbh.stmt()?
  try
    create_table(sth)?
    Debug.out("Successfully created table: psqldemo")
    populate_demo_table(sth)?
    Debug.out("Successfully written two rows")

    Debug.out("First Simple Row: " + id_from_name(sth, "First Simple Row")?.string())
//  dbh.commit()?
  else
//  dbh.rollback()?
  end
```
