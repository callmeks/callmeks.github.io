---
title: OWASP Juice Shop - MC SafeSearch
categories: [writeups,OWASP-juice-shop]
tags: [web]
---

## MC SafeSearch

```
- Difficulty: 2 star
- Category:  Sensitive Data Exposure 
- Description: Log in with MC SafeSearch's original user credentials without applying SQL Injection or any other bypass.
```

### Method to solve

- According to the description, we will need to get Mc SafeSearch credentials to login his account. 
- To do so, the first thing is to get the username or email or the user.
- This can be easily done by accessing into administration page with admin account. 
- The username would be : `mc.safesearch@juice-sh.op`
- The next part would be searching for password.
- Since there are no clue for us, the first thing would be searching the user online.
- After gathering some information, we could find out that there is a a [song](https://www.youtube.com/watch?v=v59CX2DiX0Y) related to MC SafeSearch that talks about password.
- According to the lyrics of the video, thats a potential password that we could trry.

```
[verse 1]
Online, I'm shoppin' and I'm feelin' fine
But I gotta use some passwords to keep what's mine
Passwords are codes that protect yo stuff
You gotta keep them all secret to stay safe enough
"But then how do you remember?" is the question that I get
I say, "Why not use the first name of your favorite pet?"
Mine's my dog, Mr. Noodles. It don't matter if you know
'Cause I was tricky and replaced some vowels with zeroes
```

- Based on the lyrics, the potential password would be `Mr. Noodles` or `Mr. N00dles`.
- After trying each potential password, the correct credentials would be `mc.safesearch@juice-sh.op`:`Mr. N00dles`

### Useful-links

- [Youtube Walkthrough](https://youtu.be/QgFC-056Mos)
- [MC SafeSearch lyrics](https://genius.com/Collegehumor-protect-ya-passwordz-lyrics)