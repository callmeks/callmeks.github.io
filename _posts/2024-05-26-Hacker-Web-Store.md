---
title: Hacker Web Store
categories:
  - CTF
tags:
  - nahamcon-CTF
  - Web
---

# Challenge Information

> Welcome to the hacker web store! Feel free to look around at our wonderful products, or create your own to sell.  

## Explanation

I started out by exploring whats available in the challenge. It consist of a product page, create page and admin page. It seems like I could insert information from create page and it will be updated to the product page.

![](/assets/img/posts/2024-05-26-Hacker-Web-Store/2024-05-26-Hacker-Web-Store-09-01-12.png)

I then tried to perform SQL injection and it somehow provides error when I perform SQL injection. 

![](/assets/img/posts/2024-05-26-Hacker-Web-Store/2024-05-26-Hacker-Web-Store-09-02-27.png)

Based on the SQL error, I got useful information which is the SQL query as well as it is vulnerable to SQL injection. the whole form is vulnerable to SQL injection but only the first two input could successfully retrieve result as the main SQL query is trying to save the input into the database and show in the product page. I then tried to get information such as the table, column name and the information.

SQL injection in product name input field: `','',(SELECT sqlite_version()))-- a`

![](/assets/img/posts/2024-05-26-Hacker-Web-Store/2024-05-26-Hacker-Web-Store-09-07-52.png)

By executing that SQL injection, I can easily confirmed that it is running SQLite. The SQL injection payload is created like that as it needed to match the existing SQL query which should be like this: `INSERT INTO Products (name, price, desc) VALUES ('', '', '');`. If the malicious SQL payload was added into it, it will become `INSERT INTO Products (name, price, desc) VALUES ('','',(SELECT sqlite_version()))-- a', '', '');`. There's a comment at the end of the malicious SQL query which mean anything behind it will be ignored. The bracket is to replaced the one that has been ignored by the comment. With this, I continue to get the information of the table and column.

SQL injection in product name input field: `','',(SELECT sql FROM sqlite_master))-- a`

![](/assets/img/posts/2024-05-26-Hacker-Web-Store/2024-05-26-Hacker-Web-Store-09-21-21.png)

With this, I got all the information I needed to retrieve the data.

SQL injection in product name input field: `','',(SELECT (id || ' -- ' || name || ' -- ' ||password) FROM users))-- a`

![](/assets/img/posts/2024-05-26-Hacker-Web-Store/2024-05-26-Hacker-Web-Store-09-28-06.png)

Although I got the username and password hash, there might be another result. I then tried to check if there's more usernames and passwords. 

SQL injection in product name input field: `','',(SELECT (group_concat(id) || ' --  '||group_concat(name) ||' --' ||group_concat(password)) FROM users))-- a`

```txt
1,2,3 -- Joram,James,website_admin_account --pbkdf2:sha256:600000$m28HtZYwJYMjkgJ5$2d481c9f3fe597590e4c4192f762288bf317e834030ae1e069059015fb336c34,pbkdf2:sha256:600000$GnEu1p62RUvMeuzN$262ba711033eb05835efc5a8de02f414e180b5ce0a426659d9b6f9f33bc5ec2b,pbkdf2:sha256:600000$MSok34zBufo9d1tc$b2adfafaeed459f903401ec1656f9da36f4b4c08a50427ec7841570513bf8e57
```

It seems like there's a total of 3 users which one is the admin account. I then tried to crack all 3 of the hash with the help of python script as it somehow does not work for hashcat and john. After searching a while, I managed to crack it with the help of this [tool](https://github.com/AnataarXVI/Werkzeug-Cracker) and the password is `ntadmin1234`. I then login with the creds and got the flag `flag{87257f24fd71ea9ed8aa62837e768ec0}`


## Things i learned from the challenge

- a fun SQLI
- so hard to crack that hash


