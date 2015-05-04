---
layout: post
title: "Scala and Android: AsyncTask implementation problem"
date: 2012-12-12 13:06
comments: true
categories: 
---
So I recently started some really simple Scala development for Android. However because I'm a very lucky person, I stumbled upon my first problem the very first moment I tried doing something nice.

There is a neat way for doing things in Android with AsyncTasks - this cool utility abstracts away the need for creation and management of threads for invoking stuff that would block UI thread. For more info just go to AsyncTask documentation.

So I tried doing something with this cool AsyncTask and here's the class definition:
{% codeblock lang:java %}
class ObtainRequestTokenTask() extends AsyncTask[String, Void, String] {
...
override protected def doInBackground(p1: String*): String = {
...
}
}
{% endcodeblock %}
The major problem here is that this thing compiles. Running this on Android emulator I got an exception suggesting that doInBackground is not implemented (it's abstract in AsyncTask)... So where's the problem?

Googling about this I found [this bug](https://issues.scala-lang.org/browse/SI-1459). I won't go into detail, however to make this code work you have to actually change the signature of doInBackground. Instead of String input, you need AnyRef and just cast params to the desired class.

For me this now looks something like this:
{% codeblock lang:java %}
class ObtainRequestTokenTask() extends AsyncTask[AnyRef, Void, String] {
...
override protected def doInBackground(p1: AnyRef*): String = {
p1.head.asInstanceOf[String]
...
}
}
{% endcodeblock %}