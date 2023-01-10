---
date: "2017-08-07T00:00:00Z"
title: How to create text files on Cisco routers
---

In case you will need it, here's a thing that helped me out once. In case you want to append some config lines that you have stored in a file and if you can't copy it through SFTP, you can make it locally.
To do this you need to start tclsh and create the file `puts [open "flash:filename" w+] { ` now don't press enter but just paste in your config lines and then `}` and ENTER. `tclquit` to exit tclsh.
Now you'll have your config snippet in flash:filename and you can use it to append to your config. Would be a good idea to check it with `more flash:filename` just to be sure that you have everything in place and no characters were left outside. 
As always, more info on [Cisco's webpage](https://www.cisco.com/c/en/us/support/docs/ip/telnet/116214-technote-technology-00.html)
