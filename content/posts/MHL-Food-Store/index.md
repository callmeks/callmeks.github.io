---
date: '2025-04-03T22:58:30+08:00'
title: 'MHL Food Store'
tags:
    - MHL
    - Android CTF
---

## Challenge Description

> Welcome to the Android App Security Lab: SQL Injection Challenge! Dive into the world of cybersecurity with our hands-on lab. This challenge is centered around a fictitious "Food Store" app, highlighting the critical security flaw of SQL Injection (SQLi) within the app's framework.
>
> [foodstore.apk](static/foodstore.apk)

## Solution

As usual, static analysis to understand first.

### Static Analysis

I started out by reading the `AndroidManifest.xml` code first.

```xml
<activity
    android:name="com.mobilehackinglab.foodstore.Signup"
    android:exported="false"/>
<activity
    android:name="com.mobilehackinglab.foodstore.MainActivity"
    android:exported="true"/>
<activity
    android:name="com.mobilehackinglab.foodstore.LoginActivity"
    android:exported="true">
    <intent-filter>
        <action android:name="android.intent.action.MAIN"/>
        <category android:name="android.intent.category.LAUNCHER"/>
    </intent-filter>
</activity>
```

There is a `Signup`, `MainActivity` and `LoginActivity` activity but only `Signup` activity is not exported. Reading the objective provided, this challenge will be focused in the signup function.

> Exploit a SQL Injection Vulnerability: Your mission is to manipulate the signup function in the "Food Store" Android application, allowing you to register as a Pro user, bypassing standard user restrictions.

In that case, I start reading the `Signup` activity first.

```java
public static final void onCreate$lambda$0(Signup this$0, View it) {
    Intrinsics.checkNotNullParameter(this$0, "this$0");
    EditText editText = this$0.username;
    EditText editText2 = null;
    if (editText == null) {
        Intrinsics.throwUninitializedPropertyAccessException("username");
        editText = null;
    }
    if (!(StringsKt.trim((CharSequence) editText.getText().toString()).toString().length() == 0)) {
        EditText editText3 = this$0.password;
        if (editText3 == null) {
            Intrinsics.throwUninitializedPropertyAccessException("password");
            editText3 = null;
        }
        if (!(StringsKt.trim((CharSequence) editText3.getText().toString()).toString().length() == 0)) {
            EditText editText4 = this$0.address;
            if (editText4 == null) {
                Intrinsics.throwUninitializedPropertyAccessException("address");
                editText4 = null;
            }
            if (!(StringsKt.trim((CharSequence) editText4.getText().toString()).toString().length() == 0)) {
                DBHelper dbHelper = new DBHelper(this$0);
                int i = 0;
                EditText editText5 = this$0.username;
                if (editText5 == null) {
                    Intrinsics.throwUninitializedPropertyAccessException("username");
                    editText5 = null;
                }
                String obj = editText5.getText().toString();
                EditText editText6 = this$0.password;
                if (editText6 == null) {
                    Intrinsics.throwUninitializedPropertyAccessException("password");
                    editText6 = null;
                }
                String obj2 = editText6.getText().toString();
                EditText editText7 = this$0.address;
                if (editText7 == null) {
                    Intrinsics.throwUninitializedPropertyAccessException("address");
                } else {
                    editText2 = editText7;
                }
                User newUser = new User(i, obj, obj2, editText2.getText().toString(), false, 1, null);
                dbHelper.addUser(newUser);
                Toast.makeText(this$0, "User Registered Successfully", 0).show();
                return;
            }
        }
    }
    Toast.makeText(this$0, "Please fill in all fields", 0).show();
}
```

Looking into the code, it basically just getting strings from each text field and send the information into `dbHelper.addUser(newUser)` function.

```java
public final void addUser(User user) {
    Intrinsics.checkNotNullParameter(user, "user");
    SQLiteDatabase db = getWritableDatabase();
    byte[] bytes = user.getPassword().getBytes(Charsets.UTF_8);
    Intrinsics.checkNotNullExpressionValue(bytes, "this as java.lang.String).getBytes(charset)");
    String encodedPassword = Base64.encodeToString(bytes, 0);
    String Username = user.getUsername();
    byte[] bytes2 = user.getAddress().getBytes(Charsets.UTF_8);
    Intrinsics.checkNotNullExpressionValue(bytes2, "this as java.lang.String).getBytes(charset)");
    String encodedAddress = Base64.encodeToString(bytes2, 0);
    String sql = "INSERT INTO users (username, password, address, isPro) VALUES ('" + Username + "', '" + encodedPassword + "', '" + encodedAddress + "', 0)";
    db.execSQL(sql);
    db.close();
}
```

After looking into the `addUser` function code, the first thing i noticed is this `+ Username +` which definitely vulnerable to SQL injection. Although `encodedPassword` and `encodedAddress` is also vulnerable to SQL injection, it is harder to perform attacks as the input will be base64 encode which mean I can only inject in the `Username` parameter. Now that I understand how things work, time to perform dynamic analysis to test out my theory. 

## Dynamic analysis

I immediately started out by testing the signup function. 

![alt text](img/index.png#center)

I tried out simple SQL injection technique first by putting a single quote and I got some interesting error.

```bash
android.database.sqlite.SQLiteException: near "Yg": syntax error (code 1 SQLITE_ERROR): , while compiling: INSERT INTO users (username, password, address, isPro) VALUES ('a'', 'Yg==
', 'Yg==
', 0)
```

I then tried to craft a simple payload like `a');--` to see if it accept.

```bash
android.database.sqlite.SQLiteException: 1 values for 4 columns (code 1 SQLITE_ERROR): , while compiling: INSERT INTO users (username, password, address, isPro) VALUES ('a');--', 'Yg==
', 'Yg==
', 0)
```

Based on the error, I managed to inject data into it but it needs 4 values but I only provided one. I then craft another one `a','Yg==','Yg==',1);--`. The last value need to changed to 1 because of the objective where I need to create a Pro account.

![alt text](img/index-1.png#center)

This time, I did not get any error which mean I successfully inject my payload via SQL injection. I then login using the user and see if I am Pro user.

![alt text](img/index-2.png#center)

I am now a Pro user according to the image which mean I successfully performed the SQL injection.

## Things I learned from this challenge

- SQL injection source code review in Java