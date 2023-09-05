---
title: Extracting QR from android
categories: [CTF]
tags: [Forensic]
---

## Step 1: understanding how thing works

The provided file is a directory that contains a lot of directories and files which I have no idea where to start. Since there's nothing for me to start with, I just try to search for flag or the flag format provided.

```bash
find . -name "*flag*"
./media/0/DCIM/flag.jpg
./user_de/0/com.google.android.gms/shared_prefs/gms_chimera_phenotype_flags.xml
./data/com.google.android.partnersetup/shared_prefs/partnersetup_phenotype_flags.xml
./data/com.android.vending/files/experiment-flags-regular-null-account
./data/com.android.vending/files/experiment-flags-process-stable
./data/com.google.android.gms/shared_prefs/gms_chimera_phenotype_flags.xml
./data/com.google.android.gms/shared_prefs/google_ads_flags.xml
./data/com.google.android.gms/shared_prefs/google_sdk_flags.xml
./server_configurable_flags
```

Based on the result, there is one interesting **"flag"** which is `flag.jpg`. 

## Step 2: investigating the directory and the image

lets have a look at the directory as well as the images.

```bash
ls -la media/0/DCIM 
total 14592
drwxr-xr-x  2 root root    4096 Aug 27 16:40 .
drwxr-xr-x 12 root root    4096 Jun 22 01:17 ..
-rw-r--r--  1 root root 8252866 Jun 16 17:05 0c9cbf5da66bb723a6e4280825075dab.mp4
-rw-r--r--  1 root root   85435 Aug 20 00:29 1679091c5a880faf6fb5e6087eb1b2dc.jpeg
-rw-r--r--  1 root root  112358 Aug 20 00:34 1f0e3dad99908345f7439f8ffabdffc4.jpg
-rw-r--r--  1 root root  150274 Aug 20 00:36 1ff1de774005f8da13f42943881c655f.jpg
-rw-r--r--  1 root root  477296 Jun 18 13:15 2c896f081dbbe942282f1c4776044d77.mp4
-rw-r--r--  1 root root   88662 Aug 20 00:35 37693cfc748049e45d87b8c7d8b9aacd.jpeg
-rw-r--r--  1 root root   71966 Aug 20 00:35 3c59dc048e8850243be8079a5c74d079.jpeg
-rw-r--r--  1 root root  180413 Aug 20 00:31 45c48cce2e2d7fbdea1afc51c7c6ad26.jpeg
-rw-r--r--  1 root root  223953 Jun 16 16:56 5b2e79e7b3d6b16efe74e2547fddf092.mp4
-rw-r--r--  1 root root   39032 Aug 20 00:31 6512bd43d9caa6e02c990b0a82652dca.jpeg
-rw-r--r--  1 root root   51099 Aug 20 00:34 6f4922f45568161a8cdf4ad2299f6d23.jpg
-rw-r--r--  1 root root   46113 Aug 20 00:34 70efdf2ec9b086079795c442636b55fb.jpg
-rw-r--r--  1 root root   91653 Aug 20 00:29 8f14e45fceea167a5a36dedd4bea2543.jpeg
-rw-r--r--  1 root root  271054 Aug 20 00:35 98f13708210194c475687be6106a3b84.jpeg
-rw-r--r--  1 root root  302022 Aug 20 00:33 9bf31c7ff062936a96d3c8bd1f8f2ff3.jpeg
-rw-r--r--  1 root root   60615 Aug 20 00:28 a87ff679a2f3e71d9181a67b7542122c.jpeg
-rw-r--r--  1 root root   77073 Aug 20 00:33 aab3238922bcc25a6f606eb525ffdc56.jpg
-rw-r--r--  1 root root  115334 Aug 20 00:35 b6d767d2f8ed5d21a44b0e5886680cb9.jpeg
-rw-r--r--  1 root root  183150 Aug 20 00:32 c20ad4d76fe97759aa27a0c99bff6710.jpeg
-rw-r--r--  1 root root   55272 Aug 20 00:26 c4ca4238a0b923820dcc509a6f75849b.jpg
-rw-r--r--  1 root root 3024240 Aug 20 00:32 c51ce410c124a10e0db5e4b97fc2af39.jpeg
-rw-r--r--  1 root root  151432 Aug 20 00:33 c74d97b01eae257e44aa9d5bade97baf.jpeg
-rw-r--r--  1 root root   60615 Aug 20 00:27 c81e728d9d4c2f636f067f89cc14862c.jpeg
-rw-r--r--  1 root root   85316 Aug 20 00:30 c9f0f895fb98ab9159f51fd0297e236d.jpeg
-rw-r--r--  1 root root  172537 Aug 20 00:31 d3d9446802a44259755d38e6d163e820.jpeg
-rw-r--r--  1 root root  137846 Aug 20 00:29 e4da3b7fbbce2345d7772b0674a318d5.jpeg
-rw-r--r--  1 root root  153235 Aug 20 00:28 eccbc87e4b5ce2fe28308fd9f2a7baf3.jpeg
-rw-r--r--  1 root root  134973 Aug 12 01:08 f842e5e8cdcc8b6f2c7e0ed73499e7ca.jpeg
-rw-r--r--  1 root root   24788 Aug 12 01:14 flag.jpg
```

