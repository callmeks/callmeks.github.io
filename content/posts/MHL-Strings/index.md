---
date: '2025-03-21T20:35:52+08:00'
title: 'MHL Strings'
tags:
    - MHL
    - Android CTF
---

## Challenge Description

> Welcome to the Strings Challenge! In this lab,your goal is to find the flag. The flag's format should be "MHL{...}". The challenge will give you a clear idea of how intents and intent filters work on android also you will get a hands-on experience using Frida APIs.
>
> [Strings.apk](static/Strings.apk)

## Solution

This is interesting challenge. I started with static analysis first.

### Static Analysis

As usual, I used `jadx-gui` to have a look at the source code.

```xml
 <activity
    android:name="com.mobilehackinglab.challenge.Activity2"
    android:exported="true">
    <intent-filter>
        <action android:name="android.intent.action.VIEW"/>
        <category android:name="android.intent.category.DEFAULT"/>
        <category android:name="android.intent.category.BROWSABLE"/>
        <data
            android:scheme="mhl"
            android:host="labs"/>
    </intent-filter>
</activity>
<activity
    android:name="com.mobilehackinglab.challenge.MainActivity"
    android:exported="true">
    <intent-filter>
        <action android:name="android.intent.action.MAIN"/>
        <category android:name="android.intent.category.LAUNCHER"/>
    </intent-filter>
</activity>
```

After looking into the `AndroidManifest.xml`, I found 2 interesting activities which have `exported=true`. `Activity2` has this `intent-filter` where it accept `scheme="mhl"` and `host="labs"`. Moving on, I looked into `MainActivity` source code.

```java
public final class MainActivity extends AppCompatActivity {
    private ActivityMainBinding binding;

    public final native String stringFromJNI();

    @Override // androidx.fragment.app.FragmentActivity, androidx.activity.ComponentActivity, androidx.core.app.ComponentActivity, android.app.Activity
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        ActivityMainBinding inflate = ActivityMainBinding.inflate(getLayoutInflater());
        Intrinsics.checkNotNullExpressionValue(inflate, "inflate(...)");
        this.binding = inflate;
        ActivityMainBinding activityMainBinding = null;
        if (inflate == null) {
            Intrinsics.throwUninitializedPropertyAccessException("binding");
            inflate = null;
        }
        setContentView(inflate.getRoot());
        ActivityMainBinding activityMainBinding2 = this.binding;
        if (activityMainBinding2 == null) {
            Intrinsics.throwUninitializedPropertyAccessException("binding");
        } else {
            activityMainBinding = activityMainBinding2;
        }
        activityMainBinding.sampleText.setText(stringFromJNI());
    }

    static {
        System.loadLibrary("challenge");
    }

    public final void KLOW() {
        SharedPreferences sharedPreferences = getSharedPreferences("DAD4", 0);
        SharedPreferences.Editor editor = sharedPreferences.edit();
        Intrinsics.checkNotNullExpressionValue(editor, "edit(...)");
        SimpleDateFormat sdf = new SimpleDateFormat("dd/MM/yyyy", Locale.getDefault());
        String cu_d = sdf.format(new Date());
        editor.putString("UUU0133", cu_d);
        editor.apply();
    }
}
```

After reading the code, it use this `System.loadLibrary` which mean it is using a native library. The function from native library could be easily identified as it will have the strings `native` in the function. Based on what I understand, the native library basically providing a output or string. Moving to the `KLOW` function, it seems to be not used in the `MainActivity`. The code seems to be writing information into the shared preferences file.

