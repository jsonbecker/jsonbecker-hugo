---
title: Calculating Age with Precision in R
author: Jason P. Becker
date: 2013-12-04
tags: r, rstats, age_calc, eeptools, code
---

**Update**
> Turns out the original code below was pretty messed up. All kinds of little errors I didn't catch. I've updated it below. There are a lot of options to refactor this further that I'm currently considering. Sometimes it is really hard to know just how flexible something this big really should be. I think I am going to wait until I start developing tests to see where I land. I have a feeling moving toward a more test-driven work flow is going to force me toward a different structure.

I recently updated the function I [posted] about back in June that calculates the difference between two dates in days, months, or years in R. It is still surprising to me that `difftime` can only return units from seconds up until weeks. I suspect this has to do with the challenge of properly defining a "month" or "year" as a unit of time, since these are variable.

[posted]: |filename|/calculating-age-in-r.md

While there was nothing wrong with the original function, it did irk me that it always returned an integer. In other words, function returned only complete months or years. If the start date was on `2012-12-13` and the end date was on `2013-12-03`, the function would return `0` years. Most of the time, this is the behavior I expect when calcuating age. But it is completely reasonable to want to include partial years or months, e.g. in the aforementioned example returning `0.9724605`.

So after several failed attempts because of silly errors in my algorithm, here is the final code. It will be released as part of [eeptools 0.3](https://github.com/jknowles/eeptools/) which should be avialable on CRAN soon [^moves].

[^moves]: I should note that my mobility function will also be included in eeptools 0.3. I know I still owe a post on the actual code, but it is such a complex function I have been having a terrible time trying to write clearly about it.

```r
age_calc <- function(dob, enddate=Sys.Date(), units='months', precise=TRUE){
  if (!inherits(dob, "Date") | !inherits(enddate, "Date")){
    stop("Both dob and enddate must be Date class objects")
  }
  start <- as.POSIXlt(dob)
  end <- as.POSIXlt(enddate)
  if(precise){
    start_is_leap <- ifelse(start$year %% 400 == 0, TRUE, 
                        ifelse(start$year %% 100 == 0, FALSE,
                               ifelse(start$year %% 4 == 0, TRUE, FALSE)))
    end_is_leap <- ifelse(end$year %% 400 == 0, TRUE, 
                        ifelse(end$year %% 100 == 0, FALSE,
                               ifelse(end$year %% 4 == 0, TRUE, FALSE)))
  }
  if(units=='days'){
    result <- difftime(end, start, units='days')
  }else if(units=='months'){
    months <- sapply(mapply(seq, as.POSIXct(start), as.POSIXct(end), 
                            by='months', SIMPLIFY=FALSE), 
                     length) - 1
    # length(seq(start, end, by='month')) - 1
    if(precise){
      month_length_end <- ifelse(end$mon==1, 28,
                                 ifelse(end$mon==1 & end_is_leap, 29,
                                        ifelse(end$mon %in% c(3, 5, 8, 10), 
                                               30, 31)))
      month_length_prior <- ifelse((end$mon-1)==1, 28,
                                     ifelse((end$mon-1)==1 & start_is_leap, 29,
                                            ifelse((end$mon-1) %in% c(3, 5, 8, 
                                                                      10), 
                                                   30, 31)))
      month_frac <- ifelse(end$mday > start$mday,
                           (end$mday-start$mday)/month_length_end,
                           ifelse(end$mday < start$mday, 
                            (month_length_prior - start$mday) / 
                                month_length_prior + 
                                end$mday/month_length_end, 0.0))
      result <- months + month_frac
    }else{
      result <- months
    }
  }else if(units=='years'){
    years <- sapply(mapply(seq, as.POSIXct(start), as.POSIXct(end), 
                            by='years', SIMPLIFY=FALSE), 
                     length) - 1
    if(precise){
      start_length <- ifelse(start_is_leap, 366, 365)
      end_length <- ifelse(end_is_leap, 366, 365)
      year_frac <- ifelse(start$yday < end$yday,
                          (end$yday - start$yday)/end_length,
                          ifelse(start$yday > end$yday, 
                                 (start_length-start$yday) / start_length +
                                end$yday / end_length, 0.0))
      result <- years + year_frac
    }else{
      result <- years
    }
  }else{
    stop("Unrecognized units. Please choose years, months, or days.")
  }
  return(result)
}
```