![](/assets/img/posts/2023-09-05-Extracting-QR-from-android/2023-09-05-Extracting-QR-from-android-21-00-02.png)

Based on the result, there alot of images in the same directory and the `flag.jpg` is just a rabbit hole. 

## Step 3: Check file name 

The image filename in the same directory looks suspicious as all names has the same length and they looks like hashes. Since I have no idea how to move on, lets just check the filename with any hash identifier.

```bash
hash-identifier                                             
   #########################################################################
   #     __  __                     __           ______    _____           #
   #    /\ \/\ \                   /\ \         /\__  _\  /\  _ `\         #
   #    \ \ \_\ \     __      ____ \ \ \___     \/_/\ \/  \ \ \/\ \        #
   #     \ \  _  \  /'__`\   / ,__\ \ \  _ `\      \ \ \   \ \ \ \ \       #
   #      \ \ \ \ \/\ \_\ \_/\__, `\ \ \ \ \ \      \_\ \__ \ \ \_\ \      #
   #       \ \_\ \_\ \___ \_\/\____/  \ \_\ \_\     /\_____\ \ \____/      #
   #        \/_/\/_/\/__/\/_/\/___/    \/_/\/_/     \/_____/  \/___/  v1.2 #
   #                                                             By Zion3R #
   #                                                    www.Blackploit.com #
   #                                                   Root@Blackploit.com #
   #########################################################################
--------------------------------------------------
 HASH: 0c9cbf5da66bb723a6e4280825075dab

