# ODBC Tutorial

Welcome to the Pony ODBC tutorial.  The purpose of this tutorial is to get you to the point that you are able to do all the database things you are used to being able to do in other languages, with your RDBMS of choice.

If you've used ODBC, or DBD/DBI, or even jdbc - many of these concepts will be familiar to you.

We're going to assume that you have some knowledge of interfacing with a database in some language as well as basic concepts such as databases, tables, schemas, and the like.

We currently test the pony-odbc package against PostgreSQL, MariaDB (MySQL Open Source fork), and SQLite3.

## What is ODBC and pony-odbc?

ODBC (Open Database Connectivity) is an open standard API that allows applications to access data from a variety of database management systems (DBMS) using a common set of functions and SQL syntax, regardless of the specific type, brand, or location of the database.

### ODBC's Aspirations

ODBC functions as a bridge between software applications and different databases, making it possible to write a database-agnostic application that can work with any database simply by using the appropriate ODBC driver.

It decouples the application layer from the database layer, letting developers swap database engines or migrate data sources with minimal changes to code.

However - one should note that most databases were originally developed in a non-open world where vendor-lockin was considered a feature, not a bug. As such, there are still cases where different databases will respond differently to the same code.

Examples of this include:

* Returning different ODBC Error Codes (known as SQLStates) for the same invalid operation.
* Throwing or not throwing a warning on SQL Statements like: `DROP TABLE IF EXISTS foo;`.
* Whether an error is thrown when an invalid SQL Statement is prepared, or when it is executed.
* Which ODBC layer (Environment/Database/Statement) reports the error when a failure occurs.

ODBC has rough edges, regardless of the binding language.  We have done our best to minimize them here.

### pony-odbc Aspirations

By targeting the Pony binding to ODBC we aim to be able to provide as broad a database support as possible.
