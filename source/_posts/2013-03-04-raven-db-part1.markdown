---
layout: post
title: "NoSQL for the .NET Guy: RavenDB"
date: 2013-03-04 21:16
comments: true
categories: [RavenDB, .NET, NoSQL]
---

This is part one of an ongoing series of posts in which I will be taking a delve into at a very nifty document-oriented database: [RavenDB](http://ravendb.net).

<!-- more -->

Too Many Sequels?
-----------------

DavenDB belongs to the family of databases described (a bit misleadingly) as NoSQL.

What that means to most people is that it's a database which isn't your more usual relational variety, such as SQL Server, MySQL or Oracle. The query language, or lack thereof is not really at the core of what these data stores are.

It's the lack of the enforced referential integrity in the database itself that primarily distinguishes them from an RDBMS. A high degree of normalisation is usually the goal of traditional databases, whereas the NoSQL variety is distinguished by the opposite - redundancy and read-optimisation.

As with all things under the sun they're very good for some things and terrible at others.

Anyway, I'm not trying to (or capable of) writing a novel on the history of NoSQl here.

My interest in RavenDB is a tad less academic: I've found it really fun to hack around with!

LET'S DO THIS THING.

Some Features:

*   A JSON based document-oriented database
*   Server and client written in .NET Framework v4
*   Indexes/queries written using LINQ
*   REST interface
*   Swanky Web UI management studio
*   Runs as a Windows Service, hosted in IIS, embedded or standalone application

Try Out RavenDB
---------------

Trying it out for yourself is a very simple thing - here's how you can install it and have it up and running in short order:

1.   Download the [latest stable build](http://hibernatingrhinos.com/builds/ravendb-stable)
2.   Extract the zip to a desirable location
3.   In the root of the extracted directory, run Start.cmd

This is the most straightforward way to run the software, launching the server in a command window. You should also now find your browser pointing at localhost:8080/raven/studio.html

As this is your first time in you will receive a prompt to create a test database - give it a cool name:

![Texas Forever](https://dl.dropbox.com/u/47685018/Blog/2013/03-04/new_db.png "Texas Forever")

If you double-click your new database to open it up, you can pre-populate it with some test data.

Navigate to Tasks -> Create Sample Data.

If you now click on 'Collections' you will see two items: Albums and Genres and a long list of sample documents.

Open a few of these up and take a look at JSON format in which the data is stored:

![Great Hits Indeed](https://dl.dropbox.com/u/47685018/Blog/2013/03-04/document.png "Greatest Hits Indeed")

Note that you can still have all the Id's you could ever want - here the Genre refers to another document within the database: genres/1. Remember that the database isn't enforcing the integrity of this data - that is left entirely up to us.

Note the format of the name of this resource: 

{% blockquote %}
albums/388. 
{% endblockquote %}

{resource}/{id} - That has the look of REST about it, right?

In the next post I'll take a dig into the HTTP interface that RavenDB provides by knocking up a simple .NET client application. We'll also look at the use of LINQ to write indexes/queries and have a look at Map/Reduce (if you ever did any HPC module at uni you'll be familiar with the concepts of crunching big data sets across a cluster of nodes; I swear to god that it's more fun than Fortan MPI though).

I'll also run the RavenDB server as a website hosted within IIS, so that we won't have to leave a command window open the whole time.


