---
title: Contributing to vegalite
author: Jason P. Becker
date: 2017-03-02
tags: rstats
---

One of my goals for 2017 is to contribute more to the R open source community. At the beginning of last year, I spent a little time helping to refactor [rio](https://www.github.com/leeper/rio). It was one of the more rewarding things I did in all of 2016. It wasn't a ton of work, and I feel like I gained a lot of confidence in writing R packages and using S3 methods. I wrote code that R users download and use thousands of times a month.

I have been on the lookout for a Javascript powered interactive charting library since `ggvis` was announced in 2014. But `ggvis` seems to have stalled out in favor of other projects (for now) and the evolution of `rCharts` into `htmlwidgets` left me feeling like there were far too many options and no clear choices.

What I was looking for was a plotting library to make clean, attractive graphics with tool tips that came with clear documentation and virtually no knowledge of Javascript required. Frankly, all of the `htmlwidgets` stuff was very intimidating. From my vantage point skimming blog posts and watching stuff come by on Twitter, `htmlwidgets`-based projects all felt very much directed at Javascript polyglots.

Vega and Vega-Lite had a lot of the qualities I sought in a plotting library. Reading and writing JSON is very accessible compared to learning Javascript, especially with R's excellent translation from lists to JSON. And although I know almost no Javascript, I found in both Vega and Vega-Lite easy to understand documents that felt a lot like building grammar of graphics [^thegg] plots.

[^thegg]: The `gg` in `ggplot2`.

So I decided to take the plunge-- there was a `vegalite` [package](https://github.com/hrbrmstr/vegalite) and the examples didn't look so bad. It was time to use my first `htmlwidgets` package.

Things went great. I had some simple data and I wanted to make a bar chart. I wrote:

```r
vegalite() %>%
add_data(my_df) %>%
encode_x('schools', type = 'nominal') %>%
encode_y('per_pupil', type = 'quantitative') %>%
mark_bar()
```

A bar chart was made! But then I wanted to use the font Lato, which is what we use at [Allovue](http://www.allovue.com). No worries, Vega-Lite has a property called `titleFont` for axes. So I went to do:

```r
vegalite() %>%
add_data(my_df) %>%
encode_x('schools', type = 'nominal') %>%
encode_y('per_pupil', type = 'quantitative') %>%
mark_bar() %>%
axis_x(titleFont = 'Lato')
```

Bummer. It didn't work. I almost stopped there, experiment over. But then I remembered my goal and I thought, maybe I need to learn to contribute to a package that is an `htmlwidget` and not simply use an `htmlwidget`-based package. I should at least _look_ at the code.

What I found surprised me. Under the hood, all the R package does is build up lists. It makes so much sense-- pass JSON to Javascript to process and do what's needed.

So it turned out, `vegalite` for R was a bit behind the current version of `vegalite` and didn't have the `titleFont` property yet. And with that, I made my [first commit](https://github.com/hrbrmstr/vegalite/commit/8f4d4db057985bac4fc8c5743780b4746dd56c56). All I had to do was update the function definition and add the new arguments to the axis data like so:

```r
if (!is.null(titleFont))    vl$x$encoding[[chnl]]$axis$titleFont <- titleFont
```

But why stop there? I wanted to update all of `vegalite` to use the newest available arguments. Doing so looked like a huge pain though. The original package author made these great functions like `axis_x` and `axis_y`. They both had the same arguments, the only difference was the "channel" was preset as `x` or `y` based on which function was called. Problem was that all of the arguments, all of the assignments, and all of the documentation had to be copied twice. It was worse with `encode` and `scale` which had many, many functions that are similar or identical in their "signature". No wonder the package was missing so many Vega-Lite features-- they were a total pain to add.

So as a final step, I decided I would do a light refactor across the whole package. In each of the core functions, like `encode` and `axis`, I would write a single generic function like `encode_vl()` that would hold all of the possible arguments for the [encoding portion](https://vega.github.io/vega-lite/docs/encoding.html) of Vega-Lite. Then the specific functions like `encode_x` could become wrapper functions that internally call `encode_vl` like so:

```r
encode_x <- function(vl, ...) {
  vl <- encode_vl(vl, chnl = "x", ...)
  vl
}

encode_y <- function(vl, ...) {
  vl <- encode_vl(vl, chnl ="y", ...)
  vl
}

encode_color <- function(vl, ...) {
  vl <- encode_vl(vl, chnl = "color", ...)
  vl
}
```

Now, in order to update the documentation and the arguments for `encoding`, I just have to update the `encode_vl` function. It's a really nice demonstration, in my opinion, of the power of R's `...` syntax. All of the wrapper functions can just pass whatever additional arguments the caller wants to `encode_vl` without having to explicitly list them each time.

This greatly reduced duplication in the code and made it far easier to update `vegalite` to the newest version of Vega-Lite, which I also decided to do.

Now Vega-Lite itself is embarking on a 2.0 release that I have a feeling will have some pretty big changes in store. I'm not sure if I'll be the one to update `vegalite`-- in the end, I think that Vega-Lite is too simple for the visualizations I need to do-- but I am certain whoever does the update will have a much easier go of it now than they would have just over a month ago.

Thanks to [Bob Rudis](https://github.com/hrbrmstr) for making `vegalite` and giving me free range after a couple of commits to go hog-wild on his package!