Possible Hashs:
[+] MD5
[+] Domain Cached Credentials - MD4(MD4(($pass)).(strtolower($username)))
```

Based on the result, it shows that the filename a MD5 hashes.

## Step 4: gather the file name/hashes

Since the filename is a md5 hashes, lets gather the filename together.

```bash
ls -l | rev |  cut -d " " -f 1 |rev | head -n -1 | tail -n +2 | cut -d "." -f 1
0c9cbf5da66bb723a6e4280825075dab
1679091c5a880faf6fb5e6087eb1b2dc
1f0e3dad99908345f7439f8ffabdffc4
1ff1de774005f8da13f42943881c655f
2c896f081dbbe942282f1c4776044d77
37693cfc748049e45d87b8c7d8b9aacd
3c59dc048e8850243be8079a5c74d079
45c48cce2e2d7fbdea1afc51c7c6ad26
5b2e79e7b3d6b16efe74e2547fddf092
6512bd43d9caa6e02c990b0a82652dca
6f4922f45568161a8cdf4ad2299f6d23
70efdf2ec9b086079795c442636b55fb
8f14e45fceea167a5a36dedd4bea2543
98f13708210194c475687be6106a3b84
9bf31c7ff062936a96d3c8bd1f8f2ff3
a87ff679a2f3e71d9181a67b7542122c
aab3238922bcc25a6f606eb525ffdc56
b6d767d2f8ed5d21a44b0e5886680cb9
c20ad4d76fe97759aa27a0c99bff6710
c4ca4238a0b923820dcc509a6f75849b
c51ce410c124a10e0db5e4b97fc2af39
c74d97b01eae257e44aa9d5bade97baf
c81e728d9d4c2f636f067f89cc14862c
c9f0f895fb98ab9159f51fd0297e236d
d3d9446802a44259755d38e6d163e820
e4da3b7fbbce2345d7772b0674a318d5
eccbc87e4b5ce2fe28308fd9f2a7baf3
f842e5e8cdcc8b6f2c7e0ed73499e7ca
```

## Step 5: Crack the hashes

We could try to crack the hashes and see the result.

```bash
hashcat -a 3 -m 0 temp --show
1679091c5a880faf6fb5e6087eb1b2dc:6
1f0e3dad99908345f7439f8ffabdffc4:19
1ff1de774005f8da13f42943881c655f:24
37693cfc748049e45d87b8c7d8b9aacd:23
3c59dc048e8850243be8079a5c74d079:21
45c48cce2e2d7fbdea1afc51c7c6ad26:9
6512bd43d9caa6e02c990b0a82652dca:11
6f4922f45568161a8cdf4ad2299f6d23:18
70efdf2ec9b086079795c442636b55fb:17
8f14e45fceea167a5a36dedd4bea2543:7
98f13708210194c475687be6106a3b84:20
9bf31c7ff062936a96d3c8bd1f8f2ff3:15
a87ff679a2f3e71d9181a67b7542122c:4
aab3238922bcc25a6f606eb525ffdc56:14
b6d767d2f8ed5d21a44b0e5886680cb9:22
c20ad4d76fe97759aa27a0c99bff6710:12
c4ca4238a0b923820dcc509a6f75849b:1
c51ce410c124a10e0db5e4b97fc2af39:13
c74d97b01eae257e44aa9d5bade97baf:16
c81e728d9d4c2f636f067f89cc14862c:2
c9f0f895fb98ab9159f51fd0297e236d:8
d3d9446802a44259755d38e6d163e820:10
e4da3b7fbbce2345d7772b0674a318d5:5
eccbc87e4b5ce2fe28308fd9f2a7baf3:3
```

After cracking the hashes, the results shows numbers which do not make any sense. Lets try to check if there is anything interesting in the file.

## Step 6: Analyzing all image file wich md5 hashes as file name

To anaylze the image file, there's a lot of tools that could be used. After trying out a few, there's one tool that provide interesting output.

```bash
exiftool 45c48cce2e2d7fbdea1afc51c7c6ad26.jpeg 
ExifTool Version Number         : 12.64
File Name                       : 45c48cce2e2d7fbdea1afc51c7c6ad26.jpeg
Directory                       : .
File Size                       : 180 kB
File Modification Date/Time     : 2023:08:20 00:31:01+08:00
File Access Date/Time           : 2023:09:03 20:17:38+08:00
File Inode Change Date/Time     : 2023:09:03 20:09:04+08:00
File Permissions                : -rw-r--r--
File Type                       : JPEG
File Type Extension             : jpg
MIME Type                       : image/jpeg
JFIF Version                    : 1.01
Resolution Unit                 : None
X Resolution                    : 72
Y Resolution                    : 72
Exif Byte Order                 : Big-endian (Motorola, MM)
Color Space                     : sRGB
Exif Image Width                : 640
Exif Image Height               : 953
Current IPTC Digest             : d41d8cd98f00b204e9800998ecf8427e
IPTC Digest                     : d41d8cd98f00b204e9800998ecf8427e
Comment                         : M8xVZsxVmcxVzMxV/8yAAMyAM8yAZsyAmcyAzMyA/8yqAMyqM8yqZsyqmcyqzMyq/8zVAMzVM8zVZszVmczVzMzV/8z/A
Image Width                     : 640
Image Height                    : 953
Encoding Process                : Baseline DCT, Huffman coding
Bits Per Sample                 : 8
Color Components                : 3
Y Cb Cr Sub Sampling            : YCbCr4:2:0 (2 2)
Image Size                      : 640x953
Megapixels                      : 0.610
```

Based on the result from `exiftool`, there is a comment section with a bunch of random text. After trying out with a few image file, not all have random text in comment. Only the file name that is able to be cracked contains comment. 

## Step 7: Extracting the random text from comment

Since only certain image file have comment, the next thing is to just get all the comment. Since some of the file name can be cracked and the result is a number, maybe the comment is based on the number and something interesting might appear. I created a script to help reorder the image order based on the cracked value and print the comment accordingly.

```bash
#!/bin/bash

