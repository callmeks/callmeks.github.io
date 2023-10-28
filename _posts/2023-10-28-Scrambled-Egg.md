---
title: Scrambled Egg
categories:
  - CTF
tags:
  - rev
  - ITREXC 2023
---

## Challenge Description

- Challenge Name: Scrambled Egg
> Hmm, I found  this executable that should allow us access to some juicy innovation projects. But it asks for a key. Hmm, I'm stumped. Get the Key for me, please!


## Explanation

I started out by open the given file in IDA to decompile it and view the code.

![](/assets/img/posts/2023-10-28-Scrambled-Egg/2023-10-28-Scrambled-Egg-15-35-47.png)

After understanding the graph view, it seems to be checking for different character each time.

![](/assets/img/posts/2023-10-28-Scrambled-Egg/2023-10-28-Scrambled-Egg-15-36-37.png)

The character seems to be very random to me. After trying to understand the graph view, I tried to understand the exe file by reading the pseudocode view.

![](/assets/img/posts/2023-10-28-Scrambled-Egg/2023-10-28-Scrambled-Egg-15-38-43.png)

It is easier to understand in pseudocode view. Based on what I understand, the code has a do while loop where it check if `v7` has 31 character. After checking it is 31 character, it started to check the array number of `v7` with different decimal number in a random order. I tried to convert the decimal number into character and save it into the list to print it out using python.

```py
v7 = list(range(1, 32))
v7[17] = chr(68)
v7[23] = chr(114)
v7[18] = chr(95)
v7[20] = chr(95)
v7[21] = chr(53)
v7[14] = chr(56)
v7[12] = chr(65)
v7[8] = chr(110)
v7[27] = chr(76)
v7[22] = chr(67)
v7[2] = chr(115)
v7[10] = chr(99)
v7[6] = chr(123)
v7[29] = chr(100)
v7[4] = chr(116)
v7[26] = chr(56)
v7[3] = chr(99)
v7[11] = chr(114)
v7[28] = chr(51)
v7[19] = chr(100)
v7[30] = chr(125)
v7[15] = chr(76)
v7[24] = chr(97)
v7[5] = chr(102)
v7[13] = chr(77)
v7[1] = chr(105)
v7[0] = chr(117)
v7[25] = chr(77)
v7[7] = chr(117)
v7[9] = chr(53)
v7[16] = chr(51)

print("".join(v7))
```
The result of `v7` is a flag which is also the key

```bash
python solution.py
uisctf{un5crAM8L3D_d_5CraM8L3d}

Hold Up!. Enter the key to access secret Innovation Projects: uisctf{un5crAM8L3D_d_5CraM8L3d}
Answer Received!. Checking...

Key Verified. Here are unscrambled eggs for your troubles.

                       .-~-.
                     .'     '.
                    /         \
            .-~-.  :           ;
          .'     '.|           |
         /         \           :
        :           ; .-~""~-,/
        |           /`        `'.
        :          |             \
         \         |             /
          `.     .' \          .'
            `~~~`    '-.____.-'
Art Credit: jgs
Source: https://ascii.co.uk/art/egg
```

## Things I learned from the challenge

- Sometimes try use pseudocode view in IDA as it helps alot
- Trying to write script is fun but time consuming
- Debug is not needed here ~
- I'm not good at reverse 