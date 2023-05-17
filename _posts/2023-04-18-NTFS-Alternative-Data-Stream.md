---
title: NTFS Alternative Data Stream (ADS)
categories: [Knowledge,Windows]
tags: [Knowledge]
---

# Explanation
- Can be used to hide text or command
- Can be checked with cmd `dir /r` or powershell `gi <filename> -stream *`
- To read the stream content, use powershell `gc <filename> -stream <stream name>`

# Example
- checking ADS
    - cmd
    ```cmd
    directory>dir /r
    Volume in drive D is Fake Volume
    Volume Serial Number is A0BA-07DC

    Directory of directory

    08-Apr-23  01:04 PM    <DIR>          .
    18-Apr-23  10:33 AM    <DIR>          ..
    18-Apr-23  02:38 PM           162,748 bof.png
                                        5 bof.png:test:$DATA
    ```
    
    - powershell
    ```powershell
    PS directory> gi bof.png -stream *

    PSPath        : Microsoft.PowerShell.Core\FileSystem::directory\bof.png::$DATA
    PSParentPath  : Microsoft.PowerShell.Core\FileSystem::directory
    PSChildName   : bof.png::$DATA
    PSDrive       : D
    PSProvider    : Microsoft.PowerShell.Core\FileSystem
    PSIsContainer : False
    FileName      : directory\bof.png
    Stream        : :$DATA
    Length        : 162748

    PSPath        : Microsoft.PowerShell.Core\FileSystem::directory\bof.png:test
    PSParentPath  : Microsoft.PowerShell.Core\FileSystem::directory
    PSChildName   : bof.png:test
    PSDrive       : D
    PSProvider    : Microsoft.PowerShell.Core\FileSystem
    PSIsContainer : False
    FileName      : directory\bof.png
    Stream        : test
    Length        : 5
    ```

- Read stream content

```powershell
PS directory> gc bof.png -stream test
hi
```

# Useful-links
- [Hiding data using NTFS ADS](https://www.youtube.com/watch?v=S4MBzeni9Eo&t=318s)
- [Create,open,detect,remove ADS](https://www.minitool.com/partition-disk/alternate-data-streams.html)