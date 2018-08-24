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
CREATE OR REPLACE TABLE customers ( 
   id         int IDENTITY (1,1), 
   first_name varchar(40) , 
   last_name  varchar(40) , 
   email      varchar(100) , 
   gender     varchar(6) , 
   country    varchar(40) , 
   balance DOUBLE );
```

Note: You may not be familiar with the `IDENTITY` syntax. We're telling the database to start this column with the number 1, and increment 1 for every new value. This is similar to an autoincrement keyword in other databases.

Note: The CSV we'll be loading in the rest of this tutorial, `customers.csv`, must match the table structure, including specific data types.

## 4. Copy data into the table

Let's peek at the CSV file we'll be loading.
```bash
$ head /temp/customers.csv
first_name,last_name,email,gender,country,balance
Dollie,Deackes,ddeackes0@godaddy.com,Female,Indonesia,1962.55
Rhonda,Seiler,rseiler1@ezinearticles.com,Female,Greece,1911.38
Rolland,Friel,rfriel2@w3.org,Male,Indonesia,1594.77
Ruy,Sprey,rsprey3@webs.com,Male,Pakistan,1226.00
Celestyna,Duffie,cduffie4@apple.com,Female,China,710.98
Jonathan,Wyndham,jwyndham5@blogs.com,Male,France,1919.56
Joelynn,Farryn,jfarryn6@sakura.ne.jp,Female,Poland,111.29
```
Before we load:
* We note that this CSV actually has one less column than we defined. So, we must tell the database to fill in the `id` column (identity column) by itself.
* We also note that this CSV has a header. We'll have to tell the database to skip this line, or the types will mismatch!

```sql
 COPY customers (first_name,last_name,email,gender,country,balance) FROM '/temp/customers.csv' WITH offset 2;
```

