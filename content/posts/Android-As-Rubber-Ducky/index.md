---
date: '2024-12-04T16:25:45+08:00'
title: 'Android as Rubber Ducky'
tags:
    - Android
    - guide
---

## Prerequisite 

- a physical android devices
  - rooted
  - termux app

As mentioned, the android devices must be rooted in order to work like a rubber ducky. I'll be using my old devices that I have rooted as demonstration purposes. 

Here's a quick evidence that my devices is rooted. 

![alt text](img/index.png#center)

## Setting up

Now that the android devices is rooted, I'll need to download some useful files and application that has been created by others

- [android usb gadget](https://github.com/tejado/android-usb-gadget)
- [hid-gadget-test](https://github.com/pelya/android-keyboard-gadget/raw/refs/heads/master/hid-gadget-test/hid-gadget-test)
- [poc_pc_gadget](https://github.com/androidmalware/android_hid/blob/main/part2/poc_pc_gadget)

Install the **android usb gadget** and prepare both the `hid-gadget-test` and `poc_pc_gadget` in your devices. After everything is ready, open the **android usb gadget** application.

![alt text](img/index-1.png#center)

It should have a popup where it ask about superuser rights. After that, scroll down and use the second option or add another one if there's no second option. In the second option, add keyboard will do. 

![alt text](img/index-2.png#center)

After adding it, remember to turn on by pressing the "Gadget status" and process to android terminal (termux).

![alt text](img/index-3.png#center)

Make sure to place the both the file in a same directory and both the file has execute permission.

![alt text](img/index-4.png#center)

Basically, `hid-gadget-test` is a binary and `poc_pc_gadget` is a script.

![alt text](img/index-5.png#center)

One last thing is to take note is the `/dev/hidg1`. In some cases, the `hidg` number will be different make sure to have a look. Somehow, I'll need to give all permission to everyone in order to make this works. `/dev/hidg1` will only appear if you have turn on the "Gadget status". 

![alt text](img/index-6.png#center)

Since it does not have execute permission by default, I'll need to change the permission myself.

![alt text](img/index-7.png#center)

After everything is done, I could just run the `poc_pc_gadget` to perform HID attack. remember to give it some try and error 

## POC

Here's a demonstration on how it works.

{{< video src="vid/poc" autoplay="false" controls="true" loop="true" width="100%">}}

## References

- [android usb gadget](https://github.com/tejado/android-usb-gadget)
- [hid-gadget-test](https://github.com/pelya/android-keyboard-gadget/raw/refs/heads/master/hid-gadget-test/hid-gadget-test)
- [poc_pc_gadget](https://github.com/androidmalware/android_hid/blob/main/part2/poc_pc_gadget)
- [youtube video](https://www.youtube.com/watch?v=Mek9DMGy8os)