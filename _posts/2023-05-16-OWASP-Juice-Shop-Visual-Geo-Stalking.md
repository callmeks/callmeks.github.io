---
title: OWASP Juice Shop - Visual Geo Stalking
categories: [writeups,OWASP-juice-shop]
tags: [web]
---

## Visual Geo Stalking

```
- Difficulty: 2 star
- Category:  Sensitive Data Exposure 
- Description: Determine the answer to Emma's security question by looking at an upload of her to the Photo Wall and use it to reset her password via the Forgot Password mechanism.
```

### Method to solve

- According to the description, we will need to change the password of Emma via the forgot password mechanism.
- The email or username of Emma can be retrieve from administration page with admin login.
<br>
![](/assets/img/posts/2023-05-16-OWASP-Juice-Shop-Visual-Geo-Stalking/2023-05-16-OWASP-Juice-Shop-Visual-Geo-Stalking-14-29-36.png)
<br>
- Based on the security question, it is asking for the work that Emma first do as an adult.
- The hint given from the description would be to check out the image in photo wall which she posted herself.
<br>
![](/assets/img/posts/2023-05-16-OWASP-Juice-Shop-Visual-Geo-Stalking/2023-05-16-OWASP-Juice-Shop-Visual-Geo-Stalking-14-34-20.png)
<br>
- After looking deeply into the image, there is a potential answer for the security question.
<br>
![](/assets/img/posts/2023-05-16-OWASP-Juice-Shop-Visual-Geo-Stalking/2023-05-16-OWASP-Juice-Shop-Visual-Geo-Stalking-14-35-58.png)
<br>
- Based on the images, there is a paper with the word `ITsec`. 
- This will be the correct answer for the security question.

### Useful-links
- [Youtube Walkthrough](https://youtu.be/GpjB2FL0-MY)