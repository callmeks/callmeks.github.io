---
date: '2025-06-30T14:44:58+08:00'
title: 'ICSCTF Simple Guess'
tags:
    - Android CTF
    - ICSCTF
---

## Description

> I asked for the challenge from other people so I have no idea what the description is. 
> [SimpleGuess.apk](static/SimpleGuess.apk)

## Static Analysis

As usual, I just start analyzing with `jadx-gui`. I started with the `AndroidManifest.xml`.

```xml
<activity
    android:theme="@style/Theme.SimpleGuess"
    android:label="@string/app_name"
    android:name="com.example.simpleguess.MainActivity"
    android:exported="true">
    <intent-filter>
        <action android:name="android.intent.action.MAIN"/>
        <category android:name="android.intent.category.LAUNCHER"/>
    </intent-filter>
</activity>
```

There's only one activity named `MainActivity` so I just look into this.

```java
public static /* synthetic */ SecretKeySpec gk$default(MainActivity mainActivity, String str, byte[] bArr, int i, int i2, int i3, Object obj) {
    if ((i3 & 4) != 0) {
        i = 65536;
    }
    if ((i3 & 8) != 0) {
        i2 = 256;
    }
    return mainActivity.gk(str, bArr, i, i2);
}

public final SecretKeySpec gk(String c, byte[] s, int i, int k) {
    Intrinsics.checkNotNullParameter(c, "c");
    Intrinsics.checkNotNullParameter(s, "s");
    return new SecretKeySpec(SecretKeyFactory.getInstance("PBKDF2WithHmacSHA256").generateSecret(ksp(c, s, i, k)).getEncoded(), "AES");
}

public final KeySpec ksp(String c, byte[] s, int i, int k) {
    Intrinsics.checkNotNullParameter(c, "c");
    Intrinsics.checkNotNullParameter(s, "s");
    char[] charArray = c.toCharArray();
    Intrinsics.checkNotNullExpressionValue(charArray, "toCharArray(...)");
    return new PBEKeySpec(charArray, s, i, k);
}

public final String decp(Context cot, String p) {
    Object obj;
    Intrinsics.checkNotNullParameter(cot, "cot");
    Intrinsics.checkNotNullParameter(p, "p");
    String string = cot.getString(R.string.salt);
    Intrinsics.checkNotNullExpressionValue(string, "getString(...)");
    byte[] bytes = string.getBytes(Charsets.UTF_8);
    Intrinsics.checkNotNullExpressionValue(bytes, "getBytes(...)");
    byte[] decode = Base64.decode(bytes, 0);
    String string2 = cot.getString(R.string.iv);
    Intrinsics.checkNotNullExpressionValue(string2, "getString(...)");
    byte[] bytes2 = string2.getBytes(Charsets.UTF_8);
    Intrinsics.checkNotNullExpressionValue(bytes2, "getBytes(...)");
    byte[] decode2 = Base64.decode(bytes2, 0);
    String string3 = cot.getString(R.string.ecp);
    Intrinsics.checkNotNullExpressionValue(string3, "getString(...)");
    Intrinsics.checkNotNull(decode);
    SecretKeySpec gk$default = gk$default(this, p, decode, 0, 0, 12, null);
    Cipher cipher = Cipher.getInstance("AES/CBC/PKCS5Padding");
    cipher.init(2, gk$default, new IvParameterSpec(decode2));
    try {
        Result.Companion companion = Result.INSTANCE;
        byte[] doFinal = cipher.doFinal(Base64.decode(string3, 0));
        Intrinsics.checkNotNull(doFinal);
        Log.d("Decrypted", new String(doFinal, Charsets.UTF_8));
        obj = Result.m6603constructorimpl("Thank You");
    } catch (Throwable th) {
        Result.Companion companion2 = Result.INSTANCE;
        obj = Result.m6603constructorimpl(ResultKt.createFailure(th));
    }
    return (String) (Result.m6606exceptionOrNullimpl(obj) == null ? obj : "Thank You");
}

public final String cutf(Context co, String c) {
    Intrinsics.checkNotNullParameter(co, "co");
    Intrinsics.checkNotNullParameter(c, "c");
    String str = c;
    StringBuilder sb = new StringBuilder();
    int length = str.length();
    for (int i = 0; i < length; i++) {
        char charAt = str.charAt(i);
        if (Character.isDigit(charAt)) {
            sb.append(charAt);
        }
    }
    String sb2 = sb.toString();
    Intrinsics.checkNotNullExpressionValue(sb2, "toString(...)");
    String take = StringsKt.take(sb2, 4);
    if (Intrinsics.areEqual(take, "")) {
        return "Nuh Uh";
    }
    return decp(co, take);
}
```

