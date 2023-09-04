---
title: Extracting info from raw image
categories: [CTF]
tags: [Forensic]
---

## Step 1: Information Gathering

since a file `memdump.raw` was given as a challenge, lets try to analyze it with **volatility**. There are 2 types of volatility, [volatility2](https://www.volatilityfoundation.org/26) and [volatility3](https://www.volatilityfoundation.org/3), which is basically written in python2 and python3. Both volatility will be used for testing purpose.

### Volatility 2

```bash
vol2 -f memdump.raw imageinfo
Volatility Foundation Volatility Framework 2.6
INFO    : volatility.debug    : Determining profile based on KDBG search...
          Suggested Profile(s) : Win7SP1x64, Win7SP0x64, Win2008R2SP0x64, Win2008R2SP1x64_23418, Win2008R2SP1x64, Win7SP1x64_23418
                     AS Layer1 : WindowsAMD64PagedMemory (Kernel AS)
                     AS Layer2 : FileAddressSpace (/root/bat/memdump.raw)
                      PAE type : No PAE
                           DTB : 0x187000L
                          KDBG : 0xf80002bfd120L
          Number of Processors : 1
     Image Type (Service Pack) : 1
                KPCR for CPU 0 : 0xfffff80002bff000L
             KUSER_SHARED_DATA : 0xfffff78000000000L
           Image date and time : 2023-06-20 08:28:07 UTC+0000
     Image local date and time : 2023-06-20 16:28:07 +0800
```

### Volatility 3

```bash
vol -f memdump.raw windows.info                
Volatility 3 Framework 2.4.1
Progress:  100.00               PDB scanning finished                        
Variable        Value

Kernel Base     0xf80002a1b000
DTB     0x187000
Symbols file:///usr/local/lib/python3.11/dist-packages/volatility3/symbols/windows/ntkrnlmp.pdb/ECE191A20CFF4465AE46DF96C2263845-1.json.xz
Is64Bit True
IsPAE   False
layer_name      0 WindowsIntel32e
memory_layer    1 FileLayer
KdDebuggerDataBlock     0xf80002bfd120
NTBuildLab      7601.24384.amd64fre.win7sp1_ldr_
CSDVersion      1
KdVersionBlock  0xf80002bfd0e8
Major/Minor     15.7601
MachineType     34404
KeNumberProcessors      1
SystemTime      2023-06-20 08:28:07
NtSystemRoot    C:\Windows
NtProductType   NtProductWinNt
NtMajorVersion  6
NtMinorVersion  1
PE MajorOperatingSystemVersion  6
PE MinorOperatingSystemVersion  1
PE Machine      34404
PE TimeDateStamp        Thu Feb 21 03:36:29 2019
```


After getting some information from the `memdump.raw` such as **Windows OS**, the next step would be to check the processes to have an idea about the situation.

## Step 2: Gathering and analyzing processes

In volatiltiy 2 and 3, there are several command to check processes. The example below will show the result of **PSList** and **PSTree**.

### Volatility 2

#### PSList

```bash
vol2 -f memdump.raw --profile Win7SP1x64 pslist
Volatility Foundation Volatility Framework 2.6
Offset(V)          Name                    PID   PPID   Thds     Hnds   Sess  Wow64 Start                          Exit                          
------------------ -------------------- ------ ------ ------ -------- ------ ------ ------------------------------ ------------------------------
0xfffffa8006c7fb00 System                    4      0     91      577 ------      0 2023-06-20 07:22:42 UTC+0000                                 
0xfffffa8007d39610 smss.exe                260      4      2       29 ------      0 2023-06-20 07:22:42 UTC+0000                                 
0xfffffa80089a2b00 csrss.exe               348    340      9      505      0      0 2023-06-20 07:22:43 UTC+0000                                 
0xfffffa80089ffb00 wininit.exe             388    340      3       76      0      0 2023-06-20 07:22:43 UTC+0000                                 
0xfffffa8008993b00 csrss.exe               400    380     11      352      1      0 2023-06-20 07:22:43 UTC+0000                                 
0xfffffa8008c28b00 services.exe            464    388     13      241      0      0 2023-06-20 07:22:43 UTC+0000                                 
0xfffffa8008c35b00 lsass.exe               472    388      8      767      0      0 2023-06-20 07:22:43 UTC+0000                                 
0xfffffa8008c45b00 lsm.exe                 480    388     10      153      0      0 2023-06-20 07:22:43 UTC+0000                                 
0xfffffa8008c39b00 winlogon.exe            492    380      3      113      1      0 2023-06-20 07:22:43 UTC+0000                                 
0xfffffa8008c49b00 svchost.exe             612    464     10      367      0      0 2023-06-20 07:22:43 UTC+0000                                 
0xfffffa8008d1d7e0 svchost.exe             680    464      9      309      0      0 2023-06-20 07:22:43 UTC+0000                                 
0xfffffa8008d3a6d0 svchost.exe             732    464     23      596      0      0 2023-06-20 07:22:43 UTC+0000                                 
0xfffffa8008d758b0 svchost.exe             820    464     29      630      0      0 2023-06-20 07:22:43 UTC+0000                                 
0xfffffa8008de1170 svchost.exe             892    464     39      958      0      0 2023-06-20 07:22:44 UTC+0000                                 
0xfffffa8008e17b00 svchost.exe             240    464     20      777      0      0 2023-06-20 07:22:44 UTC+0000                                 
0xfffffa8008e6eb00 svchost.exe             440    464     17      490      0      0 2023-06-20 07:22:44 UTC+0000                                 
0xfffffa8008ed6220 dwm.exe                1128    820      3       72      1      0 2023-06-20 07:22:44 UTC+0000                                 
0xfffffa8008ee4b00 explorer.exe           1152   1108     36     1013      1      0 2023-06-20 07:22:44 UTC+0000                                 
0xfffffa8008f2c260 spoolsv.exe            1188    464     13      278      0      0 2023-06-20 07:22:44 UTC+0000                                 
0xfffffa8008f4a5e0 taskhost.exe           1220    464      8      195      1      0 2023-06-20 07:22:44 UTC+0000                                 
0xfffffa8008f51320 svchost.exe            1244    464     18      327      0      0 2023-06-20 07:22:44 UTC+0000                                 
0xfffffa8008fd9060 svchost.exe            1360    464     23      318      0      0 2023-06-20 07:22:45 UTC+0000                                 
0xfffffa8009002b00 FoxitPDFReader         1404    464      3       68      0      1 2023-06-20 07:22:45 UTC+0000                                 
0xfffffa800905bb00 vmtoolsd.exe           1508   1152      8      188      1      0 2023-06-20 07:22:45 UTC+0000                                 
0xfffffa8007bb54b0 VGAuthService.         1676    464      3       87      0      0 2023-06-20 07:22:46 UTC+0000                                 
0xfffffa8009117b00 vm3dservice.ex         1708    464      4       61      0      0 2023-06-20 07:22:46 UTC+0000                                 
0xfffffa80091135f0 vmtoolsd.exe           1736    464     12      282      0      0 2023-06-20 07:22:46 UTC+0000                                 
0xfffffa8009086b00 vm3dservice.ex         1760   1708      2       53      1      0 2023-06-20 07:22:46 UTC+0000                                 
0xfffffa8008d628b0 svchost.exe            2000    464      6       97      0      0 2023-06-20 07:22:47 UTC+0000                                 
0xfffffa8008055060 WmiPrvSE.exe            332    612     10      206      0      0 2023-06-20 07:22:47 UTC+0000                                 
0xfffffa8008146b00 dllhost.exe             196    464     13      194      0      0 2023-06-20 07:22:48 UTC+0000                                 
0xfffffa800824f3f0 SearchIndexer.         2136    464     11      586      0      0 2023-06-20 07:22:52 UTC+0000                                 
0xfffffa80083acb00 msdtc.exe              2256    464     12      149      0      0 2023-06-20 07:22:52 UTC+0000                                 
0xfffffa80083e5b00 wmpnetwk.exe           2512    464     13      422      0      0 2023-06-20 07:22:53 UTC+0000                                 
0xfffffa800921fb00 svchost.exe            2624    464      8      356      0      0 2023-06-20 07:22:54 UTC+0000                                 
0xfffffa800931a750 WUDFHost.exe           3024    820      8      202      0      0 2023-06-20 07:23:14 UTC+0000                                 
0xfffffa8006da0b00 sppsvc.exe             2456    464      4      152      0      0 2023-06-20 07:24:47 UTC+0000                                 
0xfffffa80074a5b00 svchost.exe            3284    464     13      353      0      0 2023-06-20 07:24:47 UTC+0000                                 
0xfffffa8008df32b0 taskhost.exe           3356    464      5      123      1      0 2023-06-20 07:54:22 UTC+0000                                 
0xfffffa8008e9c060 MRCv120.exe            2496   1152      4      282      1      1 2023-06-20 08:08:46 UTC+0000                                 
0xfffffa800742ab00 audiodg.exe            3468    732      5      128      0      0 2023-06-20 08:22:23 UTC+0000                                 
0xfffffa8008f8ab00 WmiPrvSE.exe           3316    612      5      113      0      0 2023-06-20 08:22:56 UTC+0000                                 
0xfffffa800748fb00 FoxitPDFReader         1872   1152     18      492      1      1 2023-06-20 08:26:55 UTC+0000                                 
0xfffffa800740d060 FoxitPDFReader         4060   1872      0 --------      1      0 2023-06-20 08:27:01 UTC+0000   2023-06-20 08:27:02 UTC+0000  
0xfffffa8006db7980 FirefoxPortabl         2420   1152      6      147      1      1 2023-06-20 08:27:05 UTC+0000                                 
0xfffffa8007fe0060 firefox.exe            3716   2420     57      648      1      0 2023-06-20 08:27:05 UTC+0000                                 
0xfffffa8008d03b00 firefox.exe            4000   3716     26      313      1      0 2023-06-20 08:27:20 UTC+0000                                 
0xfffffa8008b60a80 FoxitPDFReader         3476   1152      2      113      1      1 2023-06-20 08:28:04 UTC+0000                                 
0xfffffa8008b66760 dllhost.exe            3396    612      6  7536754      1      0 2023-06-20 08:28:09 UTC+0000                                 
```

#### PSTree

```bash
vol2 -f memdump.raw --profile Win7SP1x64 pstree
Volatility Foundation Volatility Framework 2.6
Name                                                  Pid   PPid   Thds   Hnds Time
-------------------------------------------------- ------ ------ ------ ------ ----
 0xfffffa8008ee4b00:explorer.exe                     1152   1108     36   1013 2023-06-20 07:22:44 UTC+0000
. 0xfffffa8008e9c060:MRCv120.exe                     2496   1152      4    282 2023-06-20 08:08:46 UTC+0000
. 0xfffffa8008b60a80:FoxitPDFReader                  3476   1152      2    113 2023-06-20 08:28:04 UTC+0000
. 0xfffffa800748fb00:FoxitPDFReader                  1872   1152     18    492 2023-06-20 08:26:55 UTC+0000
.. 0xfffffa800740d060:FoxitPDFReader                 4060   1872      0 ------ 2023-06-20 08:27:01 UTC+0000
. 0xfffffa8006db7980:FirefoxPortabl                  2420   1152      6    147 2023-06-20 08:27:05 UTC+0000
.. 0xfffffa8007fe0060:firefox.exe                    3716   2420     57    648 2023-06-20 08:27:05 UTC+0000
... 0xfffffa8008d03b00:firefox.exe                   4000   3716     26    313 2023-06-20 08:27:20 UTC+0000
. 0xfffffa800905bb00:vmtoolsd.exe                    1508   1152      8    188 2023-06-20 07:22:45 UTC+0000
 0xfffffa8006c7fb00:System                              4      0     91    577 2023-06-20 07:22:42 UTC+0000
. 0xfffffa8007d39610:smss.exe                         260      4      2     29 2023-06-20 07:22:42 UTC+0000
 0xfffffa80089ffb00:wininit.exe                       388    340      3     76 2023-06-20 07:22:43 UTC+0000
. 0xfffffa8008c28b00:services.exe                     464    388     13    241 2023-06-20 07:22:43 UTC+0000
.. 0xfffffa80083e5b00:wmpnetwk.exe                   2512    464     13    422 2023-06-20 07:22:53 UTC+0000
.. 0xfffffa8007bb54b0:VGAuthService.                 1676    464      3     87 2023-06-20 07:22:46 UTC+0000
.. 0xfffffa800824f3f0:SearchIndexer.                 2136    464     11    586 2023-06-20 07:22:52 UTC+0000
.. 0xfffffa8008146b00:dllhost.exe                     196    464     13    194 2023-06-20 07:22:48 UTC+0000
.. 0xfffffa8006da0b00:sppsvc.exe                     2456    464      4    152 2023-06-20 07:24:47 UTC+0000
.. 0xfffffa8008df32b0:taskhost.exe                   3356    464      5    123 2023-06-20 07:54:22 UTC+0000
.. 0xfffffa8008f2c260:spoolsv.exe                    1188    464     13    278 2023-06-20 07:22:44 UTC+0000
.. 0xfffffa8008d1d7e0:svchost.exe                     680    464      9    309 2023-06-20 07:22:43 UTC+0000
.. 0xfffffa8008d3a6d0:svchost.exe                     732    464     23    596 2023-06-20 07:22:43 UTC+0000
... 0xfffffa800742ab00:audiodg.exe                   3468    732      5    128 2023-06-20 08:22:23 UTC+0000
.. 0xfffffa8009117b00:vm3dservice.ex                 1708    464      4     61 2023-06-20 07:22:46 UTC+0000
... 0xfffffa8009086b00:vm3dservice.ex                1760   1708      2     53 2023-06-20 07:22:46 UTC+0000
.. 0xfffffa8008d758b0:svchost.exe                     820    464     29    630 2023-06-20 07:22:43 UTC+0000
... 0xfffffa800931a750:WUDFHost.exe                  3024    820      8    202 2023-06-20 07:23:14 UTC+0000
... 0xfffffa8008ed6220:dwm.exe                       1128    820      3     72 2023-06-20 07:22:44 UTC+0000
.. 0xfffffa8008d628b0:svchost.exe                    2000    464      6     97 2023-06-20 07:22:47 UTC+0000
.. 0xfffffa8008e6eb00:svchost.exe                     440    464     17    490 2023-06-20 07:22:44 UTC+0000
.. 0xfffffa800921fb00:svchost.exe                    2624    464      8    356 2023-06-20 07:22:54 UTC+0000
.. 0xfffffa8008f4a5e0:taskhost.exe                   1220    464      8    195 2023-06-20 07:22:44 UTC+0000
.. 0xfffffa80091135f0:vmtoolsd.exe                   1736    464     12    282 2023-06-20 07:22:46 UTC+0000
.. 0xfffffa80074a5b00:svchost.exe                    3284    464     13    353 2023-06-20 07:24:47 UTC+0000
.. 0xfffffa8008f51320:svchost.exe                    1244    464     18    327 2023-06-20 07:22:44 UTC+0000
.. 0xfffffa80083acb00:msdtc.exe                      2256    464     12    149 2023-06-20 07:22:52 UTC+0000
.. 0xfffffa8008fd9060:svchost.exe                    1360    464     23    318 2023-06-20 07:22:45 UTC+0000
.. 0xfffffa8008c49b00:svchost.exe                     612    464     10    367 2023-06-20 07:22:43 UTC+0000
... 0xfffffa8008b66760:dllhost.exe                   3396    612      6 75...4 2023-06-20 08:28:09 UTC+0000
... 0xfffffa8008055060:WmiPrvSE.exe                   332    612     10    206 2023-06-20 07:22:47 UTC+0000
... 0xfffffa8008f8ab00:WmiPrvSE.exe                  3316    612      5    113 2023-06-20 08:22:56 UTC+0000
.. 0xfffffa8009002b00:FoxitPDFReader                 1404    464      3     68 2023-06-20 07:22:45 UTC+0000
.. 0xfffffa8008e17b00:svchost.exe                     240    464     20    777 2023-06-20 07:22:44 UTC+0000
.. 0xfffffa8008de1170:svchost.exe                     892    464     39    958 2023-06-20 07:22:44 UTC+0000
. 0xfffffa8008c35b00:lsass.exe                        472    388      8    767 2023-06-20 07:22:43 UTC+0000
. 0xfffffa8008c45b00:lsm.exe                          480    388     10    153 2023-06-20 07:22:43 UTC+0000
 0xfffffa80089a2b00:csrss.exe                         348    340      9    505 2023-06-20 07:22:43 UTC+0000
 0xfffffa8008993b00:csrss.exe                         400    380     11    352 2023-06-20 07:22:43 UTC+0000
 0xfffffa8008c39b00:winlogon.exe                      492    380      3    113 2023-06-20 07:22:43 UTC+0000
```

### Volatility 3

#### PSList

```bash
vol -f memdump.raw windows.pslist
Volatility 3 Framework 2.4.1
Progress:  100.00               PDB scanning finished                        
PID     PPID    ImageFileName   Offset(V)       Threads Handles SessionId       Wow64   CreateTime      ExitTime        File output

4       0       System  0xfa8006c7fb00  91      577     N/A     False   2023-06-20 07:22:42.000000      N/A     Disabled
260     4       smss.exe        0xfa8007d39610  2       29      N/A     False   2023-06-20 07:22:42.000000      N/A     Disabled
348     340     csrss.exe       0xfa80089a2b00  9       505     0       False   2023-06-20 07:22:43.000000      N/A     Disabled
388     340     wininit.exe     0xfa80089ffb00  3       76      0       False   2023-06-20 07:22:43.000000      N/A     Disabled
400     380     csrss.exe       0xfa8008993b00  11      352     1       False   2023-06-20 07:22:43.000000      N/A     Disabled
464     388     services.exe    0xfa8008c28b00  13      241     0       False   2023-06-20 07:22:43.000000      N/A     Disabled
472     388     lsass.exe       0xfa8008c35b00  8       767     0       False   2023-06-20 07:22:43.000000      N/A     Disabled
480     388     lsm.exe 0xfa8008c45b00  10      153     0       False   2023-06-20 07:22:43.000000      N/A     Disabled
492     380     winlogon.exe    0xfa8008c39b00  3       113     1       False   2023-06-20 07:22:43.000000      N/A     Disabled
612     464     svchost.exe     0xfa8008c49b00  10      367     0       False   2023-06-20 07:22:43.000000      N/A     Disabled
680     464     svchost.exe     0xfa8008d1d7e0  9       309     0       False   2023-06-20 07:22:43.000000      N/A     Disabled
732     464     svchost.exe     0xfa8008d3a6d0  23      596     0       False   2023-06-20 07:22:43.000000      N/A     Disabled
820     464     svchost.exe     0xfa8008d758b0  29      630     0       False   2023-06-20 07:22:43.000000      N/A     Disabled
892     464     svchost.exe     0xfa8008de1170  39      958     0       False   2023-06-20 07:22:44.000000      N/A     Disabled
240     464     svchost.exe     0xfa8008e17b00  20      777     0       False   2023-06-20 07:22:44.000000      N/A     Disabled
440     464     svchost.exe     0xfa8008e6eb00  17      490     0       False   2023-06-20 07:22:44.000000      N/A     Disabled
1128    820     dwm.exe 0xfa8008ed6220  3       72      1       False   2023-06-20 07:22:44.000000      N/A     Disabled
1152    1108    explorer.exe    0xfa8008ee4b00  36      1013    1       False   2023-06-20 07:22:44.000000      N/A     Disabled
1188    464     spoolsv.exe     0xfa8008f2c260  13      278     0       False   2023-06-20 07:22:44.000000      N/A     Disabled
1220    464     taskhost.exe    0xfa8008f4a5e0  8       195     1       False   2023-06-20 07:22:44.000000      N/A     Disabled
1244    464     svchost.exe     0xfa8008f51320  18      327     0       False   2023-06-20 07:22:44.000000      N/A     Disabled
1360    464     svchost.exe     0xfa8008fd9060  23      318     0       False   2023-06-20 07:22:45.000000      N/A     Disabled
1404    464     FoxitPDFReader  0xfa8009002b00  3       68      0       True    2023-06-20 07:22:45.000000      N/A     Disabled
1508    1152    vmtoolsd.exe    0xfa800905bb00  8       188     1       False   2023-06-20 07:22:45.000000      N/A     Disabled
1676    464     VGAuthService.  0xfa8007bb54b0  3       87      0       False   2023-06-20 07:22:46.000000      N/A     Disabled
1708    464     vm3dservice.ex  0xfa8009117b00  4       61      0       False   2023-06-20 07:22:46.000000      N/A     Disabled
1736    464     vmtoolsd.exe    0xfa80091135f0  12      282     0       False   2023-06-20 07:22:46.000000      N/A     Disabled
1760    1708    vm3dservice.ex  0xfa8009086b00  2       53      1       False   2023-06-20 07:22:46.000000      N/A     Disabled
2000    464     svchost.exe     0xfa8008d628b0  6       97      0       False   2023-06-20 07:22:47.000000      N/A     Disabled
332     612     WmiPrvSE.exe    0xfa8008055060  10      206     0       False   2023-06-20 07:22:47.000000      N/A     Disabled
196     464     dllhost.exe     0xfa8008146b00  13      194     0       False   2023-06-20 07:22:48.000000      N/A     Disabled
2136    464     SearchIndexer.  0xfa800824f3f0  11      586     0       False   2023-06-20 07:22:52.000000      N/A     Disabled
2256    464     msdtc.exe       0xfa80083acb00  12      149     0       False   2023-06-20 07:22:52.000000      N/A     Disabled
2512    464     wmpnetwk.exe    0xfa80083e5b00  13      422     0       False   2023-06-20 07:22:53.000000      N/A     Disabled
2624    464     svchost.exe     0xfa800921fb00  8       356     0       False   2023-06-20 07:22:54.000000      N/A     Disabled
3024    820     WUDFHost.exe    0xfa800931a750  8       202     0       False   2023-06-20 07:23:14.000000      N/A     Disabled
2456    464     sppsvc.exe      0xfa8006da0b00  4       152     0       False   2023-06-20 07:24:47.000000      N/A     Disabled
3284    464     svchost.exe     0xfa80074a5b00  13      353     0       False   2023-06-20 07:24:47.000000      N/A     Disabled
3356    464     taskhost.exe    0xfa8008df32b0  5       123     1       False   2023-06-20 07:54:22.000000      N/A     Disabled
2496    1152    MRCv120.exe     0xfa8008e9c060  4       282     1       True    2023-06-20 08:08:46.000000      N/A     Disabled
3468    732     audiodg.exe     0xfa800742ab00  5       128     0       False   2023-06-20 08:22:23.000000      N/A     Disabled
3316    612     WmiPrvSE.exe    0xfa8008f8ab00  5       113     0       False   2023-06-20 08:22:56.000000      N/A     Disabled
1872    1152    FoxitPDFReader  0xfa800748fb00  18      492     1       True    2023-06-20 08:26:55.000000      N/A     Disabled
4060    1872    FoxitPDFReader  0xfa800740d060  0       -       1       False   2023-06-20 08:27:01.000000      2023-06-20 08:27:02.000000      Disabled
2420    1152    FirefoxPortabl  0xfa8006db7980  6       147     1       True    2023-06-20 08:27:05.000000      N/A     Disabled
3716    2420    firefox.exe     0xfa8007fe0060  57      648     1       False   2023-06-20 08:27:05.000000      N/A     Disabled
4000    3716    firefox.exe     0xfa8008d03b00  26      313     1       False   2023-06-20 08:27:20.000000      N/A     Disabled
3476    1152    FoxitPDFReader  0xfa8008b60a80  2       113     1       True    2023-06-20 08:28:04.000000      N/A     Disabled
3396    612     dllhost.exe     0xfa8008b66760  6       7536754 1       False   2023-06-20 08:28:09.000000      N/A     Disabled
```

#### PSTree

```bash
vol -f memdump.raw windows.pstree
Volatility 3 Framework 2.4.1
Progress:  100.00               PDB scanning finished                        
PID     PPID    ImageFileName   Offset(V)       Threads Handles SessionId       Wow64   CreateTime      ExitTime

4       0       System  0xfa8006c7fb00  91      577     N/A     False   2023-06-20 07:22:42.000000      N/A
* 260   4       smss.exe        0xfa8007d39610  2       29      N/A     False   2023-06-20 07:22:42.000000      N/A
348     340     csrss.exe       0xfa80089a2b00  9       505     0       False   2023-06-20 07:22:43.000000      N/A
388     340     wininit.exe     0xfa80089ffb00  3       76      0       False   2023-06-20 07:22:43.000000      N/A
* 464   388     services.exe    0xfa8008c28b00  13      241     0       False   2023-06-20 07:22:43.000000      N/A
** 892  464     svchost.exe     0xfa8008de1170  39      958     0       False   2023-06-20 07:22:44.000000      N/A
** 1676 464     VGAuthService.  0xfa8007bb54b0  3       87      0       False   2023-06-20 07:22:46.000000      N/A
** 2456 464     sppsvc.exe      0xfa8006da0b00  4       152     0       False   2023-06-20 07:24:47.000000      N/A
** 3356 464     taskhost.exe    0xfa8008df32b0  5       123     1       False   2023-06-20 07:54:22.000000      N/A
** 1188 464     spoolsv.exe     0xfa8008f2c260  13      278     0       False   2023-06-20 07:22:44.000000      N/A
** 680  464     svchost.exe     0xfa8008d1d7e0  9       309     0       False   2023-06-20 07:22:43.000000      N/A
** 1708 464     vm3dservice.ex  0xfa8009117b00  4       61      0       False   2023-06-20 07:22:46.000000      N/A
*** 1760        1708    vm3dservice.ex  0xfa8009086b00  2       53      1       False   2023-06-20 07:22:46.000000      N/A
** 820  464     svchost.exe     0xfa8008d758b0  29      630     0       False   2023-06-20 07:22:43.000000      N/A
*** 1128        820     dwm.exe 0xfa8008ed6220  3       72      1       False   2023-06-20 07:22:44.000000      N/A
*** 3024        820     WUDFHost.exe    0xfa800931a750  8       202     0       False   2023-06-20 07:23:14.000000      N/A
** 440  464     svchost.exe     0xfa8008e6eb00  17      490     0       False   2023-06-20 07:22:44.000000      N/A
** 2624 464     svchost.exe     0xfa800921fb00  8       356     0       False   2023-06-20 07:22:54.000000      N/A
** 1220 464     taskhost.exe    0xfa8008f4a5e0  8       195     1       False   2023-06-20 07:22:44.000000      N/A
** 196  464     dllhost.exe     0xfa8008146b00  13      194     0       False   2023-06-20 07:22:48.000000      N/A
** 1736 464     vmtoolsd.exe    0xfa80091135f0  12      282     0       False   2023-06-20 07:22:46.000000      N/A
** 1360 464     svchost.exe     0xfa8008fd9060  23      318     0       False   2023-06-20 07:22:45.000000      N/A
** 2000 464     svchost.exe     0xfa8008d628b0  6       97      0       False   2023-06-20 07:22:47.000000      N/A
** 2256 464     msdtc.exe       0xfa80083acb00  12      149     0       False   2023-06-20 07:22:52.000000      N/A
** 2512 464     wmpnetwk.exe    0xfa80083e5b00  13      422     0       False   2023-06-20 07:22:53.000000      N/A
** 3284 464     svchost.exe     0xfa80074a5b00  13      353     0       False   2023-06-20 07:24:47.000000      N/A
** 2136 464     SearchIndexer.  0xfa800824f3f0  11      586     0       False   2023-06-20 07:22:52.000000      N/A
** 732  464     svchost.exe     0xfa8008d3a6d0  23      596     0       False   2023-06-20 07:22:43.000000      N/A
*** 3468        732     audiodg.exe     0xfa800742ab00  5       128     0       False   2023-06-20 08:22:23.000000      N/A
** 1244 464     svchost.exe     0xfa8008f51320  18      327     0       False   2023-06-20 07:22:44.000000      N/A
** 612  464     svchost.exe     0xfa8008c49b00  10      367     0       False   2023-06-20 07:22:43.000000      N/A
*** 3316        612     WmiPrvSE.exe    0xfa8008f8ab00  5       113     0       False   2023-06-20 08:22:56.000000      N/A
*** 332 612     WmiPrvSE.exe    0xfa8008055060  10      206     0       False   2023-06-20 07:22:47.000000      N/A
*** 3396        612     dllhost.exe     0xfa8008b66760  6       7536754 1       False   2023-06-20 08:28:09.000000      N/A
** 240  464     svchost.exe     0xfa8008e17b00  20      777     0       False   2023-06-20 07:22:44.000000      N/A
** 1404 464     FoxitPDFReader  0xfa8009002b00  3       68      0       True    2023-06-20 07:22:45.000000      N/A
* 480   388     lsm.exe 0xfa8008c45b00  10      153     0       False   2023-06-20 07:22:43.000000      N/A
* 472   388     lsass.exe       0xfa8008c35b00  8       767     0       False   2023-06-20 07:22:43.000000      N/A
400     380     csrss.exe       0xfa8008993b00  11      352     1       False   2023-06-20 07:22:43.000000      N/A
492     380     winlogon.exe    0xfa8008c39b00  3       113     1       False   2023-06-20 07:22:43.000000      N/A
1152    1108    explorer.exe    0xfa8008ee4b00  36      1013    1       False   2023-06-20 07:22:44.000000      N/A
* 2496  1152    MRCv120.exe     0xfa8008e9c060  4       282     1       True    2023-06-20 08:08:46.000000      N/A
* 1508  1152    vmtoolsd.exe    0xfa800905bb00  8       188     1       False   2023-06-20 07:22:45.000000      N/A
* 1872  1152    FoxitPDFReader  0xfa800748fb00  18      492     1       True    2023-06-20 08:26:55.000000      N/A
** 4060 1872    FoxitPDFReader  0xfa800740d060  0       -       1       False   2023-06-20 08:27:01.000000      2023-06-20 08:27:02.000000 
* 2420  1152    FirefoxPortabl  0xfa8006db7980  6       147     1       True    2023-06-20 08:27:05.000000      N/A
** 3716 2420    firefox.exe     0xfa8007fe0060  57      648     1       False   2023-06-20 08:27:05.000000      N/A
*** 4000        3716    firefox.exe     0xfa8008d03b00  26      313     1       False   2023-06-20 08:27:20.000000      N/A
* 3476  1152    FoxitPDFReader  0xfa8008b60a80  2       113     1       True    2023-06-20 08:28:04.000000      N/A
```

Based on the result provided from both volatility 2 and 3, there is several interesting processes such as `FoxitPDFReader` and `firefox.exe` which might have some information in it. Before further digging into the processes, lets try to check the command line first to see if there is any information

## Step 3: gathering and analyzing command line

To check the command line from the given file, the command used for both volatility 2 and 3 is **cmdline**.

### Volatility 2

```bash
vol2 -f memdump.raw --profile Win7SP1x64 cmdline 
Volatility Foundation Volatility Framework 2.6
************************************************************************
System pid:      4
************************************************************************
smss.exe pid:    260
************************************************************************
csrss.exe pid:    348
************************************************************************
wininit.exe pid:    388
************************************************************************
csrss.exe pid:    400
Command line : 
************************************************************************
services.exe pid:    464
Command line : 
************************************************************************
lsass.exe pid:    472
Command line : 
************************************************************************
lsm.exe pid:    480
Command line : 
************************************************************************
winlogon.exe pid:    492
************************************************************************
svchost.exe pid:    612
Command line : 
************************************************************************
svchost.exe pid:    680
Command line : 
************************************************************************
svchost.exe pid:    732
Command line : C:\Windows\System32\svchost.exe -k LocalServiceNetworkRestricted
************************************************************************
svchost.exe pid:    820
Command line : C:\Windows\System32\svchost.exe -k LocalSystemNetworkRestricted
************************************************************************
svchost.exe pid:    892
Command line : C:\Windows\system32\svchost.exe -k netsvcs
************************************************************************
svchost.exe pid:    240
Command line : 
************************************************************************
svchost.exe pid:    440
Command line : 
************************************************************************
dwm.exe pid:   1128
************************************************************************
explorer.exe pid:   1152
Command line : C:\Windows\Explorer.EXE
************************************************************************
spoolsv.exe pid:   1188
Command line : 
************************************************************************
taskhost.exe pid:   1220
Command line : 
************************************************************************
svchost.exe pid:   1244
Command line : 
************************************************************************
svchost.exe pid:   1360
Command line : 
************************************************************************
FoxitPDFReader pid:   1404
************************************************************************
vmtoolsd.exe pid:   1508
Command line : "C:\Program Files\VMware\VMware Tools\vmtoolsd.exe" -n vmusr
************************************************************************
VGAuthService. pid:   1676
************************************************************************
vm3dservice.ex pid:   1708
************************************************************************
vmtoolsd.exe pid:   1736
Command line : "C:\Program Files\VMware\VMware Tools\vmtoolsd.exe"
************************************************************************
vm3dservice.ex pid:   1760
************************************************************************
svchost.exe pid:   2000
************************************************************************
WmiPrvSE.exe pid:    332
Command line : C:\Windows\system32\wbem\wmiprvse.exe
************************************************************************
dllhost.exe pid:    196
************************************************************************
SearchIndexer. pid:   2136
Command line : C:\Windows\system32\SearchIndexer.exe /Embedding
************************************************************************
msdtc.exe pid:   2256
************************************************************************
wmpnetwk.exe pid:   2512
Command line : 
************************************************************************
svchost.exe pid:   2624
Command line : C:\Windows\System32\svchost.exe -k LocalServicePeerNet
************************************************************************
WUDFHost.exe pid:   3024
************************************************************************
sppsvc.exe pid:   2456
************************************************************************
svchost.exe pid:   3284
Command line : C:\Windows\System32\svchost.exe -k secsvcs
************************************************************************
taskhost.exe pid:   3356
************************************************************************
MRCv120.exe pid:   2496
Command line : 
************************************************************************
audiodg.exe pid:   3468
Command line : C:\Windows\system32\AUDIODG.EXE 0x814
************************************************************************
WmiPrvSE.exe pid:   3316
Command line : 
************************************************************************
FoxitPDFReader pid:   1872
Command line : 
************************************************************************
FoxitPDFReader pid:   4060
************************************************************************
FirefoxPortabl pid:   2420
************************************************************************
firefox.exe pid:   3716
Command line : 
************************************************************************
firefox.exe pid:   4000
Command line : "E:\FirefoxPortable\App\firefox64\firefox.exe" -contentproc --channel="3716.0.1257441394\170687372" -greomni "E:\FirefoxPortable\App\firefox64\omni.ja" -appomni "E:\FirefoxPortable\App\firefox64\browser\omni.ja" -appdir "E:\FirefoxPortable\App\firefox64\browser"  3716 "\\.\pipe\gecko-crash-server-pipe.3716" tab
************************************************************************
FoxitPDFReader pid:   3476
Command line : "C:\Program Files (x86)\Foxit Software\Foxit PDF Reader\FoxitPDFReader.exe" "E:\591dedcb722fc7fea0c0f378e3192d78.pdf"
************************************************************************
dllhost.exe pid:   3396
```

### Volatility 3

```bash
vol -f memdump.raw windows.cmdline
Volatility 3 Framework 2.4.1
Progress:  100.00               PDB scanning finished                        
PID     Process Args

4       System  Required memory at 0x20 is inaccessible (swapped)
260     smss.exe        Required memory at 0x7fffffd4020 is inaccessible (swapped)
348     csrss.exe       Required memory at 0x7fffffdd020 is inaccessible (swapped)
388     wininit.exe     Required memory at 0x7fffffdf020 is inaccessible (swapped)
400     csrss.exe       Required memory at 0x818e8 is inaccessible (swapped)
464     services.exe    Required memory at 0x361c28 is inaccessible (swapped)
472     lsass.exe       Required memory at 0x361c28 is inaccessible (swapped)
480     lsm.exe Required memory at 0x2b1c28 is inaccessible (swapped)
492     winlogon.exe    Required memory at 0x7fffffd7020 is inaccessible (swapped)
612     svchost.exe     Required memory at 0x894c38244489 is not valid (process exited?)
680     svchost.exe     Required memory at 0x271df8 is inaccessible (swapped)
732     svchost.exe     C:\Windows\System32\svchost.exe -k LocalServiceNetworkRestricted
820     svchost.exe     C:\Windows\System32\svchost.exe -k LocalSystemNetworkRestricted
892     svchost.exe     C:\Windows\system32\svchost.exe -k netsvcs
240     svchost.exe     Required memory at 0x421df8 is inaccessible (swapped)
440     svchost.exe     Required memory at 0x341df8 is inaccessible (swapped)
1128    dwm.exe Required memory at 0x7fffffda020 is inaccessible (swapped)
1152    explorer.exe    C:\Windows\Explorer.EXE
1188    spoolsv.exe     Required memory at 0x301d68 is inaccessible (swapped)
1220    taskhost.exe    Required memory at 0x281e48 is inaccessible (swapped)
1244    svchost.exe     Required memory at 0x311df8 is inaccessible (swapped)
1360    svchost.exe     Required memory at 0x191df8 is inaccessible (swapped)
1404    FoxitPDFReader  Required memory at 0x7efdf020 is inaccessible (swapped)
1508    vmtoolsd.exe    "C:\Program Files\VMware\VMware Tools\vmtoolsd.exe" -n vmusr
1676    VGAuthService.  Required memory at 0x7fffffdf020 is inaccessible (swapped)
1708    vm3dservice.ex  Required memory at 0x7fffffdf020 is inaccessible (swapped)
1736    vmtoolsd.exe    "C:\Program Files\VMware\VMware Tools\vmtoolsd.exe"
1760    vm3dservice.ex  Required memory at 0x7fffffd8020 is inaccessible (swapped)
2000    svchost.exe     Required memory at 0x7fffffdf020 is inaccessible (swapped)
332     WmiPrvSE.exe    C:\Windows\system32\wbem\wmiprvse.exe
196     dllhost.exe     Required memory at 0x7fffffdf020 is inaccessible (swapped)
2136    SearchIndexer.  C:\Windows\system32\SearchIndexer.exe /Embedding
2256    msdtc.exe       Required memory at 0x7fffffd4020 is inaccessible (swapped)
2512    wmpnetwk.exe    Required memory at 0x1d1df8 is inaccessible (swapped)
2624    svchost.exe     C:\Windows\System32\svchost.exe -k LocalServicePeerNet
3024    WUDFHost.exe    Required memory at 0x7fffffdd020 is inaccessible (swapped)
2456    sppsvc.exe      Required memory at 0x7fffffdb020 is inaccessible (swapped)
3284    svchost.exe     C:\Windows\System32\svchost.exe -k secsvcs
3356    taskhost.exe    Required memory at 0x7fffffde020 is inaccessible (swapped)
2496    MRCv120.exe     Required memory at 0x1f1e48 is inaccessible (swapped)
3468    audiodg.exe     C:\Windows\system32\AUDIODG.EXE 0x814
3316    WmiPrvSE.exe    Required memory at 0x251d68 is inaccessible (swapped)
1872    FoxitPDFReader  Required memory at 0x271ee8 is inaccessible (swapped)
4060    FoxitPDFReader  Required memory at 0x7efdf020 is not valid (process exited?)
2420    FirefoxPortabl  Required memory at 0x7efdf020 is inaccessible (swapped)
3716    firefox.exe     Required memory at 0x302682 is inaccessible (swapped)
4000    firefox.exe     "E:\FirefoxPortable\App\firefox64\firefox.exe" -contentproc --channel="3716.0.1257441394\170687372" -greomni "E:\FirefoxPortable\App\firefox64\omni.ja" -appomni "E:\FirefoxPortable\App\firefox64\browser\omni.ja" -appdir "E:\FirefoxPortable\App\firefox64\browser"  3716 "\\.\pipe\gecko-crash-server-pipe.3716" tab
3476    FoxitPDFReader  "C:\Program Files (x86)\Foxit Software\Foxit PDF Reader\FoxitPDFReader.exe" "E:\591dedcb722fc7fea0c0f378e3192d78.pdf"
3396    dllhost.exe     Required memory at 0x7fffffd5020 is not valid (process exited?)
```

Based on the result above, there is 2 things to take note. One of it is `firefox` is running and the another one is the `FoxitPDFReader` is reading a file named `591dedcb722fc7fea0c0f378e3192d78.pdf`. Aside from that, each of the command line has its own `PID` which is useful for dumping out more information. Moving on will be trying to dump out the information based on the processes.

## Step 4: Dumping out the memory of the processes

Based on previous information, the processes that will be dump out is `FoxitPDFReader` and the PID is `3476`. The command that will be used for volatiltiy 2 and 3 will be different

### Volatility 2

```bash
vol2 -f memdump.raw --profile Win7SP1x64 memdump -p 3476 --dump-dir=v2
Volatility Foundation Volatility Framework 2.6
************************************************************************
Writing FoxitPDFReader [  3476] to 3476.dmp


strings  v2/3476.dmp | grep pdf | head
"C:\Program Files (x86)\Foxit Software\Foxit PDF Reader\FoxitPDFReader.exe" "E:\591dedcb722fc7fea0c0f378e3192d78.pdf"
isCpdf
c:\phantompdfci\jenkins\workspace\taa-ph-auto-compile\starship\bcgcbpro\bcgglobals.cpp
c:\phantompdfci\jenkins\workspace\taa-ph-auto-compile\starship\bcgcbpro\bcgpvisualmanager.cpp
c:\phantompdfci\jenkins\workspace\taa-ph-auto-compile\starship\bcgcbpro\bcgpmenubar.cpp
c:\phantompdfci\jenkins\workspace\taa-ph-auto-compile\starship\bcgcbpro\bcgpcontrolbar.cpp
c:\phantompdfci\jenkins\workspace\taa-ph-auto-compile\starship\bcgcbpro\bcgptabwnd.cpp
c:\phantompdfci\jenkins\workspace\taa-ph-auto-compile\starship\bcgcbpro\bcgpribbonbar.cpp
c:\phantompdfci\jenkins\workspace\taa-ph-auto-compile\starship\bcgcbpro\bcgpmainclientareawnd.cpp
c:\phantompdfci\jenkins\workspace\taa-ph-auto-compile\starship\bcgcbpro\bcgpfullscreenimpl.cpp
```

### Volatility 3

```bash
vol -f memdump.raw -o v3 windows.memmap --dump --pid 3476 
Volatility 3 Framework 2.4.1
Progress:  100.00               PDB scanning finished                        
Virtual Physical        Size    Offset in File  File output

strings v3/pid.3476.dmp | grep pdf | head
"C:\Program Files (x86)\Foxit Software\Foxit PDF Reader\FoxitPDFReader.exe" "E:\591dedcb722fc7fea0c0f378e3192d78.pdf"
isCpdf
c:\phantompdfci\jenkins\workspace\taa-ph-auto-compile\starship\bcgcbpro\bcgglobals.cpp
c:\phantompdfci\jenkins\workspace\taa-ph-auto-compile\starship\bcgcbpro\bcgpvisualmanager.cpp
c:\phantompdfci\jenkins\workspace\taa-ph-auto-compile\starship\bcgcbpro\bcgpmenubar.cpp
c:\phantompdfci\jenkins\workspace\taa-ph-auto-compile\starship\bcgcbpro\bcgpcontrolbar.cpp
c:\phantompdfci\jenkins\workspace\taa-ph-auto-compile\starship\bcgcbpro\bcgptabwnd.cpp
c:\phantompdfci\jenkins\workspace\taa-ph-auto-compile\starship\bcgcbpro\bcgpribbonbar.cpp
c:\phantompdfci\jenkins\workspace\taa-ph-auto-compile\starship\bcgcbpro\bcgpmainclientareawnd.cpp
c:\phantompdfci\jenkins\workspace\taa-ph-auto-compile\starship\bcgcbpro\bcgpfullscreenimpl.cpp
```

Based on the result, it is confirmed that someone is trying to read `591dedcb722fc7fea0c0f378e3192d78.pdf`. Moving on, lets try to search the pdf file and dump it out.

## Step 5: searching the pdf file

Before dumping out the pdf file, we will need to search for the file to ensure it exist in the raw image first. To do so, both volatility will be using the same command **filescan**.

### Volatility 2

```bash
vol2 -f memdump.raw --profile Win7SP1x64 filescan | grep 591dedcb722fc7fea0c0f378e3192d78.pdf
Volatility Foundation Volatility Framework 2.6
0x000000001ea48070      3      0 R--rw- \Device\HarddiskVolume3\591dedcb722fc7fea0c0f378e3192d78.pdf
0x000000001edb2470      1      1 R--r-- \Device\HarddiskVolume3\591dedcb722fc7fea0c0f378e3192d78.pdf
```

### Volatility 3

```bash
vol -f memdump.raw windows.filescan | grep 591dedcb722fc7fea0c0f378e3192d78.pdf
0x1ea48070 100.0\591dedcb722fc7fea0c0f378e3192d78.pdf   216
0x1edb2470      \591dedcb722fc7fea0c0f378e3192d78.pdf   216
```

Now that we had confimed that the file exist in the raw image, lets try to dump it out.

## Step 6: Dump the pdf file

To dump the pdf file, the both volatility command will be the same which is **dumpfiles**.

### Volatiltiy 2

```bash
vol2 -f memdump.raw --profile Win7SP1x64 dumpfiles --dump-dir=v2 -Q 0x000000001ea48070       
Volatility Foundation Volatility Framework 2.6
DataSectionObject 0x1ea48070   None   \Device\HarddiskVolume3\591dedcb722fc7fea0c0f378e3192d78.pdf
SharedCacheMap 0x1ea48070   None   \Device\HarddiskVolume3\591dedcb722fc7fea0c0f378e3192d78.pdf

vol2 -f memdump.raw --profile Win7SP1x64 dumpfiles --dump-dir=v2 -Q 0x000000001edb2470
Volatility Foundation Volatility Framework 2.6
DataSectionObject 0x1edb2470   None   \Device\HarddiskVolume3\591dedcb722fc7fea0c0f378e3192d78.pdf
SharedCacheMap 0x1edb2470   None   \Device\HarddiskVolume3\591dedcb722fc7fea0c0f378e3192d78.pdf
```

### Volatility 3

```bash
vol -f memdump.raw -o v3 windows.dumpfiles --physaddr 0x1ea48070 
Volatility 3 Framework 2.4.1
Progress:  100.00               PDB scanning finished                        
Cache   FileObject      FileName        Result

DataSectionObject       0x1ea48070      591dedcb722fc7fea0c0f378e3192d78.pdf    file.0x1ea48070.0xfa800743b250.DataSectionObject.591dedcb722fc7fea0c0f378e3192d78.pdf.dat
SharedCacheMap  0x1ea48070      591dedcb722fc7fea0c0f378e3192d78.pdf    file.0x1ea48070.0xfa8008ba3e00.SharedCacheMap.591dedcb722fc7fea0c0f378e3192d78.pdf.vacb

vol -f memdump.raw -o v3 windows.dumpfiles --physaddr 0x1edb2470
Volatility 3 Framework 2.4.1
Progress:  100.00               PDB scanning finished                        
Cache   FileObject      FileName        Result

DataSectionObject       0x1edb2470      591dedcb722fc7fea0c0f378e3192d78.pdf    file.0x1edb2470.0xfa800743b250.DataSectionObject.591dedcb722fc7fea0c0f378e3192d78.pdf.dat
SharedCacheMap  0x1edb2470      591dedcb722fc7fea0c0f378e3192d78.pdf    file.0x1edb2470.0xfa8008ba3e00.SharedCacheMap.591dedcb722fc7fea0c0f378e3192d78.pdf.vacb
```

After dumping out the file, the next thing to do is try to analyze the file.

## Step 7: Analyzing the dumped file

this section will still be same as previous as we will analyze files that dumped from both volatility.

### Volatility 2 dumped file

```bash
file v2/*
v2/file.None.0xfffffa8008ba3e00.vacb: Zip archive data, at least v2.0 to extract, compression method=AES Encrypted
v2/file.None.0xfffffa800743b250.dat:  Zip archive data, at least v2.0 to extract, compression method=AES Encrypted
```

### Volatility 3 dumped file

```bash
file v3/*
v3/file.0x1ea48070.0xfa8008ba3e00.SharedCacheMap.591dedcb722fc7fea0c0f378e3192d78.pdf.vacb:   Zip archive data, at least v2.0 to extract, compression method=AES Encrypted
v3/file.0x1ea48070.0xfa800743b250.DataSectionObject.591dedcb722fc7fea0c0f378e3192d78.pdf.dat: Zip archive data, at least v2.0 to extract, compression method=AES Encrypted
v3/file.0x1edb2470.0xfa8008ba3e00.SharedCacheMap.591dedcb722fc7fea0c0f378e3192d78.pdf.vacb:   Zip archive data, at least v2.0 to extract, compression method=AES Encrypted
v3/file.0x1edb2470.0xfa800743b250.DataSectionObject.591dedcb722fc7fea0c0f378e3192d78.pdf.dat: Zip archive data, at least v2.0 to extract, compression method=AES Encrypted
```

Volatility 2 only have 2 files while volatility 3 have 4 files. This is because volatility 2 overwrite the file since it is same name while volatility 3 have different names. Based on the result, the files is not a pdf file but instead a zip file. The next step would be trying to unzip it. 

## Step 8: Unzip the file

Before that unzipping the file, it is compulsory to change the file extension to `.zip` to succuessfully unzip it. 

### Volatility 2 dumped file

```bash
mv file.None.0xfffffa8008ba3e00.vacb test.zip 
                                                                                                                                                             
mv file.None.0xfffffa800743b250.dat test2.zip

zip2john *.zip
test2.zip/flag.txt:$zip2$*0*3*0*810ac093ee40b9a6e100e9c136806c67*1712*27*5c46c3495afd25681727887a9c7f9b1cb6692d6fc7b9b9a6f1bb369073c1b6e8897d567bedbc2d*59f3515251e6914d983d*$/zip2$:flag.txt:test2.zip:test2.zip
Did not find End Of Central Directory.
```

### Volatility 3 dumped file

```bash
mv file.0x1ea48070.0xfa8008ba3e00.SharedCacheMap.591dedcb722fc7fea0c0f378e3192d78.pdf.vacb test.zip
                                                                                                                                                             
mv file.0x1ea48070.0xfa800743b250.DataSectionObject.591dedcb722fc7fea0c0f378e3192d78.pdf.dat test2.zip
                                                                                                                                                             
mv file.0x1edb2470.0xfa8008ba3e00.SharedCacheMap.591dedcb722fc7fea0c0f378e3192d78.pdf.vacb test3.zip
                                                                                                                                                             
mv file.0x1edb2470.0xfa800743b250.DataSectionObject.591dedcb722fc7fea0c0f378e3192d78.pdf.dat test4.zip

zip2john *.zip
test2.zip/flag.txt:$zip2$*0*3*0*810ac093ee40b9a6e100e9c136806c67*1712*27*5c46c3495afd25681727887a9c7f9b1cb6692d6fc7b9b9a6f1bb369073c1b6e8897d567bedbc2d*59f3515251e6914d983d*$/zip2$:flag.txt:test2.zip:test2.zip
Did not find End Of Central Directory.
test4.zip/flag.txt:$zip2$*0*3*0*810ac093ee40b9a6e100e9c136806c67*1712*27*5c46c3495afd25681727887a9c7f9b1cb6692d6fc7b9b9a6f1bb369073c1b6e8897d567bedbc2d*59f3515251e6914d983d*$/zip2$:flag.txt:test4.zip:test4.zip
Did not find End Of Central Directory.
```

After renaming the file, I used a tool named `zip2john` to check if the zip file is having any password. Based on the result, all the result is same as the provide the same hashes which means the zip file is password protected. Since the `firefox` processes has not gone through yet, lets try to go through it to check if there is any password. 

## Step 9: gathering informtion from firefox

Since the processes stated that someone is running firefox, the best way to gather information is by checking the browser history. By default, the history of firefox will be saved into a file named `places`. for more information about default file name for each browser history, click [here](https://docs.nxlog.co/userguide/integrate/browser-history.html). Since we are looking for files, lets search for the relevant file name

### Volatility 2

```bash
vol2 -f memdump.raw --profile Win7SP1x64 filescan | grep places.sql
Volatility Foundation Volatility Framework 2.6
0x000000001d8e3c60      1      1 R--rw- \Device\HarddiskVolume3\FirefoxPortable\Data\profile\places.sqlite
0x000000001dad8070     15      1 RW-rw- \Device\HarddiskVolume3\FirefoxPortable\Data\profile\places.sqlite-shm
0x000000001df039c0      1      1 RW-rw- \Device\HarddiskVolume3\FirefoxPortable\Data\profile\places.sqlite-wal
0x000000001e8ae250     14      1 RW-rw- \Device\HarddiskVolume3\FirefoxPortable\Data\profile\places.sqlite-wal
0x000000001eb898c0      1      1 R--rw- \Device\HarddiskVolume3\FirefoxPortable\Data\profile\places.sqlite
0x000000001ed846b0      1      1 R--rw- \Device\HarddiskVolume3\FirefoxPortable\Data\profile\places.sqlite
0x000000001edc01d0      1      1 RW-rw- \Device\HarddiskVolume3\FirefoxPortable\Data\profile\places.sqlite-wal
0x000000001ef895b0      1      1 RW-rw- \Device\HarddiskVolume3\FirefoxPortable\Data\profile\places.sqlite-wal
0x000000001fe27830      5      1 RW-rw- \Device\HarddiskVolume3\FirefoxPortable\Data\profile\places.sqlite
```

### Volatility 3

```bash
vol -f memdump.raw windows.filescan | grep places.sql
0x1d8e3c60 100.0\FirefoxPortable\Data\profile\places.sqlite     216
0x1dad8070      \FirefoxPortable\Data\profile\places.sqlite-shm 216
0x1df039c0      \FirefoxPortable\Data\profile\places.sqlite-wal 216
0x1e8ae250      \FirefoxPortable\Data\profile\places.sqlite-wal 216
0x1eb898c0      \FirefoxPortable\Data\profile\places.sqlite     216
0x1ed846b0      \FirefoxPortable\Data\profile\places.sqlite     216
0x1edc01d0      \FirefoxPortable\Data\profile\places.sqlite-wal 216
0x1ef895b0      \FirefoxPortable\Data\profile\places.sqlite-wal 216
0x1fe27830      \FirefoxPortable\Data\profile\places.sqlite     216
```

Based on the result, there are a few files that contain the names of `places`. Lets just dump one out from each volatility to check the information inside it.

## Step 10: Dump the sqlite file

The command will be same as previous for dumping the files.

### Volatility 2

```bash
vol2 -f memdump.raw --profile Win7SP1x64 dumpfiles --dump-dir=v2 -Q 0x000000001d8e3c60
Volatility Foundation Volatility Framework 2.6
DataSectionObject 0x1d8e3c60   None   \Device\HarddiskVolume3\FirefoxPortable\Data\profile\places.sqlite
SharedCacheMap 0x1d8e3c60   None   \Device\HarddiskVolume3\FirefoxPortable\Data\profile\places.sqlite
```

### Volatility 3

```bash
vol -f memdump.raw -o v3 windows.dumpfiles --physaddr 0x1d8e3c60 
Volatility 3 Framework 2.4.1
Progress:  100.00               PDB scanning finished                        
Cache   FileObject      FileName        Result

DataSectionObject       0x1d8e3c60      places.sqlite   Error dumping file
SharedCacheMap  0x1d8e3c60      places.sqlite   file.0x1d8e3c60.0xfa8009074d80.SharedCacheMap.places.sqlite.vacb
```

After dumping out the file, lets read the file. 

## Step 11: read the dumped sqlite file

### Volatility 2 dumped file

```bash
strings v2/file.None.0xfffffa8009074d80.vacb| grep google.com/search
https://www.google.com/search?hl=en-MY&gbv=2&q=amitriptyline+malaysia&oq=amitriptyline+malaysia&aqs=heirloom-srp..0l5amitriptyline malaysia - Google Searchmoc.elgoog.www.
https://www.google.com/search?hl=en-MY&gbv=2&q=passwd%3A+whodiditidk&oq=passwd%3A+whodiditidk&aqs=heirloom-srp..passwd: whodiditidk - Google Searchmoc.elgoog.www.
https://www.google.com/search?hl=en-MY&gbv=2&q=amitriptyline+overdose&oq=amitriptyline++over&aqs=heirloom-srp.0.0l5amitriptyline overdose - Google Searchmoc.elgoog.www.
https://www.google.com/search?hl=en-MY&source=hp&biw=&bih=&q=can+i+buy+amitriptyline+over+the+counter+malaysia&iflsig=AOEireoAAAAAZJFiMvT6EIZLI3gc6YwsYTCfmgRHROIi&gbv=2&oq=can+i+buy+amitriptyline+over+the+counter+malaysia&gs_l=heirloom-hp.3..0i546l5.6365.18936.0.19223.29.28.0.1.1.0.2237.12264.4j5j9j4j2j9-3.27.0....0...1ac.1.34.heirloom-hp..13.16.4581.6kGofs948xEcan i buy amitriptyline over the counter malaysia - Google Searchmoc.elgoog.www.
https://www.google.com/search?hl=en-MY&source=hp&biw=&bih=&q=amitriptyline&iflsig=AOEireoAAAAAZJE-FIBk0yE9cTyRfsJGgKzsGpPtubIo&gbv=2&oq=amitriptyline&gs_l=heirloom-hp.3..0i512i433i131l3j0i512l7.23131.26384.0.28332.8.7.0.1.1.0.153.549.6j1.7.0....0...1ac.1.34.heirloom-hp..0.8.577.n4YLioOAAPcamitriptyline - Google Searchmoc.elgoog.www.
strings v2/file.None.0xfffffa8009074d80.vacb| grep google.com/search
https://www.google.com/search?hl=en-MY&gbv=2&q=amitriptyline+malaysia&oq=amitriptyline+malaysia&aqs=heirloom-srp..0l5amitriptyline malaysia - Google Searchmoc.elgoog.www.
https://www.google.com/search?hl=en-MY&gbv=2&q=passwd%3A+whodiditidk&oq=passwd%3A+whodiditidk&aqs=heirloom-srp..passwd: whodiditidk - Google Searchmoc.elgoog.www.
https://www.google.com/search?hl=en-MY&gbv=2&q=amitriptyline+overdose&oq=amitriptyline++over&aqs=heirloom-srp.0.0l5amitriptyline overdose - Google Searchmoc.elgoog.www.
https://www.google.com/search?hl=en-MY&source=hp&biw=&bih=&q=can+i+buy+amitriptyline+over+the+counter+malaysia&iflsig=AOEireoAAAAAZJFiMvT6EIZLI3gc6YwsYTCfmgRHROIi&gbv=2&oq=can+i+buy+amitriptyline+over+the+counter+malaysia&gs_l=heirloom-hp.3..0i546l5.6365.18936.0.19223.29.28.0.1.1.0.2237.12264.4j5j9j4j2j9-3.27.0....0...1ac.1.34.heirloom-hp..13.16.4581.6kGofs948xEcan i buy amitriptyline over the counter malaysia - Google Searchmoc.elgoog.www.
https://www.google.com/search?hl=en-MY&source=hp&biw=&bih=&q=amitriptyline&iflsig=AOEireoAAAAAZJE-FIBk0yE9cTyRfsJGgKzsGpPtubIo&gbv=2&oq=amitriptyline&gs_l=heirloom-hp.3..0i512i433i131l3j0i512l7.23131.26384.0.28332.8.7.0.1.1.0.153.549.6j1.7.0....0...1ac.1.34.heirloom-hp..0.8.577.n4YLioOAAPcamitriptyline - Google Searchmoc.elgoog.www.
```

### Volatility 3 dumped file

```bash
strings v3/file.0x1d8e3c60.0xfa8009074d80.SharedCacheMap.places.sqlite.vacb| grep google.com/search
https://www.google.com/search?hl=en-MY&gbv=2&q=amitriptyline+malaysia&oq=amitriptyline+malaysia&aqs=heirloom-srp..0l5amitriptyline malaysia - Google Searchmoc.elgoog.www.
https://www.google.com/search?hl=en-MY&gbv=2&q=passwd%3A+whodiditidk&oq=passwd%3A+whodiditidk&aqs=heirloom-srp..passwd: whodiditidk - Google Searchmoc.elgoog.www.
https://www.google.com/search?hl=en-MY&gbv=2&q=amitriptyline+overdose&oq=amitriptyline++over&aqs=heirloom-srp.0.0l5amitriptyline overdose - Google Searchmoc.elgoog.www.
https://www.google.com/search?hl=en-MY&source=hp&biw=&bih=&q=can+i+buy+amitriptyline+over+the+counter+malaysia&iflsig=AOEireoAAAAAZJFiMvT6EIZLI3gc6YwsYTCfmgRHROIi&gbv=2&oq=can+i+buy+amitriptyline+over+the+counter+malaysia&gs_l=heirloom-hp.3..0i546l5.6365.18936.0.19223.29.28.0.1.1.0.2237.12264.4j5j9j4j2j9-3.27.0....0...1ac.1.34.heirloom-hp..13.16.4581.6kGofs948xEcan i buy amitriptyline over the counter malaysia - Google Searchmoc.elgoog.www.
https://www.google.com/search?hl=en-MY&source=hp&biw=&bih=&q=amitriptyline&iflsig=AOEireoAAAAAZJE-FIBk0yE9cTyRfsJGgKzsGpPtubIo&gbv=2&oq=amitriptyline&gs_l=heirloom-hp.3..0i512i433i131l3j0i512l7.23131.26384.0.28332.8.7.0.1.1.0.153.549.6j1.7.0....0...1ac.1.34.heirloom-hp..0.8.577.n4YLioOAAPcamitriptyline - Google Searchmoc.elgoog.www.
```

Based on the information provided from the `places.sqlite` file, there is a potential password provided `passwd: whodiditidk`. Lets just try to unzip the file with the password and hope it works.

## Step 12: Unzip the file with the password given

all files dumped from volatility 2 and 3 gives the same hashes which mean the password should be same. one of the filed will be used as example to unzip.

```bash
7z x test2.zip    

7-Zip [64] 16.02 : Copyright (c) 1999-2016 Igor Pavlov : 2016-05-21
p7zip Version 16.02 (locale=en_US.UTF-8,Utf16=on,HugeFiles=on,64 bits,128 CPUs Intel(R) Core(TM) i5-9300H CPU @ 2.40GHz (906EA),ASM,AES-NI)

Scanning the drive for archives:
1 file, 4096 bytes (4 KiB)

Extracting archive: test2.zip

WARNINGS:
There are data after the end of archive

--
Path = test2.zip
Type = zip
WARNINGS:
There are data after the end of archive
Physical Size = 255
Tail Size = 3841

    
Enter password (will not be echoed):
Everything is Ok

Archives with Warnings: 1

Warnings: 1
Size:       37
Compressed: 4096

cat flag.txt                        
BAT{442b6acb58091deddb8bf3b6f1062ee4}   
```

With that, we managed to solve the challenge and get the exact flag. 


## Things I learned from the challenge
- volatility 2 and 3 commands
- dumping out interesting files from raw image
- browser history default name