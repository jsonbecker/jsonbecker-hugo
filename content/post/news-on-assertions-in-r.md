---
title: News on Assertions in R
author: Jason P. Becker
date: 2015-07-05
tags: rstats
link: http://4dpiecharts.com/2015/07/03/the-state-of-assertions-in-r/
---

How many times have you written R functions that start with a bunch of code that looks like this?

```r
my_funct <- function(dob, enddate = "2015-07-05"){
if (!inherits(dob, "Date") | !inherits(enddate, "Date")){
    stop("Both dob and enddate must be Date class objects")
  } 
...
}
```

Because R was designed to be interactive, it is incredibly tolerant to bad user input. Functions are not *type safe*, meaning function arguments do not have to conform to specified data types. But most of my R code is not run interactively. I have to trust my code to run on servers on schedules or on demand as a part of production systems. So I find myself frequently writing code like the above-- manually writing type checks for safety.

There has been some great action in the R community around *assertive programming*, as you can see in the link. My favorite development, by far, are type-safe functions in the [ensurer package](https://github.com/smbache/ensurer). The above function definition can now be written like this:

```r
my_funct <- function_(dob ~ Date, enddate ~ Date: as.Date("2015-07-05"), {
  ...
})
```
All the type-checking is done.

I really like the reuse of the formula notation `~` and the use of `:` to indicate default values.

Along with packages like [testthat](https://github.com/hadley/testthat), R is really growing up and modernizing.