```Java
@Override // androidx.fragment.app.FragmentActivity, androidx.activity.ComponentActivity, androidx.core.app.ComponentActivity, android.app.Activity
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_2);
    SharedPreferences sharedPreferences = getSharedPreferences("DAD4", 0);
    String u_1 = sharedPreferences.getString("UUU0133", null);
    boolean isActionView = Intrinsics.areEqual(getIntent().getAction(), "android.intent.action.VIEW");
    boolean isU1Matching = Intrinsics.areEqual(u_1, cd());
    if (isActionView && isU1Matching) {
        Uri uri = getIntent().getData();
        if (uri != null && Intrinsics.areEqual(uri.getScheme(), "mhl") && Intrinsics.areEqual(uri.getHost(), "labs")) {
            String base64Value = uri.getLastPathSegment();
            byte[] decodedValue = Base64.decode(base64Value, 0);
            if (decodedValue != null) {
                String ds = new String(decodedValue, Charsets.UTF_8);
                byte[] bytes = "your_secret_key_1234567890123456".getBytes(Charsets.UTF_8);
                Intrinsics.checkNotNullExpressionValue(bytes, "this as java.lang.String).getBytes(charset)");
                String str = decrypt("AES/CBC/PKCS5Padding", "bqGrDKdQ8zo26HflRsGvVA==", new SecretKeySpec(bytes, "AES"));
                if (str.equals(ds)) {
                    System.loadLibrary("flag");
                    String s = getflag();
                    Toast.makeText(getApplicationContext(), s, 1).show();
                    return;
                } else {
                    finishAffinity();
                    finish();
                    System.exit(0);
                    return;
                }
            }
            finishAffinity();
            finish();
            System.exit(0);
            return;
        }
        finishAffinity();
        finish();
        System.exit(0);
        return;
    }
    finishAffinity();
    finish();
    System.exit(0);
}
```

As for the `Activity2` activity, it has this `onCreate` function which will first check for the shared preferences file and make sure the information is same. After that, it will check for the incoming intent. It will first get the data from the intent and check if the data match the URI requirement (`mhl`,`labs`) and get the last part (`getLastPathSegment`) if its URI. After getting the last part, it will just base64 decode it and use it to compare with the result of the AES decryption. if the comparision is corect, it will load another native library and get a string or output from the library. 

```Java
public final String decrypt(String algorithm, String cipherText, SecretKeySpec key) {
    Intrinsics.checkNotNullParameter(algorithm, "algorithm");
    Intrinsics.checkNotNullParameter(cipherText, "cipherText");
    Intrinsics.checkNotNullParameter(key, "key");
    Cipher cipher = Cipher.getInstance(algorithm);
    try {
        byte[] bytes = Activity2Kt.fixedIV.getBytes(Charsets.UTF_8);
        Intrinsics.checkNotNullExpressionValue(bytes, "this as java.lang.String).getBytes(charset)");
        IvParameterSpec ivSpec = new IvParameterSpec(bytes);
        cipher.init(2, key, ivSpec);
        byte[] decodedCipherText = Base64.decode(cipherText, 0);
        byte[] decrypted = cipher.doFinal(decodedCipherText);
        Intrinsics.checkNotNull(decrypted);
        return new String(decrypted, Charsets.UTF_8);
    } catch (Exception e) {
        throw new RuntimeException("Decryption failed", e);
    }
}
```

Moving the the `decrypt` function, this is basically just simple AES decryption where the encrypted strings (`bqGrDKdQ8zo26HflRsGvVA==`) and keys (`your_secret_key_1234567890123456`) is provided in the `onCreate` function. as for the IV, it is in the `Activity2Kt.fixedIV` which is `1234567890123456`. After having all this information, it is possible to decrypt and get the correct strings easily.

