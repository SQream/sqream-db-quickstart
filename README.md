# SQream DB Quickstart
This tutorial uses ClientCmd, the SQream DB command line client, to introduce key concepts and tasks, including:
* Creating a database, creating tables
* Loading CSV data into tables
* Querying
* Basic monitoring

Note: ClientCmd, the SQL client, is distrubted with SQream DB, so nothing extra is needed.

## Before you begin
Make sure you have already installed SQream DB.

The rest of this guide assumes you're running SQream DB listening on local port 5000

## 1. Log into SQream DB
1. Open a terminal window and start ClientCmd at the command prompt:
```bash
$ ClientCmd --username=<username> --password=<password> --database=<database>
```
Where:
* `<username>` is the role for signing into SQream DB. A new SQream DB installation comes with the built-in `sqream` role.
* `<password>` is the password for the SQream DB role
* `<database>` is the database name. If this is your first time, select the built-in `master` database

Note: If connecting to a different server than `localhost` and port `5000`, specify them using the `--host` and `--port` arguments. For example, `$ ClientCmd --host=192.168.0.11 --port=5112 ...`.

You should see our interactive client and the connected database name.
```
$ ClientCmd --username=sqream --password=sqream --database=master
Interactive client mode
To quit, use ^D or \q.

master=>
```

To exit the interactive client, use `\q` or <kbd>Ctrl</kbd>+<kbd>D</kbd>.

## 2. Create a new database for testing
```sql 
CREATE DATABASE test;
```

Note: Each SQL statement must be terminated with a semicolon - <kbd>;</kbd>

Upon successful creation, we will want to reconnect the client to this database. In the client, use the meta-command `\c test`. You can also re-start the client with the new `--database=test` argument.

If you successfuly connected to the database, you will see the `test=>` prompt, in place of the previous one.

## 3. Create a sample table
```sql
CREATE OR REPLACE TABLE customer (
  first_name varchar(40) ,
  last_name varchar(40) ,
  email varchar(100) ,
  city varchar(40) ,
  country varchar(30) ,
  start_date date
  );
```

Note: The CSV we'll be loading down the line must match the table structure, including specific data types.

## 4. Copy data into the table

