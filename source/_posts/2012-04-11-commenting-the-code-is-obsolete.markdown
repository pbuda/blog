---
layout: post
title: "Commenting the code is obsolete"
date: 2012-04-11 12:01
comments: true
categories: 
---

Some time ago I had a few visits from fellow developers asking me questions about that I supposedly created. Since I obviously created some of that code months or years ago, I often couldn't answer their questions because either my explanations were false (changed requirements) or I didn't even know about that particular case (new code). It struck me that they came to me to ask these questions because I was placed as the author of the class in Javadoc...

More recently I've been in situation when our team leader told us that we're not putting enough comments in our classes and the we could at least provide the @author Javadoc comment (or ASDoc in that case) so that everyone knows who to talk to about this code.

Revelation came to me sometime after that statement - why the hell do I need author information in a file?! I am checking in my code to a VCS which provides that information out of the box, moreover when the code changes it can report who changed the code! @author becomes meaningless after some time, VCS history does not - so use VCS instead.

This also applies to other comments (IMO). Instead of creating some code and heavily comment what it does (bad) I prefer to write smaller methods that tell what they're doing (good). The only thing I think a comment is necessary for is method arguments, but I could also omit them when I call my arguments with meaningful names. This does not apply to function arguments, because I have to comment that function arguments. But that can also lead to something like this:

{% codeblock lang:actionscript %}
/**
* @param callback function to call with signature 
*                 <code>function(arg1:String, arg2:Object):void</code>
*/
public function myFunction(callback:Function):void {
  callback(arg1, arg2, arg3)
}
{% endcodeblock %}

Summary: instead of extensive comments, write better code and use tools that you're given.