# Transactions, Commits, and Rollbacks

We have gone out of our way so far to ensure that none of the changes we have made to our database have been persistent. We did that so that during this tutorial we can be sure that every time we tested our program we did so from a known state. It's time to rip that band-aid off and move to transactions.

## Database Transactions

A database transaction is a sequence of one or more operations—such as reading, writing, updating, or deleting data—performed on a database as a single, logical unit of work. Transactions ensure that either all operations are successfully completed (committed) or none are applied (rolled back), maintaining data integrity even in the event of system failures. In other words, all commands in a transaction are either applied in an atomic manner, or rejected.

ODBC by default commits every command you execute after you execute it.  This is called "autocommit".

In order to implement transactions, we need to disable autocommit in Main.create:

```pony
  try
    let dbh: ODBCDbc = enh.dbc()?
    dbh.set_autocommit(false)?
    dbh.connect("psql-demo")?
```

Now remove all of the TEMPORARY keywords from the SQL create statements and run out example again multiple times.  As we didn't commit the transaction, the tables we added and inserted data on are removed automatically from the database on disconnect.

## Testing for a table's existence

In order to ensure that we do not attempt to recreate the tables if the tables have been created by a previous run, we need a function to determine if a table exists.

We're going to assume in this example that if we only commit our database when all of our tables have been successfully created, we can treat all the tables as a unit and only test for one table.

There is an API call which allows us to search for tables.  We can use this API call as a simple way to identify if a table exists:

```pony
  fun check_table_exists(sth: ODBCStmt, tablename: String): Bool ? =>
    sth.tables("", "", tablename, "TABLE")?
    var rt: Bool = sth.fetch()?
    sth.finish()?
    rt
```

`tables()?`, aka SQLTables call behaves like a SQL statement, so we must ensure that the ODBCStmt that we pass it is pristine.  We can either do that by creating a new one, or by ensuring that `finish()?` was called before it to reset it. You should not call `execute()?` after calling `tables()?` - it is implied.  We did not bind any columns to this query because as we don't need the data.  We just need to know if the data exists.

The function `fetch()?` returns `true` if there was a row of data (ie, our table exists), or `false` if there is no data.

Now we can modify our program to only create tables if the tables don't exist… and commit them if they are successful!

```pony
  if (not check_table_exists(sth, "play")?) then
    create_tables(sth)?
    dbh.commit()?
  end
```

## Testing if a table contains data

A simple SQL query will do this.

```pony
  fun check_play_populated(sth: ODBCStmt): Bool ? =>
    var cnt: SQLBigInteger = SQLBigInteger
    sth
      .> finish()?
      .> bind_column(cnt)?
      .> direct_exec("select count(*) from play")?
      .> fetch()?
      .> finish()?

    (cnt.read()? > 0)
```

Now we can likewise gate the call to `populate_play_table()?` and execute a `commit()?` on success.

```pony
  if (not check_play_populated(sth)?) then
    populate_play_table(sth)?
    dbh.commit()?
  end
```
