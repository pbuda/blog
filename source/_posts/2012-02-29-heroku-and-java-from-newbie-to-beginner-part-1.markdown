---
layout: post
title: "Heroku and Java - from newbie to beginner, part 1"
date: 2012-02-29 16:47
comments: true
categories: 
---
Recently I've heard that [Heroku](http://www.heroku.com) allows deployment of Java applications in it's Cedar stack. Having no real software idea I decided I'll give it a try and just configure SOMETHING to work on Heroku.

I have some kind of crush on ReST (I still want to learn it and practice it) so I've decided my first application will be a simple hello world using [Jersey](http://jersey.java.net) (JAX-RS implementation). So I started a [project](https://github.com/pbuda/recaps) on GitHub and started setting up Heroku CLI.

### Setting up Heroku CLI
Heroku is now easy to set up. I remember when it required Ruby env and my first encounter with Ruby was not so great (there was no installer of any sort so it was all manual - and I'm lazy) so I gave up on Heroku back then. But now installing it is a breeze - simply go to [Heroku Toolbelt](https://toolbelt.herokuapp.com) and download version for your platform. I have now set it up both on Linux Mint and Windows 7 and it works great.

### Setting up project for Heroku
My project is called [recaps](ttps://github.com/pbuda/recaps) - it's supposed to be yet another ticket management system. But that's irrelevant for now. The most important thing is that in order for Heroku to discover that our application is a Java application pom.xml file must be present. That's because Heroku uses [Maven 3](http://maven.apache.org) to build Java applications.

So to actually begin any work with Heroku you need a simple pom.xml file. In my case I've added a separate module for the application, so my main pom looks like this:

{% codeblock lang:xml %}
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.github.pbuda.recaps</groupId>
    <artifactId>recaps</artifactId>
    <packaging>pom</packaging>
    <version>0.0.1-SNAPSHOT</version>

    <inceptionYear>2012</inceptionYear>

    <developers>
        <developer>
            <name>Piotr Buda</name>
            <email>pibuda@gmail.com</email>
            <timezone>+1</timezone>
        </developer>
    </developers>

    <licenses>
        <license>
            <name>Apache License, version 2.0</name>
            <url>http://www.apache.org/licenses/LICENSE-2.0.html</url>
        </license>
    </licenses>

    <modules>
        <module>webmodule</module>
    </modules>

    <build>
        <pluginManagement>
            <plugins>
                <plugin>
                    <groupId>org.apache.maven.plugins</groupId>
                    <artifactId>maven-compiler-plugin</artifactId>
                    <configuration>
                        <source>1.6</source>
                        <target>1.6</target>
                    </configuration>
                </plugin>
            </plugins>
        </pluginManagement>
    </build>

</project>
{% endcodeblock %}

Then there is the web module, just for the sake of splitting projects:

{% codeblock lang:xml %}
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>com.github.pbuda.recaps</groupId>
        <artifactId>recaps</artifactId>
        <version>0.0.1-SNAPSHOT</version>
    </parent>

    <artifactId>webmodule</artifactId>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-dependency-plugin</artifactId>
                <version>2.4</version>
                <executions>
                    <execution>
                        <id>copy-dependencies</id>
                        <phase>package</phase>
                        <goals>
                            <goal>copy-dependencies</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
    <dependencies>
        <dependency>
            <groupId>com.sun.jersey</groupId>
            <artifactId>jersey-server</artifactId>
            <version>1.12</version>
        </dependency>
        <dependency>
            <groupId>com.sun.jersey</groupId>
            <artifactId>jersey-core</artifactId>
            <version>1.12</version>
        </dependency>
        <dependency>
            <groupId>com.sun.jersey</groupId>
            <artifactId>jersey-grizzly2</artifactId>
            <version>1.12</version>
        </dependency>
    </dependencies>

</project>
{% endcodeblock %}

My first attempt at running my sample project concentrated on setting up Jersey. After checking out docs I decided I'll use Grizzly2 HTTP server just because it's very easy to set up. I've basically pasted the docs tutorial into main Main class. There were some necessary differences, because for example port of the server is dynamically assigned by Heroku. So after very few changes, the resulting Main class looks like this:

{% codeblock lang:java %}
/*
 * Copyright 2012 Piotr Buda
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *     http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */

package com.github.pbuda.recaps;

import com.sun.jersey.api.container.grizzly2.GrizzlyServerFactory;
import com.sun.jersey.api.core.PackagesResourceConfig;
import com.sun.jersey.api.core.ResourceConfig;
import org.glassfish.grizzly.http.server.HttpServer;

import javax.ws.rs.core.UriBuilder;
import java.io.IOException;
import java.net.URI;

/**
 * Created by IntelliJ IDEA.
 * User: pbu
 * Date: 28.02.12
 * Time: 21:01
 * To change this template use File | Settings | File Templates.
 */
public class Main {
    private static URI getBaseURI(String hostname, int port) {
        return UriBuilder.fromUri("http://0.0.0.0/").port(port).build();
    }

    protected static HttpServer startServer(URI uri) throws IOException {
        System.out.println("Starting grizzly...");
        ResourceConfig rc = new PackagesResourceConfig("com.github.pbuda.recaps");
        return GrizzlyServerFactory.createHttpServer(uri, rc);
    }

    public static void main(String[] args) throws IOException {
        URI uri = getBaseURI(System.getenv("HOSTNAME"), Integer.valueOf(System.getenv("PORT")));
        HttpServer httpServer = startServer(uri);
        System.out.println(String.format("Jersey app started with WADL available at "
                + "%sapplication.wadl\nTry out %shelloworld\nHit enter to stop it...",
                uri, uri));
        while(true) {
            System.in.read();
        }
    }
}
{% endcodeblock %}

That starts up the server and registers some resources with it.

### Some Grizzly tricks

Firstly, the GrizzlyServerFactory.createHttpServer method accepts an URI which has to begin witch schema name - in this case http://. Then it has to specify host name, which at first I set up to the application name on herokuapp.com. This didn't work, but Heroku told me nicely about it: there is a notification in logs that server should bind to 0.0.0.0, so I changed the URI to http://0.0.0.0.

Secondly, the Jersey example waited for a key press to terminate the server. Unfortunately Heroku printed a message which was then passed to the application somehow and the server was terminated. To resolve this, I wrapped the System.in.read() in an endless while loop.

This of course is not the best solution, but it worked, or so it seemed. After a few hours I checked the logs of the application and they said that the application went from up to down. So I've decided to switch from Grizzly to Jetty, but that's out of topic for this post :)

Before pushing all this to Heroku I also added a Procfile:

{% codeblock %}
web:    java -cp webmodule/target/classes:webmodule/target/dependency/* com.github.pbuda.recaps.Main
{% endcodeblock %}

After pushing to Heroku the application was build and started, and request to http://growing-dawn-9158.herokuapp.com/helloworld produced some output (in this case a simple 'Message' message). Job well done :)

### Mistakes I've made and learnt from
Firstly, I forgot to add the Maven Dependency plugin, but I resolved that one before pushing to Heroku. Without it configured I couldn't add dependencies to classpath, which in turn produced ClassNotFound exceptions. It didn't occur to me at first it was required, but then I looked at Heroku example and fixed it easily.

Secondly, I didn't know that web dynos time out. After successful deployment I was sure the application was running, but because of time out, logs said the application went down. Because I wasn't aware of the fact that web dynos time out I suspected that Grizzly was simply interrupted somehow so I've decided to move to Jetty. But it happened to Jetty implementation too, so I started digging and I found relevant information :)

### Summary

I think that Heroku is great. It's free Java hosting, something that is NOT popular and yet they did it and it works quite nicely. I once tried Google App Engine, but the experience wasn't great (mind you it was some long time ago) so I've decided to give Heroku a chance and because it was actually quite simple to set up the application I think I'll stick to it for a while and play with the platform - look at all those plugins :)