---
title: Using R to Calculate Student Moblity
author: Jason P. Becker
email: jason.paul.becker@gmail.com
date: 2013-09-23
tags: education, data, rstats
status: draft
---

In a couple of previous posts, I outlined the importance of [documenting business rules for common education statistics](filename|documentation-of-business-rules-and-analysis.md) and described my take on how to [best calculate student mobility](new-calculation-for-student-mobility.md). In this post, I will be sharing two versions of R function I wrote to implement this mobility calculation, reviewing their different structure and methods to reveal how I achieved an order of magnitude speed up between the two versions. [^optimization] At the end of this post, I will propose several future routes for optimization that I believe should lead to the ability to handle millions of student records in seconds.

## Version 0: Where Do I Begin?

The first thing I tend to do is whiteboard the rules I want to use through careful consideration and constant referal back to real data sets. By staying grounded in the data, I am less likely to encounter unexpected situations during my quality control. It also makes it much easier to develop test data, since I seek out outlier records in actual data during the business rule process.

Developing test data is a key part of the development process. Without a compact, but sufficiently complex, set of data to try with a newly developed function, there is no way to know whether or not it does what I intend.

Recall the [business rules for mobility](filename|new-calculation-for-student-mobility.md) that I have proposed, all of which came out of this whiteboarding process:

1. Entering the data with an enroll date after the start of the year counts as one move.
2. Leaving the data with an exit date before the end of the year counts as one move.
3. Changing schools sometime during the year without a large gap in enrollment counts as one move.
4. Changing schools sometime during the year with a large gap in enrollment counts as two moves.
5. Adjacent enrollment records for the same student in the same school without a large gap in enrollment does not count as moving.

Test data needs to represent each of these situations so that I can confirm the function is properly implementing each rule.

Below is a copy of my test data. As an exercise, I recommend determining the number of "moves" each of these students should be credited with after applying the above stated business rules.

| Unique Student ID   | School Code   | Enrollment Date   | Exit Date    |  |
| ------------------- | ------------- | ----------------- | ------------ |  |
| 1000000             | 10101         | 2012-10-15        | 2012-11-15   |  |
| 1000000             | 10103         | 2012-01-03        | 2013-03-13   |  |
| 1000000             | 10103         | 2012-03-20        | 2013-05-13   |  |
| 1000001             | 10101         | 2012-09-01        | 2013-06-15   |  |
| 1000002             | 10102         | 2012-09-01        | 2013-01-23   |  |
| 1000003             | 10102         | 2012-09-15        | 2012-11-15   |  |
| 1000003             | 10102         | 2013-03-15        | 2013-06-15   |  |
| 1000004             | 10103         | 2013-03-15        | NA           |  |

## Version 1: A Naïve Implementation

Once I have developed business rules and a test data set, I like to quickly confirm that I can produce the desired results. That's particularly true when it comes to implementing a new, fairly complex business rules. My initial implementation of a new algorithm does not need to be efficient, easily understood, or maintainable. My goal is simply to follow my initial hunch on how to accomplish a task and get it working. Sometimes this *naïve implementation* turns out to be pretty close to my final implementation, but sometimes it can be quite far off. The main things I tend to improve with additional work are extensibility, readability, and performance.

In the case of this mobility calculation, I knew almost immediately that my initial approach was not going to have good performance characteristics. Here is a step by step discussion of Version 1.

### Function Declaration: Parameters

```r
moves_calc <- function(df, 
                       enrollby,
                       exitby,
                       gap=14,
                       sid='sid', 
                       schid='schid',
                       enroll_date='enroll_date',
                       exit_date='exit_date')){
```

I named my function `moves_calc()` to match the style of `age_calc()` which was submitted and accepted to the eeptools package. This new function has eight parameters.


`df`: a `data.frame` containing the required data to do the mobility calculation. 
 
`enrollby`: an atomic vector of type `character` or `Date` in the format `YYYY-MM-DD`. This parameter signifies the start of the school year. Students whose first enrollment is after this date will have an additional  `move` under the assumption that they enrolled somewhere prior to the first enrollment record in the data. This does not (and likely should not) match the actual first day of the school year.

