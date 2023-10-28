---
title: Function Discovery Quest
categories: [ctf]
tags:
  -  
---

## Challenge Description

- Challenge Name: Function Discovery Quest

> I did not manage to get the description but I managed to download the challenge and save it in my laptop

## Explanation

> I did not manage to solve this challenge during the CTF
{: .prompt-info }

To start off, I tried to analyze a bit of the file by using `file` command

```bash
file rev_breakme
rev_breakme: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=d707d32515fe7aaa7ece41602bd6a26d0111ebb2, for GNU/Linux 3.2.0, not stripped
```

I literally know nothing about all the information given other that it is a 64-bit executable file which could run in linux. After that, I tried to run the file to get any result.

```bash
./rev_breakme                    
Hold up!
zsh: segmentation fault  ./rev_breakme
```

The result shows segmentation fault which somehow means that there is some error in the executable file. I then used `gdb` to analyze and debug it.

```bash
gef➤  disass main
Dump of assembler code for function main:
   0x00000000000012a8 <+0>:     push   rbp
   0x00000000000012a9 <+1>:     mov    rbp,rsp
   0x00000000000012ac <+4>:     sub    rsp,0x30
   0x00000000000012b0 <+8>:     lea    rax,[rip+0xd50]        # 0x2007
   0x00000000000012b7 <+15>:    mov    rdi,rax
   0x00000000000012ba <+18>:    call   0x1040 <puts@plt>
   0x00000000000012bf <+23>:    int    0x4
   0x00000000000012c1 <+25>:    lea    rax,[rip+0xd48]        # 0x2010
   0x00000000000012c8 <+32>:    mov    rdi,rax
   0x00000000000012cb <+35>:    call   0x1040 <puts@plt>
   0x00000000000012d0 <+40>:    mov    QWORD PTR [rbp-0x30],0x624aac1c
   0x00000000000012d8 <+48>:    mov    QWORD PTR [rbp-0x28],0x6549a223
   0x00000000000012e0 <+56>:    mov    QWORD PTR [rbp-0x20],0x5e47c2c8
   0x00000000000012e8 <+64>:    mov    QWORD PTR [rbp-0x18],0x7407f1c6
   0x00000000000012f0 <+72>:    mov    QWORD PTR [rbp-0x10],0x30e10e4e
   0x00000000000012f8 <+80>:    mov    QWORD PTR [rbp-0x8],0x7c747e
   0x0000000000001300 <+88>:    lea    rax,[rip+0xd1e]        # 0x2025
   0x0000000000001307 <+95>:    mov    rdi,rax
   0x000000000000130a <+98>:    call   0x1040 <puts@plt>
   0x000000000000130f <+103>:   lea    rax,[rip+0xd21]        # 0x2037
   0x0000000000001316 <+110>:   mov    rdi,rax
   0x0000000000001319 <+113>:   call   0x1040 <puts@plt>
   0x000000000000131e <+118>:   lea    rax,[rip+0xd1c]        # 0x2041
   0x0000000000001325 <+125>:   mov    rdi,rax
   0x0000000000001328 <+128>:   call   0x1040 <puts@plt>
   0x000000000000132d <+133>:   lea    rax,[rbp-0x30]
   0x0000000000001331 <+137>:   mov    esi,0x6
   0x0000000000001336 <+142>:   mov    rdi,rax
   0x0000000000001339 <+145>:   call   0x1169 <printFlag>
   0x000000000000133e <+150>:   mov    eax,0x0
   0x0000000000001343 <+155>:   leave
   0x0000000000001344 <+156>:   ret
End of assembler dump.
```

Based on the assembler code in main function of the executable file, there is a `int 0x4` which is a software interrupt instruction. To verfiy again, I tried to rerun the executable file in gdb and the result shows that it will stop after the software interrupt instruction.

