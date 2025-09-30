# Overview

In this section we'll provide a high-level overview of how ODBC is designed and structured before we delve into how we use it.

## ODBC Architecture

As mentioned previously, ODBC is designed to be an abstraction over connectivity to databases. It consists of the following areas:

### Application

The application is what you write. This is your "client" of ODBC as it were.

### Driver Manager

The driver manager acts as an intermediary between your application and the database-specific driver.  The Driver Manager is what provides the programmatic API that we bind to.  As the programmatic API is standardized, you can use any implementation of a driver manager you wish.

Within the Driver Manager you configure individual Data Source Names (DSNs). Each DSN is a configuration entry that specifies attributes such as the vendor-specific driver to use, the database's hostname, the instance to connect to, username and password for authentication, and so on.

A Driver Manager is capable of managing connections to different databases, database instances, with different users, and even different vendors.

We have tested pony-odbc against two Driver Managers, unixODBC and iODBC.  Feel free to use whichever is better supported in your environment. Configuration of both of these Driver Managers is done via /etc or ~user dotfile versions of odbc.ini files.

### Driver

Each database type requires a specific ODBC driver. A driver is a library that translates ODBC function calls into commands understood by a particular database system. Drivers can modify application requests to comply with database-specific syntax and protocols.

This driver is typically provided by your database vendor, and they are responsible for mapping their database specific "foibles", to the ODBC standard.

In theory, these database drivers would be consistent in their behaviour. In reality, they are not.

### Database

Your actual database.

## ODBC Programmatic Structure

pony-odbc abstracts much of the following away depending on which of the APIs you choose to use. However, understanding the foundations is beneficial.

The ODBC (client) API is implemented on top of three fundamental structures: Environment, Database, and Statement.

### Environment Handle (SQL\_HANDLE\_ENV)

* The Environment Handle represents the global context for ODBC Operations.
* You typically only need one Environment handle per application.
* It is used to create your database handles, or setting or querying attributes in the Driver Manager.

### Database Handle (SQL\_HANDLE\_DBC)

* Each Database Handle identifies a specific connection to a database.
* Multiple Database Handles can be active from a single Environment Handle, so you can connect to multiple database instances from potentially multiple vendors concurrently.
* Each Database Handle handles its own connection-based attributes.  This includes the obvious such as which Driver to use to connect to which database as which user, whether your database connection is Read-Only or Read-Write, to whether statements are automatically committed or transactions are in use.

### Statement Handle (SQL\_HANDLE\_STMT)

* A Statement Handle tracks and manages a SQL statement, its preparation, execution, and associated result-sets.
* Statement Handles are allocated from Database Handles.  You can have multiple Statement Handles per Database Handle, but typically Database Drivers serials queries.
* Each Statement Handle manages its own resources specific to the provided query.
