---
title: Ping With Integer
categories: [Knowledge,bash]
tags: [Knowledge]
---

- [Explanation](#explanation)
- [Example](#example)
- [Useful-links](#useful-links)


# Explanation
- Normally, command `ping` is usually used to ping IP Address
- Instead of IP address, `ping` is also able to convert integer to IP Address and ping it.

# Example

- Ping IP address

```bash
% ping 127.0.0.1
PING 127.0.0.1 (127.0.0.1) 56(84) bytes of data.
64 bytes from 127.0.0.1: icmp_seq=1 ttl=64 time=0.049 ms
64 bytes from 127.0.0.1: icmp_seq=2 ttl=64 time=0.017 ms
64 bytes from 127.0.0.1: icmp_seq=3 ttl=64 time=0.018 ms
64 bytes from 127.0.0.1: icmp_seq=4 ttl=64 time=0.026 ms
^C
--- 127.0.0.1 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3144ms
rtt min/avg/max/mdev = 0.017/0.027/0.049/0.012 ms
```
- Ping IP address with Integer

```bash
% ping 2130706433
PING 2130706433 (127.0.0.1) 56(84) bytes of data.
64 bytes from 127.0.0.1: icmp_seq=1 ttl=64 time=0.012 ms
64 bytes from 127.0.0.1: icmp_seq=2 ttl=64 time=0.024 ms
64 bytes from 127.0.0.1: icmp_seq=3 ttl=64 time=0.021 ms
64 bytes from 127.0.0.1: icmp_seq=4 ttl=64 time=0.019 ms
^C
--- 2130706433 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3088ms
rtt min/avg/max/mdev = 0.012/0.019/0.024/0.004 ms
```


# Useful-links
- [IP to Integer](http://www.aboutmyip.com/AboutMyXApp/IP2Integer.jsp/)