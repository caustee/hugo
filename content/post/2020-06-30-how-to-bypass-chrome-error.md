---
date: "2020-06-30T00:00:00Z"
title: How to bypass Chrome error "Your connection is not private"
---

If you're an IT professional that has to deal with old systems that have self-signed certificates you most probably had to add exceptions to your browser many times. Maybe even import those self-signed certificates locally.

Unfortunately when you're working on a customer's jump host this is not an option. And starting with more recent version of Chrome this is not possible anymore, you don't have the option to Add it as an exception.


## Luckily there's a fix
All you have to do is press on `Advanced` tab and type **thisisunsafe**

You don't have to type it in the URL field, just type it in the main window of Chrome. In some older versions of Chrome the safeword was **badidea**

This, of course, should be done only if you know what you're doing, but if you reached this website I'm almost sure you do :) 

Thanks @Răzvan for the tip.
