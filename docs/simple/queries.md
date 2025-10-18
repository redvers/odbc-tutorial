# Simple Queries

Let's write a simple function to query the database for the id (I64) for a specific play in the play table.

## Preparing the Query

A reminder that our tables schema looks like this:

```sql
CREATE TEMPORARY TABLE play (
  id BIGSERIAL,
  name VARCHAR(30) UNIQUE NOT NULL
);
```

In order to fulfil our function, we will need to provide a SQLVarchar _in_, and a SQLBigInteger _out_.

```pony
fun play_id_from_name(sth: ODBCStmt, queryname: String): I64 ? -=>
  var id: SQLBigInteger = SQLBigInteger
  var name: SQLVarchar = SQLVarchar(31)
```

Like before, we need to bind our name _parameter_ to our query using `bind_parameter()?`. In addition, we need to bind a _column_ for every column that will be in the query's result set.  We do this using the somewhat intuitive `bind_column()?` function:

```pony
  sth
    .> prepare("select id from play where name = ?")?
    .> bind_parameter(name)?
    .> bind_column(id)?
```

Then we can write our value to the name _parameter_, execute the query and fetch the (singular, due to name being UNIQUE) result back.

```pony
  name.write(queryname)

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
  create_tables(sth)?
  populate_play_table(sth)?
  Debug.out(" R&J: " + play_id_from_name(sth, "Romeo and Juliet")?.string())
  Debug.out("MSND: " + play_id_from_name(sth, "A Midsummer nights dream")?.string())
  try
    play_id_from_name(sth, "I don't exist")?
  else
    Debug.out("I don't exist doesn't exist™")
  end
```
