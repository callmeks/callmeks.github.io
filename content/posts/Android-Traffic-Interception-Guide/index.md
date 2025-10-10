---
date: '2025-10-10T16:07:09+08:00'
title: 'Android Traffic Interception Guide'
tags:
    - Android
    - guide
---

## Prerequisite 

If you have look into my previous post, [Android Penetration Testing Setup Guide](../Android-Penetration-Testing-Setup-Guide/), do have a look into it to ensure that you have installed the tools.


## Summary

I created this blog post to showcase how I intercept android traffic using Burp Suite. 

## Creating an Android Emulator

To create an android emulator, I will be using Android Studio as an example.

![alt text](img/index.png#center)

Select "More Actions" > "Virtual Device Manager". This is where the android emulator is managed.

![alt text](img/index-1.png#center)

Then you will be brought to a new page. Just press on the "create virtual device" or the "+" button on top left corner.

![alt text](img/index-2.png#center)

It will then pop up this page where it is used to create a new android emulator. I choose "Small Phone" here as the size is suitable for my screen. 

![alt text](img/index-3.png#center)

The next part is selecting the system image. Select "API 31" for API and "Android Open Source" for services. You should then select "Intel x86_64 Atom System Image" for system image. Do take note that this is important as different APIs required different steps and Android Open Source comes with root by default. Once you are done, you should be able to just spawn the android emulator now. 

![alt text](img/index-4.png#center)

## Setting Up Burp Suite

Next is to make sure Burp Suite is properly set up. There will be 2 things that we will work on. The first part is to set up a proxy listener first.

![alt text](img/index-5.png#center)

To do so, go to Proxy > Proxy settings > Proxy listeners > click on the "Add" button.

![alt text](img/index-6.png#center)

Now just provide any available port number to bind and select all interfaces for address to bind. 

![alt text](img/index-7.png#center)

It should look like that if everything is set up correctly. The second part is to export the CA certiticate. 

![alt text](img/index-8.png#center)

On the same page, press on the "Import/Export CA certificate".

![alt text](img/index-9.png#center)

Select Export > Certificate in DER format.

![alt text](img/index-10.png#center)

Then just find a place to keep your CA Certificate file.

## Installing CA Cert to User Level

Now that everything is prepared, we could just proceed with installing the CA Cert into our android emulator. To do so, the first step is to copy the CA Cert from our computer to the Android emulator.


![alt text](img/index-11.png#center)

I used the following command to copy the CA Cery into Android emulator's Download folder.

```bash
adb push cert.der /sdcard/download/
```

Once its done, now in the Android Emulator, go to Settings and look for Certificate management app.

![alt text](img/index-12.png#center)

Somewhere around there, you will find "Install a certificate" to proceed with.

![alt text](img/index-13.png#center)

Now select the "CA certificate" to install your CA certificate. 

![alt text](img/index-14.png#center)

Once its done, you have check under "Trusted credentials" > "User" tab.

![alt text](img/index-15.png#center)

It should be something like this. This shows that you have successfully installed the CA Certiticate to User Level.

## Installing CA Cert to System Level

This part is abit complicated. To actually install CA Certiticate to system level, the first thing it to go into android shell with root user privilege.

![alt text](img/index-16.png#center)

Now just run the following commands.

```bash
cp /system/etc/security/cacerts/* /data/misc/user/0/cacerts-added/
mount -t tmpfs tmpfs /system/etc/security/cacerts
cp /data/misc/user/0/cacerts-added/* /system/etc/security/cacerts/

chown root:root /system/etc/security/cacerts/*
chmod 644 /system/etc/security/cacerts/*
chcon u:object_r:system_file:s0 /system/etc/security/cacerts/*
```

![alt text](img/index-17.png#center)

- command 1: copy all system level CA certificate to user level certificate folder
- command 2: create memory mount on top of the system level certificate folder
- command 3: copy all user level CA certificate to system level certificate folder 
- command 4-6: fix permissions issue and SELinux context labels

![alt text](img/index-18.png#center)

Once this is done, the Burp Suite's CA Certificate now should be installed in system level as well.

## Demonstration

Once you have set up everything accordingly, now there's one last thing that you will need to do which is to connect the proxy from your android emulator.

![alt text](img/index-19.png#center)

To do so, go to wifi settings > press on the network details. You will noticed that there's a button to click on the top right corner. 

![alt text](img/index-20.png#center)

On the Proxy, select manual and add your computer's IP address as the proxy hostname. As for proxy port, use the port number that you have set up for Burp Suite. Once its done, you can now intercept Android application's traffic. 

![alt text](img/index-21.png#center)

## Video Guide

I know its complicated and confusing so I have provided a video version as well.

{{< youtube id="m4heWC61vac" >}}


