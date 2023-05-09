---
title: OWASP Juice Shop - Password Strength
categories: [writeups,OWASP-juice-shop]
tags: [web]
---

## Password Strength

```
- Difficulty: 2 star
- Category: Broken Authentication
- Description: Log in with the administrator's user credentials without previously changing them or applying SQL Injection.
```

### Method to solve

- In this challenge, we are ask to login as admin using admin's credentials only.
- Since the category is broken authentication, the admin credential should be weak.
- We could perform brute force attacks in the login page with correct administrator email and a password list.
- The first thing is to prepare and set up burp suite.
- When everything is ready, perform a random login to intercept the request.
<br>
![](/assets/img/posts/2023-05-01-OWASP-Juice-Shop-Password-Strength/2023-05-01-OWASP-Juice-Shop-Password-Strength-13-34-43.png)
<br>
- When burp suite successfully intercept the request, send it to intruder.
<br>
![](/assets/img/posts/2023-05-01-OWASP-Juice-Shop-Password-Strength/2023-05-01-OWASP-Juice-Shop-Password-Strength-13-35-47.png)
<br>
- When the request is sent to intruder, select **Sniper** attack type and only add payload position at the password value.
- This is to allow burp suite to send multiple request with different password.
- After everything is set, go to payload tab and select a password list by adding one from list or load a external list file. 
<br>
![](/assets/img/posts/2023-05-01-OWASP-Juice-Shop-Password-Strength/2023-05-01-OWASP-Juice-Shop-Password-Strength-13-40-07.png)
<br>
- Once everything is ready, click on start attack and wait for the result.
<br>
![](/assets/img/posts/2023-05-01-OWASP-Juice-Shop-Password-Strength/2023-05-01-OWASP-Juice-Shop-Password-Strength-13-43-59.png)
<br>
- The results only shows one status 200 code which means OK.
- After performing brute force attacks, we managed to could the password `admin123`.
- Now with the correct username and password, we are able to login as admin with the credentials. 

### Code Review

- Vulnerable Code
    ```js
    User.init(
        password: {
            type: DataTypes.STRING,
            set (clearTextPassword) {
                this.setDataValue('password', security.hash(clearTextPassword))
            }
        },
    ```
    - According to the code, the password do not have any validation.
    - This allow different type of weak and common password to be registered and allow attackers to get their account easily by brute force attack. 
- Solution
    ```js
    User.init(
        password: {
            type: DataTypes.STRING,
            set (clearTextPassword) {
                validatePasswordHasAtLeastTenChar(clearTextPassword)
                validatePasswordIsNotInTopOneMillionCommonPasswordsList(clearTextPassword)
                this.setDataValue('password', security.hash(clearTextPassword))
            }
        },
    ```
    - The solution added 2 validation which force every user to have at least 10 character and ensure that the password is not in the common password list. 

### Useful-links

- [Youtube walkthrough](https://youtu.be/EwVQhykH4xU)
- [Authentication Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html)
- [Common Password List](https://www.kaggle.com/datasets/wjburns/common-password-list-rockyoutxt)


