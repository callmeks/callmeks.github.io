---
title: PHP ereg() function
categories: [Knowledge,Web]
tags: [Knowledge]
---

# Explanation

- **`ereg()`** function === **FISHY / must look into it**
- the function is used to search a string for matches to the regular expression given in pattern in a case-sensitive way
- **THIS FUNCTION WAS DEPRECATED IN PHP 5.3.0 AND REMOVED IN PHP 7.0.0**
- The regex could be easily bypassed using Null Byte (**`%00`**)

# Remediation

- STOP USING **`ereg()`** FUNCTION

# References
- [PHP tricks](https://devansh.xyz/ctfs/2021/09/11/php-tricks.html)
- [PHP - regex bypass](https://bugs.php.net/bug.php?id=44366)