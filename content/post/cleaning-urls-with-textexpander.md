---
title: Cleaning URLs with TextExpander
author: Jason P. Becker
email: jason.paul.becker@gmail.com
date: 2013-05-30
tags: textexpander, code, snippets
---

One thing I really dislike about Google Reader is it replaces the links to posts in my RSS feed. My [Pinboard account][] is littered with links that start with `http://feedproxy.google.com`. I am quite concerned that with the demise of [Google Reader][] on July 1, 2013, these redirects will no longer work.

It's not just Google that obscures the actually address of links on the internet. The popularity of using link shortening services, both to save characters on Twitter and to collect analytics, has proliferated the *Internet of Redirects*.

Worse still, after I am done cutting through redirects, I often find that the ultimate link include all kinds of extraneous attributes, most especially a barrage of `utm_*` campaign tracking.

Now, I understand why all of this is happening and the importance of the services and analytics this link cruft provides. I am quite happy to click on shortened links, move through all the redirects, and let sites know just how I found them. But quite often, like when using a bookmarking service or writing a blog post, I just want the simple, plain text URL that gets me directly to the permanent home of the content.

One part of my workflow to deal with link cruft is a TextExpander snippet I call `cleanURL`. It triggers a simple Python script that grabs the URL in my clipboard, traces through the redirects to the final destination, then strips links of campaign tracking attributes, and ultimately pastes a new URL that is much "cleaner".

Below I have provided the script. I hope it is useful to some other folks, and I would love some recommendations for additional "cleaning" that could be performed.

My next task is expanding this script to work with [Pinboard][] so that I can clean up all my links before the end of the month when Google Reader goes belly up.

    :::python
    #!/usr/bin/python
    import requests
    import sys
    from re import search
    from subprocess import check_output
    
    url = check_output('pbpaste')
    
    # Go through the redirects to get the destination URL
    r = requests.get(url)
    
    # Look for utm attributes
    match =  search(r'[?&#]utm_', r.url)
    
    # Because I'm not smart and trigger this with
    # already clean URLs
    if match:
      cleanURL = r.url.split(match.group())[0]
    else:
      cleanURL = r.url
    
    print cleanURL


[Google Reader]: http://googlereader.blogspot.com/2013/03/powering-down-google-reader.html
[Pinboard account]: http://pinboard.in/u:jasonpbecker
[Pinboard]: http://pinboard.in