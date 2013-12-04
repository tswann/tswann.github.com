---
layout: post
title: "Azure and Visual Studio 2013"
date: 2013-12-03 16:26
comments: true
categories: [Azure, Visual Studio 2013]
---
On Monday I had the pleasure of attending a Microsoft event down in the impressive [Helix](http://thehelix.ie/) theatre at Dublin City University. 

This featured Scott Guthrie on Azure accompanied by a variety of other guest speakers covering topics such as Visual Studio 2013, F# and Xamarin. Lots of interesting stuff and really well attended.

Read on below the cut for some of my notes and random thoughts.

<!-- more -->

Windows Azure
-------------

The highlight(s) of the day for me were the talks given by the main man Scott Guthrie (Windows Azure) and also Michael Koester (Visual Studio 2013).

The day began with Scott's talk and - *GASP* - network issues. I think the greatest lesson we all took away is that it's hard to demo cloud services with no internet connection. In the end, the conference centre WiFi came to the rescue and there was much rejoicing.

The Azure coverage focused mostly on the IaaS side of things. I think this was primarily down to timing issues and certainly the principle mantra of Scotts talk was __Focus on Apps, Not Infrastructure__

So the list of managed services got short shrift - Web, Mobile, Storage etc - but given that I was less familiar with the infrastucture offering that was fine by me and I found the whole thing very interesting nonetheless.

Cool stuff wot was shown:

- __Traffic Manager:__ used to determine the best data centre to use in terms of performance

- __Auto Scaler:__ configurable CPU and other resource usage sliders that can be used to set boundaries were new VMs are spun up or down. You can also scale based on particular time ranges - for example shutting down outside of working hours.

- __Virtual networks and gateways:__ You can define a network in Azure and join it via VPN. I.e. you can then access everything via local IP address. Public extensibility can be completely turned off and restricted to VPN only. This goes for web and worker roles, as well as the VMs.

All of this is configurable via the Azure dashboard, but is also scriptable at the CLI. Good for automation via powershell scripts and obviously via build tools like msbuild, or something non-awful like psake.

Interestingly it's all doable from within Visual Studio 2013 as well. That's a bit of a philosophical battle waiting to happen. It's interesting how MS are positioning VS as the package which has it all, certainly tightly coupling all this Ops business to an IDE is considered by many to be dodgy territory.

A lot of the features above were put to use in a demo around using Azure VMs for a Dev/Test example scenario.

Coolest gadgetry of the day probably goes to the Visual Studio Azure remote debugger. Scott demonstrated debugging into a process running on an Azure VM connected via VPN. He also demoed the Resource Browser in Visual Studio which allows you to browse all your Azure resources, restart VMs and so on without going through the portal. All really cool stuff.

It aint just Microsoft
----------------------

A point that Scott was keen to stress is that it's not just .NET that's running on Azure. As well as ASP.NET, websites can also be written in node.js, PHP, ruby, Python or Java.

Databases extend beyond good old SQL Server into the NoSQL space, with providers for MongoDB currently available with plans to include Redis and Couch in the near future.

A lot of what was demoed with regards to the services will be familiar to users of other cloud PaaS offerings like Heroku; the ability to push and build from a Git repository or FTP.

FREE STUFF
----------

Unfortunately we didn't get on to things like logging, telemetry, Hadoop and Active Directory single sign on - time went over and then ran out.

The most important information for MSDN subsrcibers out there is just how much you get for free. A Premium MSDN sub would allow you to run 3 VMs for 16 hours a day each month for absolutely no monies - pretty cool.


Visual Studio 2013
------------------

Michael Koester gave a talk on VS 2013 and he's a really engaging speaker! He endeared himself to the crowd almost immediately by admitting that his Munich accent makes him sound somewhat like a WWII movie villain.

I have to say that I'd taken somewhat of an issue with the new look and feel of VS and Office 365 et al.

Michael explained that this new 'Content over Chrome' philosophy was intended to make anything which isn't content "fade into the background".

Most people I've talked to find it overwhelmingly GRAY.

The Connected IDE
-----------------

The idea of Visual Studio as a connected environment, with a sign in and retention of my settings across multiple hosts is one I do find appealing I must say.

Looking at the bigger picture, it marks another step in the road of Microsoft positioning all of their traditional desktop products with online equivalents and VS is certainly no exception.

Project 'Monaco' - which is essentially Visual Studio in the browser - is currently limited to web-only, that's CSS, HTML, JavaScript and so on, but this is early days and the plan is to expand this.

It's already available to MSDN subsribers, so you should check it out.

Beyond the editor, there are some other really cool features in the works:

- __Agile Teams:__ Michael demoed a number of Agile specific features such as an integrated Kanban board (I'd like to have a play with this and see how it compares to Trello), burndown charts, and Team Room. This last one is basically Yet Another Chat Room, but it does have a lot of desirable features like history and logs, integrated activity feeds (think check-ins, work items and drill-down to TFS).

- __Code lens/HUDS:__ Think of this as little customizable plugins which git above your methods and classes just below the comments. Can be customized to included all kind of different info like test results, code metrics and so on.

- __Preview/Outline pane:__ i.e. as you will already find in Sublime Text

- __History based navigation:__ Again in line with the shift towards the browser we see the introduction of Back and Forward buttons in the solution explorer and the main window

- __Alt + F12:__ This now has a peek function were you can look at the contents of a class without actually opening a new tab.

I loved the little factoid that MS groups users into two groups when it comes to Tabs. Those who open ALL THE TABS until it becomes unmanageable and they nuke everything with a "Close All" and the OCD tab manager, who closes everything the moment they think they're done.

And The Rest
------------

Ok, this has long passed the TL/DR barrier, so I'll save my notes on the remaining sessions for another post.

That includes F# and Big Data, Gamespark and their use of Azure for game backends and analytics and also Xamarin and cross-platform development with C#.