`exitby`: an atomic vector of type `character` or `Date` in the format  `YYYY-MM-DD`. This parameter signifies the end of the school year. Students whose last exit is before this date will have an additional `move` under the assumption that they enrolled somewhere after this exit record that is excluded in the data. This date does not (and likely should not) match the actual last day of the school year.
 
`gap`: an atomic vector of type `numeric` that signifies how long a gap must exist between student records to record an additional move for that student under the assumption that they enrolled somewhere in between the two records in the data that is not recorded.

`sid`: an atomic vector of type `character` that represents the name of the vector in `df` that contains the unique student identifier. The default value is `'sid'`.

`schid`: an atomic vector of type `character` that represents the name of the vector in `df` that contains the unique school identifier. The default value is `schid`.

`enroll_date`: an atomic vector of type `character` that represents the name of the vector in `df` that contains the enrollment date for each record. The default value is `enroll_date`.

`exit_date`: an atomic vector of type `character` that represents the name of the vector in `df` that contains the exit date for each record. The default value is `exit_date`.

Most of these parameters are about providing flexibility around the naming of attributes in the data set. Although I often write functions for my own work which accept `data.frames`, I can not help but to feel this is a bad practice. Assuming particular data attributes of the right name and type does not make for generalizable code. To make up for my shortcoming in this area, I have done my best to allow other users to enter whatever data column names they want, so long as they contain the right information to run the algorithm.

The next portion of the function loads some of the required packages and is common to many of my custom functions:

```r
if("data.table" %in% rownames(installed.packages()) == FALSE){
    install.packages("data.table")
  } 
require(data.table)

if("plyr" %in% rownames(installed.packages()) == FALSE){
    install.packages("plyr")
  } 
require(plyr)
```
### Type Checking and Programmatic Defaults

Next, I do extensive type-checking to make sure that `df` is structured the way I expect it to be in order to run the algorithm. I do my best to supply humane `warning()` and `stop()` messages when things go wrong, and in some cases, set default values that may help the function run even if function is not called properly.

```r
if (!inherits(df[[enroll_date]], "Date") | !inherits(df[[exit_date]], "Date"))
    stop("Both enroll_date and exit_date must be Date objects")
```

The `enroll_date` and `exit_date` both have to be `Date` objects. I could have attempted to coerce those vectors into `Date` types using `as.Date()`, but I would rather not assume something like the date format. Since `enroll_date` and `exit_date` are the most critical attributes of each student, the function will `stop()` if they are the incorrect type, informing the analyst to clean up the data.

```r
if(missing(enrollby)){
   enrollby <- as.Date(paste(year(min(df$enroll_date, na.rm=TRUE)),
                              '-09-15', sep=''), format='%Y-%m-%d')
}else{
  if(is.na(as.Date(enrollby, format="%Y-%m-%d"))){
     enrollby <- as.Date(paste(year(min(df$enroll_date, na.rm=TRUE)),
                               '-09-15', sep=''), format='%Y-%m-%d'
     warning(paste("enrollby must be a string with format %Y-%m-%d,",
                   "defaulting to", 
                   enrollby, sep=' '))
  }else{
    enrollby <- as.Date(enrollby, format="%Y-%m-%d")
  }
}
if(missing(exitby)){
  exitby <- as.Date(paste(year(max(df$exit_date, na.rm=TRUE)),
                          '-06-01', sep=''), format='%Y-%m-%d')
}else{
  if(is.na(as.Date(exitby, format="%Y-%m-%d"))){
    exitby <- as.Date(paste(year(max(df$exit_date, na.rm=TRUE)),
                              '-06-01', sep=''), format='%Y-%m-%d')
    warning(paste("exitby must be a string with format %Y-%m-%d,",
                  "defaulting to", 
                  exitby, sep=' '))
  }else{
    exitby <- as.Date(exitby, format="%Y-%m-%d")
  }
}
if(!is.numeric(gap)){
  gap <- 14
  warning("gap was not a number, defaulting to 14 days")
}
```

For maximum flexibility, I have parameterized the `enrollby`, `exitby`, and `gap` used by the algorithm to determine student moves. An astute observer of the function declaration may have noticed I did not set default values for `enrollby` or `exitby`. This is because these dates are naturally going to be different which each year of data. As a result, I want to enforce their explicit declaration. 

