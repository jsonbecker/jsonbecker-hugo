---
title: Calculating Age in R
author: Jason P. Becker
email: jason.paul.becker@gmail.com
date: 2013-06-12
tags: rstats, r, code
---


A few months back I wrote some code to calculate age from a date of birth and arbitrary end date. It is not a real tricky task, but it is certainly one that comes up often when doing research on individual-level data.

I was a bit surprised to only find bits and pieces of code and advice on how to best go about this task. After reading through some old R-help and Stack Overflow responses on various ways to do date math in R, this is the function I wrote [^wrote]:

```r
age_calc <- function(dob, enddate=Sys.Date(), units='months'){
  if (!inherits(dob, "Date") | !inherits(enddate, "Date"))
    stop("Both dob and enddate must be Date class objects")
  start <- as.POSIXlt(dob)
  end <- as.POSIXlt(enddate)
  
  years <- end$year - start$year
  if(units=='years'){
    result <- ifelse((end$mon < start$mon) | 
                      ((end$mon == start$mon) & (end$mday < start$mday)),
                      years - 1, years)    
  }else if(units=='months'){
    months <- (years-1) * 12
    result <- months + start$mon
  }else if(units=='days'){
    result <- difftime(end, start, units='days')
  }else{
    stop("Unrecognized units. Please choose years, months, or days.")
  }
  return(result)
}
```

A few notes on proper usage and the choices I made in writing this function: 

* The parameters `dob` and `enddate` expect data that is already in one of the various classes that minimally inherits the base class `Date`. 
* This function takes advantage of the way that R treats vectors, so both `dob` and `enddate` can be a single or multi-element vector. For example `enddate` is a single date, as is the default, then the function will return a vector that calculates the difference between `dob` and that single date for each element in `dob`. If `dob` and `enddate` are both vectors with n>1, then the returned vector will contain the [element-wise][] difference between `dob` and `enddate`. When the vectors are of different sizes, the shorter vector will be repeated over until it reaches the same length as the longer vector. This is known as [recycling][], and it is the default behavior in R.
* This function always returns an integer. Calculating age in years will never return, say, 26.2. Instead, it assumes that the correct behavior for age calculations is something like a `floor` function. For examle, the function will only return 27 if `enddate` is minimally your 27th birthday. Up until that day you are considered 26. The same is true for age in months.

This is probably the first custom function in almost 3 years using R that I wrote to be truly generalizable. I was inspired by three factors. First, this is a truly frequent task that I will have to apply to many data sets in the future that I don't want to have to revisit. Second, a professional acquaintance, [Jared Knowles][], is putting together a CRAN package with various convenience functions for folks who are new to R and using it to analyze education data [^eeptools]. This seemed like an appropriate addition to that package, so I wanted to write it to that standard. In fact, it was my first (and to date, only) submitted and accepted pull request on Github. Third, it is a tiny, simple function so it was easy to wrap my head around and write it well. I will let you be the judge of my success or failure [^inspiration].

[^wrote]: I originally used `Sys.time()` not realizing there was a `Sys.Date()` function. Thanks to Jared Knowles for that edit in preparation for a CRAN check.

[^eeptools]: Check out [eeptools] on Github.

[^inspiration]: Thanks to [Matt's Stats n Stuff][] for getting me to write this post. When I saw another age calculation function pop up on the r-bloggers feed I immediately thought of this function. Matt pointed out that it was quite hard to Google for age calculations in R, lamenting that Google doesn't meaningfully crawl Github where I linked to find my code. So this post is mostly about providing some help to less experience R folks who are frantically Googling as both Matt and I did when faced with this need.

[Matt's Stats n Stuff]: http://mcfromnz.wordpress.com/2013/06/12/updated-age-calculation-function/

[element-wise]: http://heather.cs.ucdavis.edu/~matloff/r.old.html#elementwise

[recycling]: http://cran.r-project.org/doc/manuals/R-intro.html#The-recycling-rule

[Jared Knowles]: http://jaredknowles.com/
[eeptools]: https://github.com/jknowles/eeptools