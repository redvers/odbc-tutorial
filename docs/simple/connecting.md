# Connecting To Our Database

As we mentioned in our overview, before we can create any queries, we need a Statement Handle. In order to get a Statement Handle, we need a Database Handle. In order to get a Database Handle, we need an Environment Handle.

In this API, these are encapsulated as follows:

* Environment Handle, ODBCEnv
* Database Handle, ODBCDbc
* Statement Handle, ODBCStmt

## General Structure of the API

The vast majority of calls on ODBCEnv, ODBCDbc, and ODBCStmt return `Bool ?`. In other words, these are partial functions.

Structuring the API in this way means that we can serialize our calls and choose to fail / rollback a transaction if any of them fail. It makes for a clean interface.

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
      dbh.connect("psql-demo")?
    else
      Debug.out("We were enable to create our database handle")
    end
```

Let's tease this apart:

First we create our Environment Object:

```pony
  let enh: ODBCEnv = ODBCEnv
```

Then we create our Database Object and connect to our database using the DSN we configured previously in our .odbc.ini file:

```pony
    try
      let dbh: ODBCDbc = enh.dbc()?
      dbh.connect("psql-demo")?
    else
      Debug.out("We were enable to create our database handle")
    end
```

Once the `dbh.connect()?` call is complete, we have an authenticated connection to the database instance that we requested.

Next up - let's create some tables!