However, we all make mistakes. So when I check to see if `enrollby` or `exitby` are `missing()`, I do not stop the function if it returns `TRUE`. Instead, I set the value `enrollby` to September 15 in the year that matches the minimum (first) enrollment record and `exitby` to June 1 in the year that matches the maximum (last) exit record. I then pop off a `warning()` that informs the user the expected values for each parameter and what values I have defaulted them to. I chose to use `warning()` because many R users set their environment to halt at `warnings()`. They are generally not good and should be pursued and fixed. No one should depend upon the defaulting process I use in the function. But the defaults that can be determined programmatically are sensible enough that I did not feel the need to always halt the function in its place.

I also check to see if `gap` is, in fact, defined as a number. If not, I also throw a `warning()` after setting `gap` equal to default value of `14`.

Is this all of the type and error-checking I could have included? Probably not, but I think this represents a very sensible set that make this function much more generalizable outside of my coding environment. This kind of checking may be overkill for a project that is worked on independently and with a single data set, but colleagues, including your future self, will likely be thankful for their inclusion if any of your code is to be reused.

### Initializing the Results

```r
output <- data.frame(id = as.character(unique(df[[sid]])),
                     moves = vector(mode = 'numeric', 
                                    length = length(unique(df[[sid]]))))
output <- data.table(output, key='id')
df <- arrange(df, sid, enroll_date)
```
My naïve implementation uses a lot of `for` loops, a no-no when it comes to R performance. One way to make `for` loops a lot worse, and this is true in any language, is to reassign a variable within the loop. This means that each iteration has the overhead of creating and assigning that object. Especially when we are building up results for each observation, it is silly to do this. We know exactly how big the data will be and therefore only need to create the object once. We can then assign a much smaller part of that object (in this case, one value in a vector) rather than the whole object (a honking `data.table`).

Our `output` object is what the function returns. It is a simple `data.table` containing all of the unique student identifiers and the number of `moves` recorded for each student.

The last line in this code chunk ensures that the data are arranged by the unique student identifier and enrollment date. This is key since the `for` loops assume that they are traversing a student's record sequentially.

### Business Rule 1: The Latecomer
```r
for(i in 1:(length(df[[sid]])-1)){
  if(i>1 && df[sid][i,]!=df[sid][(i-1),]){
    if(df[['enroll_date']][i]>enrollby){
      output[as.character(df[[sid]][i]), moves:=moves+1L]
    }
  }else if(i==1){
    if(df[['enroll_date']][i]>enrollby){
    output[as.character(df[[sid]][i]), moves:=moves+1L]
    }
  }
``` 
The first bit of logic checks if `sid` in row `i` is not equal to the `sid` in the `i-1` row. In other words, is this the first time we are observing this student? If it is, then row `i` is the first observation for that student and therefore has the minimum enrollment date. The `enroll_date` is checked against `enrollby`. When `enroll_date` is after `enrollby`, then the `moves` attribute for that `sid` is incremented by 1. [^increments]

Now, I didn't really mention the conditional that `i>1`. This is needed because there is no `i-1` observation for the very first row of the `data.table`. Therefore, `i==1` is a special case where we once again perform the same check for `enroll_date` and `enrollby`. The `i>1` condition is before the `&&` operator, which ensures the statement after the `&&` is not evaluated when the first conditional is `FALSE`. This avoids an "out of bounds"-type error where R tries to check `df[0]`.

### Business Rule 5: The Feint
Yeah, yeah-- the business rule list above doesn't match the order of my function. That's ok. Remember, sometimes giving instructions to a computer does not follow the way you would organize instructions for humans.

Remember, the function is traversing through our `data.frame` one row at a time. First I checked to see if the function is at the first record for a particular student. Now I check to see if there are any records after the current record.

