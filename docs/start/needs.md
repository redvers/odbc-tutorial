# What You Need

In this tutorial, we are going to walk-through how to write a Pony program that connects to a PostgreSQL database from an Ubuntu host using unixODBC. From there we will write our application, perform various operations, and even build a custom SQL Type.

## Installation

In our example, we're going to assume that the database is local, the database is called "pony", and the username and password to access it are also "pony".

### Driver Manager

You can use either unixODBC or iODBC, they are functionally identical. In our example, we will use unixODBC. If you do want to use iODBC, the only differences are in the packages you install and the `use "lib:` declaration in your application (we will notate the differences when we get there).

```shell
red@panic:~$ sudo apt install unixodbc-dev unixodbc
```

### PostgreSQL Driver

```shell
red@panic:~$ sudo apt install odbc-postgresql
```

## Create the test user and database

We will assume that our postgres database is listening on 127.0.0.1 and we will connect to it via the network instead of via a unix socket:

Create the user pony:

```shell
red@panic:~$ createuser -h 127.0.0.1 -U postgres --interactive -P
Enter name of role to add: pony
Enter password for new role:
Enter it again:
Shall the new role be a superuser? (y/n) n
Shall the new role be allowed to create databases? (y/n) y
Shall the new role be allowed to create more new roles? (y/n) n
Password:
```

Create the database ponydb:

```shell
red@panic:~$ createdb -h 127.0.0.1 -U pony -O pony ponydb
Password:
```

Ensure that we can connect to it via a native postgres client

```shell
red@panic:~$ psql -h 127.0.0.1 -U pony ponydb
Password for user pony:
psql (16.10 (Ubuntu 16.10-0ubuntu0.24.04.1), server 14.5 (Debian 14.5-2.pgdg110+2))
Type "help" for help.

ponydb=> \q
red@panic:~$
```

Success!

## Example ODBC Configuration and Testing

For ease of our example, let's configure and test a postgreSQL connection. In your user's home directory create the file: .odbc.ini and populate with the following:

```ini
[psql-demo]
Driver=/usr/lib/x86_64-linux-gnu/odbc/psqlodbca.so
Description=My awesome pony database
Servername=127.0.0.1
Driver=PostgreSQL
Database=ponydb
UserName=pony
Password=pony
```

Now to test it!

```shell
red@panic:~$ isql psql-demo
+---------------------------------------+
| Connected!                            |
|                                       |
| sql-statement                         |
| help [tablename]                      |
| echo [string]                         |
| quit                                  |
|                                       |
+---------------------------------------+
SQL> select 1;
+------------+
| ?column?   |
+------------+
| 1          |
+------------+
SQLRowCount returns 1
1 rows fetched
SQL> ^C
red@panic:~$
```

If you get the above result, you have a working ODBC Driver Manager, PostgreSQL Driver, database and user.
