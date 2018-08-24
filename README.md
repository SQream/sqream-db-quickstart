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
Open a terminal window and start ClientCmd at the command prompt:
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
If you make a mistake, use <kbd>Ctrl</kbd>+<kbd>C</kbd> to escape the current statement and clean the shell.

Find the SQream DB version
```sql
master=> select show_version();
v2.13
1 row
time: 0.073726s
```

## 2. Create a new database for testing
```sql 
CREATE DATABASE test;
```

Note: Each SQL statement must be terminated with a semicolon - <kbd>;</kbd>

Now reconnect the client to this database. Use the meta-command `\c test`.
You can also re-start the client with the new `--database=test` argument.

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
Note:
* You may not be familiar with the `IDENTITY` syntax. We're telling the database to start this column with the number 1, and increment 1 for every new value. This is similar to an autoincrement keyword in other databases.
* The CSV we'll be loading in the rest of this tutorial, `customers.csv`, must match the table structure, including specific data types.

## 4. Copy data into the table

Let's peek at the CSV file we'll be loading.
```bash
$ head -n5 /temp/customers.csv
first_name,last_name,email,gender,country,balance
Dollie,Deackes,ddeackes0@godaddy.com,Female,Indonesia,1962.55
Rhonda,Seiler,rseiler1@ezinearticles.com,Female,Greece,1911.38
Rolland,Friel,rfriel2@w3.org,Male,Indonesia,1594.77
Ruy,Sprey,rsprey3@webs.com,Male,Pakistan,1226.00
```

Before we load:
* We note that this CSV actually has one less column than we defined. So, we must tell the database to fill in the `id` column (identity column) by itself.
* We also note that this CSV has a header. We'll have to tell the database to skip this line, or the types will mismatch!

```sql
 COPY customers (first_name,last_name,email,gender,country,balance) FROM '/temp/customers.csv' WITH offset 2;
```

## 5. Query the loaded data
### Count table entries
```sql
test=> SELECT count(*) from customers;
3000
1 row
time: 0.061949s
```

### See all data
Peek at the records in the table
```sql
test=> SELECT top 5 * from customers;
3001,Dollie                                  ,Deackes                                 ,ddeackes0@godaddy.com                                                                               ,Female,Indonesia                               ,1962.5500
3002,Rhonda                                  ,Seiler                                  ,rseiler1@ezinearticles.com                                                                          ,Female,Greece                                  ,1911.3800
3003,Rolland                                 ,Friel                                   ,rfriel2@w3.org                                                                                      ,Male  ,Indonesia                               ,1594.7700
3004,Ruy                                     ,Sprey                                   ,rsprey3@webs.com                                                                                    ,Male  ,Pakistan                                ,1226.0000
3005,Celestyna                               ,Duffie                                  ,cduffie4@apple.com                                                                                  ,Female,China                                   ,710.9800
5 rows
time: 0.233690s
```

### Query based on filters
Return list of customer emails who have academic e-mails, with the `LIKE` syntax

```sql
test=> select email from customers where email like '%.edu';
bedmondsong@uiuc.edu
dglabachj@stanford.edu
awells14@uiuc.edu
mschaffel2i@umich.edu
anarup2z@unc.edu
bcargenven38@msu.edu
bmaric3n@columbia.edu
npeckitt3t@ucla.edu
tpiper46@arizona.edu
pstock54@msu.edu
thupe58@stanford.edu
-- [...]
156 rows
time: 0.116133s
```

### Aggregate and summarize
Find countries with highest average account balances

```sql
test=> SELECT TOP 10 country,
.               AVG(balance) AS avg_balance
. FROM   customers
. GROUP  BY 1
. ORDER  BY 2 DESC;
South Sudan                             ,2002.0400
Kyrgyzstan                              ,1992.3400
Moldova                                 ,1947.5800
Liechtenstein                           ,1941.2800
Svalbard and Jan Mayen                  ,1899.9100
Malawi                                  ,1868.6750
Mayotte                                 ,1799.4500
Bhutan                                  ,1789.3400
Rwanda                                  ,1763.7200
Guinea                                  ,1679.3900
10 rows
time: 0.289319s
```

Note:
* Using positional arguments to group by and sort helps brevity. `GROUP BY 1` is synonymous with `GROUP BY country`, while `ORDER BY 2` is synonymous with `ORDER BY avg_balance`

## Monitor the SQream DB instance
### List open SQream DB connections
```sql
test=> SELECT show_connections();
192.168.0.186       ,13,2018-08-24 16:36:11 ,31,2018-08-24 16:36:11 ,  SELECT show_connections();                              
192.168.0.186       ,6,2018-08-24 16:34:38 ,24,2018-08-24 16:34:38 ,   COPY customers (first_name,last_name,email,gender,country,balance) FROM '/temp/customers.csv' WITH offset 2;
3 rows
time: 0.089976s
```
Column order:
* Connection host
* Connection ID
* Connection start time
* Statement ID
* Statement start time
* Statement text

### Show server status
```sql
test=> SELECT show_server_status();
sqream,0,192.168.0.186,5000,test,sqream,192.168.0.186,39,SELECT show_server_status();,24-08-2018 16:45:06,Execute,24-08-2018 16:45:06
1 row
time: 0.118321s
```

Column order:
* Service name
* Instance ID
* Connection ID
* Server IP
* Server port
* Database name
* Username
* Client IP
* Statement ID
* Statement text
* Statement start time
* Statement status
* Statement status start

## Clean up
Once you're done with your test, drop the test database.

1. Switch to the `master` database from the client: `\c master`
2. Drop the `test` database:
```sql
DROP DATABASE test;
```
You can now leave the client with `\q` or <kbd>Ctrl</kbd>+<kbd>D</kbd>.

