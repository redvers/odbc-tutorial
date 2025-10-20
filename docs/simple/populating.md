# Populating Our Tables

We're going to manually insert two of the Play names into our database to make the process as clear as possible:

## Preparing Statements

When we are executing any statements that use any kind of user-input or are going to be repeated with different data or parameters, we should always use `prepare()?` and `execute()?`, instead of `direct_exec()?`. The reasons for this are SQL Injection vulnerabilities and performance.

A prepared statement allows you to create tokens in your SQL Statement that are replaced with data at runtime. For example:

```pony
sth.prepare("insert into play (name) values (?)")?
```

This creates a _parameter_ placeholder which we need to populate with the correct value to insert into the database.

Let's write a simple function to populate the play table:

```pony
fun populate_play_table(sth: ODBCStmt)? =>
  var name: SQLVarchar = SQLVarchar(31)
  sth
    .> prepare("insert into play (name) values (?)")?
    .> bind_parameter(name)?
```

When we bind parameters we have to bind them in order.  If there were multiple parameters in our `prepare()?` statement then we would have to execute multiple `bind_parameter()?` statements, with the variables and types in the correct order. We will see this later when we populate more complex tables.

Now that we have our statement fully prepared, we can populate data and execute it:

```pony
  name.write("Romeo and Juliet")
  sth.execute()?

  name.write("A Midsummer nights dream")
  sth.execute()?
```

Please note that we are reusing the same statement handle (sth), and the same bound parameter (name). In doing so we do not have to do the SQL statement parsing, query setup, memory allocation for our buffers, and binding said newly allocated buffers.

Of course, in our example you'll need to add a call to this function on line 17 of our example:

```pony
      let sth: ODBCStmt = dbh.stmt()?
      create_tables(sth)?
      populate_play_table(sth)?
    else
```

Next up, let's query the data!
