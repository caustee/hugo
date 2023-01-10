---
date: "2017-07-07T00:00:00Z"
title: Cisco guest shell
---

So, it seems that finally Cisco decided to introduce a shell for their routers. I know this already exists for their FirePower plaform, but from my knowledge, they just made it available for routers.

### Enabling the shell

To enable the feature we simply have to enable `iox` and then we can enter linux land with `guestshell`
```
CSR01(config)#iox
CSR01(config)#do guestshell
[guestshell@guestshell ~]$
```
####Dohost command
`dohost` permits you to run IOS commands from your shell terminal.

```
[guestshell@guestshell ~]$ for x in {1..5}; do dohost "conf t ; interface l$x ; ip address 10.0.0.$x 255.255.255.255" ; done
[guestshell@guestshell ~]$
*Jul  4 22:32:39.252: %LINEPROTO-5-UPDOWN: Line protocol on Interface Loopback1, changed state to up
*Jul  4 22:32:39.253: %LINK-3-UPDOWN: Interface Loopback1, changed state to up
*Jul  4 22:32:39.332: %LINEPROTO-5-UPDOWN: Line protocol on Interface Loopback2, changed state to up
*Jul  4 22:32:39.332: %LINK-3-UPDOWN: Interface Loopback2, changed state to up
*Jul  4 22:32:39.415: %LINEPROTO-5-UPDOWN: Line protocol on Interface Loopback3, changed state to up
*Jul  4 22:32:39.415: %LINK-3-UPDOWN: Interface Loopback3, changed state to up
*Jul  4 22:32:39.496: %LINEPROTO-5-UPDOWN: Line protocol on Interface Loopback4, changed state to up
*Jul  4 22:32:39.496: %LINK-3-UPDOWN: Interface Loopback4, changed state to up
*Jul  4 22:32:39.566: %LINEPROTO-5-UPDOWN: Line protocol on Interface Loopback5, changed state to up
*Jul  4 22:32:39.567: %LINK-3-UPDOWN: Interface Loopback5, changed state to up
[guestshell@guestshell ~]$ dohost 'show ip route | b Loop'
C        10.0.0.1/32 is directly connected, Loopback1
C        10.0.0.2/32 is directly connected, Loopback2
C        10.0.0.3/32 is directly connected, Loopback3
C        10.0.0.4/32 is directly connected, Loopback4
C        10.0.0.5/32 is directly connected, Loopback5
C        10.0.0.6/32 is directly connected, Loopback6
C        10.0.0.7/32 is directly connected, Loopback7
```

I find this very cool and mostly needed. I know Juniper has had it since forever, but still is nice to see that Cisco decided to make a step forward.
You can find out more about this in their [paper](https://www.cisco.com/c/en/us/td/docs/switches/datacenter/nexus9000/sw/7-x/programmability/guide/b_Cisco_Nexus_9000_Series_NX-OS_Programmability_Guide_7x/Guest_Shell.pdf) or on [reddit](https://www.reddit.com/r/networking/comments/6lm34k/cisco_is_coming_out_of_its_shell/).

