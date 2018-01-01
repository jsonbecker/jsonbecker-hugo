---
title: 'This Year in My R Code'
author: Jason Becker
date: '2017-12-31'
tags: [rstats]
---

Looking back on 2017, there were three major trends in my R code: the end of S4, directly writing to SQL database, and `purrr` everywhere.

## The End of S4

The first package I ever wrote extensively used S4 classes. I wanted to have the security of things like `setValidity`. I liked the idea of calling `new` as it felt more like class systems I was familiar with from that one semester of Java in college. S4 felt more grown up than S3, more like it was utilizing the advantages of object oriented programming, and less exotic than R6, which in 2014 felt riskier to build with and teach future employees. Using S4 was a mistake from day one and never led to any advantages in the code I wrote.

So this year, I rewrote that original package. It's internal (and a core function) at my job so I can't share too much, but this was a long time coming. Not only did I clean up a lot of code that was just plain bad (in the way all old code is), but I got rid of S4 in favor of S3 or more functional code wherever possible. Our test coverage is far more complete, the code is far easier to extend without duplication, and it looks far more idiomatic to the standard non-BioConductor R user.

What's the lesson learned here? From a technical perspective, it would be avoid premature optimization and, of course, that everyone can and wants to throw out old code they revist with greater knowledge and context. But I know those things. What drove me to making the wrong decision here was purely imposter syndrome. I was writing code that had to be run unattended on a regular basis as a part of a product in a new job. I didn't feel up to the task, so I felt working with a new, complex, scary part of R that promised some notion of "safety" would mean I really knew what I was doing. So my takeaway from walking away from S4 is this: start small, build what you know, have confidence you can solve problems one at a time, and trust yourself.

## Directly Writing SQL

I use SQL far more than R, but almost entirely as a consumer (e.g. `SELECT` only). I've almost always directly used SQL for my queries into _other people's data_, but rarely ventured into the world of `INSERT` or `UPDATE` directly, preferring to use interfaces like `dbWriteTable`. This gets back to imposter syndrome-- there's so little damage that can be done with a `SELECT` statement, but writing into databases I don't control means taking on risk and responsiblity.

This year I said fuck it-- there's a whole lot of work and complexity going on that's entirely related to me not wanting to write `INSERT INTO` a PostgreSQL has the amazing `ON CONFLICT...`-based "upserts" now. So I started to write a lot of queries, some of them pretty complex [^lateral]. R is a great wrapper language, and it's database story is getting even better with the new DBI, odbc, and RPostgres packages. Although it's native table writing support is a little weak, there's no problem at all just using `dbSendStatement` with complex queries. I've fallen into a pattern I really like of writing temporary tables (with `dplyr::copy_to` because it's clean in a pipeline) and then executing complex SQL with `dbSendStatement`. In the future, I might be inclined to make these database functions, but either way this change has been great. I feel more confident than ever working with databases and R (my two favorite places to be) and I have been able to simplify a whole lot of code that involved passing around text files (and boy do I hate the type inference and other madness that can happen with CSVs. Oy.).

[^lateral]: I let out quite the "fuck yea!" when I got that two-common-table-expression, two joins with one lateral join in an upsert query to work.

## purrr

This is the year that `purrr` not only clicked, but became my preferred way to write code. Where there was `apply`, now there was `purrr`. Everything started to look like a list. I'm still only scratching the surface here, but I love code like this:

```
locations %>%
  filter(code %in% enrollment$locations) %$%
  code %>%
  walk(function(x) render(input = 'schprofiles.Rmd',
                          html_document(theme = NULL,
                                        template = NULL,
                                        self_contained = FALSE,
                                        css = 'static/styles.css',
                                        lib_dir = 'cache/demo/output/static/',
                                        includes = includes('fonts.html')),
                          params = list(school_code = x),
                          output_file = paste0(x,'.html'),
                          output_dir = "cache/demo/output/"))
```

It's a simple way to run through all of the `locations` (a `data.frame` with columns `code` and `name`) and render an HTML-based profile of each school (defined by having student enrollment). `walk` is beautiful, and so is `purrr`. I mean, who does need to do `map(., mutate_if, is.numeric, as.character)` 10 times a day?

## 2018 R Goals

One thing that's bittersweet is that 2017 is probably the last year in a long time that writing code is the main thing my every day job is about. With increased responsibility and the growth of my employees, I find myself reviewing code a lot more than writing it, and sometimes not even that. With that in mind, I have a few goals for 2018 that I hope will keep the part of me that loves R engaged.

First, I want to start writing command line utilities using R. I know almost nothing beyond `Rscript -e` or `./script.sh` when it comes to writing a CLI. But there are all kinds of tasks I do every day that could be written as small command line scripts. Plus, my favorite part of package authoring is writing interfaces for other people to use. How do I expect someone to want to use R and reason about a problem I'm helping to solve? It's no wonder that I work on product every day with this interest. So I figure one way to keep engaged in R is to learn how to design command line utilities in R and get good at it. Rather than write R code purely intended to be called and used from R, my R code is going to get an interface this year.

Like every year, I'd like to keep up with this blog. I never do, but this year I had a lot of encouraging signs. I actually got considerable attention for every R-related post (high hundreds of views), so I think it's time to lean into that. I'm hoping to write one R related post each week. I think the focus will help me have some chance of pulling this off. Since I also want to keep my R chops alive while I move further and further away from day to day programming responsibilities, it should be a two birds with one stone scenario. One major thing I haven't decided-- do I want to submit to r-bloggers? I'm sure it'd be a huge source of traffic, but I find it frustrating to have to click through from my RSS reader of choice when finding things there.

Lastly, I'd like to start to understand the internals of a core package I use every day. I haven't decided what that'll be. Maybe it'll be something really fundamental like `dplyr`, `DBI`, or `ggplot2`. Maybe it'll be something "simpler". But I use a lot more R code than I read. And one thing I've learned every time I've forced myself to dig in is that I understand more R than I thought and also that reading code is one of the best ways to learn more. I want to do at least one deep study that advances my sense of self-R-worth. Maybe I'll even have to take the time to learn a little C++ and understand how Rccp is being used to change the R world.

## Special Thanks

The #rstats world on Twitter has been the only reason I can get on that service anymore. It's a great and positive place where I learn a ton and I really appreciate feeling like there is a family of nerds out there talking about stuff that I feel like no one should care about. My tweets are mostly stupid musings that come to me and retweeting enraging political stuff in the dumpster fire that is Trump's America, so I'm always surprised and appreciative that anyone follows me. It's so refreshing to get away from that and just read #rstats. So thank you for inspiring me and teaching me and being a fun place to be.

