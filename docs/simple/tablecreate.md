# Creating Our Tables

In order to create our tables, we simply execute the SQL commands. As there are no parameters, we can use the function `direct_exec("my sql statement")?` which will either succeed or fail.

## Creating Statements

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
      dbh.connect("psql-demo")?
      let sth: ODBCStmt = dbh.stmt()?
      try
        create_tables(sth)?
      else
        Debug.out("Our create_tables() function threw an error")
        error
      end
    else
      Debug.out("Our application failed in some way")
    end
```

Now lets create our function to make these (temporary) tables.

```pony
  fun create_tables(sth: ODBCStmt)? =>
      .> direct_exec(
         """
         CREATE TEMPORARY TABLE play (
          id BIGSERIAL,
          name VARCHAR(30) NOT NULL
         );
         """)?
      .> direct_exec(
         """
         ALTER TABLE play ADD CONSTRAINT play_pkey PRIMARY KEY (id);
         """)?
      .> direct_exec(
         """
         CREATE TEMPORARY TABLE player (
          id BIGSERIAL,
          name VARCHAR(20) NOT NULL
         );
         """)?
      .> direct_exec(
         """
         ALTER TABLE player ADD CONSTRAINT player_pkey PRIMARY KEY (id);
         """)?
      .> direct_exec(
         """
         CREATE TEMPORARY TABLE line (
          id BIGSERIAL,
          id_play INTEGER,
          id_player INTEGER,
          playerlinenumber INTEGER,
          actsceneline VARCHAR(15),
          playerline VARCHAR(127) NOT NULL
         );
         """)?
      .> direct_exec(
         """
         ALTER TABLE line ADD CONSTRAINT line_pkey PRIMARY KEY (id);
         """)?
      .> direct_exec(
         """
         ALTER TABLE line ADD CONSTRAINT line_id_play_fkey FOREIGN KEY (id_play) REFERENCES play(id);
         """)?
      .> direct_exec(
         """
         ALTER TABLE line ADD CONSTRAINT line_id_player_fkey FOREIGN KEY (id_player) REFERENCES player(id);
         """)?
```

Since we are not reading or writing any parameters in this example, we can simply use the `direct_exec()?` function which does, as the name suggests - direct execution.
