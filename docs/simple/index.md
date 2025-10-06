# Simple API Overview

The main purpose of this Simple API is to provide 95% of the functionality required to write any database integration you need without having to expose yourself to any of the sharp corners of the C API.

## Philosophy

We have tried to stay consistent in the design of this API in order to try and make writing your code as unobtrusive as possible. Here are the principles we have tried to follow:

### Transactions and Code Blocks

When one writes a database application it is fairly common for us to batch SQL Statements into transactions. The reason for this is because we want these statements to either succeed or fail as one unit. If the statements succeed, then we `commit` the transaction. If they fail, we `rollback` the transaction, and it's like none of the statements were executed at all.

Pony has a mechanism that is well suited to that - partial functions!

```pony
try
  statement_handle.direct_exec("some SQL command")?
  statement_handle.direct_exec("some other SQL command")?
  statement_handle.direct_exec("etc etc etc ...")?
  database_handle.commit()
else
  database_handle.rollback()
end
```

### SQL Datatypes

