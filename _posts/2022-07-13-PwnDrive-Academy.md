---
layout: post
title: PwnDrive Academy
subtitle: 10.150.150.11
tags: [PwnTillDawn,Easy,Windows,PHP Reverse Shell]
# date: 13/07/2022
---
# PwnDrive Academy 
- Difficulty : `easy`
- IP address : `10.150.150.11`
- Operating System : `Windows`

# Table of Content
- [Rustscan result](#rustscan-result)
- [Nmap Result](#nmap-result)
- [Method to solve the challenge](#method-to-solve-the-challenge)
- [Flag](#flag)


## Rustscan result
```
Open 10.150.150.11:21
Open 10.150.150.11:80
Open 10.150.150.11:135
Open 10.150.150.11:139
Open 10.150.150.11:443
Open 10.150.150.11:445
Open 10.150.150.11:1433
Open 10.150.150.11:3306
Open 10.150.150.11:3389
Open 10.150.150.11:49152
Open 10.150.150.11:49155
Open 10.150.150.11:49154
Open 10.150.150.11:49153
Open 10.150.150.11:49156
Open 10.150.150.11:49157
Open 10.150.150.11:49192
```
## Nmap Result
```
Nmap scan report for 10.150.150.11
Host is up, received syn-ack (0.25s latency).
Scanned at 2022-07-12 03:08:36 UTC for 124s

PORT      STATE SERVICE            REASON  VERSION
21/tcp    open  ftp                syn-ack Xlight ftpd 3.9
80/tcp    open  http               syn-ack Apache httpd 2.4.46 ((Win64) OpenSSL/1.1.1g PHP/7.4.9)
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-favicon: Unknown favicon MD5: 3345FF745865D02B994859241BCE2B36
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.46 (Win64) OpenSSL/1.1.1g PHP/7.4.9
|_http-title: PwnDrive - Your Personal Online Storage
135/tcp   open  msrpc              syn-ack Microsoft Windows RPC
139/tcp   open  netbios-ssn        syn-ack Microsoft Windows netbios-ssn
443/tcp   open  ssl/http           syn-ack Apache httpd 2.4.46 ((Win64) OpenSSL/1.1.1g PHP/7.4.9)
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-favicon: Unknown favicon MD5: 3345FF745865D02B994859241BCE2B36
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.46 (Win64) OpenSSL/1.1.1g PHP/7.4.9
|_http-title: PwnDrive - Your Personal Online Storage
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
445/tcp   open  microsoft-ds       syn-ack Windows Server 2008 R2 Enterprise 7601 Service Pack 1 microsoft-ds
1433/tcp  open  ms-sql-s           syn-ack Microsoft SQL Server 2012 11.00.2100.00; RTM
| ms-sql-ntlm-info: 
|   Target_Name: PWNDRIVE
|   NetBIOS_Domain_Name: PWNDRIVE
|   NetBIOS_Computer_Name: PWNDRIVE
|   DNS_Domain_Name: PwnDrive
|   DNS_Computer_Name: PwnDrive
|_  Product_Version: 6.1.7601
| ssl-cert: Subject: commonName=SSL_Self_Signed_Fallback
| Issuer: commonName=SSL_Self_Signed_Fallback
| Public Key type: rsa
| Public Key bits: 1024
| Signature Algorithm: sha1WithRSAEncryption
| Not valid before: 2020-08-24T13:11:13
| Not valid after:  2050-08-24T13:11:13
| MD5:   2d8f 22bc 83a0 9877 14a4 9d97 6707 fe79
| SHA-1: 4bbd 5787 7eb1 e46e f5fc 8c3a 84c0 ae68 f149 0272
| -----BEGIN CERTIFICATE-----
| MIIB+zCCAWSgAwIBAgIQL0K65z76GL1Ivdd40Q5YYjANBgkqhkiG9w0BAQUFADA7
| MTkwNwYDVQQDHjAAUwBTAEwAXwBTAGUAbABmAF8AUwBpAGcAbgBlAGQAXwBGAGEA
| bABsAGIAYQBjAGswIBcNMjAwODI0MTMxMTEzWhgPMjA1MDA4MjQxMzExMTNaMDsx
| OTA3BgNVBAMeMABTAFMATABfAFMAZQBsAGYAXwBTAGkAZwBuAGUAZABfAEYAYQBs
| AGwAYgBhAGMAazCBnzANBgkqhkiG9w0BAQEFAAOBjQAwgYkCgYEAv5sKKwZpy8UQ
| Q552Do6cumRnAGF1v64IRQdLAOF/ZWIUKU+YGes4k5n7PYLC1wqSmexkoHL8MUom
| qSqbfehglepBTd8894ZEkPo5TOUosHSWZMZVEMZj7yF2jH85CpDh+abEGDB13Fvz
| WiZL7Twm89XV34Zlr25+n3yvBD6/TwsCAwEAATANBgkqhkiG9w0BAQUFAAOBgQA4
| AhMTYh38yAqr/fveiRfxJddOQf+WosbrnlXyyGiHzpkwfUT7rXDoz02zWYf4tr+m
| UKWFnCeZCjUJWBBdHjrJEswX7gaPFn0MaaHrZIp0q21h7XVhn1iK+ewTUA+U8rri
| BQ9koXFxdF1IVvOHymd3Ki1s/Nynww6ApU0O05k1lg==
|_-----END CERTIFICATE-----
|_ssl-date: 2022-07-12T03:54:27+00:00; +43m50s from scanner time.
3306/tcp  open  mysql              syn-ack MySQL 5.5.5-10.4.14-MariaDB
| mysql-info: 
|   Protocol: 10
|   Version: 5.5.5-10.4.14-MariaDB
|   Thread ID: 1061
|   Capabilities flags: 63486
|   Some Capabilities: ODBCClient, Speaks41ProtocolNew, InteractiveClient, ConnectWithDatabase, FoundRows, IgnoreSigpipes, Support41Auth, SupportsTransactions, Speaks41ProtocolOld, DontAllowDatabaseTableColumn, SupportsCompression, IgnoreSpaceBeforeParenthesis, SupportsLoadDataLocal, LongColumnFlag, SupportsAuthPlugins, SupportsMultipleResults, SupportsMultipleStatments
|   Status: Autocommit
|   Salt: gfL*+9{"2spWC,N&nz=]
|_  Auth Plugin Name: mysql_native_password
3389/tcp  open  ssl/ms-wbt-server? syn-ack
| rdp-ntlm-info: 
|   Target_Name: PWNDRIVE
|   NetBIOS_Domain_Name: PWNDRIVE
|   NetBIOS_Computer_Name: PWNDRIVE
|   DNS_Domain_Name: PwnDrive
|   DNS_Computer_Name: PwnDrive
|   Product_Version: 6.1.7601
|_  System_Time: 2022-07-12T03:54:17+00:00
| ssl-cert: Subject: commonName=PwnDrive
| Issuer: commonName=PwnDrive
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha1WithRSAEncryption
| Not valid before: 2022-07-11T02:41:09
| Not valid after:  2023-01-10T02:41:09
| MD5:   0d56 7277 5476 05c8 148f 6ceb 2a49 3bc8
| SHA-1: 5f1e 25fa 9615 3c33 44a6 8b7c 5a07 f0ac 6312 6619
| -----BEGIN CERTIFICATE-----
| MIIC1DCCAbygAwIBAgIQFbXvOER9gKlK/PrqKEZ5EDANBgkqhkiG9w0BAQUFADAT
| MREwDwYDVQQDEwhQd25Ecml2ZTAeFw0yMjA3MTEwMjQxMDlaFw0yMzAxMTAwMjQx
| MDlaMBMxETAPBgNVBAMTCFB3bkRyaXZlMIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8A
| MIIBCgKCAQEAtNNjoFRu/vG+J+UBUhS4MVbVOHd/+j9XuUkpnexxWAn2ypPsk78P
| 6gmIsposuHr9D7+zU9c2xKsWyiTe+lq7l4sgFNuJudpUubngu8q6VnVXHiCmKlIF
| XoZ420xxY6DpYZ9zm/OADg01KVadqCIMMFNOPsh6NcuHII+z6bcrgAGZegVQ6vUm
| vVODCGVJ/LOOR0Wo8yNjOTLRRWeKLZqB9E5F0H8DSctwF6UtIz+DAtZTuT8CAg/q
| ZiM97xDcPAHLz4ViEEqr5diL7UtKG3vAyjT17hwIRceFh67Cd2a/SoRLgMM9rEDh
| xIDT990aNKl0JJkzZn0oY576sNTGpNv4TwIDAQABoyQwIjATBgNVHSUEDDAKBggr
| BgEFBQcDATALBgNVHQ8EBAMCBDAwDQYJKoZIhvcNAQEFBQADggEBAG2KIgcLsgNW
| viVDbuB68fJdXZeOFMk/9BVD22iQmR/QEq2NSlJ45pI3Mm94MCKLPKp2ITTGV1Tc
| 80Cnx1BdE2B7x4coBesDqu7UwwsBoRI8BiNy1LonuPsinNmfkIF9Valw9l0x5IHb
| kQAY+IRc9PaRoshHl10yoH49FZgv+ORZUwwzlYQuE+6ED8TdO/AZOmunhOXX3anH
| veQdzO8E7lOiGvDED6NPvsk2I1FJcdbPR6On8K9HE28hYMWx6VI+2YKxE+eqaU15
| AYE+1ZUQ/S55TSNlsQATzjwO5UflTKzH+MhdW5lD/qw25VeiXK83YusvbbL7l9A8
| TbIJVosnlrY=
|_-----END CERTIFICATE-----
|_ssl-date: 2022-07-12T03:54:27+00:00; +43m50s from scanner time.
49152/tcp open  msrpc              syn-ack Microsoft Windows RPC
49153/tcp open  msrpc              syn-ack Microsoft Windows RPC
49154/tcp open  msrpc              syn-ack Microsoft Windows RPC
49155/tcp open  msrpc              syn-ack Microsoft Windows RPC
49156/tcp open  msrpc              syn-ack Microsoft Windows RPC
49157/tcp open  msrpc              syn-ack Microsoft Windows RPC
49192/tcp open  ms-sql-s           syn-ack Microsoft SQL Server 2012 11.00.2100; RTM
| ms-sql-ntlm-info: 
|   Target_Name: PWNDRIVE
|   NetBIOS_Domain_Name: PWNDRIVE
|   NetBIOS_Computer_Name: PWNDRIVE
|   DNS_Domain_Name: PwnDrive
|   DNS_Computer_Name: PwnDrive
|_  Product_Version: 6.1.7601
| ssl-cert: Subject: commonName=SSL_Self_Signed_Fallback
| Issuer: commonName=SSL_Self_Signed_Fallback
| Public Key type: rsa
| Public Key bits: 1024
| Signature Algorithm: sha1WithRSAEncryption
| Not valid before: 2020-08-24T13:11:13
| Not valid after:  2050-08-24T13:11:13
| MD5:   2d8f 22bc 83a0 9877 14a4 9d97 6707 fe79
| SHA-1: 4bbd 5787 7eb1 e46e f5fc 8c3a 84c0 ae68 f149 0272
| -----BEGIN CERTIFICATE-----
| MIIB+zCCAWSgAwIBAgIQL0K65z76GL1Ivdd40Q5YYjANBgkqhkiG9w0BAQUFADA7
| MTkwNwYDVQQDHjAAUwBTAEwAXwBTAGUAbABmAF8AUwBpAGcAbgBlAGQAXwBGAGEA
| bABsAGIAYQBjAGswIBcNMjAwODI0MTMxMTEzWhgPMjA1MDA4MjQxMzExMTNaMDsx
| OTA3BgNVBAMeMABTAFMATABfAFMAZQBsAGYAXwBTAGkAZwBuAGUAZABfAEYAYQBs
| AGwAYgBhAGMAazCBnzANBgkqhkiG9w0BAQEFAAOBjQAwgYkCgYEAv5sKKwZpy8UQ
| Q552Do6cumRnAGF1v64IRQdLAOF/ZWIUKU+YGes4k5n7PYLC1wqSmexkoHL8MUom
| qSqbfehglepBTd8894ZEkPo5TOUosHSWZMZVEMZj7yF2jH85CpDh+abEGDB13Fvz
| WiZL7Twm89XV34Zlr25+n3yvBD6/TwsCAwEAATANBgkqhkiG9w0BAQUFAAOBgQA4
| AhMTYh38yAqr/fveiRfxJddOQf+WosbrnlXyyGiHzpkwfUT7rXDoz02zWYf4tr+m
| UKWFnCeZCjUJWBBdHjrJEswX7gaPFn0MaaHrZIp0q21h7XVhn1iK+ewTUA+U8rri
| BQ9koXFxdF1IVvOHymd3Ki1s/Nynww6ApU0O05k1lg==
|_-----END CERTIFICATE-----
|_ssl-date: 2022-07-12T03:54:27+00:00; +43m49s from scanner time.
Service Info: OSs: Windows, Windows Server 2008 R2 - 2012; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 1h30m29s, deviation: 2h20m00s, median: 43m49s
| ms-sql-info: 
|   10.150.150.11:1433: 
|     Version: 
|       name: Microsoft SQL Server 2012 RTM
|       number: 11.00.2100.00
|       Product: Microsoft SQL Server 2012
|       Service pack level: RTM
|       Post-SP patches applied: false
|_    TCP port: 1433
| nbstat: NetBIOS name: PWNDRIVE, NetBIOS user: <unknown>, NetBIOS MAC: 00:0c:29:89:87:cb (VMware)
| Names:
|   PWNDRIVE<20>         Flags: <unique><active>
|   PWNDRIVE<00>         Flags: <unique><active>
|   WORKGROUP<00>        Flags: <group><active>
| Statistics:
|   00 0c 29 89 87 cb 00 00 00 00 00 00 00 00 00 00 00
|   00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
|_  00 00 00 00 00 00 00 00 00 00 00 00 00 00
| p2p-conficker: 
|   Checking for Conficker.C or higher...
|   Check 1 (port 52298/tcp): CLEAN (Couldn't connect)
|   Check 2 (port 44474/tcp): CLEAN (Couldn't connect)
|   Check 3 (port 8468/udp): CLEAN (Failed to receive data)
|   Check 4 (port 22295/udp): CLEAN (Timeout)
|_  0/4 checks are positive: Host is CLEAN or ports are blocked
| smb-os-discovery: 
|   OS: Windows Server 2008 R2 Enterprise 7601 Service Pack 1 (Windows Server 2008 R2 Enterprise 6.1)
|   OS CPE: cpe:/o:microsoft:windows_server_2008::sp1
|   Computer name: PwnDrive
|   NetBIOS computer name: PWNDRIVE\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2022-07-11T20:54:15-07:00
| smb-security-mode: 
|   account_used: <blank>
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2022-07-12T03:54:16
|_  start_date: 2020-08-24T13:11:20

NSE: Script Post-scanning.
NSE: Starting runlevel 1 (of 3) scan.
Initiating NSE at 03:10
Completed NSE at 03:10, 0.00s elapsed
NSE: Starting runlevel 2 (of 3) scan.
Initiating NSE at 03:10
Completed NSE at 03:10, 0.00s elapsed
NSE: Starting runlevel 3 (of 3) scan.
Initiating NSE at 03:10
Completed NSE at 03:10, 0.00s elapsed
Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 130.10 seconds
```
  
# Method to solve the challenge
- For starters, always look out for port 80 which is website of the ip address.
- ![image](https://user-images.githubusercontent.com/88197307/178403372-eb197fb4-ec11-4f76-a401-d27c88990a90.png)
- Since there is a login page, look into it and try to get access into the website by using SQL injection, default credentials or simple credentials such as `admin:admin`
- In this challenge, SQL injection is hard to perform as it detect `=` sign.
- ![image](https://user-images.githubusercontent.com/88197307/178403643-42017034-c54e-4374-ae5e-04b5fda13762.png)
- Since simple SQL injection is unable to perform, try the other easy method first before moving on to some harder technique.
- Simple credentials `admin:admin` is able to login into the system
- ![image](https://user-images.githubusercontent.com/88197307/178403959-218b3b91-c8e3-4f6c-84a4-ac1e5449d532.png)
- In this page, the admin is allowed to upload images to the web server. 
- Since it is able to upload images, it might have a chance to spawn reverse shell by uploading a malicious php file into the web server.
- ![image](https://user-images.githubusercontent.com/88197307/178404365-b0de533b-404f-4c10-9446-5b3bcaaff57f.png)
- The malicious php file was able to upload successfully.
- Since it is uploaded, clicking the php file to make it run and it should provides a reverse shell.
- *remember to use `nc -nvlp [port]` when setting up for reverse shell
- The only thing the remains are the paths which will redirected us to the uploaded php file.
- When user clicked the view button from the personal files, it will redirected us to a page which show us the source code of the file but will not triggered.
- ![image](https://user-images.githubusercontent.com/88197307/178405128-9bd1401d-e4dc-42d3-9d4d-cb9d039548f8.png)
- Based on the url, it is redirecting us to another directory which might be the place that saves the uploaded php file. 
- ![image](https://user-images.githubusercontent.com/88197307/178405232-2dc38260-dc31-44cf-8537-a0f151ec06e0.png)
- The php file was there when the path has changed.
- ![image](https://user-images.githubusercontent.com/88197307/178405331-94a42f38-ef84-4b36-a0dd-cb0639a4f09b.png)
- ![image](https://user-images.githubusercontent.com/88197307/178405365-e30c36e3-751d-489c-9876-5216a42b89fc.png)
- The php file was unable to return a reverse shell.
- This is because the php reverse shell is specifically created for linux machine and windows machines will not able to spawn a reverse shell.
- The solution to this is either chaning the source code of the php file or each online for another [php reverse shell for windows](https://github.com/Dhayalanb/windows-php-reverse-shell/blob/master/Reverse%20Shell.php).
- After uploading it the same method and run the php file, the reverse shell was managed to spawn.
- ![image](https://user-images.githubusercontent.com/88197307/178405910-28b399a9-7250-4baa-97cb-dd70d6e7cc7e.png)
- The reverse shell is spawn for user `authority\system` which is also root privilege and therefore no privilege escalation is needed.

# Flag
- Flag 1: 
- ![image](https://user-images.githubusercontent.com/88197307/178406206-8fb5414f-d76d-41a1-a069-13229ae5df8f.png)











