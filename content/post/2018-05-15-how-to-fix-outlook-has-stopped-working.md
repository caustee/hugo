---
date: "2018-05-15T00:00:00Z"
title: How to fix Microsoft Lync/Outlook has stopped working
---

![lync-error](/images/outlook-error.jpeg "Lync Error")

Recently I had this issue that ate a big part of my day. Here's how I fixed it.

#### Firstly, some background info
I'm using a Win 10 (Version 10.0.14393) OS with Office 2013 [15.0.4569.1503]. The previous day I've been prompted to restart my laptop to complete the new updates. I've left in a hurry and just closed the lid. The laptop didn't close and I had to use the power button to stop it. I knew I was in trouble at this point, but had to attend a meeting and didn't have time to wait for the updates.

The following morning Microsoft Outlook greeted me with the message in the picture. And so did Skype for Business. 

```
Faulting application name: OUTLOOK.EXE, version: 15.0.5023.1000, time stamp: 0x5aa78315
Faulting module name: MSVCR100.dll, version: 10.0.40219.325, time stamp: 0x4df2be1e
Exception code: 0xc0000005
Fault offset: 0x000173a5
Faulting process id: 0x2f94
Faulting application start time: 0x01d3ece4824e69d2
Faulting application path: C:\Program Files (x86)\Microsoft Office\Office15\OUTLOOK.EXE
Faulting module path: C:\windows\SYSTEM32\MSVCR100.dll
Report Id: 2ed214cc-58d8-11e8-822d-b4b676a83afd
Faulting package full name: 
Faulting package-relative application ID: 
```
```
Faulting application name: lync.exe, version: 15.0.5023.1000, time stamp: 0x5aa7802d
Faulting module name: MSVCR100.dll, version: 10.0.40219.325, time stamp: 0x4df2be1e
Exception code: 0xc0000005
Fault offset: 0x000173a5
Faulting process id: 0x1c78
Faulting application start time: 0x01d3ece617a52df9
Faulting application path: C:\Program Files (x86)\Microsoft Office\Office15\lync.exe
Faulting module path: C:\windows\SYSTEM32\MSVCR100.dll
Report Id: 9a24552e-58d9-11e8-822e-b4b676a83afd
Faulting package full name: 
Faulting package-relative application ID: 
```

I've tried everything I could think of (check for updates on the system to continue what was left; remove the updates one by one; update again; do a system restore with a day before; do another system restore with 5 days earlier; reinstalled the office suite. **NOTHING**. Same error.

After a system restore, the faulty module changed, but the situation was the same. Event viewer displayed the following errors:
```
Faulting application name: OUTLOOK.EXE, version: 15.0.4569.1503, time stamp: 0x52b0b282
Faulting module name: KERNELBASE.dll, version: 10.0.14393.2189, time stamp: 0x5abda7d6
Exception code: 0x80000003
Fault offset: 0x00154922
Faulting process id: 0x11ec
Faulting application start time: 0x01d3ed163b7e5cfc
Faulting application path: C:\PROGRA~2\MICROS~1\Office15\OUTLOOK.EXE
Faulting module path: C:\windows\System32\KERNELBASE.dll
Report Id: b3cacfd0-ea82-41d0-83b2-1955e51d6a53
Faulting package full name: 
Faulting package-relative application ID: 
```

I've also ran a system scan to no avail.
```
C:\windows\system32>sfc /scannow

Beginning system scan.  This process will take some time.

Beginning verification phase of system scan.
Verification 100% complete.

Windows Resource Protection found corrupt files and successfully repaired
them. Details are included in the CBS.Log windir\Logs\CBS\CBS.log. For
example C:\Windows\Logs\CBS\CBS.log. Note that logging is currently not
supported in offline servicing scenarios.
```

Microsoft Community forum didn't help much. But I've learned that you can start Outlook in safe mode (`outlook /safe`). And even though it didn't connect to the server in the beginning, it did ran without the error. During one of the reboots of Outlook I've been prompted to create a new account (in safe mode), and when I recreated my profile, BANG, it worked! Now the problem that I had was that Lync was still not working. Same error message when starting. 

## THE FIX

The fix was to simply rename/delete the .ost file for my profile (you can find it in C:\Users\/<you>\AppData\Local\Microsoft\Outlook\/<youremailaddress@domain>.ost

That was it. I hope this helps in case you run into this.


