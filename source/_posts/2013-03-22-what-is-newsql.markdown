---
layout: post
title: "What Is NewSQL?"
date: 2013-03-22 10:18
comments: true
categories: [NewSQL, VoltDB]
---
NewSQL is a term used to describe a variety of database which is even more new-fangled than NoSQL (yes, it wasn't a typo).

It's a very recent term (circa 2011) - and the group of distributed database techologies which it describes are similarly up-and-coming. I suppose the name is a rare case of calling a spade a spade in the tech world.

NewSQL challenges some of the assumptions and received wisdom around database architecture. Primarily that you have to make a choice between "schemaless and available" versus "relational and consistent".

The central claim which binds them all together is something approaching "You can have your cake and eat it." That is:

*   A Relational data model
*   ACID compliant
*   SQL interface
*   Capability for massive scale out
*   Designed to cope with the [Three Vs](http://www.datacenterknowledge.com/archives/2012/03/08/three-vs-of-big-data-volume-velocity-variety/)

<!-- more -->

Some examples of tech which currently falls under the elastically scalable RDBMS blanket:

*   [Google Spanner](http://static.googleusercontent.com/external_content/untrusted_dlcp/research.google.com/en//archive/spanner-osdi2012.pdf)
*   [NuoDB](http://www.nuodb.com/)
*   [JustOne](http://www.justonedb.com/)
*   [VoltDB](https://voltdb.com/)

A quick ReCAP
-------------

We know from CAP theorem that when designing a distributed system you have a choice to make between three qualities, of which you can pick only two:

![CAP](https://dl.dropbox.com/u/47685018/Blog/2013/03-22/CAP.png "CAP")

Item number one on the [8 fallacies of distributed computing](http://en.wikipedia.org/wiki/Fallacies_of_Distributed_Computing) is thus: _the network is reliable_.

When you combine that thought with CAP theorem it becomes apparent that partition tolerance is the obvious thing you _need_ to take into account when designing a distributed system. The network is inherently an untrustworthy knave.

In layman's terms, it goes something like this: 

Say I have two machines on which I have a replicated set of data connected by a cable. If I cut that cable with a pair of scissors - I've just partitioned my network. How will it cope with this? 

Clients can write and request data on either machine, so how will it handle those requests? We can guarantee availability and respond with data which is potentially out of date. If favouring consistency, then we have to respond with an error (neither machine can know if it's the most up-to-date since we chopped the link between them).

OK this is over-simplified, but it's the basic core of what CAP theory states.

So how can NewSQL databases promise it all? How can you have ACID behaviour coupled with the high-availability of a NoSQL-like?

Well to a certain degree it seems some partition tolerance must be sacrificed in order to achieve this best of both words. Perhaps you're not going to get the potent reliability of a [Hadoop](http://hadoop.apache.org/), but you'd first need to ask 'Do I need it?'. The NewSQL architectures are designed for high reliability, in practice that may well be high enough for your own needs.

VoltDB
------

VoltDB is somewhat distinct from the other contenders in this area by being an in-memory database.

I thought it looked like an interesting starting point at which to enter the NewSQL fray - and luckily they have a very handy VMWare dev setup available as a free download on their website. It comes in at about 1.4GB:

Head to [http://voltdb.com/community/downloads.php](http://voltdb.com/community/downloads.php) and you can find the VMWare distribution under the Community Edition.

This is a well put together demo.

![Volt Demo Dashboard](https://dl.dropbox.com/u/47685018/Blog/2013/03-22/Volt%20Dashboard.jpg "Volt Demo Dashboard")

The VM is running Ubuntu x64 and on loading it up, hit the 'Click here to Start' icon and you're launched into the dashboard above from where all the various samples and walkthroughs are easily accessible. It's pretty slick.

What's To See?
--------------

There are two demo applications which you can run, both of which illustrate different facets of the VoltDB functionality:

1.   __Voter__
     This sample app consists of two components:
     *   A VoltDB database process
     *   A simulated call centre client capturing phone calls for an imaginary American Idol
     
     Check out the dashboard which is updated with the votes in real-time:
     
     ![Voter](https://dl.dropbox.com/u/47685018/Blog/2013/03-22/Voter.jpg)
     
     A really cool demo to watch in action! 
     
     It also demonstrates how to use Volt Studio performance monitor to see a breakdown of the overall throughput in terms of how many votes the app is processing and the latency for processing each one.
     
2.   __KV Store__
     For something completely different, this example shows Volt being used as a Key/Value store with a REST interface.
     Again the benchmarking tools give live feedback on the number of GET/PUT operations per second.
     
Both demos also provide some examples of how you can use standard SQL select statements to get back information from the data store.

Expect More of This Sort of Thing
---------------------------------

There's some really cool stuff happening out there are the moment in the NewSQL space. Big Data is very much the Big Thing at the moment, and whilst there's not an exact overlap between the two, NewSQL is very much in it's orbit. There's certainly a lot of be admired in the philosophy of allowing users to retain the big investment they've already made in SQL skills and relational design, yet not denying them the new and shiny NoSQL, web-scale world along with it.

The Volt demo is really nice - totally worth a dip if you're interested in this sort of thing.

I'll get back round to RavenDB again in my next post - part 2 will be accompanying [the introduction](http://esc-plan.com/blog/2013/03/04/raven-db-part1/) shortly.

