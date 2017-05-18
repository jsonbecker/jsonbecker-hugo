---
title: Making Sense of dplyr 0.6
author: Jason P. Becker
date: 2017-05-18
tags: rstats, dplyr
---

Non-standard evaluation is one of R's best features, and also one of it's most perplexing. Recently I have been making good use of `wrapr::let` to allow me to write reusable functions without a lot of assumptions about my data. For example, let's say I always want to `group_by` schools when adding up dollars spent, but that sometimes my data calls what is conceptually a school `schools`, `school`, `location`, `cost_center`, `Loc.Name`, etc. What I have been doing is storing a set of parameters in a `list` that mapped the actual names in my data to consistent names I want to use in my code. Sometimes that comes from using `params` in an Rmd file. So the top of my file may say something like:

```yaml
params:
    school: "locations"
    amount: "dollars"
    enrollment: n
```

In my code, I may want to write a chain like

```r
create_per_pupil <- . %>%
                    group_by(school) %>%
                    summarize(per_pupil = sum(amount) / n)
pp <- district_data %>%
      create_per_pupil
```

Only my problem is that `school` isn't always `school`. In this toy case, you could use `group_by_(params$school)`, but it's pretty easy to run into limitations with the `_` functions in `dplyr` when writing functions.

Using `wrapr::let`, I can easily use the code above:

```r
let(alias = params, {
    create_per_pupil <- . %>%
                        group_by(school) %>%
                        summarize(per_pupil = sum(amount)/n)
})

pp <- district_data %>%
      create_per_pupil
```

The core of `wrapr::let` is really scary.

```r
body <- strexpr
for (ni in names(alias)) {
    value <- as.character(alias[[ni]])
    if (ni != value) {
        pattern <- paste0("\\b", ni, "\\b")
        body <- gsub(pattern, value, body)
    }
}
parse(text = body)
```

Basically let is holidng onto the code block contained within it, iterating over the list of key-value pairs that are provided, and then runs a `gsub` on word boundaries to replace all instances of the list names with their values. Yikes.

This works, I use it all over, but I have never felt confident about it.

## The New World of tidyeval

The release of dplyr 0.6 along with tidyeval brings wtih it a ton of features to making programming over dplyr functions far better supported. I am going to [read this page](http://dplyr.tidyverse.org/articles/programming.html) by Hadley Wickham at least 100 times. There are all kinds of new goodies (`!!!` looks amazing).

So how would I re-write the chain above sans `let`?

```r
create_per_pupil <- . %>%
                    group_by(!!sym(school)) %>%
                    summarize(per_pupil = sum(amount)/n)
```

If I understand `tidyeval`, then this is what's going on.

* `sym` evaluates `school` and makes the result a `symbol`
* and `!!` says, roughly "evaluate that symbol now".

This way with `params$school` having the value `"school_name"`, `sym(school)` creates evaulates that to `"school_name"` and then makes it an unquoted symbol `school_name`. Then `!!` tells R "You can evaluate this next thing in place as it is."

I originally wrote this post trying to understand `enquo`, but I never got it to work right and it makes no sense to me yet. What's great is that `rlang::sym` and `rlang::syms` with `!!` and `!!!` respectively work really well so far. There is definitely less flexibility-- with the full on `quosure` stuff you can have very complex evaluations. But I'm mostly worried about having very generic names for my data so `sym` and `syms` seems to work great.