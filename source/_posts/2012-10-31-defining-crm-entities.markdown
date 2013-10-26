---
layout: post
title: "Defining Dynamics CRM entities"
date: 2012-10-31 16:29
comments: true
categories: [Dynamics 2011, Requirements]
---
Over the last few weeks I have been tasked with configuring a sizable Dynamics CRM solution. Having been through a lengthy period of analysis it's somewhat nice to be building again. So, I thought it would be useful to reflect on some of the work that's led up to this point.

Particularly as regards defining the entities themselves. This is the guts of any CRM project, and I'm coming to some conclusions around what works and what doesn't work so well (and how I'd ideally like to do it in future).

We've used a combination of two things:  

*   A definition of the entity data. What does this entity actually represent?  
    This is where we say: What fields? Which data types are they? 
    What are my OptionSet values? So on and so forth.
*   A wireframe defining the layout of the entity's main form.

<!-- more -->

The Long Slog
-------------

All in all our solution has ~60 custom entities as well as a variety of tweaks and additions to the OOTB standard stuff like Contacts and Accounts.

Some of these have an awful lot of fields.

When it comes to manually creating all of these via the Dynamics administrative GUI, well... it's time to get the headphones in shall we say.

Somewhat unsurprisingly, all of this must be documented in Word. Each entity has a table listing the required fields, the data type, minimum/maximum values any default values and so on.

It's a lot of information to capture, and no matter how nicely it's formatted it's always going to be a dry read for the customer (and the developer, let's be honest).

{% pullquote %}
{"Dynamics has quite a tight coupling between this concept of the entity's "data type" and it's appearance in the UI."} Of course, when it comes to your documentation it need not be so.

Our preference has been to have one definition for the data and a seperate, rather 'cartoony' wireframe which exclusively depicts the layout.
{% endpullquote %}

Power Mockup
------------

[Power Mockup](http://www.powermockup.com/)* is a nice wireframing tool which I've used to define a large number of the screens/forms in our solution where specific layout requirements are warranted. It provides a set of easy to use templates from within MS PowerPoint - the learning curve is fairly non-existent and it saves to a common format.

The implication that not all entities require a tailored layout is important. If an entity only has a few fields and its function is purely as reference data item, well... it probably doesn't warrant a screen design all of its own. A bit of common sense should prevail here.

In these cases it's sufficient to set a general principle that fields will appear on a form in the order that they appear in the entity's data definition (from top to bottom, left to right).

For everything else there's a wireframe.

The function of this is to purely show field layout. The data definition already defines the data types of fields, so why duplicate that information in the wireframe?

\* _Saying Power Mockup in a Ballymena accent may lead to peer ridicule._

Auto-Magic Entity Configurator
------------------------------

So if all of the data definitions have been captured and documented as part of our analysis, why should it be that we now have to go through this list and manually enter it all once again, this time through the Dynamics UI?

{% pullquote %}
{"I'm sure there is a better solution to be had here."} My feeling is that the [Create Entity Request](http://msdn.microsoft.com/en-us/library/microsoft.xrm.sdk.messages.createentityrequest.aspx) class and similar classes used for field creation in the Xrm SDK could usefully automate the entity configuration process.
{% endpullquote %}

{% gist 3994712 %}

What you would need are two files (.csv or similar) which the Automagic Configurator (tm) can read in and parse. It would then call the Dynamics API to do the laborious creation part for you:

*   One is a 'header' file which defines the general entity information - Name, ownership level, primary attribute etc.
    You must define these separately and up front as it's the easiest way to avoid order dependencies when creating Lookup fields in the next file...

*   The second file contains the field definitions. This captures our data types, min/max values etc.

Together these files would capture essentially what we've currently been documenting in Word, but would feed directly into the build phase cutting out a lot of grunt work. I always like to avoid grunt work. That's what computers are for!

As far as I'm aware such a tool doesn't currently exist. For me it might be a case of, if it doesn't come - build it!

And so
------

The alternative to the scenario described is obviously some form of Agile. That almost goes without saying. In this blog, I'm trying to think of a way to get round doing a whole lot of CRM configuration up front at the start of a build phase after a lengthly period of analysis.

You can feasibly cut down on the overall time by doing your field creation and form definition at once from the Form designer (rather than creating all the fields and __then__ firing up the form designer).

The problem is that if you have a whole slew of custom processes for which the CRM configuration is a dependency (as part of a Portal for example) then many of your team are going to be twiddling their thumbs until the underlying entity model is in place (at least, in place to some extent). That means getting the field definitions done has to take priority over form layouts.

In other news
-------------

As previously mentioned my Raspberry Pi finally arrived last week. Unfortunately it arrived at my parent's house deep in the darkest hills. So I still haven't got to play with it yet. [XBMC](http://xbmc.org/about/) looks like it could be the way to go for me though.

I wonder how long I can keep making blog posts and working in a mention of [Dark Souls](http://en.wikipedia.org/wiki/Dark_Souls)? Let's find out!