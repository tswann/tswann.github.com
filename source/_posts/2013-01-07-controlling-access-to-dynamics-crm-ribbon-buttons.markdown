---
layout: post
title: "Controlling Access to Dynamics CRM ribbon buttons"
date: 2013-01-07 14:46
comments: true
categories: [Dynamics 2011, Security]
---

At the moment I'm working on a project which is an interesting combination of core Dynamics CRM functionality and a set of custom processes written using ASP.NET MVC 4 and Web API.

Each custom process is something like an [OOTB CRM dialog] - a sequence of screens which the user can progress backwards and forwards through using Next/Previous navigation. At the end of the flow the dialog is completed and a series of CRM records is saved off/workflow invoked or whatever.

While there is much the OOTB CRM dialogs can achieve, they weren't a fit for the level of control we required - i.e. a more complex UI and with a slicker user experience and integration with various external systems.

Each process is launched from a custom CRM ribbon button.

In this blog, I'll take a look at the access control scheme we chose to restrict access to these processes for particular users.

<!-- more -->

Using Security Roles
--------------------

We would like to be able to specify which users have access to certain ribbon buttons using the Dynamics CRM built-in security roles mechanism.

To achieve this, you will need to create a custom entity for each ribbon button you want to hide or display.

For example, for a custom 'Upload File' button on your Contact ribbon - create a 'new_rbnUploadFile' entity. (new_ being whatever your organisation prefix actually happens to be).

It's a good idea to include RBN in the name to help distinguish entities which are being used for this purpose from any other business oriented custom entities in the schema.

You don't need to add anything to this entity definition beyond creating it - it should have the bare minimum of options enabled and 'Organisation' ownership.

The purpose of this is purely so that we can see it in the Security roles matrix.

It should appear under the 'Custom' tab for a role like so:

![Custom entity permissions](https://dl.dropbox.com/u/47685018/Blog/2013/01-09/Security_Role.png "Custom entity permissions")

So to specify that this role can access the Upload File ribbon button we are setting the 'Write' permission on it's associated entity. Really we could use any of the permissions for this, but what's important is that you adopt a convention and 'Read' or 'Write'probably make more sense than the others (you don't want anyone creating instances of these entities).

Defining the Display Rule
-------------------------

To make use of this we will define a Display Rule on the Upload File button which will link it to our custom entity.

Only users or teams with a security role that has a 'Write' privilege on the RBN Upload File entity will be able to see the button.

This configuration can easily be done using [Ribbon Workbench](http://www.develop1.net/public/page/Ribbon-Workbench-for-Dynamics-CRM-2011.aspx).

We want to add the new button to the Contact form ribbon. For this we create a new Command definition as shown below to which we will add the Display Rule:

![Add Display Rule](https://dl.dropbox.com/u/47685018/Blog/2013/01-09/Add%20Command.png "Add Display Rule")

The display rule will consist of a single 'Step' of type 'Entity Privilege Rule' as shown below.

![Entity Privilege Rule](https://dl.dropbox.com/u/47685018/Blog/2013/01-09/Rule%20Type.png "Entity Privilege Rule")

This rule defines that this ribbon button will only be displayed to users with 'Write' privileges on the RBN Upload File entity.

![Rule Definitions](https://dl.dropbox.com/u/47685018/Blog/2013/01-09/Entity%20Privilege%20Rule.png)

And that's it! We now have a ribbon button which can be hidden or displayed via the user's security role.

Other Options
-------------

Annoyingly the 'Entity Privilege Rule' is not an option if you're considering enabling/disabling the button using this security role controlled method.

The best option here is probably to use the 'Custom JavaScript Rule' and pass it the custom entity name as a parameter.

