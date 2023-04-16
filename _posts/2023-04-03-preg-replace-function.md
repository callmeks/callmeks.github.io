---
title: preg_replace() function
categories: [Knowledge,Web]
tags: [Knowledge]
---


- [Explanation](#explanation)
- [Example](#example)
- [Remediation](#remediation)
- [Useful Links](#useful-links)


# Explanation

- **`preg_replace()`** function is used to compare input and replace it
- instead of the exact input, this function is just searching for the pattern and compared it

# Example

![](/assets/img/posts/2023-04-03-preg-replace-function/2023-04-03-preg-replace-function-21-45-51.png)

```bash
when our supplied input is : hellohellotherehoomantherehooman
$php main.php
Final String is : hellotherehooman
Success
```

- Based on the example, the **`preg_replace()`** function found a same pattern from the input: **`hello[hellotherehooman]therehooman`**  and it will change it to **`hello[]therehooman`**
- this allow the final string to become **`hellotherehooman`** and success

# Remediation

- use **`str_replace()`** instead of **`preg_replace()`**

# Useful Links

[preg_replace](https://www.php.net/manual/en/function.preg-replace.php)

[PHP Tricks in Web CTF challenges](https://devansh.xyz/ctfs/2021/09/11/php-tricks.html)