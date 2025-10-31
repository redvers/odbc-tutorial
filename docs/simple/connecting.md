# Connecting To Our Database

As we mentioned in our overview, before we can create any queries, we need a Statement Handle. In order to get a Statement Handle, we need a Database Handle. In order to get a Database Handle, we need an Environment Handle.

In this API, these are encapsulated as follows:

* Environment Handle, ODBCEnv
* Database Handle, ODBCDbc
* Statement Handle, ODBCStmt

## Brief Database Transaction Primer

A database transaction is a sequence of one or more operations—such as reading, writing, updating, or deleting data—performed on a database as a single, logical unit of work. Transactions ensure that either all operations are successfully completed (committed) or none are applied (rolled back), maintaining data integrity even in the event of system failures. In other words, all commands in a transaction are either applied in an atomic manner, or rejected.

ODBC by default commits every command you execute after you execute it.  This is called "autocommit".

Transactions are a best-practice, so we are going to utilize them in our tutorial from the very beginning.

In order to implement transactions, we need to disable autocommit.

## General Structure of the API

The vast majority of calls on ODBCEnv, ODBCDbc, and ODBCStmt return `Bool ?`. In other words, these are partial functions.

Structuring the API in this way means that we can serialize our API calls into blocks of SQL commands, and then choose to fail / rollback a transaction if any of them fail. It makes for a very clean interface.

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
    else
    end
```

Let's tease this apart:

First we create our Environment Object:

```pony
  let enh: ODBCEnv = ODBCEnv
```

Then we create our Database Object, set auto\_commit to false, and connect to our database using the DSN we configured previously in our .odbc.ini file:

```pony
    try
      let dbh: ODBCDbc = enh.dbc()?
      dbh.set_autocommit(false)?
      dbh.connect("psql-demo")?
    else
      Debug.out("We were enable to connect to our database")
    end
```

Once the `dbh.connect()?` call is complete, we have an authenticated connection to the database instance that we requested.

Next up - let's create our table!
