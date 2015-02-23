---
layout: post
title: "Play Framework + Knockout"
date: 2015-02-23 16:03:37 +0000
comments: true
categories: [knockout, Play, MVC, Web Development]
---
I'm going off on a bit of a tangent from my usual Big Data ramblings this week, to talk about the [Play 2](https://www.playframework.com/) framework (for Java) and integrating this with the rather neat MVVM client-side JavaScript library that is [knockout.js](http://knockoutjs.com/).

In this post I will present a short walk-through on how to set up a simple play project that uses knockout.js for dynamic data binding.

This was quite a straightforward and fun exercise - I didn't find too many tutorials on knockout + Play Java specifically.

The official Play site has a demo app using the [Scala API + MongoDB + knockout](http://typesafe.com/activator/template/play-mongo-knockout), so I recommend that if you're inclined to use Scala.

<!-- more -->

## Getting started with Play 2

Of the two technologies under discussion, I'm new to the Play framework. Thankfully it's exceptionally easy to get out of the ground and I had an application up and running in short order.

If you have successfully downloaded and installed Play via [these instructions](https://www.playframework.com/download), then the first thing I did was to create a new app from one of the 'seed' projects available via the Activator UI.

I selected the Play Java seed shown below:

![Activator UI](https://dl.dropboxusercontent.com/u/47685018/Blog/2015/02-23/Typesafe_Activator.png)

This will create a new boilerplate Play 2 app in the directory of your choosing.

I was also presented with the option to create a new project for IntelliJ, so I recommend selecting that option if you're using the IDE.

### WebJars for Fun and Profit

The first thing I wanted to do was to plug in dependencies for my two favourite client-side frameworks: knockout.js and [Twitter Bootstrap](http://getbootstrap.com/).

Fortunately this turned out to be a simple case of adding to the [SBT](http://www.scala-sbt.org/) script which manages all library dependencies in Play.

```scala
libraryDependencies ++= Seq(
  "org.webjars" %% "webjars-play" % "2.3.0-2",
  "org.webjars" % "bootstrap" % "3.1.1-2",
  "org.webjars" % "jquery" % "2.1.3",
  "org.webjars" % "knockout" % "2.3.0"
)
```

`WebJars` are a cool mechanism that allows you to manage client side dependencies packaged as JAR files.

Once added to the build file, a new line in `conf/routes` configures Play to pick up the new assets:

```
# Map the webjar static assets to the /webjars URL
GET     /webjars/*file                    controllers.WebJarAssets.at(file)
```

Within the actual HTML view templates, these assets can be loaded using a helper utility as follows:

```html
<link rel="stylesheet" media="screen" href="@routes.WebJarAssets.at(WebJarAssets.locate("bootstrap.min.css"))">
```

Having come from an ASP.NET MVC / Razor view engine background, all this was feeling pretty familiar at this point.

### A Simple Domain Model

Knockout JS provides a lightweight framework to implement [MVVM](http://en.wikipedia.org/wiki/Model_View_ViewModel) commanding and data binding in your client side code.

The first thing I needed was a simple Java object to serve as my server-side domain model object.

```java
public class User {
    public String username;
    public Long id;
    public String firstName;
    public String lastName;
}
```

I then modify the provided sample controller action to return an instance of this object instead of the default String:

```java
    public static Result index() {
        User user = new User();
        user.username = "tswann";
        user.firstName = "Tom";
        user.lastName = "Swann";
        user.id = 101L;
        return ok(index.render(user));
    }
```

This requires a corresponding change in the `index.scala.html' view template to accept an object of type User:

```scala
@(Model: models.User)
```

### Creating the ViewModel

So, at this point the object could be bound up to the view using the standard Play scala declarative data binding.

However we would like to transform this object into a knockout `observable` JavaScript ViewModel. This will allow us to take advantage of knockouts clever dependency chaining and automatic UI refresh.

In order to do this, I want to serialise the User object to JSON and then de-serialise to a JavaScript object.

There are two more dependencies require in order to achieve this.

Firstly, we need a library to serialise the User object to JSON. For this I used [Google Gson](https://code.google.com/p/google-gson/). Again, this can just be added as a new dependency in the SBT script:

```scala
libraryDependencies ++= Seq(
  "com.google.code.gson" % "gson" % "2.2.4"
)
```

On the client side, there is a knockout plugin [ko.mapping](http://knockoutjs.com/documentation/plugins-mapping.html) which exists specifically for this common scenario were you want to bind a plain JavaScript object from the server to a knockout observable, for use as a ViewModel.

> At this point I should probably note that getting to this stage was much, much less painful than the first time I tried to do the same thing
> in ASP.NET MVC 4. Now maybe it's the benefit of experience, but I think it's more that Play is just REST through and through, from the ground up.

The following JavaScript shows how to tie up the the Ser/DeSer piece that actually passes the values in the User object to knockout for binding in the UI:

```javascript
 $(function() {
    var model = @{Html(new Gson().toJson(Model))};
    var viewModel = ko.mapping.fromJS(model);

    viewModel.fullName = ko.computed(function() {
        return viewModel.firstName() + " " + viewModel.lastName();
    });

    ko.applyBindings(viewModel);
});
```

## The End Result

Now that knockout has acquired a copy of the user object, we can start to bind this up to the view HTML.

Each property of the User object is bound to the desired form control using the `data-bind` attribute as follows:

```html
<div class="form-group">
    <label>First Name:</label>
    <input type="text" data-bind="value: firstName"/>
</div>
```

What is particularly interesting in this example though is the `calculated observable` Full Name.

This property of the ViewModel dynamically returns the combination of the first and last name.

```javascript
viewModel.fullName = ko.computed(function() {
    return viewModel.firstName() + " " + viewModel.lastName();
});
```

What is particularly cool about this is that it tracks the values of the first and last name. Changing the value in either input field result in the Full Name binding being updated on the fly! Knockout gives us this behaviour for "free".

Messing about with these frameworks is a lot of fun and it really doesn't take long to get something cool up and running.

You can find all the source code from this example up on Github: [https://github.com/tswann/play-java-knockout](https://github.com/tswann/play-java-knockout)

