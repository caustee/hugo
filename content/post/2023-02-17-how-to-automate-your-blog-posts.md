---
date: "2023-02-17T00:00:00Z"
title: How to automate your blog posts
---

Since I've transformed this website in a Blogging as a Service (BaaS) thing with the help of Github Actions and Github Pages my life was much easier.

Let me help you do the same.

First thing you'll need is a domain. For this domain you will need a NS, and for this you can use CloudFlare for free. They offer a great service, btw.
```
 ~/ dig in NS costinstefan.ro +short
nola.ns.cloudflare.com.
theo.ns.cloudflare.com.
```
In CF you should define your zone, and this zone should include at least an A record that will point to Github Pages (my case).
```
 ~/ dig costinstefan.ro +short
188.114.97.8
188.114.96.8
 ~/
```

Once you do this, you can start working on Github Pages. This will be out of scope for today's tutorial, but you can find great info on the [official page](https://docs.github.com/en/pages/quickstart).

Now let's get to the fun part. You should create a repo that will keep your files. I'm using Hugo to generate my static html site. Then, on this repo you should go to Actions, create a new Workflow, and chose Hugo. And that's it, now you should have a pipeline that gets triggered automatically each time you push a change in the `main` branch. Of course, all the default settings can be changed.

Here you can get the [Github Actions docs](https://docs.github.com/en/actions).

After you create the workflow/pipeline, you should clone the repo locally and start writing your blog posts using your favourit text editor. And after you push the changes to main, the new blog posts will be online :)
