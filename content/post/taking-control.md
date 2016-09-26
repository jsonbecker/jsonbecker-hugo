---
title: Taking Control
author: Jason P. Becker
date: 2015-01-11
tags: 
---

## Severing My Daemon 

When I was in high school, I piggy-backed on a friend's website to host a page for my band. We could post pictures, show locations and dates, lyrics, and pretend like we produced music people cared about. It was mostly a fun way for me to play with the web and something to show folks when I said I played guitar and sang in a band. One day, my friend canceled his hosting. He wasn't using his site for anything and he forgot that I had been using the site. I was 18, I never thought about backups, and I had long deleted all those pesky photos taking up space on my memory cards and small local hard drive.

Four years of photos from some of the best experiences of my life are gone. No one had copies. Everyone was using the site. In the decade since, no set of pictures has ever been as valuable as the ones I lost that day.

## Who controls the past...

As you can imagine, this loss has had a profound effect on how I think about both my data and the permanence of the internet. Today, I have a deep system of backups for any digital data I produce, and I am far more likely to err on keeping data than discarding it. Things still sometimes go missing. [^stage]

[^stage]: I recently found some rare music missing that I had to retrieve through some heroic efforts that included [Archive.org]() and stalking someone from an online forum that no longer exists (successfully).

Perhaps the more lasting impact is my desire to maintain some control over all of my data. I use [Fastmail](http://www.fastmail.fm/?STKI=11467093) for my email, even after over 10 years of GMail use. [^early] I like knowing that I am storing some of my most important data in a standard way that easily syncs locally and backs up. I like that I pay directly for such an important service so that all of the incentive for my email provider is around making email work better for me. I am the customer. I use [Bittorrent Sync](http://www.getsync.com) for a good chunk of my data. I want redundancy across multiple machines and syncing, but I don't want all of my work and all of my data to depend on being on a third party server like it is with Dropbox. [^db]. I also use a [Transporter](http://www.esn.fm/sponsors/) so that some of my files are stored on a local hard drive.

[^early]: I was a very early adopter of Gmail.

[^db]: I still use Dropbox. I'm not an animal. But I like having an alternative.

## Raison D'être

Why does this blog exist? I have played with [Tumblr](http://tumblr.jsonbecker.com) in the past and I like its social and discovery tools, but I do not like the idea of pouring my thoughts into someone else's service with no guarantee of easy or clean exit. I tried using Wordpress on a self-hosted blog for a while, but I took one look at the way my blog posts were being stored in the Wordpress database and kind of freaked out. All those convenient plugins and short codes were transforming the way my actual text was stored in hard to recover way. Plus, I didn't really understand how my data was stored well enough to be comfortable I had solid back ups. I don't want to lose my writing like I lost those pictures.

This blog exists, built on [Pelican](http://www.getpelican.com), because I needed to a place to write my thoughts in plain text that was as easy to back up as it was to share with the world. I don't write often, and I feel I rarely write the "best" of my thoughts, but if I am going to take the time to put something out in the world I want to be damn sure that I control it.

## Bag End

I recently began a journey that I thought was about simplifying tools. I began using `vim` a lot more for text editing, including writing prose like this post. But I quickly found that my grasping for new ways to do work was less about simplifying and more about better control. I want to be able to work well, with little interruption, on just about any computer. I don't want to use anything that's overly expensive or available only on one platform if I can avoid it. I want to strip away dependencies as much as possible. And while much of what I already use is free software, I didn't feel like I was in *control*.

For example, `git` has been an amazing change for how I do all my work since about 2011. Github is a major part of my daily work and has saved me a bunch of money by allowing me to host this site for free. But I started getting frustrated with limitations of not having an actual server and not really having access to the power and control that a real server provides. So I recently moved this site off of Github and on to a [Digital Ocean](https://www.digitalocean.com/?refcode=7377c1fcbe67) droplet. This is my first experiment with running a Linux VPS. Despite using desktop Linux for four years full time, I have never administered a server. It feels like a skill I should have and I really like the control.

## Quentin's Land

This whole blog is about having a place I control where I can write things. I am getting better at the control part, but I really need to work on the writing things part. 

Here's what I hope to do in the next few months. I am going to choose (or write) a new theme for the site that's responsive and has a bit more detail. I am probably going to write a little bit about the cool, simple things I learned about `nginx` and how moving to my own server is helping me run this page (and other experiments) with a lot more flexibility. I am also going to try and shift some of my writing from tweetstorms to short blog posts. If I am truly trying to control my writing, I need to do a better job of thinking out loud in this space versus treating them as disposable and packing them on to Twitter. I will also be sharing more code snippets and ideas and less thoughts on policy and local (Rhode Island) politics. The code/statistics/data stuff feels easier to write and has always gotten more views and comments.

That's the plan for 2015. Time to execute.