strings=(
  "1679091c5a880faf6fb5e6087eb1b2dc:6"
  "1f0e3dad99908345f7439f8ffabdffc4:19"
  "1ff1de774005f8da13f42943881c655f:24"
  "37693cfc748049e45d87b8c7d8b9aacd:23"
  "3c59dc048e8850243be8079a5c74d079:21"
  "45c48cce2e2d7fbdea1afc51c7c6ad26:9"
  "6512bd43d9caa6e02c990b0a82652dca:11"
  "6f4922f45568161a8cdf4ad2299f6d23:18"
  "70efdf2ec9b086079795c442636b55fb:17"
  "8f14e45fceea167a5a36dedd4bea2543:7"
  "98f13708210194c475687be6106a3b84:20"
  "9bf31c7ff062936a96d3c8bd1f8f2ff3:15"
  "a87ff679a2f3e71d9181a67b7542122c:4"
  "aab3238922bcc25a6f606eb525ffdc56:14"
  "b6d767d2f8ed5d21a44b0e5886680cb9:22"
  "c20ad4d76fe97759aa27a0c99bff6710:12"
  "c4ca4238a0b923820dcc509a6f75849b:1"
  "c51ce410c124a10e0db5e4b97fc2af39:13"
  "c74d97b01eae257e44aa9d5bade97baf:16"
  "c81e728d9d4c2f636f067f89cc14862c:2"
  "c9f0f895fb98ab9159f51fd0297e236d:8"
  "d3d9446802a44259755d38e6d163e820:10"
  "e4da3b7fbbce2345d7772b0674a318d5:5"
  "eccbc87e4b5ce2fe28308fd9f2a7baf3:3"
)

get_number() {
  echo "$1" | cut -d ":" -f 2
}

sorted_strings=($(for string in "${strings[@]}"; do
  echo "$string"
done | sort -t ":" -k2,2n))

for sorted_string in "${sorted_strings[@]}"; do
  left_side=$(echo "$sorted_string" | cut -d ":" -f 1)
  exiftool $left_side.* | grep Comment | rev | cut -d " " -f 1 | rev
