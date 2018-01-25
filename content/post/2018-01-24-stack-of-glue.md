---
title: Stack of Glue
author: Jason Becker
date: '2018-01-25'
slug: stack-of-glue
tags: [rstats]
---

I have some text, but I want the content of that text to be dynamic based on data. This is a case for string interpolation. Lots of languages have the ability to write something like 

```ruby
pet = "dog"
puts "This is my {#pet}"
```

```python
pet = "dog"
print(f"This is my {pet}")
```

There have been ways to do this in R, but I've mostly hated them until `glue` came along. Using `glue` in R should look really familiar now:

```r
pet <- "dog"
glue("This is my {pet}")
```

Awesome! Now I have a way to make text bend to my bidding using data. But this is pretty simple, and we could have just used something like `paste("This is my", pet)` and been done with it.

Let me provide a little motivation in the form of `data.frame`s, `glue_data`, and some `purrr`.

Pretend we have a field in a database called `notes`. I want to set the `notes` for each entity to follow the same pattern, but use other data to fill in the blanks. Like maybe something like this:

```r
notes <- "This item price is valid through {end_date} and will then increase {price_change} to {new_price}."
```

This is a terrible contrived example, but we can imagine displaying this note to someone with different content for each item. Now in most scenarios, the right thing to do for an application is to produce this content dynamically based on what's in the database, but let's pretend no one looked far enough ahead to store this data or like notes can serve lots of different purposes using different data. So there is no place for the application to find `end_date`, `price_change`, or `new_price` in its database. Instead, this was something prepared by sales in Excel yesterday and they want these notes added to all items to warn their customers.

Here's how to take a table that has `item_id`, `end_date`, `price_change`, and `new_price` as columns and turn it into a table with `item_id`, and `notes` as columns, with your properly formatted note for each item to be updated in a database.


```r
library(glue)
library(purrr)

item_notes <- data.frame(
  item_id = seq_len(10),
  end_date = c(rep(as.Date('2018-03-01', format = '%Y-%m-%d'), 5),
               rep(as.Date('2018-03-05', format = '%Y-%m-%d'), 3),
               rep(as.Date('2018-03-09', format = '%Y-%m-%d'), 2)),
  price_change = sample(x = seq_len(5),replace = TRUE,size = 10),
  new_price = sample(x = 10:20,replace = TRUE,size = 10)
)

template <- "This item price is valid through {end_date} and will then increase {price_change} to {new_price}."

map_chr(split(item_notes, item_notes$item_id), 
    glue_data, 
    template) %>% 
stack() %>% 
rename(item_id = ind,
       notes = values)
```

What's going on here? First, I want to apply my `glue` technique to rows of a `data.frame`,
so I `split` the data into a `list` using `item_id` as the identifier. That's because at the end of all this I want to preserve that id to match back up in a database. [^rownames] The function `glue_data` works like `glue`, but it accepts things that are "listish" as it's first argument (like `data.frames` and named `lists`). So with handy `map` over my newly created `list` of "listish" data, I create a named `list` with the text I wanted to generate. I then use a base R function that's new to me `stack`, which will take a list and make each element a row in a `data.frame` with `ind` as the name of the `list` element and `values` as the value.

Now I've got a nice `data.frame`, ready to be joined with any table that has `item_id` so it can have the attached note!


[^rownames]: You can split on `row.names` if you don't have a similar identifer and just want to go from `data.frame` to a `list` of your rows.
