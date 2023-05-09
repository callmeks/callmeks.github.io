---
title: OWASP Juice Shop - Login Admin
categories: [writeups,OWASP-juice-shop]
tags: [web]
---

## Login Admin

```
- Difficulty: 2 star
- Category: Injection
- Description: Log in with the administrator's user account.
```

### Method to solve

- In this challenge, we are asked to login into administrator's user account and the given category is injection.
- With the information, the challenge is most probably related to SQL Injection.
<br>
![](/assets/img/posts/2023-04-30-OWASP-Juice-Shop-Login-Admin/2023-04-30-OWASP-Juice-Shop-Login-Admin-22-28-57.png)
<br>
- Based on the login page, we are required to provide admin's email and password.
- By using SQL injection payload, `' OR 1=1 -- -`, we are able to login into admin's account.
<br>
![](/assets/img/posts/2023-04-30-OWASP-Juice-Shop-Login-Admin/2023-04-30-OWASP-Juice-Shop-Login-Admin-22-31-05.png)
<br>
- After adding the payload into both username and password, click login button and we will be login as admin.
<br>
![](/assets/img/posts/2023-04-30-OWASP-Juice-Shop-Login-Admin/2023-04-30-OWASP-Juice-Shop-Login-Admin-22-31-54.png)
<br>


### Code Review

- Vulnerable code
    ```ts
    module.exports = function login () {
    function afterLogin (user: { data: User, bid: number }, res: Response, next: NextFunction) {
        BasketModel.findOrCreate({ where: { UserId: user.data.id } })
        .then(([basket]: [BasketModel, boolean]) => {
            const token = security.authorize(user)
            user.bid = basket.id // keep track of original basket
            security.authenticatedUsers.put(token, user)
            res.json({ authentication: { token, bid: basket.id, umail: user.data.email } })
        }).catch((error: Error) => {
            next(error)
        })
    }
    
    return (req: Request, res: Response, next: NextFunction) => {
        models.sequelize.query(`SELECT * FROM Users WHERE email = '${req.body.email || ''}' AND password = '${security.hash(req.body.password || '')}' AND deletedAt IS NULL`, { model: UserModel, plain: true }) //Vulnerable
        .then((authenticatedUser: { data: User }) => {
            const user = utils.queryResultToJson(authenticatedUser)
            if (user.data?.id && user.data.totpSecret !== '') {
            res.status(401).json({
                status: 'totp_token_required',
                data: {
                tmpToken: security.authorize({
                    userId: user.data.id,
                    type: 'password_valid_needs_second_factor_token'
                })
                }
            })
            } else if (user.data?.id) {
            afterLogin(user, res, next)
            } else {
            res.status(401).send(res.__('Invalid email or password.'))
            }
        }).catch((error: Error) => {
            next(error)
        })
    }
    ```
    - The reason it is vulnerable to SQL injection is that the username and password was directly send into SQL query without any sanitization.
    - This allow attacker to modify the SQL query from validating username and password to something else as they will have full control of the SQL query.
- Solution
    ```ts
    import {BasketModel} from "../../../models/basket";  //import basket

    module.exports = function login () {
    function afterLogin (user: { data: User, bid: number }, res: Response, next: NextFunction) {
        BasketModel.findOrCreate({ where: { UserId: user.data.id } })
        .then(([basket]: [BasketModel, boolean]) => {
            const token = security.authorize(user)
            user.bid = basket.id // keep track of original basket
            security.authenticatedUsers.put(token, user)
            res.json({ authentication: { token, bid: basket.id, umail: user.data.email } })
        }).catch((error: Error) => {
            next(error)
        })
    }
    
    return (req: Request, res: Response, next: NextFunction) => {
        models.sequelize.query(`SELECT * FROM Users WHERE email = $1 AND password = $2 AND deletedAt IS NULL`,
        { bind: [ req.body.email, security.hash(req.body.password) ], model: models.User, plain: true }) // make input as plain text to escape SQL injection
        .then((authenticatedUser: { data: User }) => {
            const user = utils.queryResultToJson(authenticatedUser)
            if (user.data?.id && user.data.totpSecret !== '') {
            res.status(401).json({
                status: 'totp_token_required',
                data: {
                tmpToken: security.authorize({
                    userId: user.data.id,
                    type: 'password_valid_needs_second_factor_token'
                })
                }
            })
            } else if (user.data?.id) {
            afterLogin(user, res, next)
            } else {
            res.status(401).send(res.__('Invalid email or password.'))
            }
        }).catch((error: Error) => {
            next(error)
        })
    }
    ```

### Useful-links

- [Youtube walkthrough](https://youtu.be/Zympr5tU_ps)
- [SQL Injection Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html)
- [SQL Injection](https://portswigger.net/web-security/sql-injection)



