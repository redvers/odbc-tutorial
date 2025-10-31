# Populating Our Tables

We're going to manually insert two rows into this table to illustrate the simplest case.

## Preparing Statements

When we are executing any statements that use any kind of user-input or are going to be repeated with different data or parameters, we should always use `prepare()?` and `execute()?`, instead of `direct_exec()?`. The reasons for this are SQL Injection vulnerabilities and performance.

A prepared statement allows you to create tokens in your SQL Statement that are replaced with data at runtime. For example:

```pony
sth.prepare("insert into psqldemo (name) values (?)")?
```

This creates a _parameter_ placeholder which we need to populate with the correct value to insert into the database.

Let's write a simple function to populate our table:

```pony
fun populate_demo_table(sth: ODBCStmt)? =>
  var name: SQLVarchar = SQLVarchar(254)
  sth
    .> prepare("insert into psqldemo (name) values (?)")?
    .> bind_parameter(name)?
```

When we bind parameters we have to bind them in order.  If there were multiple parameters in our `prepare()?` statement then we would have to execute multiple `bind_parameter()?` statements, with the variables and types in the correct order. We will see this later when we populate more fields.

Now that we have our statement fully prepared, we can populate data and execute it:

```pony
  name.write("First Simple Row")
  sth.execute()?

  name.write("Second Simple Row")
  sth
    .> execute()?
    .> finish()?
```

`name` is a pointer to a buffer which we have told our database will contain our argument. So we populate that buffer with our data, execute our query, populate it with different data, execute the query again.

The `.finish()?` call tells the database that we are no longer using our `sth` handle for that prepared statement, and that we will never execute that prepared query again.

Yes, we could have achieved the same thing with:

```pony
  sth.direct_exec("insert into psqldemo (name) values ('First Simple Row')")?
  sth.direct_exec("insert into psqldemo (name) values ('Second Simple Row')")?
```

But doing it using prepare statements means that not only do we not have to worry about SQL Injection, but:

- The SQL Statement is only being parsed once.
- The associated objects are allocated only once.
- The input/output buffers only being allocated once, and re-used.
- The binding of buffers is only done once.

When we expand to hundreds or thousands of queries, this makes a significant difference.

Of course, if we choose, we can now treat both the table creation and these row insertions as a single transaction like this:

```pony
    let sth: ODBCStmt = dbh.stmt()?
    try
      create_tables(sth)?
      populate_demo_table(sth)?
//    dbh.commit()?
    else
//    dbh.rollback()?
```

Next up, let's query the data!
