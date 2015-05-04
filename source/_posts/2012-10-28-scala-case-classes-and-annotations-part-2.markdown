---
layout: post
title: "Scala case classes and annotations, part 2"
date: 2012-10-28 23:00
comments: true
categories: 
---
Before proceeding please remember that I'm learning Scala and what follows is the result of lack of proper knowledge :)

In the previous article I wrote how to specify where annotations should be applied. But if you remember an example class I ported from Java to Scala there was one thing left to do and that was using constants in annotations.

In Java you can easily do
{% codeblock lang:java %}
public static final String USER_ID = "userId";

@Field(USER_ID)
private long userId;
{% endcodeblock %}
but in Scala this doesn't work.

First of all I tried adding constant in a class like this:
{% codeblock lang:scala %}
case class ScanningBookmark(@(Field@field)(USER_ID) userId: Long,
                            @(Field@field)(STATUS_ID) statusId: Long) {
    final val USER_ID:String = "userId"
    final val STATUS_ID:String = "statusId"
}
{% endcodeblock %}
but in Scala USER_ID and STATUS_ID are not resolved. Since normally an object is used for defining static content I then tried using a companion object to store those constants:
{% codeblock lang:scala %}
case class ScanningBookmark(@(Field@field)(ScanningBookmark.USER_ID) userId: Long,
                            @(Field@field)(ScanningBookmark.STATUS_ID) statusId: Long)
object ScanningBookmark {
    final val USER_ID:String = "userId"
    final val STATUS_ID:String = "statusId"
}
{% endcodeblock %}
but this also didn't work as it gave the following error:

{% codeblock %}
error: annotation argument needs to be a constant; found: ScanningBookmark.USER_ID
case class ScanningBookmark(@(Field@field)(ScanningBookmark.USER_ID) userId: Long,
{% endcodeblock %}
At this point I gave up. I tried looking for answer at Google but didn't find it - or as it occured later - just missed it.

The problem here is that in Scala a constant doesn't have type in definition. The last thing necessary to make those constants work was removing the String type from definitions:
{% codeblock lang:scala %}
case class ScanningBookmark(@(Field@field)(ScanningBookmark.USER_ID) userId: Long,
                            @(Field@field)(ScanningBookmark.STATUS_ID) statusId: Long)
object ScanningBookmark {
    final val USER_ID = "userId"
    final val STATUS_ID = "statusId"
}
{% endcodeblock %}
Now it compiles and works as expected.

This is the end of my short experience with Scala and annotations.