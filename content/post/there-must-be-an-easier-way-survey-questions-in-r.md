---
title: There must be an easier way... survey questions in R
date: 2012-08-22
author: Jason
category: data
tags: code, markdown, r, rstats, survey
---

So I have this great little custom function I've used when looking at survey data in R. I call this function `pull()`. The goal of `pull()` is to quickly produce frequency tables with n sizes from individual-level survey data.

Before using `pull()`, I create a big table that includes information about the survey questions I want to pull. The data are structured like this:

<table align="center">
</p>
<p>
<tbody>
</p>
<p>
<tr>
</p>
<p>
<td>
quest

</td>
</p>
<p>
<td>
survey

</td>
</p>
<p>
<td>
year

</td>
</p>
<p>
<td>
break

</td>
</p>
<p>
</tr>
</p>
<p>
<tr>
</p>
<p>
<td>
ss01985

</td>
</p>
<p>
<td>
elementary

</td>
</p>
<p>
<td>
2011\_12

</td>
</p>
<p>
<td>
schoolcode

</td>
</p>
<p>
</tr>
</p>
<p>
</tbody>
</p>
<p>
</table>
</p>

*   quest represents the question coding in the raw survey data.
*   survey is the name of the survey (in my case, the elementary school students, middle school students, high school students, parents, teachers, or administrators).
*   year is the year that the survey data are collected.
*   break is the ID I want to aggregate on like schoolcode or districtcode.

They key is that `paste(survey, year,sep='')` produces the name of the `data.frame` where I store the relevant survey data. Both quest and break are columns in the survey data.frame. Using a data.frame with this data allows me to apply through the rows and produce the table for all the relevant questions at once. `pull()` does the work of taking one row of this `data.frame` and producing the output that I'm looking for. I also use `pull()` one row at a time to save a data.frame that contains these data and do other things (like the visualizations in this post).

In some sense, `pull()` is really just a fancy version of `prop.table` that takes in passed paramaters and adds an "n" to each row and adding a "total" row. I feel as though there must be an implementation of an equivalent function in a popular package (or maybe even base) that I should be using rather than this technique. It would probably be more maintainable and easier for collaborators to work with this more common implementation, but I have no idea where to find it. So, please feel free to use the code below, but I'm actually hoping that someone will chime in and tell me I've wasted my time and I should just be using some function foo::bar.

P.S. This post is a great example of why I really need to change this blog to Markdown/R-flavored Markdown. All those inline references to functions, variables, or code should really be formatted in-line which the syntax highlighter plug-in used on this blog does not support. I'm nervous that using WP-Markdown plugin will botch formatting on older posts, so I may just need to setup a workflow where I pump out HTML from the Markdown and upload the posts from there. If anyone has experience with Markdown + Wordpress, advice is appreciated.


```r
pull <- function(rows){
  # Takes in a vector with all the information required to create crosstab with
  # percentages for a specific question for all schools.
  # Args:
  #  rows: Consists of a vector with four objects.
  #        quest: the question code from SurveyWorks
  #        level: the "level" of the survey, i.e.: elem, midd, high, teac, admn,
  #        pare, etc.
  #        year: the year the survey was administered, i.e. 2011_12
  #        sch_lea: the "break" indicator, i.e. schoolcode, districtcode, etc.
  # Returns:
  # A data.frame with a row for each "break", i.e. school, attributes for
  # each possible answer to quest, i.e. Agree and Disagree, and N size for each
  # break based on how many people responded to that question, not the survey as
  # a whole, i.e. 

  # Break each component of the vector rows into separate single-element vectors
  # for convenience and clarity.
  quest <- as.character(rows[1])
  survey <- as.character(rows[2])
  year  <- as.character(rows[3])
  break <- as.character(rows[4])
  data <- get(paste(level,year,sep=''))
  # Data is an alias for the data.frame described by level and year.
  # This alias reduces the number of "get" calls to speed up code and increase
  # clarity.
  results <- with(data,
                  dcast(data.frame(prop.table(table(data[[break]],
                                                    data[[quest]]),
                                              1))
                        ,Var1~Var2,value.var='Freq'))
  # Produces a table with the proportions for each response in wide format.
  n <- data.frame(Var1=rle(sort(
    subset(data, 
           is.na(data[[quest]])==F & is.na(data[[break]])==F)[[break]]))$values,
                  n=rle(sort(
                    subset(data,
                           is.na(data[[quest]])==F &
                             is.na(data[[break]])==F)[[break]]))$lengths)
  # Generates a data frame with each break element and the "length" of that break
  # element. rle counts the occurrences of a value in a vector in order. So first
  # you sort the vector so all common break values are adjacent then you use rle
  # to count their uninterupted appearance. The result is an rle object with 
  # two components: [[values]] which represent the values in the original, sorted
  # vector and [[length]] which is the count of their uninterupted repeated
  # appearance in that vector.
  results <- merge(results, n, by='Var1')
  # Combines N values with the results table.

  state <- data.frame(t(c(Var1='Rhode Island', 
                          prop.table(table(data[[quest]])),
                          n=dim(subset(data,is.na(data[[quest]])==F))[1])))
  names(state) <- names(results)
  for(i in 2:dim(state)[2]){
    state[,i] <- as.numeric(as.character(state[,i]))
  }
  # Because the state data.frame has only one row, R coerces to type factor.
  # If I rbind() a factor to a numeric attribute, R will coerce them both to
  # characters and refuses to convert back to type numeric.
  results <- rbind(results, state)
  results
}   
```

