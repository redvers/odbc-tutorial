# About SQL Types

As mentioned previously, all data is written in and out of the database via textual buffers.  In principle you can just use the SQLVarchar type for all input and output parameters and do the type conversions yourself.  However, we have provided the default ODBC SQL Types as follows:

| ODBC SQL Type  | Pony API Type   | Pony Type | Max String Size |
|----------------|-----------------|-----------|-----------------|
| SMALLINT       | SQLSmallInteger | I16       | 8               |
| INTEGER        | SQLInteger      | I32       | 15              |
| BIGINT         | SQLBigInteger   | I64       | 22              |
| FLOAT          | SQLFloat        | F32       | 20              |
| REAL           | SQLReal         | F64       | 30              |
| VARCHAR        | SQLVarchar      | String    | N/A             |

## All about NULL

For every SQL Datatype we create we have to also support a Nullable version. Instead of doing this by defining duplicate types for everything, we chose instead to take the following approach:

## Reading NULLs

```pony
var my_sql_integer: SQLInteger = SQLInteger
/*
 * Stuff happens here which we shall explain on the
 * next page which will populate this variable with
 * the result of our query.                         */

if (my_sql_integer.is_null()) then
  // The value returned was SQLNull from the database
else
  var myint: I32 = my_sql_integer.read()?
end
```

If your schema has marked a column as `NOT NULL`, then you can safely call `read()?` without testing for NULL.

If however you try to `read()?` the value directly without testing for NULL and it is NULL, the function will error.

## Writing NULLs

All pony SQL Types default to NULL, so all that needs be done is to create your object and not set a value.

If you are reusing an object, you can either call `reset()` or `null()` (if you want to be more explicit).
