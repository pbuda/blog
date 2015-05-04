---
layout: post
title: "Visitor Pattern"
date: 2011-12-15 16:09
comments: true
categories: 
---
Some time ago I was supposed to add a boolean flag to a DTO. Nothing fancy until I found out that domain stuff related to that flag is not really defined and is based on actual type of some property...

My first idea was to simply use _instanceof_ to check whether the type of that property should change the flag to _true_ or _false_. I think I don't have to tell you this looks ugly at best...

{% codeblock lang:java %}
boolean myFlag = object.getProperty() instanceof TrueFlag;
{% endcodeblock %}

Even if it's not ugly, when more and more policies appear, this condition can change making a long, hard to maintain and read piece of code. Then a friend suggested to use the [Visitor Pattern](http://en.wikipedia.org/wiki/Visitor_pattern). It's a very simple pattern that basically allows to add operations to an existing object without modifying the object itself.

What I did was that I implemented the visitor and for each accept(object) method I simply set the required flag to true or false. Easy, clean solution that is easy to maintain, test and read.

So, when you try to do something with your existing object, instead of adding new methods to it, check out the [Visitor Pattern](http://en.wikipedia.org/wiki/Visitor_pattern) - maybe it'll help you.