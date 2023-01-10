---
date: "2020-02-16T00:00:00Z"
title: How to fix do-agent exited with a return code 100
---


So recently I've encountered the error in the subject. This came after 2 major changes (OS upgrade and PHP upgrade). So definitely it had to do with one of them. My money were on the first one.
I've looked around on the internet and the only thing I could see was this old article that had no solution: https://www.digitalocean.com/community/questions/do-agent-exited-with-a-return-code-100

So I knew I had to take the matters in my own hands. I've started by looking at that cron entry that was failing.
```run-parts: /etc/cron.daily/do-agent exited with return code 100```

The cron itself was pretty simple, just a bash script.
```
root@server:/etc/logrotate.d# cat /etc/cron.daily/do-agent
#!/bin/sh
/bin/bash /opt/digitalocean/do-agent/scripts/update.sh >/dev/null 2>&1
```

The `update.sh` script is also very simple:
```
root@server:/etc/logrotate.d# cat /opt/digitalocean/do-agent/scripts/update.sh
#!/bin/bash
# vim: noexpandtab

main() {
        # add some jitter to prevent overloading the packaging machines
        sleep $(( RANDOM % 900 ))

        export DEBIAN_FRONTEND="noninteractive"
        if command -v apt-get 2&>/dev/null; then
                apt-get -qq update -o Dir::Etc::SourceParts=/dev/null -o APT::Get::List-Cleanup=no -o Dir::Etc::SourceList="sources.list.d/digitalocean-agent.list"
                apt-get -o Dpkg::Options::="--force-confdef" -o Dpkg::Options::="--force-confold" -qq install -y --only-upgrade do-agent
        elif command -v yum 2&>/dev/null; then
                yum -q -y --disablerepo="*" --enablerepo="digitalocean-agent" makecache
                yum -q -y update do-agent
        fi
}

main
root@server:/etc/logrotate.d#
```

I've ran the command manually and after a long (timeout?) period it exited with status code 100. So just like in my emails.

I went further on with my troubleshooting and added debug to bash and removed the redirect to /dev/null part. And this time I've noticed that there was no timeout, but just a random sleep that's at the beginning of update.sh

```
root@server:/etc/apt/sources.list.d# time /bin/bash -x /opt/digitalocean/do-agent/scripts/update.sh
+ main
+ sleep 650
```

If you don't want to wait 650 seconds then you can stop and start it again, hoping to get a shorter number, or just comment that part in the script. But I didn't want to alter the script, just to fix it. So I chose to stop and re-start it till I get a shorter sleep timer.

```
root@server:/etc/apt/sources.list.d# time /bin/bash -x /opt/digitalocean/do-agent/scripts/update.sh
+ main
+ sleep 812
^C

real    0m1.218s
user    0m0.004s
sys     0m0.000s
```
```
root@server:/etc/apt/sources.list.d# time /bin/bash -x /opt/digitalocean/do-agent/scripts/update.sh
+ main
+ sleep 13
+ export DEBIAN_FRONTEND=noninteractive
+ DEBIAN_FRONTEND=noninteractive
+ command -v apt-get 2
+ apt-get -qq update -o Dir::Etc::SourceParts=/dev/null -o APT::Get::List-Cleanup=no -o Dir::Etc::SourceList=sources.list.d/digitalocean-agent.list
+ apt-get -o Dpkg::Options::=--force-confdef -o Dpkg::Options::=--force-confold -qq install -y --only-upgrade do-agent
Setting up apache2 (2.4.29-1ubuntu4.11) ...
ERROR: Module reqtimeout does not exist!
dpkg: error processing package apache2 (--configure):
 installed apache2 package post-installation script subprocess returned error exit status 1
Errors were encountered while processing:
 apache2
E: Sub-process /usr/bin/dpkg returned an error code (1)

real    0m22.398s
user    0m8.120s
sys     0m0.876s
root@server:/etc/apt/sources.list.d# echo $?
100
```

Now I have it :) It seems that the update script fails due to some apache2 module missing. And this is normal, because I don't have apache on my server.

So to fix it I just had to remove all apache-related packages from the machine. Which were installed by the latest PHP upgrade. So it wasn't due to OS upgrade like I first suspected :) 

```
root@server:~# apt-get purge apache2*
Reading package lists... Done
Building dependency tree
Reading state information... Done
Note, selecting 'apache2-ssl-dev' for glob 'apache2*'
Note, selecting 'apache2.2-common' for glob 'apache2*'
Note, selecting 'apache2-suexec-pristine' for glob 'apache2*'
Note, selecting 'apache2-api-20120211' for glob 'apache2*'
Note, selecting 'apache2-data' for glob 'apache2*'
Note, selecting 'apache2-api-20120211-openssl1.1' for glob 'apache2*'
Note, selecting 'apache2-suexec' for glob 'apache2*'
Note, selecting 'apache2-bin' for glob 'apache2*'
Note, selecting 'apache2.2-bin' for glob 'apache2*'
Note, selecting 'apache2-dbg' for glob 'apache2*'
Note, selecting 'apache2-dev' for glob 'apache2*'
Note, selecting 'apache2-doc' for glob 'apache2*'
Note, selecting 'apache2-suexec-custom' for glob 'apache2*'
Note, selecting 'apache2' for glob 'apache2*'
Note, selecting 'apache2-utils' for glob 'apache2*'
Note, selecting 'apache2-mpm-itk' for glob 'apache2*'
Package 'apache2.2-bin' is not installed, so not removed
Package 'apache2.2-common' is not installed, so not removed
Note, selecting 'apache2-bin' instead of 'apache2-api-20120211'
Note, selecting 'apache2-bin' instead of 'apache2-api-20120211-openssl1.1'
Package 'apache2-mpm-itk' is not installed, so not removed
Package 'apache2-dbg' is not installed, so not removed
Package 'apache2-dev' is not installed, so not removed
Package 'apache2-doc' is not installed, so not removed
Package 'apache2-ssl-dev' is not installed, so not removed
Package 'apache2-suexec-custom' is not installed, so not removed
Package 'apache2-suexec-pristine' is not installed, so not removed
```

This time the script ended successfuly with status code 0.

```
root@server:~# time /bin/bash -x /opt/digitalocean/do-agent/scripts/update.sh
+ main
+ sleep 61
+ export DEBIAN_FRONTEND=noninteractive
+ DEBIAN_FRONTEND=noninteractive
+ command -v apt-get 2
+ apt-get -qq update -o Dir::Etc::SourceParts=/dev/null -o APT::Get::List-Cleanup=no -o Dir::Etc::SourceList=sources.list.d/digitalocean-agent.list
+ apt-get -o Dpkg::Options::=--force-confdef -o Dpkg::Options::=--force-confold -qq install -y --only-upgrade do-agent

real    1m6.107s
user    0m4.184s
sys     0m0.552s
root@server:~# echo $?
0
```

Your error might be related to something else, but anyhow, in case you run into this sort of issues I hope this helps you speed-up the troubleshooting phase.
