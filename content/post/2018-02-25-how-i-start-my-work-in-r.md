---
title: How I start my work in R
author: Jason Becker
date: '2018-02-25'
slug: how-i-start-my-work-in-r
categories: []
tags:
  - rstats
---

## Ideation

At the start of every project, there's a blinking cursor.

Actually, that's almost never true for me. If I start staring at a blinking cursor, I'm almost guaranteed to keep looking at a blinking cursor, often for hours. The real work almost always starts weeks or months before I actually type anything. I think it's easy for folks for whom their ultimate product is a bunch of code or an analysis report to undervalue that our work is creative. Writing a package or doing data analysis is still fundamentally creative work. We're in the business of using computers to generate evidence to support insights into how things work. If all there was to it was a procedural search through models, then this would all have been automated already.

When I think, "How do I wish I could write my code to solves this problem?" I know that I am getting a great idea for a package. Often, I'm staring at a function I just wrote to make my work easier and start to think "This is still too specific to my work." I can start to see the steps of generalizing my solution a little bit further. Then I start to see how further generalization of this function will require supporting scaffolding and steps that would have been valuable. I start to think through what other problems exist in data sets unlike my own or future data I expect to work with. And I ask myself again and again, "How do I wish I could write my code to solves this problem?"

Data analysis almost always starts with an existing hypothesis of interest. My guiding thoughts are "What do I need to know to understand this data? What kind of evidence would convince me?" Sometimes the first thoughts are how I would model the data, but most of the time I begin to picture 2-3 data visualizations that would present the main results of my work. Nothing I produce is meant to convince an academic audience or even other data professionals of my results. Everything I make is about delivering value back to the folks who generate the data I use in the first place. I am trying to deliver value back to organizations by using data on their current work to inform future work. So my hypotheses are "What decisions are they making with this data? What decisions are they making without this data that should be informed by it? How can I analyze and present results to influence and improve both of these processes?" The answer to that is rarely a table of model specifications. But even if your audience is one of peer technical experts, I think it's valuable to start with what someone should learn from your analysis and how can you present that most clearly and convincingly to that audience.

Don't rush this process. If you don't know where you're heading, it's hard to do a good job getting there. That doesn't mean that once I *do* start writing code, I always know exactly what I am going to do. But I find it far easier to design the right data product if I have a few guiding light ideas of what I want to accomplish from the start.

## Design

The next step is not writing code, but it may still happen in your code editor of choice. Once I have some concept of where I am headed, I start to write out my ideas for the project in a `README.md` in a new RStudio project. Now is the time to describe who your work is for and how you expect them to interact with that work. Similar to something like a "project charter", your README should talk about what the goals are for the project, what form the project will take (a package? an Rmd -> pdf report? a website? a model to be deployed into production for use in this part of the application?), and who the audience is for the end product. If you're working with collaborators, this is a great way to level-set and build common understanding. If you're not working with collaborators, this is a great way to articulate the scope of the project and hold yourself accountable to that scope. It also is helpful for communicating to managers, mentors, and others who may eventually interact with your work even if they will be less involved at the inception.

For a package, I would write out the primary functions you expect someone to interact with and how those functions interact with each other. Use your first README to specify that this package will have functions to get data from a source, process that data into a more easy to use format, validate that data prior to analysis, and produce common descriptive statistics and visuals that you'd want to produce before using that data set for something more complex. That's just an example, but now you have the skeletons for your first functions: `fetch`, `transform`, `validate`, and `describe`. Maybe each of those functions will need multiple variants. Maybe `validate` will get folded into a step at `fetch`. You're not guaranteed to get this stuff right from the start, but you're far more likely to design a clear, clean API made with composable functions that each help with one part of the process if you think this through before writing your first function. Like I said earlier, I often think of writing a package when I look at one of my existing functions and realize I can generalize it further. Who among us hasn't written a monster function that does all of the work of `fetch`, `transform`, `validate`, and `describe` all at once?

## Design Your Data

I always write up a data model at the start of a new project. What are the main data entities I'll be working with? What properties do I expect they will have? How do I expect them to relate to one another? Even when writing a package, I want to think about "What are the ideal inputs and outputs for this function?" 

Importantly, when what I have in mind is a visualization, I actually fake data and get things working in `ggplot` or `highcharter`, depending on what the final product will be. Why? I want to make sure the visual is compelling with a fairly realistic set of data. I also want to know how to organize my data to make that visualization easy to achieve. It helps me to define the output of my other work far more clearly.

In many cases, I want to store my data in a database, so I want to start with a simple design of the tables I expect to have, along with validity and referential constraints I want to apply. If I understand what data I will have, how it is related, what are valid values, and how and where I expect the data set to expand, I find it far easier to write useful functions and reproducible work. I think this is perhaps the most unique thing I do and it comes from spending a lot of time thinking about data architectures in general. If I'm analyzing school district data, I want to understand what district level properties and measures I'll have, what school properties and measures I'll have, what student property and measures I'll have, what teacher properties and measures I'll have, etc. Even if the analysis is coming from or will ultimately produce a single, flattened out, big rectangle of data, I crave [normality](https://en.wikipedia.org/wiki/Database_normalization).

## Make Files

So now my README defines a purpose, it talks about how I expect someone to interact with my code or what outputs they should expect from the analysis, and has a description of the data to be used and how it's organized. Only then do I start to write `.R` files in my `R/` directory. Even then I'm probably not writing code but instead pseudocode outlines of how I want things to work, or fake example data to be used later. I'm not much of a [test-driven development](https://en.wikipedia.org/wiki/Test-driven_development) person, but the first code I write looks a lot like test data and basic functions that are meeting some test assertions. Here's some small bit of data, can I pass it into this function and get what I want out? What if I create this failure state? What if I can't assume column are right?

Writing code is far more fun when I know where I am heading. So that's how I start my work.
