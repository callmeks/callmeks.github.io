---
title: OWASP Juice Shop - View Basket
categories: [writeups,OWASP-juice-shop]
tags: [web]
---

## View Basket

```
- Difficulty: 2 star
- Category: Broken Access Control
- Description: View another user's shopping basket.
```

### Method to solve

- In this challenge, we will be using chrome inspect element which is different from firefox inspect element.
- There is also another solution which uses Burp Suite but it will not be used here.
- Based on the challenge description, we are asked to view other user's shopping basket.
- Our first step would be to login as any user and view our own basket.
<br>
![](/assets/img/posts/2023-05-09-OWASP-Juice-Shop-View-Basket/2023-05-09-OWASP-Juice-Shop-View-Basket-19-17-01.png)
<br>
- According to our basket, we currently only have one item in our basket.
- To view other customer basket, we will need to take a look at inspect element.
- According to the request, we could notice the requests used to get our basket.
<br>
![](/assets/img/posts/2023-05-09-OWASP-Juice-Shop-View-Basket/2023-05-09-OWASP-Juice-Shop-View-Basket-19-28-58.png)
<br>
![](/assets/img/posts/2023-05-09-OWASP-Juice-Shop-View-Basket/2023-05-09-OWASP-Juice-Shop-View-Basket-19-28-26.png)
<br>
- Based on the headers and response of the request, we could understand that the requested URL ended with one.
- To view other customer basket, we could try to change the request URL to other numbers. 
- To do so, right click on the requests and copy as fetch.
<br>
![](/assets/img/posts/2023-05-09-OWASP-Juice-Shop-View-Basket/2023-05-09-OWASP-Juice-Shop-View-Basket-20-30-51.png)
<br>
- Next, paste the request into console tab and modify the number from one to three.
<br>
![](/assets/img/posts/2023-05-09-OWASP-Juice-Shop-View-Basket/2023-05-09-OWASP-Juice-Shop-View-Basket-20-32-12.png)
<br>
- Once the number is changed, click enter to send the requests.
- Now go back to Network tab, there will be a new requests.
<br>
![](/assets/img/posts/2023-05-09-OWASP-Juice-Shop-View-Basket/2023-05-09-OWASP-Juice-Shop-View-Basket-20-35-18.png)
<br>
![](/assets/img/posts/2023-05-09-OWASP-Juice-Shop-View-Basket/2023-05-09-OWASP-Juice-Shop-View-Basket-20-36-13.png)
<br>
- Based on the result, we are now able to view other customer basket by changing the requests header.
- Although it is not as good as viewing it in the website, we are still able to get our own item.

### Useful-links
- [Youtube walkthrough](https://youtu.be/gKayL6VneFU)
- 