```r
  if(df[sid][i,]==df[sid][(i+1),]){
    if(as.numeric(difftime(df[['enroll_date']][i+1], 
                           df[['exit_date']][i], units='days')) < gap &
       df[schid][(i+1),]==df[schid][i,]){
        next
    }else if ...
```
For the case where the `i+1` record has the same `sid`, then the `enroll_date` of `i+1` is subtracted from the `exit_date` of `i` and checked against `gap`. If it is both less than `gap` and the `schid` of `i+1` is the same as `i`, then `next`, which basically breaks out of this conditional and moves on without altering moves. In other words, students who are in the same school with only a few days between the time they exited are not counting has having moved.

The `...` above is not the special `...` in R, rather, I'm continuing that line below.

### Business Rule 3: The Smooth Mover

```r
  }else if(as.numeric(difftime(df[['enroll_date']][i+1], 
                               df[['exit_date']][i], 
                               units='days')) < gap){
    output[as.character(df[[sid]][i]), moves:=moves+1L] 
  }else{ ...
```

Here we have the simple case where a student has moved to another school (recall, this is still within the `if` conditional where the next record is the same student as the current record) with a very short period of time between the `exit_date` at the current record and the `enroll_date` of the next record. This is considered a "seamless" move from one school to another, and therefore that student's moves are incremented by 1.

### Business Rule 4: The Long Hop
Our final scenario for a student moving between schools is when the gap between the `exit_date` at the `i` school and the `enroll_date` at the `i+1` school is large, defined as `> gap`. In this scenario, the assumption is that the student moved to a jurisdiction outside of the data set, such as out of district for district-level data or out of state for state level data, and enrolled in at least one school not present in their enrollment record. The result is these students receive `2 moves`-- one out from the `i` school to a missing school and one in to the `i+1` school from the missing school.

The code looks like this (again a repeat from the `else{...` above which was not using the `...` character):

```r
  }else{
    output[as.character(df[[sid]][i]), moves:=moves+2L] 
  }
}else...
```
This ends with a `}` which closes the `if` conditional that checked if the `i+1` student was the same as the `i` student, leaving only one more business rule to check.

### Business Rule 2: The Early Summer
```r
}else{
  if(is.na(df[['exit_date']][i])){
    next
  }else if(df[['exit_date']][i] < exitby){
        output[as.character(df[[sid]][i]), moves:=moves+1L]
  }
}
```
Recall that this `else` block is only called if `sid` of the `i+1` record is not the same as `i`. This means that this is the final entry for a particular student. First, I check to see if that student has a missing `exit_date` and if so charges no `move` to the student implementing the `next` statement to break out of this iteration of the loop. Students never have missing `enroll_date` for any of the data I have seen over 8 years. This is because most systems minimally autogenerate the `enroll_date` for the current date when a student first enters a student information system. However, sometimes districts forget to properly exit a student and are unable to supply an accurate `exit_date`. In a very small number of cases I have seen these missing dates. So I do not want the function to fail in this scenario. My solution here was simply to break out and move to the next iteration of the loop.

Finally, I apply the last rule, which compares the final `exit_date` for a student to `exitby`, incrementing `moves` if the student left prior to the end of the year and likely enrolled elsewhere before the summer.

The last step is to close the `for` loop and `return` our result:
```r
  }
  return(output)
}
```

## Version 2: 10x Speed And More Readable
The second version of this code is vastly quicker.

The opening portion of the code, including the error checking is essentially a repeat of before, as is the initialization of the output.

