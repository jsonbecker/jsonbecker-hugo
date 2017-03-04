---
title: Labeling Data with purrr
author: Jason P. Becker
date: 2017-03-03
tags: rstats
---

Here's a fun common task. I have a data set that has a bunch of codes like:

| Name    | Abbr | Code |
| :-----  | :--: | ---: |
| Alabama | AL |  01 |
| Alaska |  AK |  02 |
| Arizona | AZ |  04 |
| Arkansas |  AR |  05 |
| California |  CA |  06 |
| Colorado |  CO |  08 |
| Connecticut | CT |  09 |
| Delaware |  DE |  10 |

All of your data is labeled with the `code` value. In this case, you want to do a `join` so that you can use the actual names because it's 2017 and we're not animals.

But what if your data, like the accounting data we deal with at [Allovue](http://www.allovue.com), has lots of code fields. You probably either have one table that contains all of the look ups in "long" format, where there is a column that represents which column in your data the code is for like this:

| code    | type | name |
| :-----  | :--: | ---: |
| 01      | fips | Alabama |
| 02      | fips | Alaska |

Alternatively, you may have a lookup table per data element (so one called fips, one called account, one called function, etc).

I bet most folks do the following in this scenario:

```r
account <- left_join(account, account_lookup)
account <- left_join(account, fips)

## Maybe this instead ##
account %<>%
  left_join(account_lookup) %>%
  left_join(fips)
```

I want to encourage you to do this a little different using `purrr`. Here's some annotated code that uses `reduce_right` to make magic.

```r
# Load a directory of .csv files that has each of the lookup tables
lookups <- map(dir('data/lookups'), read.csv, stringsAsFactors = FALSE)
# Alternatively if you have a single lookup table with code_type as your
# data attribute you're looking up
# lookups <- split(lookups, code_type)
lookups$real_data <- read.csv('data/real_data.csv', 
                              stringsAsFactors = FALSE)
real_data <- reduce_right(lookups, left_join)
```

Boom, now you went from data with attributes like `funds_code`, `function_code`, `state_code` to data that also has `funds_name`, `function_name`, `state_name`[^namingconventions]. What's great is that this same code can be reused no matter how many fields require a hookup. I'm oftent dealing with accounting data where the accounts are defined by a different number of data fields, but my code doesn't care at all.

[^namingconventions]: My recommendation is to use consistent naming conventions like `_code` and `_name` so that knowing how to do the lookups is really straightforward. This is not unlike the convention with Microsoft SQL where the primary key of a table is named `Id` and a foreign key to that table is named `TableNameId`. Anything that helps you figure out how to put things together without thinking is worth it.

