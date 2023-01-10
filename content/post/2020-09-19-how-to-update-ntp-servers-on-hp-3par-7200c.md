---
date: "2020-09-19T00:00:00Z"
title: How to change NTP server on HP 3PAR 7200c
---

Recently I had to add more NTP servers to a HP 3PAR 7200c storage. Here's how I did it:
```
3PAR_hostname cli% setnet ntp -add 10.90.221.13
NTP server successfully updated.
```

Show current NTP server(s):
```
3PAR_hostname cli% shownet
IP Address  Netmask/PrefixLen Nodes Active Speed Duplex AutoNeg Status
10.23.155.8   255.255.255.192    01      1  1000 Full   Yes     Active

Default route :                                     10.23.155.1
NTP server    :   {} 10.102.102.132 10.102.102.133 10.90.221.13
DNS server    :                                            None
```

Remove one NTP server:
```
3PAR_hostname cli% setnet ntp -remove 10.90.221.13
NTP server successfully updated.

3PAR_hostname cli% shownet
IP Address  Netmask/PrefixLen Nodes Active Speed Duplex AutoNeg Status
10.23.155.8   255.255.255.192    01      1  1000 Full   Yes     Active

Default route :                        10.23.155.1
NTP server    :   {} 10.102.102.132 10.102.102.133
DNS server    :                               None
3PAR_hostname cli%
```

You can't add more than 1 server at a time:
```
3PAR_hostname cli% setnet ntp -add 10.90.221.13 10.90.221.29 10.90.221.45
setnet: Incorrect number of arguments.
```

And the maximum ammount of severs is 4, in case you were asking:
```
3PAR_hostname cli% setnet ntp -add 10.90.221.61
Cannot configure more than 4 NTP servers.
3PAR_hostname cli% setnet ntp -remove 10.102.102.132
NTP server successfully updated.
```

That's it! That simple. More info can be found in the original [documentation](https://support.hpe.com/hpesc/public/docDisplay?docId=c03583615&docLocale=en_US)
