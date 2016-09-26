---
title: Appreciating the Beauty of dplyr
author: Jason P. Becker
date: 2014-02-18
tags: rstats, dplyr, hadleyverse
---

[Hadley Wickham](http://had.co.nz) has [once again](https://github.com/hadley)[^seriously] made R ridiculously better. Not only is `dplyr` incredibly fast, but the new syntax allows for some really complex operations to be expressed in a ridiculously beautiful way.

Consider a data set, `course`, with a student identifier, `sid`, a course identifier, `courseno`, a quarter, `quarter`, and a grade on a scale of 0 to 4, `gpa`. What if I wanted to know the number of a courses a student has failed over the entire year, as defined by having an overall grade of less than a 1.0?

In dplyr:

```r
course %.% 
group_by(sid, courseno) %.%
summarise(gpa = mean(gpa)) %.%
filter(gpa <= 1.0) %.%
summarise(fails = n())
```

I refuse to even sully this post with the way I would have solved this problem in the past.

[^seriously]: Seriously, how many of the packages he has managed/written are indispensable to using R today? It is no exaggeration to say that the world would have many more Stata, SPSS, and SAS users if not for Hadleyverse.