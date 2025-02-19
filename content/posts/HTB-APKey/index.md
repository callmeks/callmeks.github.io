---
date: '2025-02-19T20:05:18+08:00'
title: 'HTB APKey'
tags:
    - Android CTF
    - HTB
---

## Challenge Description

> This app contains some unique keys. Can you get one?
>
> [APKey.apk](static/APKey.apk)

## Solution

### Static Analysis

The first step is always static analysis. I started by using `jadx-gui` to see the decompiled Java code. Looking into the `AndroidManifest.xml`, it looks like there's only one activity which is `MainActivity`.


```xml
<application
    android:theme="@style/Theme.APKey"
    android:label="@string/app_name"
    android:icon="@mipmap/ic_launcher"
    android:allowBackup="true"
    android:supportsRtl="true"
    android:roundIcon="@mipmap/ic_launcher_round"
    android:appComponentFactory="androidx.core.app.CoreComponentFactory">
    <activity android:name="com.example.apkey.MainActivity">
        <intent-filter>
            <action android:name="android.intent.action.MAIN"/>
            <category android:name="android.intent.category.LAUNCHER"/>
        </intent-filter>
    </activity>
</application>
```

Since it's gonna be `MainActivity`, we could have a look at the code. Inside the code, we have something interesting.


```java
public void onClick(View view) {
    Toast makeText;
    String str;
    try {
        if (MainActivity.this.f928c.getText().toString().equals("admin")) {
            MainActivity mainActivity = MainActivity.this;
            b bVar = mainActivity.e;
            String obj = mainActivity.d.getText().toString();
            try {
                MessageDigest messageDigest = MessageDigest.getInstance("MD5");
                messageDigest.update(obj.getBytes());
                byte[] digest = messageDigest.digest();
                StringBuffer stringBuffer = new StringBuffer();
                for (byte b2 : digest) {
                    stringBuffer.append(Integer.toHexString(b2 & 255));
                }
                str = stringBuffer.toString();
            } catch (NoSuchAlgorithmException e) {
                e.printStackTrace();
                str = "";
            }
            if (str.equals("a2a3d412e92d896134d9c9126d756f")) {
                Context applicationContext = MainActivity.this.getApplicationContext();
                MainActivity mainActivity2 = MainActivity.this;
                b bVar2 = mainActivity2.e;
                g gVar = mainActivity2.f;
                makeText = Toast.makeText(applicationContext, b.a(g.a()), 1);
                makeText.show();
            }
        }
        makeText = Toast.makeText(MainActivity.this.getApplicationContext(), "Wrong Credentials!", 0);
        makeText.show();
    } catch (Exception e2) {
        e2.printStackTrace();
    }
}
```

Looking into this part of the code, I could see there's a string comparison for `admin` and a function that seems to be MD5 but the strings is 31 characters. Well since I have no idea, I decided to look into the `toast.makeText` as it seems like the function to get flag. It's playing around `b.a(g.a())`.

```java
public class g {
    public static String a() {
        StringBuilder sb = new StringBuilder();
        ArrayList arrayList = new ArrayList();
        arrayList.add("722gFc");
        arrayList.add("n778Hk");
        arrayList.add("jvC5bH");
        arrayList.add("lSu6G6");
        arrayList.add("HG36Hj");
        arrayList.add("97y43E");
        arrayList.add("kjHf5d");
        arrayList.add("85tR5d");
        arrayList.add("1UlBm2");
        arrayList.add("kI94fD");
        sb.append((String) arrayList.get(8));
        sb.append(h.a());
        sb.append(i.a());
        sb.append(f.a());
        sb.append(e.a());
        ArrayList arrayList2 = new ArrayList();
        arrayList2.add("ue7888");
        arrayList2.add("6HxWkw");
        arrayList2.add("gGhy77");
        arrayList2.add("837gtG");
        arrayList2.add("HyTg67");
        arrayList2.add("GHR673");
        arrayList2.add("ftr56r");
        arrayList2.add("kikoi9");
        arrayList2.add("kdoO0o");
        arrayList2.add("2DabnR");
        sb.append((String) arrayList2.get(9));
        sb.append(c.a());
        ArrayList arrayList3 = new ArrayList();
        arrayList3.add("jH67k8");
        arrayList3.add("8Huk89");
        arrayList3.add("fr5GtE");
        arrayList3.add("Hg5f6Y");
        arrayList3.add("o0J8G5");
        arrayList3.add("Wod2bk");
        arrayList3.add("Yuu7Y5");
        arrayList3.add("kI9ko0");
        arrayList3.add("dS4Er5");
        arrayList3.add("h93Fr5");
        sb.append((String) arrayList3.get(5));
        sb.append(d.a());
        sb.append(a.a());
        return sb.toString();
    }

    public static String b() {
        return String.valueOf(d.a().charAt(1)) + String.valueOf(i.a().charAt(2)) + String.valueOf(i.a().charAt(1));
    }
}
```

Looking into `g`, there's function `a` and `b` and function `a` seems to be something I want but it has a lot of random functions from different class.

### Dynamic Analysis

Since there's a lot of random function, I decided to perform dynamic analysis by using `frida` to force executing the Java code. To do so, I'll need to use `objection` to patch the APK. 

```powershell
PS D:\> objection patchapk -s .\APKey.apk
No architecture specified. Determining it using `adb`...
Detected target device architecture as: arm64-v8a
...
```

After patching and installing it to my devices, it should be ready to use. Since it is patched using `objection`, I'll need to use `frida` to hook it to let the APK running as intended.

```powershell
 frida -U -N com.example.apkey -l .\frida.js
     ____
    / _  |   Frida 16.5.9 - A world-class dynamic instrumentation toolkit
   | (_| |
    > _  |   Commands:
   /_/ |_|       help      -> Displays the help system
   . . . .       object?   -> Display information about 'object'
   . . . .       exit/quit -> Exit
   . . . .
   . . . .   More info at https://frida.re/docs/home/
   . . . .
   . . . .   Connected to POCOPHONE F1 (id=d211a91c)
```

After hooking it, I could see the app working as intended.

![alt text](img/index.png#center)

This looks like it's related to the function where it checks for `admin` and a weird 31 hash. Since this is useless, I then continue with what i wanted to do which is using `frida` to execute the Java function.

```js
Java.perform(()=> {
    let g = Java.use("c.b.a.g");
    let test = g.a();
    console.log("g.a() result = " + test);
});
```

By using this `frida` script, I could instantly get the result of the Java code.

```powershell
[POCOPHONE F1::com.example.apkey ]-> %reload
g.a() result = 1UlBm2kHtZuVrSE6qY6HxWkwHyeaX92DabnRFlEGyLWod2bkwAxcoc85S94kFpV1
```

This seems to be another weird strings which I have no idea what it is. I then remembered that the function is `b.a(g.a())` which means this strings works as a key for `b.a()` functions. I then changed my script accordingly.

```js
Java.perform(()=> {
    let g = Java.use("c.b.a.g");
    let b = Java.use("c.b.a.b");
    let test = g.a();
    console.log("g.a() result = " + test);
    let result = b.a(test);
    console.log("b.a() result = " + result);
});
```

I then get some good result from the frida script.

```powershell
[POCOPHONE F1::com.example.apkey ]-> %reload
g.a() result = 1UlBm2kHtZuVrSE6qY6HxWkwHyeaX92DabnRFlEGyLWod2bkwAxcoc85S94kFpV1
b.a() result = HTB{m0r3_0bfusc4t1on_w0uld_n0t_hurt}
```

## Things I learned from this challenge

- Frida Scripting