done
```

Here's the result after using the script

```bash
bash test.sh            
R0lGODlhZABkAPcAAAAAAAAAMwAAZgAAmQAAzAAA/wArAAArMwArZgArmQArzAAr/wBVAABVMwBVZgBVmQBVzABV/wCAA
ACAMwCAZgCAmQCAzACA/wCqAACqMwCqZgCqmQCqzACq/wDVAADVMwDVZgDVmQDVzADV/wD/AAD/MwD/ZgD/mQD/zAD//z
MAADMAMzMAZjMAmTMAzDMA/zMrADMrMzMrZjMrmTMrzDMr/zNVADNVMzNVZjNVmTNVzDNV/zOAADOAMzOAZjOAmTOAzDO
A/zOqADOqMzOqZjOqmTOqzDOq/zPVADPVMzPVZjPVmTPVzDPV/zP/ADP/MzP/ZjP/mTP/zDP//2YAAGYAM2YAZmYAmWYA
zGYA/2YrAGYrM2YrZmYrmWYrzGYr/2ZVAGZVM2ZVZmZVmWZVzGZV/2aAAGaAM2aAZmaAmWaAzGaA/2aqAGaqM2aqZmaqm
WaqzGaq/2bVAGbVM2bVZmbVmWbVzGbV/2b/AGb/M2b/Zmb/mWb/zGb//5kAAJkAM5kAZpkAmZkAzJkA/5krAJkrM5krZp
krmZkrzJkr/5lVAJlVM5lVZplVmZlVzJlV/5mAAJmAM5mAZpmAmZmAzJmA/5mqAJmqM5mqZpmqmZmqzJmq/5nVAJnVM5n
VZpnVmZnVzJnV/5n/AJn/M5n/Zpn/mZn/zJn//8wAAMwAM8wAZswAmcwAzMwA/8wrAMwrM8wrZswrmcwrzMwr/8xVAMxV
M8xVZsxVmcxVzMxV/8yAAMyAM8yAZsyAmcyAzMyA/8yqAMyqM8yqZsyqmcyqzMyq/8zVAMzVM8zVZszVmczVzMzV/8z/A
Mz/M8z/Zsz/mcz/zMz///8AAP8AM/8AZv8Amf8AzP8A//8rAP8rM/8rZv8rmf8rzP8r//9VAP9VM/9VZv9Vmf9VzP9V//
+AAP+AM/+AZv+Amf+AzP+A//+qAP+qM/+qZv+qmf+qzP+q///VAP/VM//VZv/Vmf/VzP/V////AP//M///Zv//mf//zP/
//wAAAAAAAAAAAAAAACH5BAEAAPwALAAAAABkAGQAAAj/AAEIHEiwoMGD+xIqXJiQ4MKDBR8OlAixosWLGB0y3Kix4UWK
AEBmHEnS4kaOExViBCmypMuXJxl23LdSZUqPL2vG3EnzZk+BLHnK9BlU6EmIRmPOXEpU6MyiSVsCjTp0Ks6QNq0mfZoVK
9WuXKMy1Rp2Z9myRpGCRdj1K9SxXn/G1SkXrtqrbtvqxbu3Lturdv/Kzcu38OC+dO/6NQj1K9y3PhMLlkz2I+KmKCubXK
u5IuTNhtFeBr14ruXQpw8bVho5tWLKpkk/dtpa9mTXmFWbJfnZM+fYvlf3ZT2y9+vbtrU2Psp7dPDSUiMOn545o3Hk0vM
G7nwWO2HohLcD/+/OmPpWwY5rZwdffbZ29FTFRye+XjTPnNad976Ov/9z97oB6N+AuHW2n3MEJqjegahxN+B36ZFGH4QU
VtgefNVZqGGFdE244YfvScicciD+BhtsywG2oH4mFphfg3HNxyKMJUV3nIEj1rcijc2pmBxw9Am4Y1XPeZjhjOeReOSF5
d2Xm5BK7hZliqXZpxuDV9I2pXkd5mhkgLl9+aWVYw7Jn4KpRQgkl+qh+aJbUMaIoJtpxjckeePR2aSUay7Jp41tYgghgF
SeKR6ZduLoZ5k/KmYhoeYZGuieaU3qkow8Oqjpf5bW+NuZmFbpXYvFfTrnpoDueWiP7IGpKnYndv9qqZpiniqnj3m+Sqm
Wt2aZaa83crqrk1HiGWqosW65KJuoeklqrushC2uduK66KZ7yPYvttLoWaSquqUb7bLjk5phttYKiGKmtQc6aqLB9/vkt
kecmCW+h7Jrrrlgu4vtru83y2+iTLsaJH5X1lguurZ4K96++3RqcE8L70sttvAT6W6K66Crra4mV/qiwvCCrGezItZbM6
8nj8mryg+sSi3HFenb7XZzS1mzmu1gGq7PGjM4csaAh91o0ys76ieiyAkNrpbExkwynrB5D7bC9N1NtNLNC39tyxzq/yb
Go14ZdKtiSlm12sjqq7fTabOPM87yQov21q1srPerVcd9mGWbSrdZN9tuKfhy03Ey27fWv2xb7d+IjSzxsq0DT3S/DBE+
+dOCXM565uFcDnLfUVmPNtehZF145jClTPjffjxNJseOTH3366x83nmDawH4uOty8t6478JjT3rvxcEsOeu6fYxQQADs=
```

Since the output looks like another hashes, the next thing is tried to look into it.

## Step 8: Analyze the hashes

After trying a few times, I manage to find a interesting text in the output.

![](/assets/img/posts/2023-09-05-Extracting-QR-from-android/2023-09-05-Extracting-QR-from-android-21-55-01.png)

After using base64 to decode, there is a `GIF` text in the output. After noticing that, the first thing that comes into my mind is that it will be a GIF. With that concept in mind, the next thing will be trying to change the result to GIF.

![](/assets/img/posts/2023-09-05-Extracting-QR-from-android/2023-09-05-Extracting-QR-from-android-21-57-58.png)

With trying to get the GIF, a QR code appear. The next thing to do will be scan the QR code and hope for something useful

## Step 9: Scan the QR Code

After scanning, there is a invite code given to us.

![](/assets/img/posts/2023-09-05-Extracting-QR-from-android/2023-09-05-Extracting-QR-from-android-22-01-21.png)

The invite code looks like a discord link. The next step would most likely be joining the server.

## Step 10: Join and play with the Discord server

After joining the discord server, there is a bot in the discord server. Since I no idea what to do next, i tried to talk to the discord bot and I have some interesting result.

![](/assets/img/posts/2023-09-05-Extracting-QR-from-android/2023-09-05-Extracting-QR-from-android-22-03-51.png)

With that, I finally found the flag.

## Things i learned from the challenge
- hiding a image/GIF by base64 encoding it and split it into the comment of each file.
- Image file with weird names, especially those looks like hashes.
- android file structure, where the image that save by a user is stored in `/media/0/DCIM/`.