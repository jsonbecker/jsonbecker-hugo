---
title: R, Docker, Circle, and Environments 
author: Jason P. Becker
date: 2017-07-21 
tags: rstats, circleci, docker
---

My latest project at work involves (surprise!) an R package that interacts with a database. For the most part, that's nothing new for me. Almost all the work I've done in R in the last 7 years has interacted with databases in some way. What was new for this project is that the database would not be remote, but instead would be running alongside my code in a linked Docker container.

## A quick step back about Docker

Docker is something you use if you want to be cool on Hacker News. But Docker is also a great way to have a reproducible environment to run your code in, from the operating system up. A full review of Docker is beyond the scope of this post (maybe [check this out](https://www.docker.com/what-container)), but I would think of it like this: if you run your code in a Docker container, you can guarantee your code works because you're creating a reproducible environment that can be spun up anywhere. Think of it like making an R package instead of writing an analysis script. Installing the package means you get all your dependency packages and have confidence the functions contained within will work on different machines. Docker takes that to the next level and includes operating system level dependencies like drivers and network configurations in addition to just the thing your R functions use.

## Some challenges with testing in R

Like many folks, I use `devtools` and `testthat` extensively when developing packages. I strive for as-near-as-feasible 100% coverage with my tests, and I am constantly hitting Cmd + Shift + T while writing code in RStudio or running `devtools::test()`. I even use `Check` in the Build pane in RStudio and `goodpractice::gp()` to keep me honest even if my code won't make it to CRAN. But I ran into a few things working with CircleCI running my tests inside of a docker container that pushed me to learn a few critical pieces of information about testing in R.

### Achieving exit status 1

Only two ways of running tests (that I can tell) will result in a returning an exit status code of 1 (error in Unix systems) and therefore cause a build to fail in a continuous integration system. Without that exit status, failing tests won't fail a build, so don't run `devtools::test()` and think you're good to go.

This means using `R CMD build . && R CMD check *tar.gz` or `testthat::test_package($MY_PACKAGE)` are your best bet in most cases. I prefer using `testthat::test_package()` because `R CMD check` cuts off a ton of useful information about test failures without digging into the `*.Rcheck` folder. Since I want to see information about test failures directly in my CI tool, this is a pain. Also, although not released yet, because `testthat::test_package()` supports alternative reporters, I can have jUnit output, which plays very nicely with many CI tools.

### Methods for S4

The `methods` package is not loaded using `Rscript -e`, so if you use `S4` classes make sure you call `library(methods);` as part of your tests. [^methods]

[^methods]: This is only a problem for my older packages. I've long since decided S4 is horrible and not worth it. Just use S3, although R6 looks very attractive.

### Environment Variables and R CMD check

When using `R CMD check` and other functions that call to that program, your environment variables from the OS may not "make it" through to R. That means calls to `Sys.getenv()` when using `devtools::test()` might work, but using `testthat::test_package()` or `R CMD check` may fail.

This was a big thing I ran into. The way I know the host address and port to talk to in the database container running along side my code is using environment variables. All of my tests that were testing against a test database containers were failing for a while and I couldn't figure out why. The key content was on [this page about R startup](https://stat.ethz.ch/R-manual/R-devel/library/base/html/Startup.html). 

> R CMD check and R CMD build do not always read the standard startup files, but they do always read specific Renviron files. The location of these can be controlled by the environment variables R_CHECK_ENVIRON and R_BUILD_ENVIRON. If these are set their value is used as the path for the Renviron file; otherwise, files ‘~/.R/check.Renviron’ or ‘~/.R/build.Renviron’ or sub-architecture-specific versions are employed.

So it turns out I had to get my environment variables of interest into the `R_CHECK_ENVIRON`. At first I tried this by using `env > ~/.R/check.Renviron` but it turns out that `docker run` runs commands as `root`, and R doesn't like that very much. Instead, I had to specify `R_CHECK_ENVIRON=some_path` and then used `env > $R_CHECK_ENVIRON` to make sure that my environment variables were available during testing.

In the end, I have everything set up quite nice. Here are some snippets that might help.

## circle.yml

At the top I specify my `R_CHECK_ENVIRON`

```yml
machine:
  services:
    - docker
  environment:
    R_CHECK_ENVIRON: /var/$MY_PACKAGE/check.Renviron
```

I run my actual tests roughly like so:

```yml
test:
  override:
    - docker run --link my_database_container -it -e R_CHECK_ENVIRON=$R_CHECK_ENVIRON my_container:my_tag /bin/bash ./scripts/run_r_tests.sh
```

Docker adds critical environment variables to the container when using `--link` that point to the host and port I can use to find the database container. 

## run_r_tests.sh

I use a small script that takes care of dumping my environment properly and sets me up to take advantage of `test_package()`'s reporter option rather than directly writing my commands in line with `docker run`.

```bash
#! /bin/bash
# dump environment into R check.Renviron
env > /var/my_package/check.Renviron

Rscript -e "library(devtools);devtools::install();library(testthat);library(my_package);test_package('my_package', reporter = 'Summary')"
```

To be honest, I'm not convinced I need to do either the `install()` step or `library(my_package)`. Also, you can run `R CMD build . && R CMD check *tar.gz` instead of using the `Rscript` line. I am also considering copying the `.Rcheck` folder to `$CIRCLE_ARTIFACTS` so that I can download it as desired. To do that, you can just add:

```bash
mkdir -p $CIRCLE_ARTIFACTS/test_results
cp -r *.Rcheck $CIRCLE_ARTIFACTS/test_results
```

I hope that some of this information is useful if you're thinking about mixing R, continuous integration, and Docker. If not, at least when I start searching the internet for this information next time, at least this post will show up and remind me of what I used to know.
