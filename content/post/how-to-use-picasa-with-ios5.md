---
title: How to use Picasa with iOS5
date: 2011-12-11
author: Jason
category: Apple
tags: Apple, import photos, ios5, ipad, ipad 2, iphone, iphone 4s, iphone4s, iphoto, ipod touch, picasa, solved
---

I'm mostly writing this post because I had a fairly hard time finding a resolution to a real pesky error. For some reason, my [iPhone 4S][] was recognized by [Picasa][] but always failed to import photos. Whenever I tried, the Picasa was clearly scanning through the files and then presented this error:

> An error has occurred while attempting to import. Either the source is
> unavailable or the destination is full or read only (1).

The resolution was found on [this page][] posted by [Tradeinstyle][]. A slightly more thorough explanation of the solution below.

If you are seeing this error, what appears to have happened is that several images are "corrupted" in some way on your iPhone. Unfortunately, this requires opening up [iPhoto][]. Once in iPhoto, you should be at the import screen and see all the pictures available on your phone. Several of these pictures will have a thumbnail consisting only of a dotted-line forming a square-- a blank thumbnail. You'll want to import these photos and, after clicking import, be sure to select the option that removes them from your iPhone. Now move these imported photos from your newly create iPhoto library into your normal Pictures folder (or wherever you're watching for pictures in Picasa). They'll load just fine. Exit iPhoto, delete your iPhoto Library (probably located in \~/Pictures/iPhoto\\ Library) to avoid duplicates and open Picasa. Because the "offending" pictures have now been removed, Picasa should be able to easily import your photos.

This problem does not appear to be specific to the iPhone 4s and is probably applicable to [all][] [iOS5][] [devices][].

[iPhone 4S]: http://www.apple.com/iphone/
[Picasa]: http://picasa.google.com/
[this page]: http://www.google.com/support/forum/p/Picasa/thread?tid=36adf4b3d0209e55&hl=en
[Tradeinstyle]: http://www.google.com/support/forum/p/Picasa/user?userid=14055753863597455136&hl=en
[iPhoto]: http://www.apple.com/ilife/iphoto/
[all]: http://www.apple.com/ipodtouch/
[iOS5]: http://www.apple.com/ios/
[devices]: http://www.apple.com/ipad/
