---
title: OWASP Juice Shop - Admin Section
categories: [writeups,OWASP-juice-shop]
tags: [web]
---

## Admin Section

```
- Difficulty: 2 star
- Category: Broken Access Control
- Access the administration section of the store.
```

### Method to solve

- In this challenge, we are ask to access the administration section of the store.
- Since we are going to access administration section, we will need to login as admin.
- To login as admin, there are a few method such as SQL injection or brute force attack.
- After login in as admin, we could try to read the `main.js` file in inspect element.
<br>
![](/assets/img/posts/2023-05-02-OWASP-Juice-Shop-Admin-Section/2023-05-02-OWASP-Juice-Shop-Admin-Section-14-16-59.png)
<br>
- The source code provide from useful information about having a path named administration.
- We could try then try to access this page as this should be the administration section of the store.
<br>
![](/assets/img/posts/2023-05-02-OWASP-Juice-Shop-Admin-Section/2023-05-02-OWASP-Juice-Shop-Admin-Section-14-19-02.png)
<br>


### Code Review

- Vulnerable Code
    ```js
    const routes: Routes = [
        {
            path: 'administration',
            component: AdministrationComponent,
            canActivate: [AdminGuard]
        },
    ]
    ```
    - The code here is not vulnerable but administration section should not be inside the web application
    - This is to reduce the risk of getting more sensitive information as admin
- Solution
    ```js
    const routes: Routes = [
        /* TODO: Externalize admin functions into separate application
        that is only accessible inside corporate network
        */

        //{
        //    path: 'administration',
        //    component: AdministrationComponent,
        //    canActivate: [AdminGuard]
        //},
    ]
    ``` 

### Useful-links

- [Youtube walkthrough](https://www.youtube.com/@callmeks24) - Coming Soon
- [Authorization Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authorization_Cheat_Sheet.html)