```r
moves_calc <- function(df, 
                       enrollby,
                       exitby,
                       gap=14,
                       sid='sasid', 
                       schid='schno',
                       enroll_date='enroll_date',
                       exit_date='exit_date'){
  if("data.table" %in% rownames(installed.packages()) == FALSE){
    install.packages("data.table")
  } 
  require(data.table)
  if (!inherits(df[[enroll_date]], "Date") | !inherits(df[[exit_date]], "Date"))
      stop("Both enroll_date and exit_date must be Date objects")
  if(missing(enrollby)){
    enrollby <- as.Date(paste(year(min(df[[enroll_date]], na.rm=TRUE)),
                              '-09-15', sep=''), format='%Y-%m-%d')
  }else{
    if(is.na(as.Date(enrollby, format="%Y-%m-%d"))){
      enrollby <- as.Date(paste(year(min(df[[enroll_date]], na.rm=TRUE)),
                                '-09-15', sep=''), format='%Y-%m-%d')
      warning(paste("enrollby must be a string with format %Y-%m-%d,",
                    "defaulting to", 
                    enrollby, sep=' '))
    }else{
      enrollby <- as.Date(enrollby, format="%Y-%m-%d")
    }
  }
  if(missing(exitby)){
    exitby <- as.Date(paste(year(max(df[[exit_date]], na.rm=TRUE)),
                            '-06-01', sep=''), format='%Y-%m-%d')
  }else{
    if(is.na(as.Date(exitby, format="%Y-%m-%d"))){
      exitby <- as.Date(paste(year(max(df[[exit_date]], na.rm=TRUE)),
                                '-06-01', sep=''), format='%Y-%m-%d')
      warning(paste("exitby must be a string with format %Y-%m-%d,",
                    "defaulting to", 
                    exitby, sep=' '))
    }else{
      exitby <- as.Date(exitby, format="%Y-%m-%d")
    }
  }
  if(!is.numeric(gap)){
    gap <- 14
    warning("gap was not a number, defaulting to 14 days")
  }
  output <- data.frame(id = as.character(unique(df[[sid]])),
                       moves = vector(mode = 'numeric', 
                                      length = length(unique(df[[sid]]))))
```                                      

Where things start to get interesting is in the calculation of the number of student moves.

### Handling Missing Data
One of the clever bits of code I forgot about when I initially tried to refactor Version 1 appears under "Business Rule 2: The Early Summer". When the `exit_date` is missing, this code simply breaks out of the loop:

```r
  if(is.na(df[['exit_date']][i])){
    next
```

Because the new code will not be utilizing `for` loops or really any more of the basic control flow, I had to device a different way to treat missing data. The steps to apply the business rules that I present below will fail spectacularly with missing data.

So the first thing that I do is select the students who have missing data, assign the `moves` in the `output` to `NA`, and then subset the data to exclude these students.

```r
incomplete <- df[!complete.cases(df[, c(enroll_date, exit_date)]), ]
if(dim(incomplete)[1]>0){
  output[which(output[['id']] %in% incomplete[[sid]]),][['moves']] <- NA
}
output <- data.table(output, key='id')
df <- df[complete.cases(df[, c(enroll_date, exit_date)]), ]
dt <- data.table(df, key=sid)
```

### Woe with `data.table`
Now with the data complete and in a `data.table`, I have to do a little bit of work to assist with my frustrations with `data.table`. Because `data.table` does a lot of work with the `[` operator, I find it very challenging to use a string argument to reference a column in the data. So I just gave up and internally rename these attributes.

```r
dt$sasid <- as.factor(as.character(dt$sasid))
setnames(dt, names(dt)[which(names(dt) %in% enroll_date)], "enroll_date")
setnames(dt, names(dt)[which(names(dt) %in% exit_date)], "exit_date")
```

### Magic with `data.table`: Business Rules 1 and 2 in two lines each
Despite by challenges with the way that `data.table` re-imagines `[`, it does allow for clear, simple syntax for complex processes. Gone are the `for` loops and conditional blocks. How does `data.table` allow me to quickly identified whether or not a students first or last enrollment are before or after my cutoffs?

```r
first <- dt[, list(enroll_date=min(enroll_date)), by=sid]
output[id %in% first[enroll_date>enrollby][[sid]], moves:=moves+1L]
last <- dt[, list(exit_date=max(exit_date)), by=sid]  
output[id %in% last[exit_date<exitby][[sid]], moves:=moves+1L]
```
Line 1 creates a `data.table` with the student identifier and a new `enroll_date` column that is equal to the minimum `enroll_date` for that student.

The second line is very challenging to parse if you've never used `data.table`. The first argument for `[` in `data.table` is a subset/select function. In this case,

```r
id %in% first[enroll_date>enrollby][[sid]]
```

means,

>Select the rows in `first` where the `enroll_date` attribute (which was previously assigned as the minimum `enroll_date`) is less than the global function argument `enrollby` and check if the `id` of `output` is in the `sid` vector.

So `output` is being subset to only include those records that meet that condition, in other words, the students who should have a move because they entered the school year late. 

