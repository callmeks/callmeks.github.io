---
title: ElMariachi-PC 
categories: [writeups,PwnTillDawn]
tags: [b2r,Windows,ThinVnc]
---

# Portal
- Difficulty : `easy`
- IP address : `10.150.150.69`
- Operating System : `Windows`

# Table of Content
- [Portal](#portal)
- [Table of Content](#table-of-content)
- [Nmap Result](#nmap-result)
- [Method to solve the challenge](#method-to-solve-the-challenge)

# Nmap Result
```
Nmap scan report for 10.150.150.69
Host is up, received conn-refused (0.26s latency).
Scanned at 2022-07-14 09:14:17 UTC for 151s

PORT      STATE SERVICE       REASON  VERSION
135/tcp   open  msrpc         syn-ack Microsoft Windows RPC
139/tcp   open  netbios-ssn   syn-ack Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds? syn-ack
3389/tcp  open  ms-wbt-server syn-ack Microsoft Terminal Services
| rdp-ntlm-info: 
|   Target_Name: ELMARIACHI-PC
|   NetBIOS_Domain_Name: ELMARIACHI-PC
|   NetBIOS_Computer_Name: ELMARIACHI-PC
|   DNS_Domain_Name: ElMariachi-PC
|   DNS_Computer_Name: ElMariachi-PC
|   Product_Version: 10.0.17763
|_  System_Time: 2022-07-14T10:00:01+00:00
| ssl-cert: Subject: commonName=ElMariachi-PC
| Issuer: commonName=ElMariachi-PC
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2022-07-13T09:10:22
| Not valid after:  2023-01-12T09:10:22
| MD5:   577e d21f 3299 8905 aa9d 96e6 28b6 b80a
| SHA-1: 55a9 c83b aa6e 14c2 ba31 05cc a14d c7dd 740b e983
| -----BEGIN CERTIFICATE-----
| MIIC3jCCAcagAwIBAgIQGeBlYhxedK9OMLG7KnpsHDANBgkqhkiG9w0BAQsFADAY
| MRYwFAYDVQQDEw1FbE1hcmlhY2hpLVBDMB4XDTIyMDcxMzA5MTAyMloXDTIzMDEx
| MjA5MTAyMlowGDEWMBQGA1UEAxMNRWxNYXJpYWNoaS1QQzCCASIwDQYJKoZIhvcN
| AQEBBQADggEPADCCAQoCggEBAMI5fpKebaSrdU2B37AFTWuTY/rKbiiManE34yVi
| BkZRGuVIl4ZNi3UkQRk5vNuMWYVGO2Zxjv0D5PCxVE8WoaNaAjF7Gstu8Lhp5rAg
| k4iIKU2CWSbNMGPoS9qzOl94r0YKXTVVNtQyEWm904lFYZDcxJe4DVetdB3oB8w7
| xuLaWE/DACYI6r21OZkKZKS+Cu6dCpR0tnHnCJTbArSNOVgecPnqNDjhdqHH7Qmq
| xd+JisKrabCWohEm34wSF5yECUSQ3LDB9PZlxpfDH2GhOYOXnut0pfjuVAD7+cf+
| 1CzqalWUQK6o9rSrgQD/Up4rdmHI5qyQ+Yk6SFBFZEmGza0CAwEAAaMkMCIwEwYD
| VR0lBAwwCgYIKwYBBQUHAwEwCwYDVR0PBAQDAgQwMA0GCSqGSIb3DQEBCwUAA4IB
| AQAaOzZUPC3XTVsnny9y2JanZuxuzfwSjWyb93JBW1D8+FtZXVVgMxoN6a0IafZ2
| +hfGzNtYA/V/FMELODZ1WYY6fYNpzCXAs/32zjhCvXKJkPUO6agX6BxapKQXuo8E
| 85wWsU7qR9p+rL87mVyaHeIc4NtDzWyHIShkL9ufOKixYAgK1XxgETl2y62G0cF9
| obVlinSx5rxMVE5vcu4bq722DNlcDotn9VduTa01dh/YQDfAyKOaUVr+pABVFQwI
| u8+FC3bAUuxPwIU/TSIVVmFHKUEZoPNBpwtL5B8REyCTN4mDoC22wtyuTs/B8hLk
| +aY7wH+RlikXKQ28onz4vT7G
|_-----END CERTIFICATE-----
|_ssl-date: 2022-07-14T10:00:30+00:00; +43m43s from scanner time.
60000/tcp open  unknown       syn-ack
| fingerprint-strings: 
|   FourOhFourRequest: 
|     HTTP/1.1 404 Not Found
|     Content-Type: text/html
|     Content-Length: 177
|     Connection: Keep-Alive
|     <HTML><HEAD><TITLE>404 Not Found</TITLE></HEAD><BODY><H1>404 Not Found</H1>The requested URL nice%20ports%2C/Tri%6Eity.txt%2ebak was not found on this server.<P></BODY></HTML>
|   GetRequest: 
|     HTTP/1.1 401 Access Denied
|     Content-Type: text/html
|     Content-Length: 144
|     Connection: Keep-Alive
|     WWW-Authenticate: Digest realm="ThinVNC", qop="auth", nonce="58iP9oPa5UDo2UcCg9rlQA==", opaque="m2yqFi2usv3AY2yatYSTRmyNPAplB8C1oC"
|_    <HTML><HEAD><TITLE>401 Access Denied</TITLE></HEAD><BODY><H1>401 Access Denied</H1>The requested URL requires authorization.<P></BODY></HTML>
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port60000-TCP:V=7.80%I=7%D=7/14%Time=62CFDE81%P=x86_64-alpine-linux-mus
SF:l%r(GetRequest,179,"HTTP/1\.1\x20401\x20Access\x20Denied\r\nContent-Typ
SF:e:\x20text/html\r\nContent-Length:\x20144\r\nConnection:\x20Keep-Alive\
SF:r\nWWW-Authenticate:\x20Digest\x20realm=\"ThinVNC\",\x20qop=\"auth\",\x
SF:20nonce=\"58iP9oPa5UDo2UcCg9rlQA==\",\x20opaque=\"m2yqFi2usv3AY2yatYSTR
SF:myNPAplB8C1oC\"\r\n\r\n<HTML><HEAD><TITLE>401\x20Access\x20Denied</TITL
SF:E></HEAD><BODY><H1>401\x20Access\x20Denied</H1>The\x20requested\x20URL\
SF:x20\x20requires\x20authorization\.<P></BODY></HTML>\r\n")%r(FourOhFourR
SF:equest,111,"HTTP/1\.1\x20404\x20Not\x20Found\r\nContent-Type:\x20text/h
SF:tml\r\nContent-Length:\x20177\r\nConnection:\x20Keep-Alive\r\n\r\n<HTML
SF:><HEAD><TITLE>404\x20Not\x20Found</TITLE></HEAD><BODY><H1>404\x20Not\x2
SF:0Found</H1>The\x20requested\x20URL\x20nice%20ports%2C/Tri%6Eity\.txt%2e
SF:bak\x20was\x20not\x20found\x20on\x20this\x20server\.<P></BODY></HTML>\r
SF:\n");
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 43m43s, deviation: 0s, median: 43m43s
| p2p-conficker: 
|   Checking for Conficker.C or higher...
|   Check 1 (port 48578/tcp): CLEAN (Couldn't connect)
|   Check 2 (port 56639/tcp): CLEAN (Couldn't connect)
|   Check 3 (port 63404/udp): CLEAN (Timeout)
|   Check 4 (port 43022/udp): CLEAN (Failed to receive data)
|_  0/4 checks are positive: Host is CLEAN or ports are blocked
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2022-07-14T10:00:05
|_  start_date: N/A

NSE: Script Post-scanning.
NSE: Starting runlevel 1 (of 3) scan.
Initiating NSE at 09:16
Completed NSE at 09:16, 0.00s elapsed
NSE: Starting runlevel 2 (of 3) scan.
Initiating NSE at 09:16
Completed NSE at 09:16, 0.00s elapsed
NSE: Starting runlevel 3 (of 3) scan.
Initiating NSE at 09:16
Completed NSE at 09:16, 0.00s elapsed
Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 152.47 seconds
```

# Method to solve the challenge
This challenge was stated easy but it is abit difficult compared to the previous machine that i have done. In this machine, the web server was hosted at port 60000 but it required username and password to further access.
<br>
![](/assets/img/posts/2022-07-14-ElMariachi-PC-1.png) 
<br>
After looking around for my tips, i found out a great tip which is `ThinVnc`. This is a great tips for me as it is vulnerable. `ThinVnc` is vulnerable and it will somehow return the username and password needed to login into the web server.
<br>
![](/assets/img/posts/2022-07-14-ElMariachi-PC-2.png)
<br>
By intercepting the request and resend it with burpsuite, we managed to get the credentials `desperado:TooComplicatedToGuessMeAhahahahahahahh` Since we have the credentials, we can continue to move on and login into the web server. 
<br>
![](/assets/img/posts/2022-07-14-ElMariachi-PC-3.png)
<br>
After login into the web server, i have no idea what to do and i just clicked connect button. It appears to be a rdp which spawn a computer inside the browser. The flag is just inside the browser. 
<br>
![](/assets/img/posts/2022-07-14-ElMariachi-PC-4.png)
<br>
