---
date: "2017-08-07T00:00:00Z"
title: Password recovery procedure on Cisco Catalyst 3500-XL
---


Just got a used Catalyst 3500-XL switch and don't have credentials? Did you forget your auth credentials or you just want to have fun?

##### Accessing a Cisco switch when you don't have the right credentials is hard but not impossible. 
*If you have issues connecting the console cable, try to install the [drivers](https://software.cisco.com/portal/psn/download/login.html?login=true&referer=https://software.cisco.com/download/release.html?mdfid=282979369&softwareid=282855122&release=3.1&dwnld=true&dwldImageGuid=1BEFCB8F748762B8CC378ABA82D6E48B040A80BF&atcFlag=N) first.* 

This will work only if you have access to the box itself. If so, you'll need to connect a rollover cable and follow the instructions:

Now you have to keep the Mode button pressed, while you plug the power cord to start the device.
You keep the button pressed till Port 1X light will be turned off.
After you release it, your prompt on the console connection will become `switch:`
Here you have to initialize the flash image, by using the command: `switch flash_init`

You can browse the `flash:` and check for `startup-config`, then rename it to something else `switch: rename flash:/config.text flash:/config.text.old`. 

After that you'll be able to boot the switch with `boot` command. The IOS won't be able to find the config file and it will default to Setup wizard. Just refuse it politely by typing `no` and when the prompt will become `Switch>`, type `enable`, restore the config file: `Switch#rename flash:/config.text.old flash:/config.text`

Now you have to load the original config `Switch#copy flash:/config.text flash:/running-config` and enter into `configure terminal` to change the tricky password.

That's all! Don't forget to check config register and try to remember the password  this time :)

~~~~~~~~~~~~~~
switch: rename flash:/config.text flash:/config.text.old
switch: dir flash:
Directory of flash:/

2    -rwx  1751867   <date>               c3500XL-c3h2s-mz.120-5.WC3b.bin
3    -rwx  94370     <date>               c3500XL-diag-mz-120-5.WC2a
4    -rwx  347       <date>               env_vars
5    -rwx  109       <date>               info
6    -rwx  25        <date>               snmpengineid
7    drwx  640       <date>               html
18   -rwx  109       <date>               info.ver
19   -rwx  6218      <date>               config.text.old
21   -rwx  616       <date>               vlan.dat

349696 bytes available (3262976 bytes used)
switch: boot
Loading "flash:c3500XL-c3h2s-mz.120-5.WC3b.bin"...############################################

File "flash:c3500XL-c3h2s-mz.120-5.WC3b.bin" uncompressed and installed, entry poin
executing...

              Restricted Rights Legend

Use, duplication, or disclosure by the Government is
subject to restrictions as set forth in subparagraph
(c) of the Commercial Computer Software - Restricted
Rights clause at FAR sec. 52.227-19 and subparagraph
(c) (1) (ii) of the Rights in Technical Data and Computer
Software clause at DFARS sec. 252.227-7013.

           cisco Systems, Inc.
           170 West Tasman Drive
           San Jose, California 95134-1706

Cisco Internetwork Operating System Software
IOS (tm) C3500XL Software (C3500XL-C3H2S-M), Version 12.0(5)WC3b, RELEASE SOFTWARE
Copyright (c) 1986-2002 by cisco Systems, Inc.
Compiled Fri 15-Feb-02 10:51 by antonino
Image text-base: 0x00003000, data-base: 0x00337600


Initializing C3500XL flash...
flashfs[1]: 19 files, 2 directories
flashfs[1]: 0 orphaned files, 0 orphaned directories
flashfs[1]: Total bytes: 3612672
flashfs[1]: Bytes used: 3262976
flashfs[1]: Bytes available: 349696
flashfs[1]: flashfs fsck took 4 seconds.
flashfs[1]: Initialization complete.
...done Initializing C3500XL flash.
C3500XL POST: System Board Test: Passed
C3500XL POST: Daughter Card Test: Passed
C3500XL POST: CPU Buffer Test: Passed
C3500XL POST: CPU Notify RAM Test: Passed
C3500XL POST: CPU Interface Test: Passed
C3500XL POST: Testing Switch Core: Passed
C3500XL POST: Testing Buffer Table: Passed
C3500XL POST: Data Buffer Test: Passed
C3500XL POST: Configuring Switch Parameters: Passed
C3500XL POST: Ethernet Controller Test: Passed
C3500XL POST: MII Test: Passed
cisco WS-C3524-PWR-XL (PowerPC403) processor (revision 0x01) with 8192K/1024K bytes
Processor board ID CHK0624W0BZ, with hardware revision 0x00
Last reset from power-on

Processor is running Enterprise Edition Software
Cluster command switch capable
Cluster member switch capable
24 FastEthernet/IEEE 802.3 interface(s)
2 Gigabit Ethernet/IEEE 802.3 interface(s)

32K bytes of flash-simulated non-volatile configuration memory.
...
C3500XL INIT: Complete

00:00:20: %SYS-5-RESTART: System restarted --
Cisco Internetwork Operating System Software
IOS (tm) C3500XL Software (C3500XL-C3H2S-M), Version 12.0(5)WC3b, RELEASE SOFTWARE
Copyright (c) 1986-2002 by cisco Systems, Inc.
Compiled Fri 15-Feb-02 10:51 by antonino

         --- System Configuration Dialog ---

At any point you may enter a question mark '?' for help.
Use ctrl-c to abort configuration dialog at any prompt.
Default settings are in square brackets '[]'.

Continue with configuration dialog? [yes/no]: no
Press RETURN to get started.


Switch>ena
Switch#
Switch#rename flash:config.text.old flash:config.text
Destination filename [config.text]?
Switch#copy flash:config.text system:/running-config
Destination filename [running-config]?
6218 bytes copied in 2.753 secs (3109 bytes/sec)
ORIGINAL-NAME#conf t
ORIGINAL-NAME(conf)# enable secret ANOTHERPASSWORD
~~~~~~~~~~~~~~
