---
title: Naming Manual Colors with ggplot2
author: Jason Becker
date: '2018-01-03'
slug: naming-manual-colors-with-ggplot2
categories: []
tags: [rstats, ggplot2]
---

I have been using `ggplot2` for 7 years I think. In all that time, I've been frustrated that I can never figure out what order to put my color values in for `scale_*_manual`. Not only is the order mapping seemingly random to me, I know that sometimes if I change something about how I'm treating the data, the order switches up.

Countless hours could have been saved if I knew that this one, in hindsight, obvious thing was possible.

Whenever using `scale_*_manual`, you can directly reference a color using a character vector and then name your `value` in the `scale_` call like so:

```r
geom_blah(aes(color = 'good')) +
geom_blah(aes(color = 'bad')) +
scale_blah_manual(values = c(good = 'green', bad = 'red'))
```

Obviously this is a toy example, but holy game changer.
