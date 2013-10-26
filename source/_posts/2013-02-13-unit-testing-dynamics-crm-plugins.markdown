---
layout: post
title: "Unit Testing Dynamics CRM Plugins"
date: 2013-02-13 12:35
comments: true
categories: [Dynamics 2011, TDD, Moq]
---

In this post I'm going to take a look at the dark art of writing unit tests for Dynamics CRM 2011 plugin classes.

OK, so it's not really all that dark, but it can be somewhat tricky if you're not familiar with how the plugin execution pipeline works and what kind of inputs an executing plugin expects to receive.

I have seen approaches to this which involve serializing the plugin context or using [Moles](http://research.microsoft.com/en-us/projects/moles/) (not an option for everyone now that this has been deprecated by the pricey [Fakes](http://msdn.microsoft.com/en-us/library/hh549175.aspx) framework). 

My approach is quite simple and make use of [Moq](http://code.google.com/p/moq/). Rhino Mocks, NSubstitute or other feature-equivalent mocking frameworks should work just as well, so go with your preference.

<!-- more -->

Mocking Dynamics
----------------

![Mocking Dynamics CRM](https://dl.dropbox.com/u/47685018/Blog/2013/02-14/mocked.png "Yes, I've been waiting to use this")

Plugins by their nature are stateless and you must invoke a fair bit of mocking voodoo to set up the external pipeline execution context in which they run.

I use a base class for all my plugin tests which contains the following core:

{%gist 4954043 %}

This allows us to hide away some of the usual mock object setup code - the objects which will be common to every plugin - the IServiceProvider, IPluginExecutionContext and IOrganisationService.

In our individual plugin classes we can then mock the typical input parameters to the plugin context.

An Example
----------

Let's say we have some imaginary business pals. What they would like is for a bit of validation to be triggered whenever someone attempts to delete a Contact record from the CRM database.

If the Contact has any open Task activities outstanding, we would like the system to display an error message and prevent the user from doing so.

We could of course have a piece of custom JavaScript to do this, but we want the logic to be triggered _any time_ a delete occurs, be it via the front end or the API.

The following plugin code will do just that:

{%gist 4954084 %}

For the sake of brevity I have cut all the boiler plate gunk that the  [Visual Studio CRM Dev  Toolkit](http://msdn.microsoft.com/en-gb/library/hh547459.aspx) generates when you create a new plugin, leaving only the custom business logic that I actually added myself.

So how about writing an automated unit test for this code?

The purpose of providing mock objects for your unit tests is to isolate them - removing any dependence on external resources. 

As mentioned, plugins are stateless - all of their dependencies are supplied to the plugin Execute() method by the plugin execution context object which is passed at runtime.

A few of the things that we will need to Mock include:

*   The Contact object which is causing the plugin to fire
*   The parameters which identify the action we want to trigger for that contact, namely:

    1. The PrimaryEntityName - 'contact' in this scenario.
    2. The 'Message' which identifies the data operation. In our case this is 'Delete'.
    3. The 'Stage' at which it should execute. We have supplied 10, which means 'Pre-Validation'. Our base test class contains some useful constants for these values.
    4. The Guid of the calling user. Not used in our example, but could be useful if your code contains logic which is conditional on the use of a particular user account.
   
*   The methods of the IOrganisationService which our code requires. This is generally going to be a biggie. Our example plugin calls out to RetrieveMultiple, a method which takes a QueryExpression to execute against the CRM server and return a list of entities. We must mock this with an in-memory collection. Typically this is the approach you will take for all such external CRM server calls.

Writing a Test
--------------

So, how do we go about this? Thankfully Moq makes it pretty easy to create a lot of dependencies quickly and fluently.

{% gist 4954297 %}

The Setup method above sets some specific values for your suite of tests - the entity type, user and validation stage.

Below is an example test which sets up the scenario where the validation check fails - the call to RetrieveMultiple will successfully return a matching open Task associated with the Contact:

{% gist 4954348 %}

As you can see there's not much to it - we just set up the OrganisationService mock object to return a dummy task whenever it gets called within our plugin code under test.

Write ALL THE TESTS
---------------------

I think that goes some way to illustrating that Dynamics plugins can do the TDD thing just like anything else - all you need are some great tools and [Moq](http://code.google.com/p/moq/) is certainly one such.

If you're interested, the full source code for this example can be downloaded from github: [https://github.com/tswann/x1bplan.plugintesting.git](https://github.com/tswann/x1bplan.plugintesting.git).

I've used a simple example based on the OOTB Contact entity, so it should work when deployed to any freshly created CRM organisation.

Until next time!

![Praise the Sun](https://dl.dropbox.com/u/47685018/Blog/2013/02-14/thesun.jpg "I love this game")