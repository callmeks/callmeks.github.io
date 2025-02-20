---
date: '2025-02-19T22:38:23+08:00'
title: 'HTB SAW'
tags:
    - Android CTF
    - HTB
---

## Challenge Description

> The malware forensics lab identified a new technique for hiding and executing code dynamically. A sample that seems to use this technique has just arrived in their queue. Can you help them?
>
> [SAW.apk](static/SAW.apk)

## Solution

This is something that I think quite hard but yea another fun challenge.

### Static Analysis

As usual, I started with `jadx-gui` for reading the decompiled Java code. 

```xml
<activity android:name="com.stego.saw.MainActivity">
    <intent-filter>
        <action android:name="android.intent.action.MAIN"/>
        <category android:name="android.intent.category.LAUNCHER"/>
    </intent-filter>
</activity>
```

It seems like there's only one activity to focus on. I then have a look in it. Inside the `MainActivity.java`, there's a few that I think its interesting and useful.

```java
public native String a(String str, String str2);

static {
    System.loadLibrary("default");
}
```

This is the code where it use native library. This means that I'll need to have a look at the native library which can be found in `Resources > lib > x86 (or any other) > libdefault.so` inside the `jadx-gui`.

```java
Bundle extras = getIntent().getExtras();
if (extras == null) {
    finish();
    return;
}
if (!extras.getString("open").equalsIgnoreCase("sesame")) {
    finish();
    return;
}
```

This code is interesting as it requires user to provide extra strings inside the Intent in order to open the apps. This means that opening directly is impossible in this case. 

```java
public void f() {
    WindowManager windowManager = (WindowManager) getSystemService("window");
    WindowManager.LayoutParams layoutParams = new WindowManager.LayoutParams(200, 200, 2038, 8, -2);
    layoutParams.gravity = 17;
    Button button = new Button(getApplicationContext());
    button.setOnClickListener(new View.OnClickListener() { // from class: com.stego.saw.MainActivity.2
        @Override // android.view.View.OnClickListener
        public void onClick(View view) {
            MainActivity.this.alert();
        }
    });
    windowManager.addView(button, layoutParams);
}
```

A weird `f` function that seems to be related to window manager. After asking big boss CHATGPT, it's trying to create an overlay button. The number `2038` is [TYPE_APPLICATION_OVERLAY](https://developer.android.com/reference/android/view/WindowManager.LayoutParams#TYPE_APPLICATION_OVERLAY) and it requires some permission to make it work.

```java
public final String alert() {
    final EditText editText = new EditText(this);
    new AlertDialog.Builder(this).setTitle("XOR XOR XOR").setMessage("XOR ME !").setView(editText).setPositiveButton("XORIFY", new DialogInterface.OnClickListener() { // from class: com.stego.saw.MainActivity.4
        @Override // android.content.DialogInterface.OnClickListener
        public void onClick(DialogInterface dialogInterface, int i) {
            MainActivity.this.answer = editText.getText().toString();
            MainActivity mainActivity = MainActivity.this;
            mainActivity.a(mainActivity.FILE_PATH_PREFIX, MainActivity.this.answer);
        }
    }).setNegativeButton("Cancel", new DialogInterface.OnClickListener() { // from class: com.stego.saw.MainActivity.3
        @Override // android.content.DialogInterface.OnClickListener
        public void onClick(DialogInterface dialogInterface, int i) {
            MainActivity.this.finish();
        }
    }).show();
    return this.answer;
}
```

This one is the `alert` function where it seems to be some kind of XOR ?? From what I know, this seems to be taking one input and try to send it into `a(<filepath>,<input>)` and the `a` function should be coming from native library. 

### Exploring Native Library

This can be easily done by exporting the `libdefault.so` from `jadx-gui` and open it in `ghidra`.

```c
void a(_JNIEnv *param_1,_jobject *param_2,_jstring *param_3,_jstring *param_4)

{
  char *pcVar1;
  char *pcVar2;
  
  pcVar1 = (char *)(**(code **)(*(int *)param_1 + 0x2a4))(param_1,param_3,0,0x10ab1);
  pcVar2 = (char *)(**(code **)(*(int *)param_1 + 0x2a4))(param_1,param_4,0);
  _Z1aP7_JNIEnvP8_1(pcVar1,pcVar2);
  (**(code **)(*(int *)param_1 + 0x2a8))(param_1,param_3,pcVar1);
  (**(code **)(*(int *)param_1 + 0x29c))(param_1,pcVar1);
  return;
}
```

After going through abit, I started by looking into the function `a`. Based on the Java code, it takes in 2 input. The function `a` decompiled by `ghidra` has 4 argument, which I assume the last 2 is the one that the input is placed. Based on the code, it seems like it's taking `param_3` and `param_4` as `pcVar1` and `pcVar2` which then put into function `_Z1aP7_JNIEnvP8_1`. Focusing on the function, I noticed that theres a weird function that seems to be interested. 

