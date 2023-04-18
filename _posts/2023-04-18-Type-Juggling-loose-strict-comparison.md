---
title: Type Juggling & Loose/Strict comparison
categories: [Knowledge,Web]
tags: [Knowledge]
---

- [Explanation](#explanation)
- [Example](#example)
- [Remediation](#remediation)
- [Useful-links](#useful-links)


# Explanation
- Combination of type juggling + loose / strict comparison may lead to Authentication Bypass or account takeover
- Type Juggling
  - not a vulnerability
  - A PHP feature
  - Since PHP is a loosely type language, thing may go different
    
    ```php
    <?php
    $var1 = 1;
    $var2 = "10 ABC";
    $var3 = "12 DEF";
    $var4 = "Go abc";
    $var5 = "0e test";
    $var6 = "0*test";

    echo $var1+$var2; //11 (10 is taken; others are ignored)
    echo $var1+$var3; //13 (12 is taken; others are ignored)
    echo $var1+$var4; //1  (First word is not integer; $var4 will be ignored)
    echo $var1+$var5; //1  (First word is not integer; $var5 will be ignored)
    echo $var1+$var6; //1  (First word is not integer; $var6 will be ignored)

    ?>

    ```
    
  - If the first word of the strings is a number, PHP will automatically use it as a integer.
  - If the first word of the strings is not a number, PHP will automatically set it as 0

- Loose Comparison
  - not a vulnerability
  - **`==`** or **`!=`**
  - only **value** is check; the type of variable will be **ignored**
  - use **`0e`** to bypass
  ![](/assets/img/posts/2023-04-18-Type-Juggling-loose-strict-comparison/2023-04-18-Type-Juggling-loose-strict-comparison-13-42-32.png)
        
- Strict Comparison
  - not a vulnerability
  - **`===`** or **`!=`**

    ![](/assets/img/posts/2023-04-18-Type-Juggling-loose-strict-comparison/2023-04-18-Type-Juggling-loose-strict-comparison-13-40-53.png)

# Example
- Loose Comparison
    
    ```php
    <?php
    if (hash("md5"), $_GET['passwd']) == "0")
        {
            echo "win":
        } else {
            echo "lost";
        }

    ?>
    ```

  - the example given is a perfect example of loose comparison and type juggling
  - in order to **`"win"`** , the md5 hash value of the input must be 0 which is impossible.
  - since loose comparison, just generate a md5 value which have **`0e`**
  - **`e`** in PHP = exponential operator
  - this will turn whole md5 value to 0
  - example of md5 value that turns to **`0e`** : **`240610708`** = `0e462097431906509019562988736854`
  - *pattern of **`0e\d+`** must be satisfied to pass the condition as true*

- Strict Comparison
    
    ```php
    <?php
    require 'flag.php';

    if (isset($_GET['name']) and isset($GET['password'])) {
        if($GET['name'] == $_GET['password'])
            print 'both cannot be same';
        else if (sha1($_GET['name']) === sha1($_GET['password']))
            die('Flag: '.$flag);
        else
            print 'Invalid password';
    }
    ?>
    ```
    
  - **`0e`** will not work here
  - this should be impossible to print the flag as sha1 of 2 different value is 100% different
  - must control **value** and **type** of the variable
  - SHA1 check will only be executed if the type of variables = strings
  - bypass it with array if got SHA1
    
    ```php
    <?php

    $name = array('test');
    $password = array('test2');

    if (isset($name) and isset($password)) {
        if($name == $password)
            print 'both cannot be same';
        else if (sha1($name) === sha1($password))
            print 'gongxi';
        else
            print 'Invalid password';
    }
    ?>

    result = gongxi
    ```

# Remediation
- Use strict comparison
- Do not predefine the strings to other same types

# Useful-links
- [CTF-Katana](https://github.com/JohnHammond/ctf-katana#php)
- [PayloadAllTheThing](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Type%20Juggling/README.md)
- [Hacktricks](https://book.hacktricks.xyz/network-services-pentesting/pentesting-web/php-tricks-esp)
- [PHP Type juggling](https://www.invicti.com/blog/web-security/php-type-juggling-vulnerabilities/)