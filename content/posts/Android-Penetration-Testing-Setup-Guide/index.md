---
date: '2025-10-08T15:37:04+08:00'
title: 'Android Penetration Testing Setup Guide'
tags:
    - Android
    - guide
---

## Summary

I created this blog post to showcase how I setup my android penetration testing environment in Windows.

### Tools

The list of tools that I will be installing: 

  - Android Studio
    - build-tools
    - platform-tools
    - android virtual device (AVD)
  - OpenJDK
  - Apktool
  - Jadx / Jadx-gui
  - Frida (Required Python)
  - Objection (Required Python)
  - Burp Suite CE

Tools that are required to add into system environment (PATH) to make it work:

  - OpenJDK (java.exe)
  - Apktool
  - Jadx
  - build-tools from [Android Studio](https://developer.android.com/tools#tools-build)
  - platform-tools from [Android Studio](https://developer.android.com/tools#tools-platform)

## Video

To make things easier, I decided to just demo in a video instead.

{{< youtube id="3IZEvlg34sU" >}}

## For more information

  - Android Studio: [https://developer.android.com/studio](https://developer.android.com/studio)
    - managing AVD: [https://developer.android.com/studio/run/managing-avds](https://developer.android.com/studio/run/managing-avds)
  - OpenJDK: [https://openjdk.org/](https://openjdk.org/)
    - install guide: [https://openjdk.org/install/](https://openjdk.org/install/)
  - Apktool: [https://apktool.org/](https://apktool.org/)
    - install guide: [https://apktool.org/docs/install/](https://apktool.org/docs/install/)
  - Jadx: [https://github.com/skylot/jadx](https://github.com/skylot/jadx)
  - Frida (Required Python): [https://frida.re/](https://frida.re/)
  - Objection (Required Python): [https://github.com/sensepost/objection](https://github.com/sensepost/objection)
  - Burp Suite CE: [https://portswigger.net/burp/communitydownload](https://portswigger.net/burp/communitydownload)