---
date: "2020-04-29T00:00:00Z"
title: How to quickly enable a low-resource webserver
---

Have you ever wanted to just start a Webserver in a second, for a quick and dirty task but didn't want to install a full LAMP/LEMP stack?
This is possible with the help of a python module and here it is in action:


```
costin@ubuntu:~$ mkdir testare
costin@ubuntu:~$ cd testare/
costin@ubuntu:~/testare$ echo MERGE! > index.html
costin@ubuntu:~/testare$ pushd $PWD; python3 -m http.server; popd
~/testare ~/testare
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
192.168.1.126 - - [29/Apr/2020 14:16:10] "GET / HTTP/1.1" 200 -
192.168.1.126 - - [29/Apr/2020 14:16:10] code 404, message File not found
192.168.1.126 - - [29/Apr/2020 14:16:10] "GET /favicon.ico HTTP/1.1" 404 -
```
```
root@ubuntu:~# curl http://192.168.1.94:8000/
MERGE!
```

Oh and this is cross-platform, of course. If you have python installed on a Windows machine.

Not to mention that this is possible even from low-priviledge user accounts.

For more details about this module please see [the official documentation](https://docs.python.org/3/library/http.server.html).

2025 EDIT:
Here I'm adding the commands for doing the same in PowerShell. What I did here was to use ALL interfaces (through + sign).
```
$port = 8080
$listener = New-Object System.Net.HttpListener
$listener.Prefixes.Add("http://+:$port/")
$listener.Start()
```
