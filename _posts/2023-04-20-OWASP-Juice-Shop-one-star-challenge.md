---
title: OWASP Juice Shop Challenge - 1 star
categories: [writeups,OWASP-juice-shop]
tags: [web]
---

## Bonus Payload

```
- Difficulty: 1 star
- Category: XSS
- Description: Use the bonus payload <iframe width="100%" height="166" scrolling="no" frameborder="no" allow="autoplay" src="https://w.soundcloud.com/player/?url=https%3A//api.soundcloud.com/tracks/771984076&color=%23ff5500&auto_play=true&hide_related=false&show_comments=true&show_user=true&show_reposts=false&show_teaser=true"></iframe> in the DOM XSS challenge.
```

### Method to solve

- In this challenge, the main idea is to run the given payload into DOM XSS challenge.
- This payload could be triggered in the search function of OWASP Juice Shop on the top right corner.
<br>
![](/assets/img/posts/2023-04-20-OWASP-Juice-Shop-one-star-challenge/2023-04-20-OWASP-Juice-Shop-one-star-challenge-14-55-59.png)
<br>
- To run the code, just paste the payload into the search function and click enter.
<br>
![](/assets/img/posts/2023-04-20-OWASP-Juice-Shop-one-star-challenge/2023-04-20-OWASP-Juice-Shop-one-star-challenge-15-08-01.png)
<br>
- the result will appear a song related to OWASP juice shop.

### Code Review

- Vulnerable code
    ```js
    filterTable () {
        let queryParam: string = this.route.snapshot.queryParams.q
        if (queryParam) {
            queryParam = queryParam.trim()
            this.dataSource.filter = queryParam.toLowerCase()
            this.searchValue = this.sanitizer.bypassSecurityTrustHtml(queryParam) // vulnerable
            this.gridDataSource.subscribe((result: any) => {
            if (result.length === 0) {
                this.emptyState = true
            } else {
                this.emptyState = false
            }
            })
        } else {
            this.dataSource.filter = ''
            this.searchValue = undefined
            this.emptyState = false
        }
        }
    ```
    - The reason it is vulnerable to XSS is becasue the code uses `bypassSecurityTrustHtml`. 
    - Angular by default has their own sanitization for the value passed in.
    - The code `bypassSecurityTrustHtml` allows all html value to bypass the sanitization.
    - The correct method to fix this vulnerability is to remove the code.
- Solution
    ```js
    filterTable () {
        let queryParam: string = this.route.snapshot.queryParams.q
        if (queryParam) {
            queryParam = queryParam.trim()
            this.dataSource.filter = queryParam.toLowerCase()
            this.searchValue = queryParam   // remove bypassSecurityTrustHtml
            this.gridDataSource.subscribe((result: any) => {
            if (result.length === 0) {
                this.emptyState = true
            } else {
                this.emptyState = false
            }
            })
        } else {
            this.dataSource.filter = ''
            this.searchValue = undefined
            this.emptyState = false
        }
        }
    ```

### Useful-links

