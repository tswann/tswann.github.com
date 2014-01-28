---
layout: post
title: "Dynamics CRM Data Migration"
date: 2014-01-25 20:07
comments: true
categories: [Dynamics 2011, Data Migration, Performance]
---
For some bizarre reason I'm ridiculously early for a flight. Hence some blogging time!

This is a much belated follow up to my previous post [Dynamics CRM Performance Tuning](http://esc-plan.com/blog/2013/10/23/dynamics-crm-performance-tuning/).

I'm going to take a look at some of the issues involved in planning and executing a large migration exercise into Dynamics CRM 2011 from one or more legacy systems.

I'll cover aspects of planning, designing and developing a solution, and look at how to address some of the more pressing non-functional issues like performance and scalability when dealing with large volumes of data.

Dear Reader, let's do this thing.
<!-- more -->
The Migration Scenario
----------------------

Generally speaking, legacy LOB systems are often messy affairs when it comes to data quality.

Over a large number of years of development, support and unnumbered godless hacks, there _will_ be everything and anything in there. You never know quite what murky, red-eyed creations are lurking in the corners of these old data stores!

![Here Be Ninjas](http://i.imgur.com/lV3vFPg.jpg?1 "Here Be Ninjas")

When migrating (or indeed, just integrating) with such systems, we need to be on our toes with regards to issues of data integrity, data loss, data cleansing, data creation... basically a long list of non-trivial stuff.

A common scenario for undertaking a data migration exercise is one where the customers current system (or systems) are no longer fit for purpose.

There could be many reasons why this might be the case; here are just a few:

- Mis-alignment with business goals (e.g. the business may wish to adopt a more lean, customer-centric philosophy. Old systems often tend to be product-centric. This is a common scenario when migrating to Dynamics CRM.)
- Older technology does not offer easy extensibility.
- Costly licensing/support.
- Language requires specialised skills making it difficult to come by individuals capable of maintaining it.

Every Dynamics project I've worked on has had some element of migration, from ongoing integration with legacy systems to their wholesale replacement. 

Many of the issues we'll look at are pretty fundamental to any data migration exercise, though obviously Dynamics has it's own particular considerations and technical quirks.

In this post I'll try to cover both. The scenario is of a customer wishing to migrate a large set of their data from multiple legacy systems into a brand new solution based around Dynamics CRM 2011. We're dealing with CRM On-Premise here, but a lot of this will apply equally well to the On-line (cloud) version.

Project Approach
----------------

Let's say that our new system is in the process of being designed and built and that our task to migrate records from the old world to the new will be progressing in parallel with the main stream of development.

Starting to think about data migration early in the lifecycle of the project is critical. In the beginning we will have a limited understanding of the legacy systems, and on the other side of the coin the customer will be trying to understand their requirements for the new system and how their existing data can be made to fit.

It can be hard to get a sense early on of how much effort will be involved in data migration, particularly if the requirements of the new system are liable to shift (_hint_: they will).

An iterative process which allows for regular feedback in both directions will save you a world of pain and make that process of reaching mutual understanding with the customer go that much smoother.

Defining The Process
----------------------------

__ETL__ (Extract Transform Load) is a label applied to processes which essentially involve the transfer of data from point A to point B via some set of transformation rules. This is the conceptual frame for how our migration exercise will work.

In our case point A encompasses several legacy systems, let's call them Legacy X and Legacy Y. Point B is the Dynamics CRM 2011 database.

We have a technical limitation in that our interaction with Dynamics is via it's web services - direct access to the database is unsupported. So that will be our channel for getting data in. Obviously a web service is much slower on the face of it than a straight DB insert, so this will impact on the design of our load process and how we scale for performance.

Here's a high-level overview of how our ETL process will look:

!["Example Migration Process"](https://dl.dropboxusercontent.com/u/47685018/Blog/2014/01-26/Migration_ETL.png "Example Migration Process")

1. Source System Extracts
-------------------------

The intial step in the process is to extract the data from the customers existing source systems.

How this is accomplished will depend on the nature of the source systems and who is responsible for producing the extract - you, the customer or some third party. Typically you'll agree on the subset of source tables which will come across and the format of the data within each file.

In my example we receive two sets of CSV files from each system, one file for each source table.

At this point I want to bring those files into SQL Server so that I can easily analyse them as my ETL process progresses. Each system is assigned a set of tables which each correspond to a file. Each table is prefixed to identify the source. In this case that's __X_ __ and __Y_ __.

To insert the data into SQL Server I use good old [BCP](http://technet.microsoft.com/en-us/library/ms162802.aspx). Each input file has an associated FMT (format) file which tells BCP how to map the source columns to the destination table. 

I can also redirect any failures out to a log file; so this is the first step in the process where I can validate the quality of the data and get a feel for any work that I will need to do around field seperators, quoted strings etc.

2. Transformation
-----------------

The second step is the where the bulk of the effort is going to be - devising a set of transformation rules to convert sets of old records in the X and Y import tables and inserting them into the staging tables (STG) ready for load.

The staging tables are effectively a mirror of the target entities in Dynamics CRM, though limited to only the subset of fields which we actually need to populate.

Transformation rules can be developed as SQL Server stored procedures which select from the necessary source tables, apply some mapping rules and then insert into the relevant stg_ table in the new format.

For an example at the level of a single field, a character based status code 'A' which denotes an active record in the old system might well translate into an numeric OptionSet code in CRM (e.g. 948100000)

3. Load
-------

My load step is accomplished by a custom C# utility which sucks the records out of the staging tables using Entity Framework and inserts them into Dynamics CRM via the SOAP web service endpoint.

If you build a level of convention into the format of the staging tables, then this final step should be relatively trivial.

I name each stg_* table after the corresponding record in Dynamics CRM into which it will be loaded. For example, stg_Account, stg_Contact and so on. Each staging table column is given the same name as the corresponding column in Dynamics CRM.

Building linkages between different entity types can be achieved in the staging tables by generating the GUIDs for each record up front during step 2. You need to be careful of the order in which you load your entities into Dynamics CRM to avoid missing reference errors. For example if loading in Contacts which have a parent Account, you will need to have loaded the Accounts in first. 

A good idea is to make the list of entities which your application will load configurable via App.config. This will allow you to easily test subsets of the entities independently of the others.

Given that I have already defined my staging schema as part of the exercise to transform the source data, I make use of [Entity Framework Database First](http://msdn.microsoft.com/en-gb/data/jj206878.aspx) to automagically generate a set of classes in C# which I can use to extract the information from the staging tables.

Because of the naming conventions I have adopted in my data layer, I can use [AutoMapper](https://github.com/AutoMapper/AutoMapper) to easily map the generated POCOS to the early-bound Dynamics CRM classes (also generated) with minimal code.

{% gist 8649357 %}

The gist above shows how you can add mappings to an AutoMapper profile. Basic types can be mapped without supplying an explicit mapping since our source and destination field names match.

Note as well how we can supply a recognised prefix - this means that our stg_ tables can leave out the 'new_' (or whatever letters your Organisation is using) that Dynamics CRM prepends onto all custom fields.

OptionSets and EntityReferences will need either a Converter class to handle them implicitly or they can be mapped explicitly as in the example above.

Non-Functional Requirements
---------------------------

The performance achieved when loading into Dynamics CRM via the SOAP web service can be slow if you are dealing with a single Application server instance. Gains can be made by scaling up the DB and App servers, however the biggest gains will come from installing multiple instances of the App server in a load balanced configuration and running your Load process in parallel against each one. This might be your only option to achieve practical timings when dealing with big sets of records (say 50 million + and if you only have a weekend in which to complete your migration).

Some other performance points to consider:

- Turning off plugins and workflows. This can be done programatically via the API, so include pre and post migration steps in your Loader code to disable and re-enable these.
- Disable any nightly backup/maintenance jobs. This is kind of an obvious one, but make sure there's an explicit task for someone to ensure these are turned off. The last thing you want is some overnight job killing your app if the run time for your load is long.
- If using Entity Framework to load your data, remember that this is just a bulk read-only task. Therefore make sure you turn context tracking off, otherwise your garbage collector will scream into a pillow and jump off the roof. Below is an example method to get all records to type T with context tracking turned off. Pretty straightforward.

{% gist 8674031 %}


Alternatives
------------

Taking a step back for a moment, you should note that I have developed individual steps for each part of the ETL process over which I have very fine grained control. You could also go down the [SSIS](http://technet.microsoft.com/en-us/library/ms141026.aspx) route to manage the whole process for you end-to-end. 

There's a lot to be said for this option if you have SSIS skills and want to keep the whole flow within a single tool. Personally (and - __DISCLAIMER__ - not being an SSIS expert) I found it a bit restrictive. There is an option for connection to CRM in this [tool from Kingswaysoft](http://www.kingswaysoft.com/products/ssis-integration-toolkit-for-microsoft-dynamics-crm). This provides a graphical way to build mappings between the staging tables and CRM. 

Again, under the hood this is using the same Dynamics CRM web services API that any custom code would use. Personally I would rather write a minimal amount of code using something like AutoMapper than go clicking lots of checkboxes in a GUI, but __that's just me__.

Finally
-------

That's my Dynamics CRM data migration approach at a fairly high level. I'd be genuinely interested to learn how others have approached this, so please leave a comment if you've been through the mill on this yourself.

Back to staring at the flight board!
