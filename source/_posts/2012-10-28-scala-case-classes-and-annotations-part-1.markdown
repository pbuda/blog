---
layout: post
title: "Scala case classes and annotations, part 1"
date: 2012-10-28 19:23
comments: true
categories: 
---
Recently in Twittory I moved from Java to Scala. Well, to be honest I just switched from Java to Scala syntax and will move on to adapt to functional programming principles.

There was one glitch during the process. I am using Spring Data for MongoDb and to ease up a few queries I used statics to define field names of objects I store.

Here's an example: very simple class that I use to store tweet scanning bookmarks:
{% codeblock lang:java %}
public class ScanningBookmark {
    public static final String USER_ID = "userId";

    public static final String STATUS_ID = "statusId";

    private ScanningBookmark(long userId, long statusId) {
        this.userId = userId;
        this.statusId = statusId;
    }

    @Field(USER_ID)
    private long userId;

    public long getUserId() {
        return userId;
    }

    @Field(STATUS_ID)
    private long statusId;

    public long getStatusId() {
        return statusId;
    }
}
{% endcodeblock %}
Storing this thing with Spring Data. The static fields defining field names are here so that somewhere else I can define a query using those fields. The @Field annotations allow customization of field names in collections and while here they are the same as actual field names, they could be set differently.

Now when I was rewriting this to Scala, it's quite obvious I ended up with a case class.
{% codeblock lang:scala %}
case class ScanningBookmark(userId: Long, statusId: Long)
{% endcodeblock %}
But the problem with case class is that adding @Field annotations doesn't work anymore. This is because writing like this:
{% codeblock lang:scala %}
case class ScanningBookmark(@Field("userId") userId: Long,
                            @Field("statusId") statusId: Long)
{% endcodeblock %}
places the @Field annotation on constructor arguments, fields and Scala accessors which probably makes Spring Data scanner fail. The result is that these fields are no longer customizable and I wanted to somehow remedy that.

Playing with @BeanProperty annotation didn't solve the problem because apparently the @Field annotation is then also placed on generated accessors. But solution to this problem is actually quite easy. In Scala there are target meta-annotations that can be put on the annotation type when instantiating the annotation. There are six of those: @beanGetter, @beanSetter, @field, @getter, @setter, @param. Consult the [target package](http://www.scala-lang.org/api/2.9.2/scala/annotation/target/package.html) for more info.

The solution? It's very simple:
{% codeblock lang:scala %}
case class ScanningBookmark(@(Field@field)("userId2") userId: Long, 
                            @(Field@field)("statusId2") statusId: Long)
{% endcodeblock %}
And that's it! But there's one last thing to make it behave like Java version. In Java I used statics to define field values (so that there's no problem when some name changes). So I want something like this:
{% codeblock lang:scala %}
case class ScanningBookmark(@(Field@field)(USER_ID) userId: Long, 
                            @(Field@field)(STATUS_ID) statusId: Long) {
  final val USER_ID: String = "userId2"
  final val STATUS_ID: String = "statusId2"
}
{% endcodeblock %}
But this is not how things work in Scala and it took a while to learn that :) But this is for another short post.