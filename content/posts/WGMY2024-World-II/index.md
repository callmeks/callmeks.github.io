---
date: '2024-12-29T13:40:56+08:00'
title: 'WGMY2024 World II'
tags:
    - android CTF
---

## Challenge Description

> Welp, time to do it again.
>
> Unable to install? That is a part of the challenge, try to overcome it.
>
> Author: Trailbl4z3r & Monaruku
>
>hint: 
> Tbh this is not a natively built app, more like something just wrapped into an app
>
> [World-II.apk](static/World_II.apk)

## Solution

Since it's an APK file, Lets start with `jadx-gui` for static analysis. In terms of static analysis, it's always best to check out `AndroidManifest.xml` first.

![alt text](img/index.png#center)

Based on the code, there's only one activity which is `systems.altimit.rpgmakermv.WebPlayerActivity`. Double click and see the activity code. There's a lot of function in the `WebPlayerActivity` but I'll be focusing on `onCreate` function first as that it is the starting point for all activities. 

![alt text](img/index-1.png#center)

According to the code, it is trying to construct the game by getting the value of `id.application.rpgmakermv.R.string.mv_project_index`. I then tried to have a look and search for the strings values. By using the global search feature in `jadx-gui`, I managed to get the strings value. 

![alt text](img/index-2.png#center)

Now that I know the value is `//android_asset/www/index.html`, my first thought is checking out for the files using the global search feature.

![alt text](img/index-3.png#center)

While the `index.html` does not have a lot of code, I noticed that it's using `main.js`. I then look for it and have a look in it.

![alt text](img/index-4.png#center)

Looking into `main.js`, there's a lot of code and several script as well. I then look into each of it and see which is interesting.

![alt text](img/index-5.png#center)

Looking into `rmmz_managers.js`, I noticed that database file that are stored as json file. I then look into `Actors.json` as it seems like the game character's database file.

![alt text](img/index-6.png#center)

Since this is a json file, I think that I could modify the json file and repack the apk to make the me win easily. Since I have limited information at the moment, I tried to run the apk and see how the game works. 


![alt text](img/index-7.png#center)

Here's how the main page looks like, which is similar to the `index.html`. I then started to play around to get more information.

![alt text](img/index-8.png#center)

Here's my current stat and equipment. I then look into the `Actors.json` to compare and identify which value I should modify. I noticed that the `equip` is the current weapon and body that the character has. The weapon database file is `Weapons.json`. 

![alt text](img/index-9.png#center)

The id of the weapon is same as the number provided in `equip` in `Actors.json`. My next thought was to increase the damage of the weapon `Dragon Blade` since I could just edit the json file and repack it. While I have no idea, the `params` definitely consist the damage of the weapon. I then randomly edit every `params` into high value. To do so, I'll need to unpack first.

```powershell
PS D:\Desktop\Android\world2> apktool d .\World_II.apk
I: Using Apktool 2.10.0 on World_II.apk with 8 thread(s).
I: Baksmaling classes.dex...
I: Loading resource table...
I: Decoding file-resources...
I: Loading resource table from file: C:\Users\a\AppData\Local\apktool\framework\1.apk
I: Decoding values */* XMLs...
I: Decoding AndroidManifest.xml with resources...
I: Regular manifest package...
I: Copying assets and libs...
I: Copying unknown files...
I: Copying original files...
Press any key to continue . . .
```

After unpack the APK file, search for `Weapons.json` and modfiy the `params` value of `Dragon Blade` to 999. After modifying it, repack it.

```powershell
PS D:\Desktop\Android\world2> apktool b .\World_II\ -o world-2-modified.apk
I: Using Apktool 2.10.0 with 8 thread(s).
I: Checking whether sources has changed...
I: Smaling smali folder into classes.dex...
I: Checking whether resources has changed...
I: Building resources...
I: Building apk file...
I: Copying unknown files/dir...
I: Built apk into: world-2-modified.apk
Press any key to continue . . .
```

After repacking the APK, remember to sign it. To do so, create a keystore.

```powershell
PS D:\Desktop\Android\world2> keytool -genkey -v -keystore modify.keystore -alias modify_key -keyalg RSA -keysize 2048 -validity 10000
Enter keystore password:
Re-enter new password:
Enter the distinguished name. Provide a single dot (.) to leave a sub-component empty or press ENTER to use the default value in braces.
What is your first and last name?
What is the name of your organizational unit?
What is the name of your organization?
What is the name of your City or Locality?
What is the name of your State or Province?
What is the two-letter country code for this unit?
Is CN=a, OU=Unknown, O=Unknown, L=Unknown, ST=Unknown, C=Unknown correct?
  [no]:  yes
Generating 2,048 bit RSA key pair and self-signed certificate (SHA384withRSA) with a validity of 10,000 days
        for: CN=a, OU=Unknown, O=Unknown, L=Unknown, ST=Unknown, C=Unknown
[Storing modify.keystore]
```

After the keystore is created, sign the APK file.


```powershell
PS D:\Desktop\Android\world2> apksigner sign --ks .\modify.keystore .\world-2-modified.apk
Keystore password for signer #1:
```

After that, Just run the game and hope everything is working.

![alt text](img/index-10.png#center)

As shown in image, I managed to change the weapon stats by modifying the `params` value. Now that my stats are high, I could easily win the game.

![alt text](img/index-11.png#center)

After I win each boss, I'll get a partial flag.

- Flag 1

![alt text](img/index-12.png#center)

- Flag 2

![alt text](img/index-13.png#center)

- Flag 3

![alt text](img/index-14.png#center)

- Flag 4
    - This part of flag appear in the map after winning boss 4

![alt text](img/index-15.png#center)

- Flag 5
  - After winning the boss, you will need to talk to a dude and it will give you `23 7 13 25` which is order of the alphabelt `wgmy`.
  - Then a QR will be provided.
  - `4f51785}`

![alt text](img/index-16.png#center)

## Things I learned from this challenge

- reading code using static analysis
- patching the APK
- Cheat the game to win 
