---
layout: post
title: "Simple Hive UDFs"
date: 2014-12-23 12:23:19 +0000
comments: true
categories: [Big Data, Hadoop, Agile]
---
The lack of new content on this blog in recent months is probably a testament to my generally high level of busyness in recent months. Since I attended the [Thoughtworks Big Data briefing](http://esc-plan.com/blog/2014/07/04/thoughtworks-big-data-briefing/) back in July it's been full steam ahead with helping to build up the Big Data practice in my place of work and getting immersed in a varied platter of shiny new technologies.

In October I was in NYC for the hugely interesting [Strata & Hadoop World](http://strataconf.com) conference, which is undoubtedly worth a post in itself and something I should really have written nearer the time. But yeah - busy busy.

So with some time off arriving, I thought I would try and get back in the groove again and write up a short post on some of the Hadoop work I've been doing.
<!-- more-->

Extending Hive
-------------

One tool which I've used extensively is the [Hive](https://hive.apache.org) data warehouse. This meant getting to grips with the ins and outs of Hive QL, the SQL-live query language which provides analysts with a more usable abstraction over the [MapReduce](http://en.wikipedia.org/wiki/MapReduce) execution engine.

<img style="float: left; margin-right: 5px" src="https://dl.dropboxusercontent.com/u/47685018/Blog/2014/12-22/hive.jpg">

Hive QL has a library of functions which provide the majority of the standard functionality you would expect from an ANSI SQL compliant interface. However, what it tends to lack are out-of-the-box implementations of common functions that an analyst might expect having come from a SQL Server or Oracle background.

Recently I've encountered a few requirements to implement fuzzy-matching algorithms such as Jaro-Winkler or Soundex on the Hadoop stack.

For use cases like ETL de-duplication or fraud detection these algorithms are fairly standard, so it's not unreasonable to expect this functionality if you are moving this type of workload to a Hadoop cluster.

This post will show how to use [Hive UDFs](https://cwiki.apache.org/confluence/display/Hive/HivePlugins) to simply plug in an implementation of the Soundex algorithm for use in Hive queries.

The Simple UDF API
-----------------
There are several User-Defined Function extension points in Hive. This post is going to focus on the simplest, by extending org.apache.hadoop.hive.ql.exec.UDF

The snippet below shows the classes that our SoundexUDF requires - there isn't much to it:

```java
import org.apache.commons.codec.language.Soundex;
import org.apache.hadoop.hive.ql.exec.Description;
import org.apache.hadoop.hive.ql.exec.UDF;
import org.apache.hadoop.io.Text;
```

[Soundex](http://en.wikipedia.org/wiki/Soundex) is a phonetic algorithm which can be used to determine how similar English words sound - typically names. It does this by generating a 'soundex code' which is a character code representing how the name sounds. Very similar names should result in the same code.

The implementation that I have used is the one supplied by [Apache Commons](http://commons.apache.org/proper/commons-codec/apidocs/org/apache/commons/codec/language/Soundex.html)

Re-using well tested, common implementations which perform some transformation on scalar types is an approach which fits the UDF pattern very well.

Our function simply wraps the soundex method in the Commons class. We receive some text and return the soundex code:

```java
@Description(name = "soundex",
value = "_FUNC_(string) - Retrieves the Soundex code for a given string.",
extended = "Example:\n"
            + " SELECT _FUNC_(input_string) FROM src;")
public final class SoundexUDF extends UDF {

    public Text evaluate(final Text text) {
        if (text == null) {
            return null;
        }

        String soundex = new Soundex().soundex(text.toString());
        return new Text(soundex);
    }
}
```

The @Description attribute is used by the Hive interpreter to provide information to the 'describe function' or 'describe function extended' commands from the beeline shell or with Hue. This provides usage information to the users of the function.

Trying it out
-----------

When building in Eclipse I use the [Maven Assembly Plugin](http://maven.apache.org/plugins/maven-assembly-plugin/) to ensure that I include the requisite dependencies in my jar file (a so called 'fat jar').

You can find a full sample of the [source code](https://github.com/tswann/x1bplan-soudexudf) for this UDF class and associated pom.xml file on github.

This sample was build against the version of Hadoop included on the [Cloudera CDH 5.1](http://www.cloudera.com/content/cloudera/en/downloads/quickstart_vms/cdh-5-2-x.html) quickstart virtual machine.

We're going to use beeline to install and test the new soundex function. The old Hive shell has been deprecated in favour of the HiveServer2 compatible beeline CLI. Check out [this post](http://blog.cloudera.com/blog/2014/02/migrating-from-hive-cli-to-beeline-a-primer/) for more information.

```
[cloudera@quickstart target]$ beeline
Beeline version 0.12.0-cdh5.1.0 by Apache Hive
beeline> !connect jdbc:hive2://localhost:10000
0: jdbc:hive2://localhost:10000>
```

From the beeline prompt we can add the newly built jar file, assuming it was placed in the cloudera users home directory on the quickstart VM:

```
0: jdbc:hive2://localhost:10000> add jar /home/cloudera/soundex-udf-0.1.jar;
```

Once the jar file has been successfully registered we can now execute the following statement to add a new function which uses the custom class:

```
0: jdbc:hive2://localhost:10000> create temporary function soundex as 'com.x1bplan.hive.udf.SoundexUDF';
```
You should now be able to execute a describe statement and see the usage text which we included in the class definition:

```
0: jdbc:hive2://localhost:10000> describe function extended soundex;
+-------------------------------------------------------------------+
|                             tab_name                              |
+-------------------------------------------------------------------+
| soundex(string) - Retrieves the Soundex code for a given string.  |
| Synonyms: soudex                                                  |
| Example:                                                          |
|  SELECT soundex(input_string) FROM src;                           |
+-------------------------------------------------------------------+
4 rows selected (0.126 seconds)
```
And that really, is that. You can how try out the function by calling it as part of a SELECT statement and passing a text field for evaluation.

Hadoop Business Processes
-----------------------

UDFs allow you to easily extend Hive with additional capability that might be missing from the core Apache functions.

I've been giving some thought to the problem of building a reliable and maintainable code base on the Hadoop platform.

Potentially you risk throwing together Hive, Pig and Impala scripts, shell, Python, Java MapReduce and all the other various kinds of extensibility points that the platform offers up and ending up with a real mish-mash which is difficult to reason about and to maintain.

Frameworks like [Cascading](http://www.cascading.org) would appear to offer a solution - a framework to help orchestrate the myriad of abstractions and execution engines that go with Hadoop by building a consistent and testable Java code base.

I'll be taking a look at this along with [Lingual](http://www.cascading.org/projects/lingual/) (ANSI SQL) and [Pattern](http://www.cascading.org/projects/pattern/) (PMML based machine learning) in the near future and hopefully get another post out before the holidays come to an end.

There's also the Scala abstraction called Scalding, but I'll see how I get on - there's Christmas dinner to be eaten at some point.