```c
undefined4 _Z1aP7_JNIEnvP8_1(char *param_1,char *param_2)
...
if (((int)*param_2 ^ l) == m) {
    if ((((((int)param_2[1] ^ DAT_00013a18) == DAT_00013a38) &&
        (((int)param_2[2] ^ DAT_00013a1c) == DAT_00013a3c)) &&
        (((int)param_2[3] ^ DAT_00013a20) == DAT_00013a40)) &&
    (((((int)param_2[4] ^ DAT_00013a24) == DAT_00013a44 &&
        (((int)param_2[5] ^ DAT_00013a28) == DAT_00013a48)) &&
        (((int)param_2[6] ^ DAT_00013a2c) == DAT_00013a4c)))) {
    uVar13 = 1;
    ppuVar12 = apuStack_c80 + 0x314;
    if (((int)param_2[7] ^ DAT_00013a30) == DAT_00013a50)
```

While its abit messy, I noticed that its trying to take the characters of `param_2` and try to XOR with some random variable. After understanding it, it seems like the variable consist of some hex numbers. here's all the hex number after getting it.

```txt
l = 0x0A
DAT_00013a18 = 0x0B
DAT_00013a1c = 0x18
DAT_00013a20 = 0x0F
DAT_00013a24 = 0x5E
DAT_00013a28 = 0x31
DAT_00013a2c = 0x0C
DAT_00013a30 = 0x0F

m = 0x6C
DAT_00013a38 = 0x67
DAT_00013a3c = 0x28
DAT_00013a40 = 0x6E
DAT_00013a44 = 0x2A
DAT_00013a48 = 0x58
DAT_00013a4c = 0x62
DAT_00013a50 = 0x68
```

Now that I have the hex, I could try to get the correct input by just XOR. 

```c
sVar9 = strlen(param_1);
local_2c = (char *)calloc(sVar9 + 2,1);
strcpy(local_2c,param_1);
sVar9 = strlen(local_2c);
(local_2c + sVar9)[0] = 'h';
(local_2c + sVar9)[1] = '\0';
pFVar10 = fopen(local_2c,"wb");
if (pFVar10 == (FILE *)0x0) {
    uVar13 = 0;
    ppuVar12 = local_28;
}
else {
    pcVar11 = (char *)0xfffff3a0;
    local_24 = pFVar10;
    do {
    local_2c = pcVar11;
    fputc(*(int *)((int)(local_20 + 0x318) + (int)pcVar11),local_24);
    pcVar11 = local_2c + 4;
    } while (pcVar11 != (char *)0x0);
    fclose(local_24);
    ppuVar12 = local_28;
}
```

Moving on to the next part of the code where it uses the `param_1`, It seem's like it is trying to open a file and write something into it. Since I have no idea what's the remaining, I then tried to see how things works first.

### Dynamic Analysis

Lets start by opening the application. Remember that it requires some extra intent, so I will need to open it using `adb` instead of just clicking it.

```powershell
PS D:\> adb shell am start -n com.stego.saw/.MainActivity --es "open" "sesame"
Starting: Intent { cmp=com.stego.saw/.MainActivity (has extras) }
```

