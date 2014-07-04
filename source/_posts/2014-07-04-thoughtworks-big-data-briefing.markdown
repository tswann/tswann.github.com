---
layout: post
title: "Thoughtworks Big Data Briefing"
date: 2014-07-04 16:00
comments: true
categories: [Big Data, Hadoop, Agile]
---
Last night I arrived in Manchester to attend [Thoughtworks](http://www.thoughtworks.com/events) quarterly tech briefing. What dragged me all the way from Belfast? David Elliman speaking on the subject of Big Data.

It was well worth the trip. There was a serious amount of content packed into the two hours.

Below the jump you can find my notes and additional ruminating.

<!-- more -->
The 'Big' In Big Data misses the point
--------------------------------------
Taking the title of a previous [blog post](http://www.thoughtworks.com/insights/blog/big-big-data-misses-point) by David highlighting the vagueness of the term as a starting point, the first 'section' of the talk focused on the definition itself, the underlying problem and the opportunity.

Big Data seems to be a term universally disliked, at least by people who like clear definitions and dislike buzzword bingo - largely engineers I guess.

We started with the traditional definition of the three V's, that is:

* Volume: amount of stuff
* Velocity: high rate of new stuff arriving
* Variety: stuff from many sources

You can start appending additional V words which aren't actually measures of magnitude to this definition (veracity, verisimilitude, I dunno...) At which point we've clearly given up on the definition being useful and entered [Monty Python](https://www.youtube.com/watch?v=PPN3KTtrnZM&feature=kp) territory.

However, the 'V' which _is_ of primary interest to us is indeed none of these, but "Value".

This point is addressed shortly, but first a hail of bullets for part 1 of the session:

* The Big Data opportunity cuts across a wide spectrum of industries, but principally:

    * Retail
    * Finance
    * Healthcare

* The Internet Of Things - that is, connected autonomous devices - is set to explode. 50 billion connected devices by 2020.

* [Wikibon](http://wikibon.org/) forecast of Big Data industry growth:
    * In 2012: $5.1 billion
    * In 2017: $53.4 billion

The last time I saw this 2017 figure wheeled out it was more like $30b, so either we're getting worse at forecasting or the growth really is picking up considerable momentum - something David has noticed as well and commented on.

Setting Out
-----------

So back to that 'V' of value. The key takeaway from the next part of the talk was that normal Agile practice applies in any kind of analytics delivery.

* Start small
* Create value
* Iterate and build on it

Immediately reaching for the Hadoop cluster, and the big scaling infrastructure toolset is probably not where you want to start out. Build small, simple models frequently.

If you can define a concrete problem and start delivering immediate value then it becomes easier for an organisation to take the next step up in scale.

What 'Value' we're talking about in this context was left unmentioned, probably because there is no general answer.

Though at the Big Data/Data Science end of the analytics scale, we probably mean increased efficiency, MI, automation.

A few enumerated steps, to move us towards a better understanding of what we need to do:

1. What are your analysis goals?
--------------------------------
--------------------------------

There are four broad categories of analytics that an organisation can choose to implement:

* Descriptive _(What happened?)_
* Diagnostic _(Why did it happen?)_
* Predictive _(What is likely to happen?)_
* Prescriptive _(How can we make things happen?)_

![Analysis Goals](https://dl.dropboxusercontent.com/u/47685018/img/analytics%20goals.png "Analysis Goals")

The question we have to answer is where does the requirements of any given organisation fall on the chart above?

Value and complexity both increase for each distinct flavour of analytics, and we can expect a greater level of differentiation from what our competitors are doing as we move further to the top right.

As an aside, this comes back to an argument around what sets Data Science appart from traditional statistical modeling.

One line of thought is that when we step into building predictive systems, we have moved away from vanilla statistics and into the world of Data Science.

We're considering a system which not only ingests data for analysis, but also outputs data which is in turn fed back into the model.

A nice quote from [Cathy O'Neil](http://en.wikipedia.org/wiki/Cathy_O'Neil) in her book "Doing Data Science":

>This is very different from predicting the weather, say, where your model doesn’t influence the outcome at all. For example, you might predict it will rain next week, and unless you have some powers we don’t know about, you’re not going to cause it to rain.

Behaviour (of users) influences the system and the system in turn influences behaviour. This is the concept of the "data product", be it Netflix recommendations or a smart electricity grid.


2. Start With Standard Tools
----------------------------
----------------------------

The bulk of innovation in analytics and Big Data tooling has been driven by open source.

_Embrace freely available, common tools._

In line with the goal to start small and build, most of us already know the type of tools which are useful for cleaning and working with structured and semi-structured data. Tools which have stood the test of time:

* The shell: grep, sed, awk. There's not much you can't do with text using this toolset alone.

* [R](http://www.r-project.org/) - Statistical programming language, and you may be surprised to learn, the 2nd most popular data analysis tool after SQL

* [Python](https://www.python.org/) - Pythons library support for mathematical and scientific functionality makes it a popular choice in this domain

* [D3.js](http://d3js.org/) - powerful JavaScript library for data visualisation

There is an interesting 80/20 rule when it comes to analytics activities:

* 80% of effort is data cleaning
* 20% is doing something with it

3. Use The Right Structure
--------------------------
--------------------------

Remember [Knuth](http://www-cs-faculty.stanford.edu/~uno/). Picking the correct data structure(s) is a massive factor in framing and driving your analytics exercise. Consider how you currently model data and ask how you could do it differently.

The example given was [Neo4j](http://neo4j.com/) and graphing relationships between employees in a global organisational structure.

The data structure maps to the problem domain. There is no 'analytics' here as such, but by applying a structure to the data which fits our goal of mapping relationships - the choice of structure itself gets us 90% of the way. To do the same thing in a traditional RDBMS or key/value store would require a lot of hoop jumping.

Scaling Out
-----------

The next step - and the "big" challenge - is to take our ideas and models which have been based on a statistical sampling and prove them at scale.

There is an impedence mismatch here between what works for a sample, and what will work for a 'Big Data' set. I've seen this in the development of Hadoop jobs, which I'll come back to.

The Cloud is our friend. The Cloud affords us the possibility of on-demand access to compute resources that would otherwise prove to be a prohibitive cost. An interesting figure shown was that 47% of cloud vendors revenue comes from servicing the needs of Big Data operations. The two things are conceptually a snug fit.

At the 'Big Data' scale, infrastructure is an intrisic part of the question. Hadoop was first and foremost the response to an engineering problem.

Parallelism is hard. Nework programming is hard. The goal of Hadoop is to abstract these concerns away from the user and allow them to concentrate on the challenge of the analytics problem itself, and not get bogged down in failure modes and plumbing.

To quote from Eric Sammer's excellent book [Hadoop Operations](http://shop.oreilly.com/product/0636920025085.do), 25% of the physical storage space in a Hadoop cluster should be reserved for intermediate data. That's all the throw-away results which MapReduce generates as it gets to the _real_ answer which you want to come out the other side. Big Data analysis itself generates an awful lot of data.

In a massively scaled system, even a small amount of latency can be amplified exponentially. That could be the act of boxing/unboxing Java types unnecessarily in a MapReduce job or it could be shuffling data across the network without any form of compression.

These are examples of the type of problem that can arise from taking a solution that works on a sample and bringing it to the big data domain without considering the scale issue.

I guess you could draw a parallel with the fallacies of network programming - if bandwidth isn't infinite then neither is memory!

Architecture
------------

The final section of the talk was reserved for the question of Big Data architecture.

[The Lambda Architecture](http://lambda-architecture.net/) is a term used for describing an architecture that tries to consolidate the two primary types of processing that can be brought to bear on big data sets; that is batch and real-time.

MapReduce is fundamentally a batch processing system geared towards analysing whole data sets - so historical, complete picture kind of analysis.

The need for real-time turnaround has led to the development of event processing frameworks like Apache Storm, Spark and alternatives to the traditional abstraction layers on top of MapReduce, Hive and Pig. Alternatives like Cloudera's Impala.

One slide which was very funny was a comparison of the Hadoop ecosystem and vendor landscape in 2011 and what it looks like now - that is, a fine grained mosaic of logos which you can barely read. The pace of change and engagement in this field at the moment is incredible - incredibly confusing!

We've already passed well into TL;DR territory here, so I may return with a seperate post on the Lambda architecture when I've read up and know a bit more.

And Finally
-----------

All in all a very informative and thought-provoking talk from [Thoughworks](http://www.thoughtworks.com/).

And we haven't even got to security, privacy, new frameworks like Google Cascading. This stuff fills days of many a conference, never mind an evening get together.

As for me, you'll see less CRM content and more on Hadoop on this blog over the coming months as I delve deeper into the Big Data and analytics landscape.
Hadoop and whichever of those other myriad platforms, tools and vendors land within my field of vision.

Exciting times.