![alt text](img/index.png#center)

Now that I have everything I needed, time to move on to dynamic analysis to test if my understanding is correct or not. 

### Dynamic Analysis

I first started by running the apk and see if my assumption is correct. 

![alt text](img/index-1.png#center)

The strings seems to be coming out from the native library which what I expected.

```bash
beryllium:/ # ls -la /data/data/com.mobilehackinglab.challenge/
total 40
drwx------   4 u0_a229 u0_a229        4096 2025-03-21 21:24 .
drwxrwx--x 322 system  system        20480 2025-03-20 23:07 ..
drwxrws--x   2 u0_a229 u0_a229_cache  4096 2025-03-21 21:24 cache
drwxrwx--x   2 u0_a229 u0_a229        4096 2025-03-21 21:24 files
```

I then tried to see if the `KLOW` function is executed or not by looking for the shared preferences file. It seems like it is not executed. I then tried to use `frida` to execute the function. 

```js
Java.perform(()=> {

    Java.choose("com.mobilehackinglab.challenge.MainActivity", {
        onMatch: function (instance) {
            console.log("Found MainActivity instance:", instance);
            console.log("called KLOW function:", instance.KLOW());
        },
        onComplete: function() {}
    });

});
```

By using this frida script, I managed to execute the `KLOW` function. 

```bash
beryllium:/ # ls -la /data/data/com.mobilehackinglab.challenge/shared_prefs/
total 16
drwxrwx--x 2 u0_a229 u0_a229 4096 2025-03-21 21:29 .
drwx------ 6 u0_a229 u0_a229 4096 2025-03-21 21:28 ..
-rw-rw---- 1 u0_a229 u0_a229  117 2025-03-21 21:29 DAD4.xml
```

Moving on to the `Activity2` activity, the only way to trigger it is abusing the `exported=true` and providing the required intent. I managed to use frida script to send the intent which match the intent filter and the required information.

```js
setTimeout( ()=> {
    const ctx = Java.use("android.app.ActivityThread")
    .currentActivityThread()
    .getApplication()
    .getApplicationContext();
    const A2 = Java.use("com.mobilehackinglab.challenge.Activity2");
    let intent = Java.use("android.content.Intent").$new(ctx, A2.class);
    intent.setAction("android.intent.action.VIEW");
    intent.setData(Java.use("android.net.Uri").parse("mhl://labs/bWhsX3NlY3JldF8xMzM3"));
    intent.setFlags(0x10000000); // FLAG_ACTIVITY_NEW_TASK
    ctx.startActivity(intent);
    console.log("Intent Sent",intent);
},3000);
```

This is the code to basically send the intent. The data is set to provide the required information `mhl_secret_1337` in base64 encoded. Here's what happen after reaching the code where it will load the other native library and peform a toast.

![alt text](img/index-2.png#center)

Although I managed to get the string `success`, I still could not get the flag. I then tried to see what happened using `frida-trace` to check what is happening.

```powershell
PS E:\Desktop\Android\frida-script> frida-trace -U -N com.mobilehackinglab.challenge -I "libflag.so"
Instrumenting...

Started tracing 14 functions. Web UI available at http://localhost:59982/
 12103 ms  Java_com_mobilehackinglab_challenge_Activity2_getflag()
 12103 ms     | _Z9flag_fillv()
 12103 ms     | _Z6flag13v()
 12103 ms     | _Z3ZGSi()
 12103 ms     | _Z3tyyii()
 12103 ms     | _Z4asdsi()
 12103 ms     | _Z3zhsv()
 12103 ms     | _Z2x1ii()
 12103 ms     | _Z2ssi()
 12103 ms     | _Z9flag_fillv()
 12103 ms     | _Z6flag13v()
 12103 ms     | _Z6flag13v()
 12103 ms     | _Z6flag13v()
 12103 ms     | _Z3ZGSi()
 12103 ms     | _Z3tyyii()
 12103 ms     | _Z4asdsi()
 12103 ms     | _Z3zhsv()
 12103 ms     | _Z5dddffv()
 12103 ms     | _Z2ttv()
 12103 ms     | _Z2ddv()
 12103 ms     | _Z2x1ii()
 12103 ms     | _Z2ssi()
 12103 ms     | _Z2x1ii()
 12103 ms     | _Z2ssi()
 12103 ms     | _Z5dddffv()
 12103 ms     | _Z3zhsv()
```

It looks like there's a lot of function executed in the native library. I then look into the hints and rules provided.

> - Reverse engineer the application and search for exported activities.
> - Understand the code and find a method to invoke the exported activity.
> - Utilize Frida for tracing or employ Frida's memory scanning.
> - Don't have to spend time on static analysis of the Android library, as the code is obfuscated.
> - The flag follows the format "MHL{...}".
> - Do not attempt to patch the application.

Based on the hints and rules, it looks like i will need to focus on the frida's memory scanning. I looked into the [frida official documentation](https://frida.re/docs/javascript-api/#memory) to understand more. It has some code example as well. After understanding it, it basically used to looks for specific "pattern". The pattern is basically the character that I wanted to find.

```js
setTimeout(() => {
    let m = Process.getModuleByName("libflag.so");
    let pattern = "4d 48 4c";
    Memory.scan(m.base, m.size, pattern, {
        onMatch(address, size) {
            console.log('Memory.scan() found match at', address,'with size', size);
            console.log('Flag found: ',Memory.readCString(address));
            console.log(hexdump(address));
        }
    });
}, 1000);
```

After modifying abit of the code, here's how it looks like. The pattern is actually `MHL` but in hex and it will try to search for this hex in the memory. Here's how the result looks like.

```bash
[POCOPHONE F1::com.mobilehackinglab.challenge ]-> Memory.scan() found match at 0x7e5276805c with size 3
Flag found:  MHL{IN_THE_MEMORY}
             0  1  2  3  4  5  6  7  8  9  A  B  C  D  E  F  0123456789ABCDEF
7e5276805c  4d 48 4c 7b 49 4e 5f 54 48 45 5f 4d 45 4d 4f 52  MHL{IN_THE_MEMOR
7e5276806c  59 7d 00 00 00 00 00 00 00 00 00 00 00 00 00 00  Y}..............
7e5276807c  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
7e5276808c  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
7e5276809c  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
7e527680ac  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
7e527680bc  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
7e527680cc  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
7e527680dc  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
7e527680ec  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
7e527680fc  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
7e5276810c  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
7e5276811c  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
7e5276812c  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
7e5276813c  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
7e5276814c  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
```

### Frida Script

Here's the full frida script to let everything run smoothly and get the flag.

```js
Java.perform(function () {

    Java.choose("com.mobilehackinglab.challenge.MainActivity", {
        onMatch: function (instance) {
            console.log("Found MainActivity instance:", instance);
            console.log("called KLOW function:", instance.KLOW());
        },
        onComplete: function() {
            sendIntent();
        }
    });

});
  
function sendIntent(){
    setTimeout( ()=> {
        const ctx = Java.use("android.app.ActivityThread")
        .currentActivityThread()
        .getApplication()
        .getApplicationContext();
        const A2 = Java.use("com.mobilehackinglab.challenge.Activity2");
        let intent = Java.use("android.content.Intent").$new(ctx, A2.class);
        intent.setAction("android.intent.action.VIEW");
        intent.setData(Java.use("android.net.Uri").parse("mhl://labs/bWhsX3NlY3JldF8xMzM3"));
        intent.setFlags(0x10000000); // FLAG_ACTIVITY_NEW_TASK
        ctx.startActivity(intent);
        console.log("Intent Sent",intent);
        readMemoryfromNativeLib();
    },3000);
}

function readMemoryfromNativeLib(){
    setTimeout(() => {
        let m = Process.getModuleByName("libflag.so");
        let pattern = "4d 48 4c 7b";
        Memory.scan(m.base, m.size, pattern, {
            onMatch(address, size) {
                console.log('Memory.scan() found match at', address,'with size', size);
                console.log('Flag found: ',Memory.readCString(address));
                console.log(hexdump(address));
            }
        });
    }, 1000);
      
}
```

## Things I learned from this challenge

- frida script to execute a function, send a intent and read memory from native library.