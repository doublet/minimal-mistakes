---
layout: post
title: "Making Bootstrap 3.1.x play nicely with Play Framework 2.2.x"
modified: 2014-04-28 09:44:33 +0200
category: WebDevelopment
tags: [Java, Play Framework, Web development]
image:
  feature: playplusbootstrap.png
  credit: 
  creditlink: 
comments: true
share: 
---

[Bootstrap](http://getbootstrap.com/) is a nice CSS framework and it can really help you when you need something quick. I recently started working with the [Play framework](http://www.playframework.com/) and wanted to use the latest version of Bootstrap. Here's how to get it working.


# Assumptions

I'm going to assume you installed the Play Framework (most versions in the 2.x range will be similar, but I'm going to use version 2.2.2).

The version of Bootstrap we're going to install is 3.1.1, but as you'll see, using other versions or even upgrading a Bootstrap version is as easy as changing a number.

# Bootstrap WebJar

[WebJars](http://www.webjars.org/) are client-side packages (such as Bootstrap or jQuery) that are packaged in JAR files for easy management.

## Adding a WebJar to Play

[sbt](http://www.scala-sbt.org/) is the build tool for Play. We can add WebJars to our `build.sbt`. `sbt` will read this file and use [Apache Maven](https://maven.apache.org/) to fetch the dependencies specified (including our WebJars).

First off all, add the following line to your `build.sbt`:

{% highlight scala linenos %}
{% raw %}
"org.webjars" %% "webjars-play" % "2.2.1-2",
{% endraw %}
{% endhighlight %}

Let's go to the [WebJars homepage](http://www.webjars.org/). Scroll down and find bootstrap (or any other package you like), it will have an entry like this:

{% highlight scala linenos %}
{% raw %}
"org.webjars" % "bootstrap" % "3.1.1-1"
{% endraw %}
{% endhighlight %}


Add this to your `build.sbt`.


Here's a sample `build.sbt`:

{% highlight scala linenos %}
{% raw %}
name := "My Cool Test Site"

version := "1.0-SNAPSHOT"

libraryDependencies ++= Seq(
  javaJdbc,
  javaEbean,
  cache,
  "org.webjars" %% "webjars-play" % "2.2.1-2",
  "org.webjars" % "bootstrap" % "3.1.1"
)
play.Project.playJavaSettings
{% endraw %}
{% endhighlight %}

Now do go in the Play console, do `clean`, then `run`. Play will re-fetch all dependencies.

## Adding a route for the WebJar

Add the following to your `conf/routes`:

{% highlight scala linenos %}
{% raw %}
# WebJar routing (for bootstrap, ...)
GET		/webjars/*file					controllers.WebJarAssets.at(file)
{% endraw %}
{% endhighlight %}

# Referencing Bootstrap from your views

You can use `@routes.WebJarAssets.at(WebJarAssets.locate("..."))` to reference a file from your views. We can find a list of files that are included 

<figure>
	<a href="http://i.imgur.com/coDervE.png"><img src="http://i.imgur.com/coDervE.png"></a>
	<figcaption><a href="http://i.imgur.com/coDervE.png" title="A list of files that are included in the Bootstrap 3.1.1-1 WebJar">A list of files that are included in the Bootstrap 3.1.1-1 WebJar</a>.</figcaption>
</figure>

To reference the file `css/bootstrap.min.css`, we can use `@routes.WebJarAssets.at(WebJarAssets.locate("css/bootstrap.min.css"))`.

# Styling forms

Styling forms is hard because we can't use the default input helpers. Luckily, Play provides a Bootstrap helper... which hasn't been updated in 2 years. It's still usable, but it was created for Bootstrap 2.x, which had different class names for forms.

## Writing a custom FieldConstructor

We're going to write our own field constructor now so we don't have to write lots of boiler-plate code for each form input.

Make a new file called `bootstrapFieldConstructorTemplate.scala.html` in `app/views` and add the following:

{% highlight scala linenos %}
{% raw %}
@(elements: helper.FieldElements)
<div class="form-group @if(elements.hasErrors) {has-error}">
  <label for="@elements.id" class="control-label">@elements.label</label>
  @elements.input
  <span class="help-block">@elements.infos.mkString(", ")</span>
  <span class="help-block">@elements.errors.mkString(", ")</span>
</div>
{% endraw %}
{% endhighlight %}

## Using the FieldConstructor

In your view, import the following:

{% highlight scala linenos %}
{% raw %}
@import helper._
@implicitField = @{ FieldConstructor(bootstrapFieldConstructorTemplate.f) }
{% endraw %}
{% endhighlight %}

Now, when you use something like `@helper.inputText(orderForm("id"))`, Play will use the field constructor you specified.

## The final touch

The only thing left to do is to add the `form-control` class on the `<input>` tag. We can do that like this:

{% highlight scala linenos %}
{% raw %}
@helper.inputText(orderForm("id"), 'class -> "form-control")
{% endraw %}
{% endhighlight %}

Here's the final view:

{% highlight scala linenos %}
{% raw %}
@(orderForm: Form[Order])
@import helper._

@implicitField = @{ FieldConstructor(bootstrapFieldConstructorTemplate.f) }

<h2>A form</h2>

@helper.form(action = routes.Application.index()) {
  @helper.inputText(orderForm("id"), 'class -> "form-control")
}
{% endraw %}
{% endhighlight %}

## Github repo

The code for this article is available as a Github repo over [here](https://github.com/doublet/Play-BootstrapWebJar).

# Updating Bootstrap

When a new Bootstrap version comes out, the first thing you'll want to do is to check the WebJars site. Usually new versions should be available there pretty quick. You can than add the new version number to your `build.sbt`. After that, do a `play clean` and a `play cleanFiles`, then `play run`. That should work just fine.