![alt text](img/index.png#center)

After this, the next thing looks like some click me button. I tried clicking it but it does not show anything. It seems to be the overlay function `f` which requires some additional permission. After some research, I came across this [article](https://stackoverflow.com/questions/36016369/system-alert-window-how-to-get-this-permission-automatically-on-android-6-0-an) which could manually enable the setting. Mine is located at `Privacy Protection > special permission > Display over other apps`. 

![alt text](img/index-1.png#center)

After enabling it, clicking the button now appear another new square.

![alt text](img/index-2.png#center)

I then click it again and it shows another overlay screen.

![alt text](img/index-3.png#center)

This looks like the input that will be used to XOR. I then tried to get the correct input first be decrypting it.

```py
l = 0x0A
DAT_00013a18 = 0x0B
DAT_00013a1c = 0x18
DAT_00013a20 = 0x0F
DAT_00013a24 = 0x5E
DAT_00013a28 = 0x31
DAT_00013a2c = 0x0C
DAT_00013a30 = 0x0F

m = 0x6C
DAT_00013a38 = 0x67
DAT_00013a3c = 0x28
DAT_00013a40 = 0x6E
DAT_00013a44 = 0x2A
DAT_00013a48 = 0x58
DAT_00013a4c = 0x62
DAT_00013a50 = 0x68

param_2 = [
    m ^ l,                
    DAT_00013a38 ^ DAT_00013a18,
    DAT_00013a3c ^ DAT_00013a1c,
    DAT_00013a40 ^ DAT_00013a20,
    DAT_00013a44 ^ DAT_00013a24,
    DAT_00013a48 ^ DAT_00013a28,
    DAT_00013a4c ^ DAT_00013a2c,
    DAT_00013a50 ^ DAT_00013a30
]

param_2_str = "".join(chr(x) for x in param_2)
print(f"param_2: {param_2_str}")

## param_2: fl0ating
```

Now I have a potential string, I tried to use this as the input and see what happened. Somehow, nothing happened and I just assume everything is working as intended. Now I need to search for the file location. it should be in `/data/data/io.stego.saw/` as the path is taken from `getApplicationContext().getApplicationInfo().dataDir + File.separatorChar`.

```bash
beryllium:/data/data/com.stego.saw # ls -la
total 44
drwx------   4 u0_a318 u0_a318        4096 2025-02-20 15:41 .
drwxrwx--x 324 system  system        20480 2025-02-19 21:53 ..
drwxrws--x   2 u0_a318 u0_a318_cache  4096 2025-02-19 21:14 cache
drwxrws--x   2 u0_a318 u0_a318_cache  4096 2025-02-19 21:14 code_cache
-rw-------   1 u0_a318 u0_a318         792 2025-02-20 15:41 h
```

Now there's a `h` file which looks similar according to the native library. I then tried to read the file to see what it is.

```bash
beryllium:/data/data/com.stego.saw # file h
h: Android dex file, version 035
beryllium:/data/data/com.stego.saw # strings h
<init>
HTB{SawS0DCLing}
Ljava/io/PrintStream;
Ljava/lang/Object;
Ljava/lang/String;
Ljava/lang/System;
[Ljava/lang/String;
abcde.java
logprint
main
println
```

Although it's an Android dex file, I managed to get some strings from it which one of it is the flag. 

## Interesting Finding 1

Since this challenge focus on the native library and I had fully understand how it works, I think it is possible to use frida script to skip the overlay permission.

```js
Java.perform(()=> {
    Java.scheduleOnMainThread(()=> {
        let MainActivity = Java.use("com.stego.saw.MainActivity");
        let yeet = MainActivity.$new();
        let test = yeet.a("/data/data/com.stego.saw/","fl0ating");
        console.log("yeet.a() result = " + test);
    });

    Java.perform(function () {
        var openFunc = Module.findExportByName(null, "open");
        var readFunc = Module.findExportByName(null, "read");
    
        if (!openFunc || !readFunc) {
            console.log("[!] Failed to find open() or read(). Exiting.");
            return;
        }
    
        var filePath = "/data/data/com.stego.saw/h"; 
    
        // Open the file
        var libc = Module.findExportByName(null, "open");
        var fd = new NativeFunction(libc, "int", ["pointer", "int"])(Memory.allocUtf8String(filePath), 0);

    
        // Read from the file
        var buffer = Memory.alloc(1024);  
        var bytesRead = new NativeFunction(readFunc, "int", ["int", "pointer", "int"])(fd, buffer, 1024);
    
        if (bytesRead > 0) {
            var rawData = Memory.readByteArray(buffer, bytesRead);

            var asciiString = Array.prototype.map.call(new Uint8Array(rawData), function (byte) {
                return (byte >= 32 && byte <= 126) ? String.fromCharCode(byte) : '.';
            }).join('');
            console.log("[+] File Content (ASCII Only):\n" + asciiString);
        } else {
            console.log("[-] No data read from file.");
        }
    });
    
});
```

By using this frida script, it is possible to directly read the flag in file directly. 

## Interesting Finding 2

Another method is to abuse the native library by importing into my own project. This is much more complicated as it need to code using either Kotlin or Java. This requires abit more step to do so.

First step is to build a project with an exact same app name which in this case `com.stego.saw`.

![alt text](img/index-4.png#center)

After creating the project, add all the native library under the folder name of `jniLibs`.

![alt text](img/index-5.png#center)

After adding it, sync the project first and then start adding the required code to use the native library.

```kotlin
external fun a(str: String?, str2: String?): String?

companion object {
    init {
        System.loadLibrary("default")
    }
}
```

After this, the remaining part should be simple as all I need to do now is code for the button and text.

```kotlin
val context = LocalContext.current
var result by remember { mutableStateOf("Waiting for result...") }
var fleg by remember { mutableStateOf("Waiting for fleg") }


Column(
    modifier = Modifier.fillMaxSize().padding(16.dp),
    verticalArrangement = Arrangement.Center
) {
    Button(onClick = {
        result = MainActivity().a(context.applicationInfo.dataDir + File.separator,"fl0ating").toString()
        val file = File(context.applicationInfo.dataDir + File.separator+"h")
        fleg = if (file.exists()) file.readText() else "File not found"

    }) { Text(text="test") }
    Text(text = result)
    Text(text = fleg)
}
```

Focus on the `onclick` where it has 2 important function there. First is using the function `a` to provide the required filepath and also the correct strings. The another function is for us to easily read the file. Here's an example of the result.

![alt text](img/index-6.png#center)

The first solution is still important as the remaining solution could only works after understanding how everything works. 

** Here's my full source for those that are interested : https://github.com/callmeks/htbsaw

## Things I learned from this challenge

- `adb` to provide Intent extras
- understanding the Java code by providing the correct permission
- reading native library using `ghidra`
- Frida scripting could used to directly call certain function when needed
- creating a fake apk to use the native library  