The second argument of `[` for `data.tables` is explained in this footnote [^increments] if you're not familiar with it.

### Recursion. Which is also known as recursion.
The logic for Business Rules 3-5 are substantially more complex. At first it was not plainly obvious how to avoid a slow `for` loop for this process. Each of the rules on switching schools requires an awareness of context-- how does one record of a student compare to the very next record for that student? 

The breakthrough was thinking back to my single semester of computer science and the concept of recursion. I created a new function inside of this function that can count how many moves are associated with a set of enrollment records, ignoring the considerations in Business Rules 1 and 2. Here's my solution. I decided to include inline comments because I think it's easier to understand that way.

```r
school_switch <- function(dt, x=0){
  # This function accepts a data.table dt and initializes the output to 0.
    if(dim(dt)[1]<2){
    # When there is only one enrollment record, there are no school changes to
    # apply rules 3-5. Therefore, the function returns the value of x. If the
    # initial data.table contains a student with just one enrollment record, 
    # this function will return 0 since we initialize x as 0.
      return(x)
    }else{
      # More than one record, find the minimum exit_date which is the "first"
      # record
      exit <- min(dt[, exit_date])
      # Find out which school the "first" record was at.
      exit_school <- dt[exit_date==exit][[schid]]
      # Select which rows come after the "first" record and only keep them
      # in the data.table
      rows <- dt[, enroll_date] > exit
      dt <- dt[rows,]
      # Find the minimum enrollment date in the subsetted table. This is the
      # enrollment that follows the identified exit record
      enroll <- min(dt[, enroll_date])
      # Find the school associated with that enrollment date
      enroll_school <- dt[enroll_date==enroll][[schid]]
      # When the difference between the enrollment and exit dates are less than
      # the gap and the schools are the same, there are no moves. We assign y,
      # our count of moves to x, whatever the number of moves were in this call
      # of school_switch
      if(difftime(min(dt[, enroll_date], na.rm=TRUE), exit) < gap &
         exit_school==enroll_school){
        y = x
      # When the difference in days is less than the gap (and the schools are
      # different), then our number of moves are incremented by 1.
      }else if(difftime(min(dt[, enroll_date], na.rm=TRUE), exit) < gap){
        y = x + 1L
      }else{
      # Whenever the dates are separated by more than the gap, regardless of which
      # school a student is enrolled in at either point, we increment by two.
        y = x + 2L
      }
      # Explained below outside of the code block.
      school_switch(dt, y)
    }
  }
```

The recursive aspect of this method is calling `school_switch` within `school_switch` once the function reaches its end. Because I subset out the row with the minimum `exit_date`, the `data.table` has one row processed with each iteration of `school_switch`. By passing the number of moves, `y` back into `school_switch`, I am "saving" my work from each iteration. Only when a single row remains for a particular student does the function `return` a value.

This function is called using `data.table`'s special `.SD` object, which accesses the subset of the full `data.table` when using the `by` argument.

```r
dt[, moves:= school_switch(.SD), by=sid]
```

This calls `school_switch` after splitting the `data.table` by each `sid` and then stitches the work back together, split-apply-combine style, resulting in a `data.table` with a set of `moves` per student identifier. With a little bit of clean up, I can simply add these moves to those recorded earlier in `output` based on Business Rules 1 and 2.

```r
  dt <- dt[,list(switches=unique(moves)), by=sid]
  output[dt, moves:=moves+switches]
  return(output)
}
```

## Quick and Dirty `system.time`

[^optimization]:  On a mid-2012 Macbook Air, the current mobility calculation is very effective with tens of thousands of student records and practical for use in the low-hundreds of thousands of records range.

[^increments]: I thought I was going to use `data.table` for some of its speedier features as I wrote this initial function. I didn't in this go (though I do in Version 2). However, I do find the `data.table` syntax for assigning values to be really convenient, particularly the `:=` operator which is common in several other languages. In `data.table`, the syntax `dt[,name:=value]` assigns `value` to an exist (or new) column called `name`. Because of the need `select` operator in `data.table`, I can just use `dt[id,moves:=moves+1L]` to select only the rows where the table `key`, in this case `sid`, matches `id`, and then increment `moves`. Nice.