```bash
[ Legend: Modified register | Code | Heap | Stack | String ]
──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── registers ────
$rax   : 0x9               
$rbx   : 0x00007fffffffdf58  →  0x00007fffffffe2c4  →  "/root/ctf/itrexc/rev_breakme"
$rcx   : 0x00007ffff7ec1b00  →  0x5877fffff0003d48 ("H="?)
$rdx   : 0x0               
$rsp   : 0x00007fffffffde10  →  0x0000000000000000
$rbp   : 0x00007fffffffde40  →  0x0000000000000001
$rsi   : 0x00005555555592a0  →  "Hold up!\n"
$rdi   : 0x00007ffff7f9fa30  →  0x0000000000000000
$rip   : 0x00005555555552c1  →  <main+25> lea rax, [rip+0xd48]        # 0x555555556010
$r8    : 0x400             
$r9    : 0x410             
$r10   : 0x1000            
$r11   : 0x202             
$r12   : 0x0               
$r13   : 0x00007fffffffdf68  →  0x00007fffffffe2e1  →  "COLORFGBG=15;0"
$r14   : 0x0000555555557dd8  →  0x0000555555555120  →  <__do_global_dtors_aux+0> endbr64 
$r15   : 0x00007ffff7ffd000  →  0x00007ffff7ffe2d0  →  0x0000555555554000  →   jg 0x555555554047
$eflags: [zero carry parity adjust sign trap INTERRUPT direction overflow resume virtualx86 identification]
$cs: 0x33 $ss: 0x2b $ds: 0x00 $es: 0x00 $fs: 0x00 $gs: 0x00 
──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── stack ────
0x00007fffffffde10│+0x0000: 0x0000000000000000   ← $rsp
0x00007fffffffde18│+0x0008: 0x0000000000000000
0x00007fffffffde20│+0x0010: 0x0000000000000000
0x00007fffffffde28│+0x0018: 0x0000000000000000
0x00007fffffffde30│+0x0020: 0x0000000000000000
0x00007fffffffde38│+0x0028: 0x0000000000000000
0x00007fffffffde40│+0x0030: 0x0000000000000001   ← $rbp
0x00007fffffffde48│+0x0038: 0x00007ffff7df16ca  →  <__libc_start_call_main+122> mov edi, eax
────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── code:x86:64 ────
   0x5555555552b7 <main+15>        mov    rdi, rax
   0x5555555552ba <main+18>        call   0x555555555040 <puts@plt>
   0x5555555552bf <main+23>        int    0x4
 → 0x5555555552c1 <main+25>        lea    rax, [rip+0xd48]        # 0x555555556010
   0x5555555552c8 <main+32>        mov    rdi, rax
   0x5555555552cb <main+35>        call   0x555555555040 <puts@plt>
   0x5555555552d0 <main+40>        mov    QWORD PTR [rbp-0x30], 0x624aac1c
   0x5555555552d8 <main+48>        mov    QWORD PTR [rbp-0x28], 0x6549a223
   0x5555555552e0 <main+56>        mov    QWORD PTR [rbp-0x20], 0x5e47c2c8
────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── threads ────
[#0] Id 1, Name: "rev_breakme", stopped 0x5555555552c1 in main (), reason: SIGSEGV
──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── trace ────
[#0] 0x5555555552c1 → main()
───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
gef➤  ni

Program terminated with signal SIGSEGV, Segmentation fault.
The program no longer exists.
```

Since it stopped after that instruction, I search for ways in gdb where I could ignore or skip. There is a method where I could jump to specific address in gdb. After doing so, I manage to get the flag.

```bash
gef➤  b main
Breakpoint 1 at 0x5555555552ac
gef➤  b *main+23
Breakpoint 2 at 0x5555555552bf
gef➤  b *main+25
Breakpoint 3 at 0x5555555552c1
gef➤  r
Starting program: /root/ctf/itrexc/rev_breakme 
[Thread debugging using libthread_db enabled]
Using host libthread_db library "/lib/x86_64-linux-gnu/libthread_db.so.1".

Breakpoint 1, 0x00005555555552ac in main ()
gef➤  c
Continuing.
Hold up!

Breakpoint 2, 0x00005555555552bf in main ()
gef➤  jump *main+25
Continuing at 0x5555555552c1.
gef➤  c
Continuing.
Program continues...
hey you found me!
Congrats!
Here a flag for you....!
uisctf{f1nd_4_funct1on}
[Inferior 1 (process 48705) exited normally]

```


## Things I learned from the challenge
- **`INT` is interrupt in assembly language instruction and not integer**
- It is possible to skip or jump to certain address in gdb using `jump`
