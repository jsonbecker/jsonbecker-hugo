---
title: Contributing to vegalite
author: Jason P. Becker
date: 2017-02-25  
tags: 
draft: true
---

One of my goals for 2017 is to contribute more to the R open source community. At the beginning of last year, I spent a little time helping to refactor [rio](https://www.github.com/leeper/rio). It was one of the more rewarding things I did in all of 2016. It wasn't a ton of work, it taught just that much more about writing R packages and using S3 methods, and work I did is being downloaded thousands of times a month.

I have been on the lookout for a Javascript powered interactive charting library since `ggvis` was announced in 2014. But `ggvis` seems to have stalled out in favor of other projects (for now) and the evolution of `rCharts` into `htmlwidgets` left me feeling like there were far too many options and no clear choices.

What I was looking for was a plotting library to make clean, attractive graphics with tool tips that came with clear documentation and virtually no knowledge of Javascript required. Frankly, all of the `htmlwidgets` stuff was very intimidating. From my vantage point skimmingblog posts and watching stuff come by on Twitter, `htmlwidgets`-based projects all felt very much directed at Javascript polyglots, and none of the packages felt like they were taken seriously as opposed to used as demonstration projects.

At the same time, I had been watching Vega and Vega-lite from afar. This project had a lot of the qualities I sought in a plotting library, because reading and writing JSON is very accessible compared to learning Javascript, especially with R's excellent translation from lists to JSON.
