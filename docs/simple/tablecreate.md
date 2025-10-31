# Creating Our Table

In order to create our tables, we simply execute the SQL commands. As there are no parameters, we can use the function `direct_exec("my sql statement")?` which will either succeed or fail.

## Creating Statements

There are two SQL Statements needed to create our table.  The first creates the table, the second `ALTER`s the table to add a PRIMARY KEY constraint.  We need both of these functions to either succeed together, or fail together.  If we don't, we could end up in a situation where the table exists, but the primary key could cause data inconsistency.

For ease of following this tutorial, we will put the `commit()?` and `rollback()?` calls in the sources for illustration, but leave them commented out. This will result in no changes being committed, effectively making the tables `TEMPORARY` so we can run them again and again without having to drop them between runs.

In our application, let's make a Statement Handle and pass it to a function which will create our tables. Here is our full example thus far:

```pony
use "debug"
use "pony-odbc"
use "lib:odbc"

actor Main
  let env: Env

  new create(env': Env) =>
    env = env'

    let enh: ODBCEnv = ODBCEnv
    try
      let dbh: ODBCDbc = enh.dbc()?
      dbh.set_autocommit(false)?
      dbh.connect("psql-demo")?
      let sth: ODBCStmt = dbh.stmt()?
      try
        create_table(sth)?
        Debug.out("Successfully created table: psqldemo")
//      dbh.commit()?
      else
        Debug.out("Our create_tables() function threw an error")
//      dbh.rollback()?
        error
      end
    else
      Debug.out("Our application failed in some way")
    end
```

So to tease this out:

```pony
      try
        create_tables(sth)?
        Debug.out("Successfully created table: psqldemo")
//      dbh.commit()?
      else
        Debug.out("Our create_tables() function threw an error")
//      dbh.rollback()?
```

Were the commit/rollback calls uncommented:

If the `create_table(sth)?` fully succeeded, the `dbh.commit()?` function would be called and the changes committed to the database.  If it failed, the `dbh.rollback()?` function would be called - leaving us in a known state.

Now lets create our function to make these (temporary) tables.

```pony
  fun create_table(sth: ODBCStmt)? =>
    sth
      .> direct_exec(
         """
         CREATE TABLE psqldemo (
           id BIGSERIAL,
           name VARCHAR(254) UNIQUE NOT NULL,
           xmlfragment XML,
           jsonfragment JSON,
           insert_ts TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT current_timestamp
         );
         """)?
      .> direct_exec(
         """
         ALTER TABLE psqldemo ADD CONSTRAINT psqldemo_pkey PRIMARY KEY (id);
         """)?
      .> finish()?
```

When we are executing SQL commands that don't require any parameters, we use the `direct_exec()?` function, which executes the command immediately.

Yes, you *can* place parameters in the SQL statement, but there are very good security and performance reasons to *NOT* do this, which we will cover in the next section.
