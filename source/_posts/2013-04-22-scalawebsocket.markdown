---
layout: post
title: "ScalaWebSocket"
date: 2013-04-24 16:45
comments: true
tags:
- scala
- websocket
categories:  scala
---

### ScalaWebSocket

On April 21st I pushed the [ScalaWebSocket](https://github.com/pbuda/scalawebsocket) library to GitHub. But what is it? As the name suggests, it's WebSockets for Scala.

There are already a few implementations of WebSocket for Java, there is also the Scala project called [wCS](https://github.com/jfarcand/WCS) but all of them support Javaish style of passing the anonymous implementations of some kind of Listener interface. I didn't want that as I prefer the functional style of handlers.

ScalaWebSocket is a thin wrapper around [async-http-client](https://github.com/AsyncHttpClient/async-http-client) and it exposes a more Scala-like vocabulary to interact with WebSockets.

### Installation

To start using it in your project

{% codeblock Dependency in SBT %}
libraryDependncies += "eu.piotrbuda" %% "scalawebsocket" % "0.1.0"
{% endcodeblock %}

{% codeblock Dependency in Maven %}
<dependency>
  <groupId>eu.piotrbuda</groupId>
  <artifactId>scalawebsocket_2.10</artifactId>
  <version>0.1.0</version>
</dependency>
{% endcodeblock %}

### Examples

Usage of this library is very simple.

{% codeblock Open a WebSocket connection lang:scala %}
WebSocket().open("ws://echo.websocket.org/").sendText("text").close().shutdown()
{% endcodeblock %}

{% codeblock Listen for text messages lang:scala %}
WebSocket().open("ws://echo.websocket.org/")
.onTextMessage(msg => doSomethingWithMessage(msg))
{% endcodeblock %}

{% codeblock Add several listeners lang:scala %}
WebSocket().open("ws://echo.websocket.org/")
.onTextMessage(msg => doSomethingWithMessage(msg))
.onBinaryMessage(msg => doSomethingWithBinaryMessage(msg))
{% endcodeblock %}

### Future

I need this library to talk to SocketIO servers, so in version 0.2.0 I will implement some basic support for SocketIO.
 For now, please use it and report any issues you have. My goal for this library is to make WebSocket natural in Scala.
