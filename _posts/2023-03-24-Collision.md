---
title: Collision
categories: [writeups,pwnable.kr]
tags: [pwn]
---

# Table of Content
- [Table of Content](#table-of-content)
- [Description](#description)
- [Program Code](#program-code)
- [Solution](#solution)
  - [Solution.py](#solutionpy)
- [References](#references)

# Description
>Daddy told me about cool MD5 hash collision today.<br>
>I wanna do something like that too!

# Program Code
```c
#include <stdio.h>
#include <string.h>
unsigned long hashcode = 0x21DD09EC;
unsigned long check_password(const char* p){
        int* ip = (int*)p;
        int i;
        int res=0;
        for(i=0; i<5; i++){
                res += ip[i];
        }
        return res;
}

int main(int argc, char* argv[]){
        if(argc<2){
                printf("usage : %s [passcode]\n", argv[0]);
                return 0;
        }
        if(strlen(argv[1]) != 20){
                printf("passcode length should be 20 bytes\n");
                return 0;
        }

        if(hashcode == check_password( argv[1] )){
                system("/bin/cat flag");
                return 0;
        }
        else
                printf("wrong passcode.\n");
        return 0;
}
```
# Solution
- First of all, the code is trying to check if **`hashcode`** is same as the result of **`check_password`** function
- argument input must be exactly 20
- the **`check_password`** function is trying to make the input into pointer and add into **`res`** 5 times based on the input.
- Example of code with helps of printing

```c
#include <stdio.h>
#include <string.h>
unsigned long hashcode = 0x21DD09EC; 
unsigned long check_password(const char* p){
        int* ip = (int*)p;
        int i;
        int res=0;
        for(i=0; i<5; i++){
                res += ip[i];
                printf("ip = %p\n",ip[i]); //newly added
        }
        printf("res = %p\n",res); //newly added
        return res;
}

int main(int argc, char* argv[]){
        if(argc<2){
                printf("usage : %s [passcode]\n", argv[0]);
                return 0;
        }
        if(strlen(argv[1]) != 20){
                printf("passcode length should be 20 bytes\n");
                return 0;
        }

        if(hashcode == check_password( argv[1] )){
                system("/bin/cat flag");
                return 0;
        }
        else
                printf("wrong passcode.\n");
        return 0;
}
```
- Output of the example code

```bash
$ ./test.out AAAABBBBCCCCAAAABBBB
ip = 0x41414141
ip = 0x42424242
ip = 0x43434343
ip = 0x41414141
ip = 0x42424242
res = 0x4a4a4a49
wrong passcode.
```

- With the help of printing output, it is easier to understand as we just need to make sure the **`res`** to be same with **`hashcode`**
- In order to do it, we will just dividing **`hashcode`** into 5 which will have remainder
- after dividing it, multiply by 4 and take **`hashcode`** to minus to value
- the remaining output will be the fifth character
- payload should be something like this : **`hashcode/4 + hashcode/4 + hashcode/4 + hashcode/4 + remaining_output`**
- It is hard to get the real output as it might not be ascii character
- we can use pwntools to help and get the flag

## Solution.py
```python
from pwn import *

hashcode = 0x21DD09EC
divided = hashcode/5
total = divided*4
remaining = hashcode - total

payload = p32(divided)
payload = payload * 4
payload = payload + p32(remaining)

p = process(["/home/col/col",payload])
print(p.recvall())
p.close()
```

# References
- [https://www.w3schools.com/c/c_pointers.php](https://www.w3schools.com/c/c_pointers.php)
- [https://www.jdoodle.com/c-online-compiler/](https://www.jdoodle.com/c-online-compiler/)
- [Youtube Walkthrough by me](https://youtu.be/tBjats_SduE)