Based on the code, it is something related to AES and PBKDFwithHmacSHA256. This is definitely some kind of cryptography. I then identified the required values in `strings.xml`.

```xml
<string name="ecp">M4EKATajtPe4ry4Vs3W0SQNNoIdSZnDdtdAArgeVZRX1WVod+/IOHiQ8uz3XeAJW</string>
<string name="iv">KF/M4Oz7SyDQOY5PWF76yw==</string>
<string name="salt">S7n8CyjFt28W6JOssy1OPg==</string>
```

Since this is something that could be decrypt and I know the algorithm, I can just looks for the decryption method and write a script accordingly. Aside of that, I noticed this in `MainActivity$onCreate$1`.

```java
if (rememberedValue == Composer.INSTANCE.getEmpty()) {
    rememberedValue = SnapshotStateKt__SnapshotStateKt.mutableStateOf$default("0000", null, 2, null);
    composer.updateRememberedValue(rememberedValue);
}
```

Based on this, I can easily understand that I would need to brute force from 0000 to 9999. Now that I have all the information needed, I can just write a script easily.

```python
import base64
from Crypto.Cipher import AES
from Crypto.Protocol.KDF import PBKDF2
from Crypto.Hash import SHA256
from Crypto.Util.Padding import unpad

# === Encrypted input from Android app ===
ciphertext_b64 = "M4EKATajtPe4ry4Vs3W0SQNNoIdSZnDdtdAArgeVZRX1WVod+/IOHiQ8uz3XeAJW"
iv_b64         = "KF/M4Oz7SyDQOY5PWF76yw=="
salt_b64       = "S7n8CyjFt28W6JOssy1OPg=="

# === Decode base64 to raw bytes ===
ciphertext = base64.b64decode(ciphertext_b64)
iv         = base64.b64decode(iv_b64)
salt       = base64.b64decode(salt_b64)

# === PBKDF2 parameters as per gk$default() ===
iterations = 65536
key_len    = 32

# === Try a single PIN as password ===
def try_pin(pin):
    password = pin.encode()
    key = PBKDF2(password, salt, dkLen=key_len, count=iterations, hmac_hash_module=SHA256)
    cipher = AES.new(key, AES.MODE_CBC, iv)
    try:
        decrypted = cipher.decrypt(ciphertext)
        plaintext = unpad(decrypted, AES.block_size)
        return plaintext.decode('utf-8', errors='ignore')
    except (ValueError, UnicodeDecodeError):
        return None

# === Brute-force 0000 to 9999 ===
def brute_force_all_pins():
    for i in range(10000):
        pin = f"{i:04d}"
        result = try_pin(pin)
        if result:
            if "prelim" in result:
                print(f"[+] PIN: {pin} -> {result}")
                break

# === Entry point ===
if __name__ == "__main__":
    brute_force_all_pins()
```

Based on this script, I can easily get the flag.

```bash
python test.py
[+] PIN: 1435 -> prelim{All_Y0u_N33d_1s_F0ur_D1g1tS}
```

## Things I learned from this challenge

- AES decryption with PBKDFwithHmacSHA256