- [DomSanitizer Security risk](https://angular.io/api/platform-browser/DomSanitizer#security-risk)
- [DOM XSS Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/DOM_based_XSS_Prevention_Cheat_Sheet.html)

## Bully Chatbot

```
- Difficulty: 1 star
- Category: Miscellaneous
- Description: Receive a coupon code from the support chatbot.
```

### Method to solve

 - Based on the description, we will need to get a coupon from the support chatbot.
 - The tags given includes brute force which means that we might need to keep asking for coupon from the chatbot.
 - User must be logged in in order to use support chatbot. 
 - By constantly asking coupon from the chatbot, it will give us a 10% coupon.
<br>
![](/assets/img/posts/2023-04-20-OWASP-Juice-Shop-one-star-challenge/2023-04-20-OWASP-Juice-Shop-one-star-challenge-16-26-16.png)
<br>

## Confidential Document

```
- Difficulty: 1 star
- Category: Sensitive Data Exposure
- Description: Access a confidential document.
```

### Method to solve

- In this challenge, the goal is to access confidential document.
- Since no hint, try to look around the juice shop.
- At the about us page, there is a link which allow us to download their terms of use.
<br>
![](/assets/img/posts/2023-04-20-OWASP-Juice-Shop-one-star-challenge/2023-04-20-OWASP-Juice-Shop-one-star-challenge-16-41-12.png)
<br>
- The link given `http://localhost:3000/ftp/legal.md` in about us page consist of a subdirectory named `ftp`. 
- Since there is another subdirectory, it is worth looking into it.
<br>
![](/assets/img/posts/2023-04-20-OWASP-Juice-Shop-one-star-challenge/2023-04-20-OWASP-Juice-Shop-one-star-challenge-16-48-19.png)
<br>
- The result shows several files inside the FTP subdirectory. 
- These are some sensitive information in the FTP subdirectory which are not supposed to be available to the public. 
- In this challenge, the file `acquisitions.md` is the confidential information. 

### Code Review

- Vulnerable Code
    ```js
    /* /ftp directory browsing and file download */
    app.use('/ftp', serveIndexMiddleware, serveIndex('ftp', { icons: true }))
    app.use('/ftp(?!/quarantine)/:file', fileServer())
    app.use('/ftp/quarantine/:file', quarantineServer())
    ```
    - The code above allow user to have access into FTP subdirectory. 
    - Although listing the subdirectory is not vulnerable, the subdirectory might contains sensitive information which will be leaked out. 

- Solution
  - It is better to **remove all the code** above to avoid public user to have access into FTP subdirectory.
  - If the web application need to provide information, the information should be place in a static subdirectory.


## DOM XSS

```
- Difficulty: 1 star
- Category: XSS
- Description: Perform a DOM XSS attack with <iframe src="javascript:alert(`xss`)">.
```

### Method to solve

- Search for common place such as message boards, comments or search feature. 
<br>
![](/assets/img/posts/2023-04-20-OWASP-Juice-Shop-one-star-challenge/2023-04-20-OWASP-Juice-Shop-one-star-challenge-18-33-10.png)
<br>
- Based on the result, DOM XSS successfully triggered using search feature.

### Code Review

- Vulnerable Code
    ```js
    filterTable () {
        let queryParam: string = this.route.snapshot.queryParams.q
        if (queryParam) {
            queryParam = queryParam.trim()
            this.dataSource.filter = queryParam.toLowerCase()
            this.searchValue = this.sanitizer.bypassSecurityTrustHtml(queryParam) // vulnerable
            this.gridDataSource.subscribe((result: any) => {
            if (result.length === 0) {
                this.emptyState = true
            } else {
                this.emptyState = false
            }
            })
        } else {
            this.dataSource.filter = ''
            this.searchValue = undefined
            this.emptyState = false
        }
        }
    ```
    - The code `bypassSecurityTrustHtml` will allow HTML code to bypass security check which causes XSS.
- Solution
    ```js
    filterTable () {
        let queryParam: string = this.route.snapshot.queryParams.q
        if (queryParam) {
            queryParam = queryParam.trim()
            this.dataSource.filter = queryParam.toLowerCase()
            this.searchValue = queryParam   // remove bypassSecurityTrustHtml
            this.gridDataSource.subscribe((result: any) => {
            if (result.length === 0) {
                this.emptyState = true
            } else {
                this.emptyState = false
            }
            })
        } else {
            this.dataSource.filter = ''
            this.searchValue = undefined
            this.emptyState = false
        }
        }
    ```
### Useful-links

- [DomSanitizer Security risk](https://angular.io/api/platform-browser/DomSanitizer#security-risk)
- [DOM XSS Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/DOM_based_XSS_Prevention_Cheat_Sheet.html)


## Error Handling

```
- Difficulty: 1 star
- Category: Security Misconfiguration
- Description: Provoke an error that is neither very gracefully nor consistently handled.
```

### Method to solve

- According to the description, the challenge will be solved easily by provoking an error in the juice shop.
- To solve this challenge, it is as simple as getting any error code such as 400 or 500 from the juice shop. 
- This can be easily done by randomly searching for subdirectory.
<br>
![](/assets/img/posts/2023-04-20-OWASP-Juice-Shop-one-star-challenge/2023-04-20-OWASP-Juice-Shop-one-star-challenge-18-49-57.png)
<br>
- By accessing to the website above, the error page is successfully triggered and the challenge is solved.

### Useful-links

- [Error Code](https://developers.google.com/webmaster-tools/v1/errors)
- [Error handling cheat sheet](https://cheatsheetseries.owasp.org/cheatsheets/Error_Handling_Cheat_Sheet.html)


## Exposed Metrics

```
- Difficulty: 1 star
- Category: Sensitive Data Exposure
- Description: Find the endpoint that serves usage data to be scraped by a popular monitoring system.
```

### Method to solve

- According to the description, the [popular monitoring system](https://github.com/prometheus/prometheus) was given a link. 
- Based on the challenge, the metrics should be exposed to the public.
- The challenge will be searching for metrics.
- The link given is a github repository which have its own github pages.
<br>
![](/assets/img/posts/2023-04-20-OWASP-Juice-Shop-one-star-challenge/2023-04-20-OWASP-Juice-Shop-one-star-challenge-19-03-53.png)
<br>
- After looking into the github pages, there is a get started button which provides some tutorial.
- After looking around at the getting started page, there is an interesting information.
<br>
![](/assets/img/posts/2023-04-20-OWASP-Juice-Shop-one-star-challenge/2023-04-20-OWASP-Juice-Shop-one-star-challenge-19-06-02.png)
<br>
- According to the github pages, metrics could be easily found by accessing to `website/metrics`
- this means that the juice shop metrics should also able to access the same way.
<br>
![](/assets/img/posts/2023-04-20-OWASP-Juice-Shop-one-star-challenge/2023-04-20-OWASP-Juice-Shop-one-star-challenge-19-08-51.png)

### Code Review
- Vulnerable Code
    ```js
    /* Serve metrics */
    let metricsUpdateLoop: any
    const Metrics = metrics.observeMetrics()
    app.get('/metrics', metrics.serveMetrics()) //vulnerable
    errorhandler.title = `${config.get('application.name')} (Express ${utils.version('express')})`
    
    const registerWebsocketEvents = require('./lib/startup/registerWebsocketEvents')
    const customizeApplication = require('./lib/startup/customizeApplication')
    
    export async function start (readyCallback: Function) {
    const datacreatorEnd = startupGauge.startTimer({ task: 'datacreator' })
    await sequelize.sync({ force: true })
    await datacreator()
    datacreatorEnd()
    const port = process.env.PORT ?? config.get('server.port')
    process.env.BASE_PATH = process.env.BASE_PATH ?? config.get('server.basePath')
    
    metricsUpdateLoop = Metrics.updateLoop()
    
    server.listen(port, () => {
        logger.info(colors.cyan(`Server listening on port ${colors.bold(port)}`))
        startupGauge.set({ task: 'ready' }, (Date.now() - startTime) / 1000)
        if (process.env.BASE_PATH !== '') {
        logger.info(colors.cyan(`Server using proxy base path ${colors.bold(process.env.BASE_PATH)} for redirects`))
        }
        registerWebsocketEvents(server)
        if (readyCallback) {
        readyCallback()
        }
    })
    
    }
    
    export function close (exitCode: number | undefined) {
    if (server) {
        clearInterval(metricsUpdateLoop)
        server.close()
    }
    if (exitCode !== undefined) {
        process.exit(exitCode)
    }
    }
    ```
    - Based on the code, it allows metrics to be accessed by everyone.
    - It is not recommended to allow everyone to access to avoid any information leaked. 
- Solution
    ```js
    /* Serve metrics */
    let metricsUpdateLoop: any
    const Metrics = metrics.observeMetrics()
    app.get('/metrics', security.isAdmin(), metrics.serveMetrics()) // add security check
    errorhandler.title = `${config.get('application.name')} (Express ${utils.version('express')})`
    
    const registerWebsocketEvents = require('./lib/startup/registerWebsocketEvents')
    const customizeApplication = require('./lib/startup/customizeApplication')
    
    export async function start (readyCallback: Function) {
    const datacreatorEnd = startupGauge.startTimer({ task: 'datacreator' })
    await sequelize.sync({ force: true })
    await datacreator()
    datacreatorEnd()
    const port = process.env.PORT ?? config.get('server.port')
    process.env.BASE_PATH = process.env.BASE_PATH ?? config.get('server.basePath')
    
    metricsUpdateLoop = Metrics.updateLoop()
    
    server.listen(port, () => {
        logger.info(colors.cyan(`Server listening on port ${colors.bold(port)}`))
        startupGauge.set({ task: 'ready' }, (Date.now() - startTime) / 1000)
        if (process.env.BASE_PATH !== '') {
        logger.info(colors.cyan(`Server using proxy base path ${colors.bold(process.env.BASE_PATH)} for redirects`))
        }
        registerWebsocketEvents(server)
        if (readyCallback) {
        readyCallback()
        }
    })
    
    }
    
    export function close (exitCode: number | undefined) {
    if (server) {
        clearInterval(metricsUpdateLoop)
        server.close()
    }
    if (exitCode !== undefined) {
        process.exit(exitCode)
    }
    }
    ```
    - Since metrics still needed to be accessed by admin, the best method will be adding a security check to ensure that only admin is allowed to access. 

## Mass Dispel

```
- Difficulty: 1 star
- Category: Miscellaneous
- Description: Close multiple "Challenge solved"-notifications in one go.
```

### Method to solve

- According to the description, it requires the user to close all notification with a click.
<br>
![](/assets/img/posts/2023-04-20-OWASP-Juice-Shop-one-star-challenge/2023-04-20-OWASP-Juice-Shop-one-star-challenge-20-45-55.png)
<br>
- This can be easily done by holding [shift] key when clicking close icon.

## Missing Encoding

```
- Difficulty: 1 star
- Category: Improper Input Validation
- Description: Retrieve the photo of Bjoern's cat in "melee combat-mode".
```

### Method to solve

- According to the description, we are supposed to get the photo.
- This challenge could be completed without any account logged in. 
<br>
![](/assets/img/posts/2023-04-20-OWASP-Juice-Shop-one-star-challenge/2023-04-20-OWASP-Juice-Shop-one-star-challenge-21-15-19.png)
<br>
- According to the photo wall of juice shop, the cat image is not able to be retrieved. 
- Since it is not able to retrieve, the next step should be look around the code with inspect element.
```html
<img _ngcontent-awl-c240="" class="image" src="assets/public/images/uploads/😼-#zatschi-#whoneedsfourlegs-1572600969477.jpg" alt="😼 #zatschi #whoneedsfourlegs">
```
- According to the element code, the `src` contains a cat icon and hashtag.
- This might be the reason why it is not possible to retrieve the image.
- Since the category is improper input validation and the name is related to encoding, the first thing that could try is URL encoding
```html
<img _ngcontent-awl-c240="" class="image" src="assets/public/images/uploads/😼-#zatschi-#whoneedsfourlegs-1572600969477.jpg" alt="😼 #zatschi #whoneedsfourlegs">

<img _ngcontent-awl-c240="" class="image" src="assets%2Fpublic%2Fimages%2Fuploads%2F%F0%9F%98%BC-%23zatschi-%23whoneedsfourlegs-1572600969477.jpg" alt="😼 #zatschi #whoneedsfourlegs">

```
- After changing the `src` code to URL encoded, the image is successfully retrieved.
<br>
![](/assets/img/posts/2023-04-20-OWASP-Juice-Shop-one-star-challenge/2023-04-20-OWASP-Juice-Shop-one-star-challenge-21-24-22.png)
<br>

### Useful-links
- [URL Encode](https://www.urlencoder.org/)

## Outdated Whitelist

```
- Difficulty: 1 star
- Category: Unvalidated Redirects
- Description: Let us redirect you to one of our crypto currency addresses which are not promoted any longer.
```

### Method to solve

- In this challenge, we are supposed to saerch for the juice shop crypto currency. 
- Both HTML and JavaScript allow redirects but works on different code. 
- The method to solve challenge would be searching for both HTML and JavaScript code for redirect.
- There are a few words which is worth looking for when searching through the code such as **crypto**, **redirect**, **bitcoin** or **href**.
- To search HTML code, right click the page and view the source code.
- To search JavaScript code, right click to inspect element and change to source tab for Chrome or debugger tab to Firefox.
<br>
![](/assets/img/posts/2023-04-20-OWASP-Juice-Shop-one-star-challenge/2023-04-20-OWASP-Juice-Shop-one-star-challenge-21-55-28.png)
<br>
- After searching for awhile, there is a URL redirect which provide a link to juice shop crpyto wallet link.
- heres the result of the crypto wallet link.
<br>
![](/assets/img/posts/2023-04-20-OWASP-Juice-Shop-one-star-challenge/2023-04-20-OWASP-Juice-Shop-one-star-challenge-21-57-03.png)
<br>

### Code Review

- Vulnerable Code
    ```js
    const redirectAllowlist = new Set([
    'https://github.com/bkimminich/juice-shop',
    'https://blockchain.info/address/1AbKfgvw9psQ41NbLi8kufDQTezwG8DRZm', // crypto address
    'https://explorer.dash.org/address/Xr556RzuwX6hg5EGpkybbv5RanJoZN17kW', // crypto address
    'https://etherscan.io/address/0x0f933ab9fcaaa782d0279c300d73750e1311eae6', // crypto address
    'http://shop.spreadshirt.com/juiceshop',
    'http://shop.spreadshirt.de/juiceshop',
    'https://www.stickeryou.com/products/owasp-juice-shop/794',
    'http://leanpub.com/juice-shop'
    ])
    exports.redirectAllowlist = redirectAllowlist
    
    exports.isRedirectAllowed = (url: string) => {
    let allowed = false
    for (const allowedUrl of redirectAllowlist) {
        allowed = allowed || url.includes(allowedUrl)
    }
    return allowed
    }
    ```
    - Based on the code, there are a few crypto link is in the redirect allow list.
- Solution
    ```js
    const redirectAllowlist = new Set([
    'https://github.com/bkimminich/juice-shop',
    'http://shop.spreadshirt.com/juiceshop',
    'http://shop.spreadshirt.de/juiceshop',
    'https://www.stickeryou.com/products/owasp-juice-shop/794',
    'http://leanpub.com/juice-shop'
    ])
    exports.redirectAllowlist = redirectAllowlist
    
    exports.isRedirectAllowed = (url: string) => {
    let allowed = false
    for (const allowedUrl of redirectAllowlist) {
        allowed = allowed || url.includes(allowedUrl)
    }
    return allowed
    }
    ```
    - the solution is to remove all the crypto link.

### Useful-links
- [Unvalidated redirects and forwards cheat sheet](https://cheatsheetseries.owasp.org/cheatsheets/Unvalidated_Redirects_and_Forwards_Cheat_Sheet.html)

## Privary Policy

```
- Difficulty: 1 star
- Category: Miscellaneous
- Description: Read our privacy policy.
```

### Method to solve

- This is just a simple challenge which requires user to read the privary policy.
- To read privary policy, user will need to be logged into any account first.
<br>
![](/assets/img/posts/2023-04-20-OWASP-Juice-Shop-one-star-challenge/2023-04-20-OWASP-Juice-Shop-one-star-challenge-22-51-22.png)
<br>

## Repetitive Registration

```
- Difficulty: 1 star
- Category: Improper Input Validation
- Description: Follow the DRY principle while registering a user.
```

### Method to solve

- According to the description, register a user by following DRY principle.
- DRY principle = Dont Repeat Yourself
- In a register user page, the only criteria that will repear is password.
<br>
![](/assets/img/posts/2023-04-20-OWASP-Juice-Shop-one-star-challenge/2023-04-20-OWASP-Juice-Shop-one-star-challenge-22-54-01.png)
<br>
- Since this challenge do not allow us to repeat, we will need to provide a different password for both field. 
<br>
![](/assets/img/posts/2023-04-20-OWASP-Juice-Shop-one-star-challenge/2023-04-20-OWASP-Juice-Shop-one-star-challenge-22-55-59.png)
<br>
- If we tried to provide different password, it will provide an error message which says that password not match.
<br>
![](/assets/img/posts/2023-04-20-OWASP-Juice-Shop-one-star-challenge/2023-04-20-OWASP-Juice-Shop-one-star-challenge-22-57-18.png)
<br>
![](/assets/img/posts/2023-04-20-OWASP-Juice-Shop-one-star-challenge/2023-04-20-OWASP-Juice-Shop-one-star-challenge-22-57-32.png)
<br>
- If we tried to put the same password for both input first and change the real password later, we managed to bypass the repeat password check. 
- If we tried to register a user with this method, the challenge will be solved. 

### Useful-links
- [Input Validation Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Input_Validation_Cheat_Sheet.html)

## Score Board

```
- Difficulty: 1 star
- Category: Miscellaneous
- Description: Find the carefully hidden 'Score Board' page.
```

### Method to solve

- To solve this challenge, using some directory brute forcing tools or guessing the correct name will do.
- The correct answer is : `http://localhost:3000/#/score-board`


## Zero Stars

```
- Difficulty: 1 star
- Category: Improper Input Validation
- Description: Give a devastating zero-star feedback to the store.
```

### Method to solve

- This challenge requries us to provide 0 star feedback to store.
- This challenge can be access without user login.
<br>
![](/assets/img/posts/2023-04-20-OWASP-Juice-Shop-one-star-challenge/2023-04-20-OWASP-Juice-Shop-one-star-challenge-23-04-19.png)
<br>
- Based on the customer feedback, the least rating given is 1 which means that it is impossible to get give a 0 star rating.
- The solution for this challenge would be modifying the request.
- To modify the request, we will first need to inspect element while providing a feedback.
<br>
![](/assets/img/posts/2023-04-20-OWASP-Juice-Shop-one-star-challenge/2023-04-20-OWASP-Juice-Shop-one-star-challenge-23-12-01.png)
<br>
- After randomly providing a feedback, we will get 3 different request from network tab of inspect element.
<br>
![](/assets/img/posts/2023-04-20-OWASP-Juice-Shop-one-star-challenge/2023-04-20-OWASP-Juice-Shop-one-star-challenge-23-12-50.png)
<br>
![](/assets/img/posts/2023-04-20-OWASP-Juice-Shop-one-star-challenge/2023-04-20-OWASP-Juice-Shop-one-star-challenge-23-16-26.png)
<br>
- Based on the payload, the rating is set as 3.
- We will just need to modify the rating to 0 and resend.
- To do so, we will first need to copy as fetch.
- Next we will need to paste it in javascript console panel.
![](/assets/img/posts/2023-04-20-OWASP-Juice-Shop-one-star-challenge/2023-04-20-OWASP-Juice-Shop-one-star-challenge-23-20-34.png)
- Remember to change the rating to 0 and hit enter.
- Now back to network tab, we will see that we have successfully send a 0 star rating. 
<br>
![](/assets/img/posts/2023-04-20-OWASP-Juice-Shop-one-star-challenge/2023-04-20-OWASP-Juice-Shop-one-star-challenge-23-21-37.png)
<br>
![](/assets/img/posts/2023-04-20-OWASP-Juice-Shop-one-star-challenge/2023-04-20-OWASP-Juice-Shop-one-star-challenge-23-21-50.png)
<br>
- There are other method to solve as well such as intercepting request with Burp Suite or sending a POST request using `curl` command.