---
title: Functions as Arguments in R
author: Jason P. Becker
date: 2017-05-29
tags: rstats
---

Sometimes, silly small things about code I write just delight me. There are [lots of ways to time things](http://www.alexejgossmann.com/benchmarking_r/) in R. [^tictoc] Tools like `microbenchmark` are great for profiling code, but what I do all the time is log how long database queries that are scheduled to run each night are taking.

[^tictoc]: `tictoc` is new to me, but I'm glad it is. I would have probably never written the code in this post if it existed, and then I would be sad and this blog post wouldn't exist.

It is really easy to use calls to `Sys.time` and `difftime` when working interactively, but I didn't want to pepper all of my code with the same log statements all over the place. So instead, I wrote a function.

Almost all of `timing` is straightforward to even a novice R user. I record what time it is using `Sys.time`, do a little formatting work to make things look the way I want for reading logs, and pass in an optional message. 

The form of `timing` was easy for me to sketch out: [^realize]

```r
timing <- function(STUFF, msg = '') {
  start_time <- format(Sys.time(), '%a %b %d %X %Y')
  start_msg <- paste('Starting', msg,
                     'at:', start_time, '\n')
  cat(start_msg)
  # Call my function here
  end_time <- format(Sys.time(), '%a %b %d %X %Y')
  end_msg <- paste('Completed', 'at:', end_time, '\n')
  cat(end_msg)
  elapsed <- difftime(as.POSIXlt(end_time, format = '%a %b %d %X %Y'),
                      as.POSIXlt(start_time, format = '%a %b %d %X %Y'))
  cat('Elapsed Time: ', format(unclass(elapsed), digits = getOption('digits')),
      ' ', attr(elapsed, 'units'), '\n\n\n', sep = '')
  result
}
```

[^realize]: Yes, I realize that having the calls to `paste` and `cat` after setting `start_time` technically add those calls to the stack of stuff being timed and both of those things could occur after function execution. For my purposes, the timing does not have to be nearly that precise and the timing of those functions will contribute virtually nothing. So I opted for what I think is the clearer style of code as well as ensuring that live monitoring would inform me of what's currently running.

The thing I needed to learn when I wrote `timing` a few years back was how to fill in `STUFF` and `# Call my function here`. 

Did you know that you can pass a function as an argument in another function in R? I had been using `*apply` with its `FUN` argument all over the place, but never _really_ thought about it until I wrote `timing`. Of course in R you can pass a function name, and I even know how to pass arguments to that function-- just like `apply`, just declare a function with the magical `...` and pass that along to the fucntion being passed in.

So from there, it was clear to see how I'd want my function declartion to look. It would definitely have the form `function(f, ..., msg = '')`, where `f` was some function and `...` were the arguments for that function. What I didn't know was how to properly call that function. Normally, I'd write something like `mean(...)`, but I don't know what `f` is in this case!

As it turns out, the first thing I tried worked, much to my surprise. R actually makes this super easy-- you can just write `f(...)`, and `f` will be replaced with whatever the argument is to `f`! This just tickles me. It's stupid elegant to my eyes.

```r
timing <- function(f, ..., msg = '') {
  start_time <- format(Sys.time(), '%a %b %d %X %Y')
  start_msg <- paste('Starting', msg,
                     'at:', start_time, '\n')
  cat(start_msg)
  x <- f(...)
  end_time <- format(Sys.time(), '%a %b %d %X %Y')
  end_msg <- paste('Completed', 'at:', end_time, '\n')
  cat(end_msg)
  elapsed <- difftime(as.POSIXlt(end_time, format = '%a %b %d %X %Y'),
                      as.POSIXlt(start_time, format = '%a %b %d %X %Y'))
  cat('Elapsed Time: ', format(unclass(elapsed), digits = getOption('digits')),
      ' ', attr(elapsed, 'units'), '\n\n\n', sep = '')
  x
}
```

Now I can monitor the run time of any function by wrapping it in `timing`. For example:

```r
timing(read.csv, 'my_big_file.csv', header = TRUE, stringsAsFactors = FALSE)`
```

And here's an example of the output from a job that ran this morning:

```plaintext
Starting queries/accounts.sql at: Mon May 29 06:24:12 AM 2017
Completed at: Mon May 29 06:24:41 AM 2017
Elapsed Time: 29 secs
```