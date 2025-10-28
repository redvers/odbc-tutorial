# Simple API Tutorial

In this tutorial we're going to:

- Create a table
- Write to a table
- Perform queries
- Extend our program with three custom SQL types that are unsupported by the ODBC standard.

## Schema

Our very simple table (psqldemo) is defined as follows:

| Column Name  | SQLType        | Nullable | Default           |
|--------------|----------------|----------|-------------------|
| id           | bigint         | No       | Autoincrements    |
| name         | varchar(254)   | No       |                   |
| xmlfragment  | xml            | Yes      | NULL              |
| jsonfragment | json           | Yes      | NULL              |
| insert\_ts   | timestamp w/TZ | No       | Current Timestamp |

The first two fields use standard ODBC SQL datatypes.  The last three we will be building custom SQL types for.

### Table: psqldemo

The SQL required to create our example table is below. Usually the tables are created independently of applications, but for completeness - we will have our program create our table.


```sql
CREATE TABLE psqldemo (
  id BIGSERIAL,
  name VARCHAR(254) UNIQUE NOT NULL,
  xmlfragment XML,
  jsonfragment JSON,
  insert_ts TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT current_timestamp
);

ALTER TABLE psqldemo ADD CONSTRAINT psqldemo_pkey PRIMARY KEY (id);
```
