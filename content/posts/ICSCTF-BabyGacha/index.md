---
date: '2025-06-30T15:48:09+08:00'
title: 'ICSCTF BabyGacha'
tags:
    - Android CTF
    - ICSCTF
---

## Description

> I asked for the challenge from other people so I have no idea what the description is.
> [EmojiGachaRPG.apk](static/EmojiGachaRPG.apk)

## Static Analysis

I started out by decompiling it using `jadx-gui` and looked into the `AndroidManifest.xml`

```xml
<uses-permission android:name="android.permission.INTERNET"/>

<activity
    android:theme="@style/Theme.UltraAddictiveGachaGame"
    android:name="com.honque.ultraaddictivegachagame.MainActivity"
    android:exported="true">
    <intent-filter>
        <action android:name="android.intent.action.MAIN"/>
        <category android:name="android.intent.category.LAUNCHER"/>
    </intent-filter>
</activity>
<activity
    android:name="androidx.compose.ui.tooling.PreviewActivity"
    android:exported="true"/>
<activity
    android:theme="@android:style/Theme.Material.Light.NoActionBar"
    android:name="androidx.activity.ComponentActivity"
    android:exported="true"/>
```

Based on the information found, the application requires internet connection and the only interesting activity is the `MainActivity`.

```java
@Override // androidx.activity.ComponentActivity, androidx.core.app.ComponentActivity, android.app.Activity
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    EdgeToEdge.enable$default(this, null, null, 3, null);
    Context applicationContext = getApplicationContext();
    Intrinsics.checkNotNullExpressionValue(applicationContext, "getApplicationContext(...)");
    GameRepositoryKt.setGameRepository(new GameRepository(applicationContext));
    CharacterMasterList.INSTANCE.loadCharactersFromXml(this);
    BuildersKt__Builders_commonKt.launch$default(LifecycleOwnerKt.getLifecycleScope(this), null, null, new MainActivity$onCreate$1(null), 3, null);
    ComponentActivityKt.setContent$default(this, null, ComposableSingletons$MainActivityKt.INSTANCE.m7438getLambda2$app_debug(), 1, null);
}

@Override // android.app.Activity
protected void onStop() {
    super.onStop();
    GameRepositoryKt.getGameRepository().savePlayerData();
    Log.i("GachaGame", "Game data saved onStop.");
}
```

After looking into it, it seems to run some other functions. I then decide to play around with the application since Im too lazy o read the code.

## Dynamic analysis

![alt text](img/index.png#center)

So this is the main page of the application. After clicking around, I noticed something important.

![alt text](img/index-1.png#center)

So basically, I need the exact gold to buy the flag. The first thing that came into my mind was manipulating the gold value. To perform that, I tried looking into the code again.

```java
public final int getCurrency() {
    IntState $this$getValue$iv = currency;
    return $this$getValue$iv.getIntValue();
}

public final void setCurrency(int i) {
    MutableIntState $this$setValue$iv = currency;
    $this$setValue$iv.setIntValue(i);
}
```

When I looked into `Player`, I noticed this code which I assume it is related to the money or `player gold`. I then just use frida to change my `player gold` to the required ammount will do.

```js
Java.perform(()=>{
    let Player = Java.use("com.honque.ultraaddictivegachagame.Player");
    Player["getCurrency"].implementation = function () {
        console.log(`Player.getCurrency is called`);
        let result = this["getCurrency"]();
        console.log(`Player.getCurrency result=${result}`);
        return result;
    };
})
```

```bash
 frida -U -N com.honque.ultraaddictivegachagame -l test.js
     ____
    / _  |   Frida 16.5.9 - A world-class dynamic instrumentation toolkit
   | (_| |
    > _  |   Commands:
   /_/ |_|       help      -> Displays the help system
   . . . .       object?   -> Display information about 'object'
   . . . .       exit/quit -> Exit
   . . . .
   . . . .   More info at https://frida.re/docs/home/
   . . . .
   . . . .   Connected to POCOPHONE F1 (id=d211a91c)
[POCOPHONE F1::com.honque.ultraaddictivegachagame ]-> Player.getCurrency is called
Player.getCurrency result=90
Player.getCurrency is called
Player.getCurrency result=90
Player.getCurrency is called
Player.getCurrency result=80
Player.getCurrency is called
Player.getCurrency result=80
Player.getCurrency is called
Player.getCurrency result=80
```

![alt text](img/index-2.png#center)

Now that I can read the result, I can just modify the return value will do.

```js
Java.perform(()=>{
    let Player = Java.use("com.honque.ultraaddictivegachagame.Player");
    Player["getCurrency"].implementation = function () {
        console.log(`Player.getCurrency is called`);
        let result = this["getCurrency"]();
        console.log(`Player.getCurrency result=${result}`);
        return 214783647;
    };
})

```

![alt text](img/index-3.png#center)

by using this simple method, I can easily buy the flag. since the server is down when I am writing this, I could not get the correct flag. 

## Things I learned from this challenge

- frida intercepting and changing return value