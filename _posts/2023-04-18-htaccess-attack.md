---
title: htaccess attack
categories: [Knowledge,Web]
tags: [Knowledge]
---

- [Explanation](#explanation)
- [Example](#example)
- [Remediation](#remediation)
- [References](#references)


# Explanation

- .htaccess = Hypertext Access
- a configuration file used by apache-based web servers
- .htaccess can be used to make the server behave in a certain way
- can used to add different type of extension as php file

# Example
- Example of htaccess for file upload

```bash
------------------------
.htaccess
------------------------

------------------------
AddType application/x-http-php .<random extension>
------------------------
```

- Example of ctf which used rewrite rule

```bash
------------------------
.htaccess
------------------------

------------------------
RewriteEngine On
RewriteCond %{THE_REQUEST} flag
RewriteRule ".*" "-" [F]
------------------------

------------------------
bypass it with url encode
------------------------

------------------------
curl http://website/%66lag.txt
------------------------
```

# Remediation

- disallow .htaccess file upload 

# References
- [Utilizing htaccess for exploitation](https://medium.com/@insecurity_92477/utilizing-htaccess-for-exploitation-purposes-part-1-5733dd7fc8eb)
- [Bypass website file upload - youtube](https://www.youtube.com/watch?v=xZd1JWmLGLk)
- [CTF example with different htaccess rule](https://ctftime.org/writeup/35661)