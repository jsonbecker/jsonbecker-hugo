---
title: Foreign Keys and assertr
author: Jason P. Becker
date: 2017-04-02 21:19:30
tags: rstats, assertr
---

I have been fascinated with assertive programming in R since [this] from 2015 [^missing]. Tony Fischetti wrote a [great blog post](http://www.onthelambda.com/2017/03/20/data-validation-with-the-assertr-package/) to announce `assertr` 2.0's release on CRAN that really clarified the package's design.

UseRs often do crazy things that no sane developer in another language would do. Today I decided to build a way to check foreign key constraints in R to help me learn the `assertr` package.

## What do you mean, foreign key constraints?

Well, in many ways this is an extension of my [last post] on using `purrr::reduce`. I have a set of data with codes (like FIPS codes, or user ids, etc) and I want to make sure that all of those codes are "real" codes (as in I have a defintion for that value). So I may have a FIPS code `data.frame` with `fips_code` and `name` as the columns or a user `data.frame` with columns `id`, `fname`, `lname`, `email`.

In a database, I might have a foreign key constraint on my table that just has codes so that I could not create a row that uses an `id` or `code` value or whatever that did not exist in my lookup table. Of course in R, our data is disconnected and non-relational. New users may exist in my dataset that weren't there the last time I downloaded the `users` table, for example.

## Ok, so these are just collections of enumerated values

Yup! That's right! In some ways like R's _beloved_ `factors`, I want to have problems when my data contains values that don't have a corresponding row in another `data.frame`, just like trying to insert a value into a `factor` that isn't an existing level.

`assertr` anticipates just this, with the `in_set` helper. This way I can `assert` that my data is in a defined set of values or get an error.

```r
> my_df <- data.frame(x = c(0,1,1,2))
> assert(df, in_set(0,1), x)
Column 'x' violates assertion 'in_set(0, 1)' 1 time
  index value
1     4     2
Error: assertr stopped execution
```

## Please Don't stop()

By default, `assert` raises an error with an incredibly helpful message. It tells you which column the assertion was on, what the assertion was, how many times that assertion failed, and then returns the column index and value of the failed cases.

Even better, `assert` has an argument for `error_fun`, which, combined with some built in functions, can allow for all kinds of fun behavior when an assertion fails. What if, for example, I actually want to collect that error message for later and not have a hard stop if an assertion failed?

By using `error_append`, `assert` will return the original `data.frame` when there's a failure with a special attribute called `assertr_errors` that can be accessed later with all the information about failed assertions.

```r
> my_df %<>%
+   assert(in_set(0,1), x, error_fun = error_append) %>%
+   verify(x == 1, error_fun = error_append)
> my_df
  x
1 0
2 1
3 1
4 2
> attr(my_df, 'assertr_errors')
[[1]]
Column 'x' violates assertion 'in_set(0, 1)' 1 time
  index value
1     4     2

[[2]]
Column 'x' violates assertion 'in_set(0, 1)' 1 time
  index value
1     4     2

[[3]]
verification [x == 1] failed! (2 failures)
```

(Ok I cheated there folks. I used `verify`, a new function from `assertr` and a bunch of `magrittr` pipes like `%<>%`)

## Enough with the toy examples

Ok, so here's the code I wrote today. This started as a huge mess I ended up turning into two functions. First `is_valid_fk` provides a straight forward way to get `TRUE` or `FALSE` on whether or not all of your codes/ids exist in a lookup `data.frame`.


```r
is_valid_fk <- function(data, key, values,
                        error_fun = error_logical,
                        success_fun = success_logical){

  assert_(data, in_set(values), key,
          error_fun = error_fun, success_fun = success_fun)

}
```

The first argument `data` is your `data.frame`, the second argument `key` is the foreign key column in `data`, and `values` are all valide values for `key`. Defaulting the `error_fun` and `success_fun` to `*_logical` means a single boolean is the expected response.

But I don't really want to do these one column at a time. I want to check if all of the foreign keys in a table are good to go. I also don't want a boolean, I want to get back all the errors in a useable format. So I wrote `all_valid_fk`.

Let's take it one bit at a time. 

```r
all_valid_fk <- function(data, fk_list, id = 'code') {
```

1. `data` is the `data.frame` we're checking foreign keys in.
2. `fk_list` is a list of `data.frames`. Each element is named for the `key` that it looks up; each `data.frame` contains the valid values for that `key` named...
3. `id`, the name of the column in each `data.frame` in the list `fk_list` that corresponds to the valid `keys`.

```r
verify(data, do.call(has_all_names, as.list(names(fk_list))))
```

Right away, I want to know if my data has all the values my `fk_list` says it should. I have to do some `do.call` magic because `has_all_names` wants something like `has_all_names('this', 'that', 'the_other')` not `has_all_names(c('this', 'that', 'the_other')`.

The next part is where the magic happens.

```r
accumulated_errors <- map(names(fk_list),
                            ~ is_valid_fk(data,
                                          key = .x,
                                          values = fk_list[[.x]][[id]],
                                          error_fun = error_append,
                                          success_fun = success_continue)) %>%
                        map(attr, 'assertr_errors') %>%
                        reduce(append)
```
Using `map`, I am able to call `is_valid_fk` on each of the columns in `data` that have a corresponding lookup table in `fk_list`. The valid values are `fk_list[[.x]][[id]]`, where `.x` is the name of the `data.frame` in `fk_list` (which corresponds to the name of the code we're looking up in `data` and exists for sure, thanks to that `verify` call) and `id` is the name of the key in that `data.frame` as stated earlier. I've replaced `error_fun` and `success_fun` so that the code does not exist `map` as soon there are any problems. Instead, the data is returned for each assertion with the error attribute if one exists. [^memory] Immediately, `map` is called on the resulting list of `data.frame`s to collect the `assertr_errors`, which are `reduce`d using `append` into a flattened list.

If there are no errors accumulated, `accumulated_errors` is `NULL`, and the function exits early.

```r
if(is.null(accumulated_errors)) return(list())
```

I could have stopped here and returned all the messages in `accumulated_errors`. But I don't like all that text, I want something neater to work with later. The structure I decided on was a list of `data.frame`s, with each element named for the column with the failed foreign key assertion and the contents being the index and value that failed the constraint. 

By calling `str` on `data.frame`s returned by assertion, I was able to see that the `index` and `value` tables printed in the failed `assert` messages are contained in `error_df`. So next I extract each of those `data.frame`s into a single list.

```r
reporter <- accumulated_errors %>%
            map('error_df') %>%
            map(~ map_df(.x, as.character)) # because factors suck
```

I'm almost done. I have no way of identifying which column created each of those `error_df` in `reporter`. So to name each element based on the column that failed the foreign key contraint, I have to extract data from the `message` attribute. Here's what I came up with.

```r
names(reporter) <- accumulated_errors %>%
                   map_chr('message') %>%
                   gsub("^Column \'([a-zA-Z]+)\' .*$", '\\1', x = .)
reporter
```

So let's create some fake data and run `all_valid_fk` to see the results:

```r
> df <- data.frame(functions = c('1001','1002', '3001', '3002'),
                   objects = c('100','102', '103', '139'),
                   actuals = c(10000, 2431, 809, 50000),
                   stringsAsFactors = FALSE)

> chart <- list(functions = data.frame(code = c('1001', '1002', '3001'),
                                       name = c('Foo', 'Bar', 'Baz'),
                                       stringsAsFactors = FALSE),
                objects =   data.frame(code = c('100', '102', '103'),
                                       name = c('Mom', 'Dad', 'Baby'),
                                       stringsAsFactors = FALSE))
> all_valid_fk(data = df, fk_list = chart, id = 'code')
$functions
# A tibble: 1 × 2
  index value
  <chr> <chr>
1     4  3002

$objects
# A tibble: 1 × 2
  index value
  <chr> <chr>
1     4   139
```

Beautiful!

And here's `all_valid_fk` in one big chunk.

```r
all_valid_fk <- function(data, fk_list, id = 'code') {
  verify(data, do.call(has_all_names, as.list(names(fk_list))))

  accumulated_errors <- map(names(fk_list),
                            ~ is_valid_fk(data,
                                          key = .x,
                                          values = fk_list[[.x]][[id]],
                                          error_fun = error_append,
                                          success_fun = success_continue)) %>%
                        map(attr, 'assertr_errors') %>%
                        reduce(append)

  if(is.null(accumulated_errors)) return(list())

  reporter <- accumulated_errors %>%
              map('error_df') %>%
              map(~ map_df(.x, as.character))

  names(reporter) <- accumulated_errors %>%
                     map_chr('message') %>%
                     gsub("^Column \'([a-zA-Z]+)\' .*$", '\\1', x = .)
  reporter
}
```

[this]: {{< relref "news-on-assertions-in-r.md" >}}
[last post]: {{< relref "a-gateway-to-purrr.md" >}}
[^missing]: I appear to have forgotten to build link post types into my Hugo blog, so the missing link from that post is [here](http://4dpiecharts.com/2015/07/03/the-state-of-assertions-in-r/).
[^memory]: I am a little concerned about memory here. Eight assertions would mean, at least briefly, eight copies of the same `data.frame` copied here without the need for that actual data. There is probably a better way.