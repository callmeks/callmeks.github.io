---
title: EWF Image Forensic
categories: [CTF]
tags: [Forensic]
---

## Step 1: Identify the given files

To start off, the first thing to do is usually analyzing the file type that is given.

```bash
file *     
Alan's Private Life.E01:     EWF/Expert Witness/EnCase image file format
Alan's Private Life.E01.txt: Unicode text, UTF-8 (with BOM) text, with CRLF line terminators
Alan's Private Life.E02:     EWF/Expert Witness/EnCase image file format
```

Based on the result, we have EWF image file format which is a new things for me. After searching for information, it is also a type of image file type that contains useful information. 

## Step 2: Mount the EWF image

Based on the investigation on EWF image, it is possible to view the directories and files inside by mounting the EWF image in linux. There is a tool for it which is `ewfmount`. This tool by default is not installed in my version of kali linux. To install it, just run `apt install ewf-tools`.

```bash
ewfmount Alan\'s\ Private\ Life.E01 test 
ewfmount 20140814
```

After running the command, the output should be something like `ewfmount <numbers>`. the mounted image will be in `test` directory based on the command above. 

## Step 3: Analyzing the mounted image

although the EWF image is mounted successfully, we still could not read the content inside.

```bash
fdisk -l test/ewf1 
Disk test/ewf1: 1.85 GiB, 1988624384 bytes, 3884032 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x00ebb9c3

Device      Boot Start     End Sectors  Size Id Type
test/ewf1p1 *       32 3884031 3884000  1.9G  e W95 FAT16 (LBA)
```

To read the content inside, we will need to mount into another directory again.

## Step 4: Mount ewf1 into another directory

This time, we will just use the default `mount` command with specific flag.

```bash
mount -o ro,offset=$((32*512)) test/ewf1 test2
```

The things to take note here is the offset `32` which coulld be different. The offset `32` is retrieve from the previous step. 

## Step 4: Analyzing the mounted directory

After the mounting process is successful, `test2` will act like a directory and there will be information inside. Since the mount process is successful, we will need to search for interesting information inside the directory.

```bash
exiftool test2/zip-tie.jpg 
ExifTool Version Number         : 12.64
File Name                       : zip-tie.jpg
Directory                       : test2
File Size                       : 17 kB
File Modification Date/Time     : 2023:08:21 16:18:30+08:00
File Access Date/Time           : 2023:08:29 08:00:00+08:00
File Inode Change Date/Time     : 2023:08:21 16:18:30+08:00
File Permissions                : -rwxr-xr-x
File Type                       : JPEG
File Type Extension             : jpg
MIME Type                       : image/jpeg
JFIF Version                    : 1.01
Resolution Unit                 : inches
X Resolution                    : 72
Y Resolution                    : 72
XMP Toolkit                     : Image::ExifTool 12.57
Description                     : YzB2M3J0XzBwZXI0dGl2Mw==
Image Width                     : 709
Image Height                    : 357
Encoding Process                : Baseline DCT, Huffman coding
Bits Per Sample                 : 8
Color Components                : 3
Y Cb Cr Sub Sampling            : YCbCr4:2:0 (2 2)
Image Size                      : 709x357
Megapixels                      : 0.253
```

There is something interesting in the description of `zip-tie.jpg`. We could try to further analyze it.

## Step 5: Further analyzing the information

Since the description looks like a base64 encoding, we could try to decode it.

```bash
exiftool test2/zip-tie.jpg | grep Description | rev | cut -d " " -f 1 | rev | base64 -d
c0v3rt_0per4tiv3
```

After encoding it, we manage to get this interesting stuff which is actually a flag for the challenge. 

## Things i learned from the challenge
- [mounting EWF image](https://book.hacktricks.xyz/generic-methodologies-and-resources/basic-forensic-methodology/image-acquisition-and-mount)