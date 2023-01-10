---
date: "2017-05-04T00:00:00Z"
title: How to configure Cisco devices remotely
---

So, let's assume you are a network engineer that has to configure something on a new Cisco switch/router. You don't have console access and just connect to the box via ssh. You either have a typo in your commands or don't copy the whole line of your pre-tested commands and after you press enter BANG, you've lost connection and start sweating. You know the feeling, right? Me neither...

The classic solution to this is to schedule a reload in something like 10 minutes: `reload in 10`

This works great, but the problem is that you have to wait for the box to come back alive (and users will be disconnected during the reload).
Another method is to revert the config with no reload. For this you have to first configure a local copy of config on the disk:

```
Router#mkdir Backup
Create directory filename [Backup]?
Created dir bootflash:/Backup
Router#conf t
Router(config)#archive
Router(config-archive)#path bootflash:/Backup/backup
Router(config-archive)#write-memory
```

And then go to config mode like this:

```
Router#configure terminal revert timer 2
Rollback Confirmed Change: Backing up current running config to bootflash:/Backup/backup-May--4-13-16-19.360-0

Enter configuration commands, one per line.  End with CNTL/Z.
```

And then in 2 minutes you'll have to confirm the change with `config confirm`. If not, it will revert to the backup it just created (bootflash:/Backup/backup-May--4-13-16-19.360-0 in my case)

That's it! Simple as that, you can even create an alias for this:

```Router(config)#alias exec cfg conf term revert timer 2```
```
Router#cfg

*May  4 14:05:33.658: %SYS-5-CONFIG_I: Configured from console by consoleRollback Confirmed Change: Backing up current running config to bootflash:/Backup/backup-May--4-14-05-34.882-1

Enter configuration commands, one per line.  End with CNTL/Z.
Router(config)#
*May  4 14:05:35.179: %ARCHIVE_DIFF-5-ROLLBK_CNFMD_CHG_BACKUP: Backing up current running config to bootflash:/Backup/backup-May--4-14-05-34.882-1
*May  4 14:05:35.179: %ARCHIVE_DIFF-5-ROLLBK_CNFMD_CHG_START_ABSTIMER: User: console(Priv: 15, View: 0): Scheduled to rollback to config bootflash:/Backup/backup-May--4-14-05-34.882-1 in 2 minutes

```


