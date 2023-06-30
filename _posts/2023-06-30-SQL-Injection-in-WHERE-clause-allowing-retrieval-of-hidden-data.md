---
title: SQL injection vulnerability in WHERE clause allowing retrieval of hidden data
categories: [PortSwigger Academy,SQL Injection]
tags: [SQL-Injection]
---

## Description

- **This lab contains a SQL injection vulnerability in the product category filter. When the user selects a category, the application carries out a SQL query like the following:**
- `SELECT * FROM products WHERE category = 'Gifts' AND released = 1`
- **To solve the lab, perform a SQL injection attack that causes the application to display one or more unreleased products.**


## Method to solve

- the first thing to do is always try to play around with the website.

![](/assets/img/posts/2023-06-30-SQL-Injection-in-WHERE-clause-allowing-retrieval-of-hidden-data/2023-06-30-SQL-Injection-in-WHERE-clause-allowing-retrieval-of-hidden-data-21-11-23.png)

- Based on the description, we will need to perform SQL injection in category filter.

![](/assets/img/posts/2023-06-30-SQL-Injection-in-WHERE-clause-allowing-retrieval-of-hidden-data/2023-06-30-SQL-Injection-in-WHERE-clause-allowing-retrieval-of-hidden-data-21-12-00.png)

- The SQL query is triggered based on the GET request send in the URL to get the category of the product.
- The description has given us the SQL query.
- Since `released=1` is products that has been released, changing it to 0 might retrieve different product.
- We could just perform SQL injection at the URL to retrieve hidden data.


![](/assets/img/posts/2023-06-30-SQL-Injection-in-WHERE-clause-allowing-retrieval-of-hidden-data/2023-06-30-SQL-Injection-in-WHERE-clause-allowing-retrieval-of-hidden-data-21-19-02.png)

![](/assets/img/posts/2023-06-30-SQL-Injection-in-WHERE-clause-allowing-retrieval-of-hidden-data/2023-06-30-SQL-Injection-in-WHERE-clause-allowing-retrieval-of-hidden-data-21-20-22.png)

![](/assets/img/posts/2023-06-30-SQL-Injection-in-WHERE-clause-allowing-retrieval-of-hidden-data/2023-06-30-SQL-Injection-in-WHERE-clause-allowing-retrieval-of-hidden-data-21-28-44.png)

- We managed to get different products by performing SQL injection.
