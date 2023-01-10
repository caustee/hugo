---
date: "2017-04-06T00:00:00Z"
title: New website - powered by Jekyll
---

I've recently moved my website from [AnchorCMS](http://anchorcms.com) to [Jekyll](http://jekyllrb.com). I didn't think it was necessary to have a WordPress solution only for a personal blog that has less than 10 visits/month so I went for a static website that is not tied to mysql and php. With this change I've also switched from Apache2 to Nginx. I hope that with this change my tiny VPS is not going to need much resources in the future.
I still have to tweak the CSS settings, so expect more changes soon enough.

I took the head logo from [Jonathan Robson](http://jnrbsn.com/) and the theme from [John Otander](http://johnotander.com/). So I'd like to thank him with this occasion.  

## How I did it

First thing I did was to google it :) I found a couple of good tutorials, one of them is this one [from ines](https://ines.io/blog/the-ultimate-guide-static-websites-jekyll) which is comprehensive enough to get a good understanding about Jekyll. 

Another good resource is this youtube video by [Martino Jones](https://www.youtube.com/watch?v=HOg8jWZ3lt0).

I highly recommend to go through them both before installing it on your machine.


Ok, so you've got the theory now, let's get ready to do this!

## Installing

### Dependencies:

Here is the list from the official [documentation](https://jekyllrb.com/docs/installation/#requirements)

> * GNU/Linux, Unix, or macOS
> * Ruby version 2.0 or above, including all development headers
> * RubyGems
> * GCC and Make (in case your system doesn’t have them installed, which you can check by running gcc -v and make -v in your system’s command line interface)

Here it really depends on what OS you're using. I've installed it on an Ubuntu so the following instructions will be for Ubuntu. 

```
$ sudo gem install jekyll
```

## Starting your blog

```
$ jekyll build
```

```
$ jekyll serve --host 0.0.0.0
```
You will want to use --host 0.0.0.0 in order to have the server running on all IP addresses that are assigned to your server.


## Customization 
Next thing after you install Jekyll is to search for a theme that fits you. There are many theme sites out there, but I found this [one](http://drjekyllthemes.github.io/) that had enough blog themes. 

You can also customize it by going under the hood, down in the CSS area. But unfortunately I'm unable to understand CSS yet, so I won't talk about it :)

Go ahead and try it! But first measure the resources used by your current setup to see if it makes a different to you.

Edit: I've removed the YouTube iframe because it added 500KB to the webpage unnecessarily, now this page has less than 700KB. I got inspired by [this article](http://idlewords.com/talks/website_obesity.htm) and I'm trying to help.
