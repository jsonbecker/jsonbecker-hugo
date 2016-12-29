---
title: A Gateway Drug to purrr
author: Jason P. Becker
date: 2016-12-29
tags: 
---

A lot of the data I work with uses numeric codes rather than text to describe features of each record. For example, financial data often has a fund code that represents the account's source of dollars and an object code that signals what is bought (e.g. salaries, benefits, supplies). This is a little like the `factor` data type in `R`, which to the frustration of many modern analysts is internally an integer that mapped to a character label (which is a level) with a fixed number of possible values.

I am often looking at data stored like this:

fund_code | object_code | debit | credit 
:-------- | :---------- | ----: | -----: 
1000      | 2121        | 0     | 10000  
1000      | 2122        | 1000  | 0      

with the labels stored in another set of tables:

fund_code | fund_name  
:------- | :-------
1000      | General   

and

object_code | object_name
:------- | :--------
2121        | Social Security
2122        | Life Insurance

Before `purrr`, I might have done a series of `dplyr::left_join` or `merge` to combine these data sets and get the labels in the same `data.frame` as my data.

But no longer!

{{< tweet 813793775260733441 >}}

Now, I can just create a `list`, add all the data to it, and use `purrr:reduce` to bring the data together. Incredibly convenient when up to 9 codes might exist for a single record!

```r
# Assume each code-name pairing is in a CSV file in a directory
data_codes <- lapply(dir('codes/are/here/', full.names = TRUE ), 
                     readr::read_csv)
data_codes$transactions <- readr::read_csv('my_main_data_table.csv')
transactions <- purrr:reduce_right(data_codes, dplyr::left_join)
```