---
title: A New Calculation for Student Mobility
author: Jason P. Becker
email: jason.paul.becker@gmail.com
date: 2013-09-17
tags: education, data, rstats
summary: A brief discussion on the complexity of determining the number of schools a student has attended within a single school year using a minimal set of information.
---

How do we calculate student mobility? I am currently soliciting responses from other data professionals across the country. But when I needed to produce mobility numbers for some of my work a couple of months ago, I decided to develop a set of business rules without any exposure to how the federal government, states, or other existing systems define mobility. [^ignorance]

I am fairly proud of my work on mobility. This post will review how I defined student mobility. I am hopeful that it matches or bests current techniques for calculating the number of schools a student has attended. In my next post, I will share the first two major versions of my implementation of these mobility business rules in `R`. [^learningexperience] Together, these posts will represent the work I referred to in my previous post on the [importance of documenting business rules and sharing code](|filename|documentation-of-business-rules-and-analysis.md).

## The Rules
Working with district data presents a woefully incomplete picture of the education mobile students receive. Particularly in a state like Rhode Island, where our districts are only a few miles wide, there is substantial interdistrict mobility. When a student moves across district lines, their enrollment is not recorded in local district data. However, even with state level data, highly mobile students cross state lines and present incomplete data. A key consideration for calculating how many schools a student has attended in a particular year is capturing "missing" data sensibly.

The typical structure of enrollment records looks something like this:

| Unique Student ID | School Code | Enrollment Date | Exit Date ||
|-------------------|-------------|-----------------|------------|
| 1000000           | 10101       | 2012-09-01      | 2012-11-15 |
| 1000000           | 10103       | 2012-11-16      | 2013-06-15 |

A compound key for this data consists of the Unique Student ID, School Code, and Enrollment Date, meaning that each row must be a unique combination of these three factors. The data above shows a simple case of a student enrolling at the start of the school year, switching schools once with no gap in enrollment, and continuing at the new school until the end of the school year. For the purposes of mobility, I would define the above as having moved one time.

But it is easy to see how some very complex scenarios could quickly arise. What if student `1000000`'s record looked like this?

| Unique Student ID | School Code | Enrollment Date | Exit Date ||
|-------------------|-------------|-----------------|------------|
| 1000000           | 10101       | 2012-10-15      | 2012-11-15 |
| 1000000           | 10103       | 2013-01-03      | 2013-03-13 |
| 1000000           | 10103       | 2013-03-20      | 2013-05-13 |

There are several features that make it challenging to assign a number of "moves" to this student. First, the student does not enroll in school until October 15, 2012. This is nearly six weeks into the typical school year in the Northeastern United States. Should we assume that this student has enrolled in no school at all prior to October 15th or should we assume that the student was enrolled in a school that was outside of this district and therefore missing in the data? Next, we notice the enrollment gap between November 15, 2012 and January 3, 2013. Is it right to assume that the student has moved only once in this period of time with a gap of enrollment of over a month and a half? Then we notice that the student exited school `10103` on March 13, 2013 but was re-enrolled in the same school a week later on March 20, 2013. Has the student truly "moved" in this period? Lastly, the student exits the district on May 13, 2013 for the final time. This is nearly a month before the end of school. Has this student moved to a different school?

There is an element missing that most enrollment data has which can enrich our understanding of this student's record. All district collect an exit type, which explains if a student is leaving to enroll in another school within the district, another school in a different district in the same state, another school in a different state, a private school, etc. It also defines whether a student is dropping out, graduating, or has entered the juvenile justice system, for example. However, it has been my experience that this data is reported inconsistently and unreliably. Frequently a student will be reported as changing schools within the district without a subsequent enrollment record, or reported as leaving the district but enroll within the same district a few days later. Therefore, I think that we should try and infer the number of schools that a student has attended using soley the enrollment date, exit date, and school code for each student record. This data is far more reliable for a host of reasons, and, ultimately, provides us with all the information we need to make intelligent decisions.

My proposed set of business rules examines `school code`, `enrollment date`, and `exit date` against three parameters: `enrollment by`, `exit by`, and `gap`. 
Each students minimum enrollment date is compared to `enrollment by`. If that student entered the data set for the first time before the `enrollment by`, the assumption is that this record represents the first time the student enrolls in any school for that year, and therefore the student has 0 `moves`. If the student enrolls for the first time after `enrollment by`, then the record is considered the second school a student has attended and their `moves` attribute is incremented by `1`. Similarly, if a student's maximium `exit date` is after `exit by`, then this considered to be the student's last school enrolled in for the year and they are credited with `0` `moves`, but if `exit date` is prior to `exit by`, then that student's `moves` is incremented by `1`.

That takes care of the "ends", but what happens as students switch schools in the "middle"? I proposed that each `exit date` is compared to the subsequent `enrollment date`. If `enrollment date` occurs within `gap` days of the previous `exit date`, and the `school code` of enrollment is not the same as the `school code` of exit, then a student's `moves` are incremented by `1`. If the `school codes` are identical and the difference between dates is less than `gap`, then the student is said to have not moved at all. If the difference between the `enrollment date` and the previous `exit date` is greater than `gap`, then the student's `moves` is incremented by `2`, the assumption being that the student likely attended a different school between the two observations in the data.

Whereas calculating student mobility may have seemed a simple matter of counting the number of records in the enrollment file, clearly there is a level of complexity this would fail to capture.

Check back in a few days to see my next post where I will share my initial implementation of these business rules and how I achieved an 10x speed up with a massive code refactor.

[^ignorance]: My ignorance was intentional. It is good to stretch those brain muscles that think through sticky problems like developing business rules for a key statistic. I can't be sure that I have developed the most considered and complete set of rules for mobility, which is why I'm now soliciting other's views, but I am hopeful my solution is at least as good.

[^learningexperience]: I think showing my first two implementation of these business rules is an excellent opportunity to review several key design considerations when programming in `R`. From version 1 to version 2 I achieved a 10x speedup due to a complete refactor that avoided `for` loops, used `data.table`, and included some clever use of recursion.
