---
date: "2017-12-05T00:00:00Z"
title: Udemy offline
---

Here's another cool and helpful thing I just discovered that might help you stranger of the internet.

Recently I joined a class on Udemy.com, and because I have other tasks at the moment and can't dedicate 100% to it, I wanted to download the videos so I started looking for ways how to do it.
The [official answer](https://support.udemy.com/hc/en-us/articles/229231167-Can-I-Download-a-Course-to-my-Computer-) is that they don't have anything against it, but it depends on the instructors. In my case the option to download the video was not present, plus that I wanted to automate the process and not download the videos one by one.

So, first I read about garbage solutions, like installing bluestacks to emulate an Android environment and then install the Udemy app, from where you can enable offline videos that will save them locally. This solution required too many steps and the end result would have been a bunch of files in different folders with a generic filename like video.mp4; again, not optimal.

So the winning solution comes from [youtube-dl](https://github.com/rg3/youtube-dl). With it you can download videos from [many sources](http://rg3.github.io/youtube-dl/supportedsites.html), including Udemy, of course. 
Here's the link on [gist](https://gist.github.com/barbietunnie/8531d9c26cd1c0668e7278c7c4ba5853) on how you should use it.
Basically, you just need to provide it the url + your USER and PASS plus the CLASS NAME. 

```
youtube-dl -u USER -p PASS -o "./%(playlist)s/%(chapter_number)s-%(chapter)s/%(autonumber)03d-%(title)s.%(ext)s" https://www.udemy.com/CLASS-NAME/learn/v4/content
```

After it finishes you'll find a new folder in your current directory, called "CLASS-NAME" and a new directory for each section. Happy studying! :) 

##### Disclaimer
I'm not supporting piracy. This method is intended only for personal use. And it works only for the courses that you already paid. It will not work if you don't have an Udemy account and access to the course.

