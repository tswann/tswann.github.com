---
layout: post
title: "Dynamics CRM 2011 Performance Tuning"
date: 2013-10-23 12:53
comments: true
categories: [Dynamics 2011, Performance, SQL Server]
---

In recent months I've been dealing with some big sets of data as part of a Dynamics CRM 2011 project.

This post is a bit of a brain dump containing some lessons learned and some useful techniques when tuning for performance.

I hope to get a somewhat more regular stream of content on here in the coming months. I've got an eye on Dynamics 2013 and some other non-CRM related gubbins that I've been pondering as well.

<!-- more -->

Tuning Views
------------

{% blockquote %}
Just a note on CRM version - this tuning was carried out against an on-premise installation running CRM 2011 Rollup 11. Subsequent rollups have included updates specifically to address application performance, so some of this could potentially be of limited relevance in more current versions. Some of it will I'm sure still be generally applicable, and probably deep in the realm of common sense.
{% endblockquote %}

Good view design in Dynamics CRM is about really filtering out the noise and presenting users with the information which is most relevant to them.

For big data sets (multiple millions of records) it's rare that you would want a view that didn't do some serious filtering on this, whether that be restricting the view to records belonging to the user, a given date range or a particular option set range.

Of course views in Dynamics CRM are paged; the default page size is 50 with an option to increase this to a maximum of 250 which gets applied for all entities across the board.

If you fire up SQL Server Profiler and take a peek under the hood what you find is that the queries being executed against the database apply a TOP using this configurable value.

Now by and large you would think this to be absolutely fine. However it's possible to get into difficulties and I've seen a variation of around 20 seconds in the speed of a view loading which was purely down to the sort order of the columns!

As it turns out, sorting on an OptionSet field can totally cripple the performance of a view and throwing no amount of indexes at the problem will avail you.

Behold, the offender:

{% gist 0cc4eea312d1dac7829d %}

@new_TypeX in the above statement equals the textual description of each value in the OptionSet.

So essentially the entire list has this sort applied, before the TOP clause.

If you use a default system generated view which say filters to show only Active records, and that count of Active records is in the order of millions, then this is going to cause you a major performance problem.

You might not even realise it either. A sort order of some kind is mandatory on a CRM view, and by default the view designer applies this to the first column you add. So if that first column just so happens to be an OptionSet then you could well have this one lurking in your system, just waiting for the record counts to get adequately large to start seriously killing the view response times.

Changing your sort order to a non-option set field - say a date - and applying an index to that field in the database will get you right back down to sub 1 second response times. So it's easily enough recitified; it's just a bit non-obvious and easy to 

API Considerations
------------------

Strategies for Data Migration
-----------------------------

And Finally
-----------