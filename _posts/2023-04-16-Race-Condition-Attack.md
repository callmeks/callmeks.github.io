---
title: Raise Condition Attack
categories: [Knowledge,Web]
tags: [Knowledge]
---

- [Explanation](#explanation)
- [Example](#example)
- [Remediation](#remediation)
- [Useful-links](#useful-links)


# Explanation
- Happens when the few second gaps when file is uploaded into the server temporary before verifying
- Since the file is uploaded for a few second before verifying, it is possible to brute force to trigger the uploaded file and execute the file.
- the other option is to make use of large size of file to make the verifying process much slower

# Example
- Can try the PortSwigger lab for example. Links given below

# Remediation
- make sure to verify the file first before uploaded into the server. 

# Useful-links
- [Race Condition Attack](https://research.securitum.com/race-condition-attack-exemplary-use-in-web-application/)
- [Lab for race condition attack](https://portswigger.net/web-security/file-upload/lab-file-upload-web-shell-upload-via-race-condition)
- [How to check Race Conditions in Web Applications](https://krevetk0.medium.com/how-to-check-race-conditions-in-web-applications-338f73937992)