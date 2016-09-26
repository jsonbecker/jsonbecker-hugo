---
title: Algorithms Design and Analysis Part I
date: 2012-06-13
author: Jason
category: Education, Free Online Courses
summary: Day one of a MOOC I thought I would take. I enjoyed it, but never continued because of the time committment.
---

Tonight I started Coursera.org's [Algorithms: Design and Analysis Part I][]. This class should pick up right about where I left off my computer science education. I took [CS15][] as a sophomore in college but didn't have the time to take [CS16: Introduction to Algorithms and Data Structures][]. So, while it's been almost 6 years since I have formally taken a computer science class, it is time to continue my education.

I plan to write about once a week about my experience. This will serve both as an opportunity to work out ideas spurred by the course as well as a review of the growing area of free, online courses that started way back in 2002 with MIT's [OpenCourseWare][] and continues today with upshots [Udacity][] and [Coursera][], among [other players][]. Given the emphasis being placed on the potential for technology as disruptive to classroom teaching over the last 50 years, the topic seems worthy of some experiential learning by a budding young education researcher/wonk.

## Introduction and About the Course

The Introduction video was a bit scary. Although the content was simple, Professor Tim Roughgarden is a fast talker and he does seem to skip some of the small steps that really trip me up when learning math from lectures. For example, in discussing the first recursive method to $n$-digit multiplication, Professor Roughgarden suddenly throws in a $10^n$ and $10^{n/2}$ term that I just couldn't trace. I kept watching the video waiting for an explanation and pondering it in
my mind when a few minutes later it hit me; the two terms kept the place information lost when a number is split into its constituent digits [^constituent].

The About this Course video, however, provided some good advice I intend on following; although there will be no code written as a part of this course to be language neutral, I will be attempting to code each of the described algorithms on my own. Professor Roughgarden's assumption is that this is within the skills of students taking this class. Generally, I believe I am capable of achieving this in at least *some* language. Currently, I prefer to use R. This is not because R is best suited to this kind of work. Rather, it is because I am relatively new to R, and I think that learning to program some fundamental computational tasks will be good for learning the ins and outs of the language.

However, I think I may switch over to using Python later in the course. Why? Because I feel like learning Python and Udacity happens to have a [course up already to do just that][]. My hope is to incorporate free online learning into my routine just like I include reading dead-tree books, Google Reader, and mess around on Twitter. So while I can't swear that I'll actually start moving through these two courses (and two more I'm interested in starting June 25), I feel having complimentary, simultaneous course work will push me. Each class should reinforce the other and I should see the most benefit if I keep up with both.

Finally, this class is a big time commitment. The first week has 3.5 hours of lecture time allotted. A typical Brown class would meet for only about 2.5 hours a week (three 50 minute classes or two 80 minute classes). That means a lot of time, not including homework or spending time actually coding and implementing the introduced algorithms. Although some of this material is "optional" (about an hour), that's still pretty intimidating for a free, online, spare time class. Make no mistake, if time commitment is any indicator, this will be every bit as challenging (to actually learn) as a real college course that last this many weeks.

[Algorithms: Design and Analysis Part I]: http://www.algo-class.org
[CS15]: http://www.cs.brown.edu/courses/csci0150.html
[CS16: Introduction to Algorithms and Data Structures]: http://www.cs.brown.du/courses/csci0160html
[OpenCourseWare]: http://ocw.mit.edu/index.htm
[Udacity]: http://udacity.com/
[Coursera]: http://www.coursera.org
[other players]: http://www.codeacademy.com
[course up already to do just that]: http://www.udacity.com/view#Course/cs101/

[^constituent]: The algorithm called to split an $n$ digit number $x$ into two, $n/2$ digit numbers. What was unstated, but of course true, is that this transformation must result in an expression that was equal to $x$. Of course, $x=10^n * a+10^\frac{n}{2} * b$ , because the leading digit of $a$ must be in the $n^{th}$ place and the leading digit of $b$ must be in the $\frac{n}{2}^{th}$ place. Nothing about this is complex to me, but it was not obvious at the speed of conversation. I think working out an actual example of a 4-digit number multiplication, as Professor Roughgarden had with the "primitive" multiplication algorithm, would have made this far clearer.
