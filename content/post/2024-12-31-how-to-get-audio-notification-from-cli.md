---
date: "2024-12-31T00:00:00Z"
title: How to get audio notification from CLI when server or service is reachable
---

If you're working on fixing an issue on a server and want to be notified as soon as this is reachable, you can run an audio alert. Below I have a list of variations of the [ Bell Character](https://en.wikipedia.org/wiki/Bell_character#Usage).
* Windows / PowerShell:
```
 while ($true) {if (Test-Connection -quiet 1.1.1.1) { [console]::beep() }}
```
* Windows / cmd:
```
for /L %n in () do @ping -n 1 1.1.1.1 && echo ^G
```
* WSL2 / Bash:
```
while true; do sudo ping -q 1.1.1.1 -w1 >/dev/null && tput bel; sleep 5; done
```
*on WSL ping needs eleventation so sudo is mandatory.*
* Linux / Bash:
```
while true; do ping -n1 1.1.1.1 && tput bel; done
```
* Mac / zsh:
```
ping -A 1.1.1.1
```
*side note here, it worked only on iterm, not on Alacritty. So that's the reason I would rather not use the builtin audible option. After playing with this for this blog post I've found a [way](https://github.com/alacritty/alacritty/issues/1528#issuecomment-979722149) on how to configure Alacritty to send the bell sound.*
```
while true; do ping -n1 1.1.1.1 && osascript -e 'beep'; done
```


These command will try to reach the IP address 1.1.1.1 and when it will be available will send a beep signal. This, of course, works if you have system beep enabled.

The same options could be used if you want to be notified when the service is available too, not just the server.

* Windows:
```
while ($true) {if (TNC -ComputerName $IP -Port $PORT) {[console]::beep(1000,500)}}
```
* Linux/Mac:
```
while true; do nc -zv $IP $PORT && echo -e '\a'; sleep 5; done
```



As you can see, I'm a big fan of oneliners. Even though a proper function can be created for this usecase, in 99% of time I prefer the quick and dirty version.
