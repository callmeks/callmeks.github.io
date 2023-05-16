---
title: OWASP Juice Shop - Meta Geo Stalking
categories: [writeups,OWASP-juice-shop]
tags: [web]
---

## Meta Geo Stalking

```
- Difficulty: 2 star
- Category:  Sensitive Data Exposure 
- Description: Determine the answer to John's security question by looking at an upload of him to the Photo Wall and use it to reset his password via the Forgot Password mechanism.
```

### Method to solve

- Based on the description, we are required to change the password of John account with forgot password mechanism.
- The hint given is a photo that he uploaded in Photo Wall.
- Based on the photo wall, there are a few images but there are only one which have the name similar to john.
- The first thing will be download it are have a look.
<br>
![](/assets/img/posts/2023-05-16-OWASP-Juice-Shop-Meta-Geo-Stalking/2023-05-16-OWASP-Juice-Shop-Meta-Geo-Stalking-14-08-14.png)
<br>
- The images seems to be nothing special.
- Since we are stuck in this phase, we could try and get the security question of john to reset password.
- The email will be `john@juice-sh.op` which can be easily retrieve by accessing the administration page with admin account.
<br>
![](/assets/img/posts/2023-05-16-OWASP-Juice-Shop-Meta-Geo-Stalking/2023-05-16-OWASP-Juice-Shop-Meta-Geo-Stalking-14-11-01.png)
<br>
- According to the security question, it is asking the favourite place to go hiking.
- Although the image was taken in a forest, there are no other clues which could get us a name.
- Instead, we could have a look at the metadata of the images and hope for something useful.

```bash
>> exiftool favorite-hiking-place.png 
ExifTool Version Number         : 12.60
File Name                       : favorite-hiking-place.png
Directory                       : .
File Size                       : 667 kB
File Modification Date/Time     : 2023:05:16 11:34:59+08:00
File Access Date/Time           : 2023:05:16 11:39:35+08:00
File Inode Change Date/Time     : 2023:05:16 11:34:59+08:00
File Permissions                : -rw-r--r--
File Type                       : PNG
File Type Extension             : png
MIME Type                       : image/png
Image Width                     : 471
Image Height                    : 627
Bit Depth                       : 8
Color Type                      : RGB
Compression                     : Deflate/Inflate
Filter                          : Adaptive
Interlace                       : Noninterlaced
Exif Byte Order                 : Little-endian (Intel, II)
Resolution Unit                 : inches
Y Cb Cr Positioning             : Centered
GPS Version ID                  : 2.2.0.0
GPS Latitude Ref                : North
GPS Longitude Ref               : West
GPS Map Datum                   : WGS-84
Thumbnail Offset                : 224
Thumbnail Length                : 4531
SRGB Rendering                  : Perceptual
Gamma                           : 2.2
Pixels Per Unit X               : 3779
Pixels Per Unit Y               : 3779
Pixel Units                     : meters
Image Size                      : 471x627
Megapixels                      : 0.295
Thumbnail Image                 : (Binary data 4531 bytes, use -b option to extract)
GPS Latitude                    : 36 deg 57' 31.38" N
GPS Longitude                   : 84 deg 20' 53.58" W
GPS Position                    : 36 deg 57' 31.38" N, 84 deg 20' 53.58" W
```

- Based on the information found, there is a interesting information for us which is related to GPS.
- With the GPS Position given to us, we could search it in any map and look for potential answer.
- To search it in google map, change `deg` to `°`.
<br>
![](/assets/img/posts/2023-05-16-OWASP-Juice-Shop-Meta-Geo-Stalking/2023-05-16-OWASP-Juice-Shop-Meta-Geo-Stalking-14-15-27.png)
<br>
- Based on the map result, there are a few potential places such as Twin Branch Shelter Campground and Daniel Boone Nat'l Forest.
- After trying all the potential places from the result, the correct answer will be Daniel Boone Nat'l Forest.
- By using the correct answer, we are able to reset john password.

### Useful-links
- [Youtube Walkthrough](https://youtu.be/2uS1e5mEg4o)
- [Google Map Result](https://www.google.com/maps/place/36%C2%B057'31.4%22N+84%C2%B020'53.6%22W/@36.9754495,-84.3826992,13z/data=!4m4!3m3!8m2!3d36.9587222!4d-84.3482222)