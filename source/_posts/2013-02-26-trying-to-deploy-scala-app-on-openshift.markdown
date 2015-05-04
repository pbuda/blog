---
layout: post
title: "Trying to deploy Scala app on OpenShift"
date: 2013-02-26 10:35
comments: true
tags:
- scala
- openshift
- sbt
- spray
- cloud
categories: scala
---
### The beggining
This is a simple log of what I have done during past few days trying to deploy a simple app to [OpenShift](https://openshift.redhat.com/app/). I thought it would be quite easy but apparently this is beyond my knowledge. My application is a simple Hello World app. The only real requirement is that I wanted this app to compile with Scala 2.10.

### Setup
I won't discuss how to set up the application on [OpenShift](https://openshift.redhat.com/app/) (unless it is really needed). The only thing worth mentioning is that you will need a Do-It-Yourself type.

### SBT
There is no [SBT](http://www.scala-sbt.org/) on [OpenShift](https://openshift.redhat.com/app/). That's right, you have to get it yourself. [OpenShift](https://openshift.redhat.com/app/) however provides a nice way of storing things with its data directory (available at $OPENSHIFT_DATA_DIR) and it's action hooks are the way to go in this case. After some trial and error this is the script I came up with.
{%codeblock pre_build script%}
cd $OPENSHIFT_DATA_DIR

if [[ -d sbt ]]; then
  echo "SBT installed"
else
  curl -o sbt.tgz http://scalasbt.artifactoryonline.com/scalasbt/sbt-native-packages/org/scala-sbt/sbt/0.12.2/sbt.tgz
  tar zxvf sbt.tgz sbt
  rm sbt.tgz
fi
{%endcodeblock%}
This script basically downloads SBT from its site but only if SBT folder is not present in the data directory. Then the downloaded tgz archive gets unpacked and SBT is ready to be used.

### Building
Now this is the part I spent most time with and unfortunately I can't make it fully work :( There were several issues with building using [SBT](http://www.scala-sbt.org/), but after some time it almost worked. Here's the build script I developed that semi-works.
{% codeblock build script %}
SBT_PATH=$OPENSHIFT_DATA_DIR/sbt
SBT_DIR=$OPENSHIFT_DATA_DIR/.sbt
IVY_DIR=$OPENSHIFT_DATA_DIR/.ivy

cd $OPENSHIFT_REPO_DIR

$SBT_PATH/bin/sbt -sbt-dir $SBT_DIR -ivy $IVY_DIR start-script
{% endcodeblock %}
This took a while to figure out. Let's go line by line to see what everything is needed for. Lines 1, 2 and 3 define three variables that are required to run [SBT](http://www.scala-sbt.org/) (they could have been moved to pre_build script to keep everything SBT related in one place). 
* Line 1 simply defines the path to [SBT](http://www.scala-sbt.org/) directory.
* Line 2 is a variable with path to folder [SBT](http://www.scala-sbt.org/) uses to store its data. On [OpenShift](https://openshift.redhat.com/app/) you don't have permission to write to your home directory, hence a need for custom dir.
* Line 3 is like line 2 except for Ivy cache.
* Line 5 then switches to the repository directory. This is the place where your sources reside and this is mostly the place where you should start your build at.
* Line 7 is a simple invocation of the SBT with the predefined settings. I'm generating a start script here (using xsbt-start-script-plugin) and that's why I only include this goal.

This somewhat works. Considering the requirements we now have a project that uses Scala 2.10. But there are now two things to consider.

At first in pre_build I downloaded SBT 0.12.2 and in project build.settings I had SBT setup to 0.12.1. This triggered a compilation of 'compiler-interface'.
{% codeblock %}
remote: [info] Compiling 1 Scala source to /var/lib/openshift/512617424382ec272a0000b1/app-root/runtime/repo/target/scala-2.10/classes...
remote: [info] 'compiler-interface' not yet compiled for Scala 2.10.0. Compiling...
{% endcodeblock %}
But this failed. Compilation wouldn't finish and the process was killed. Small fix in build.properties and upping the actual [SBT](http://www.scala-sbt.org/) version to 0.12.2 fixed this - no more recompilation of 'compiler-interface'.

When you push your project to [OpenShift](https://openshift.redhat.com/app/) now, it downloads SBT, unpacks it and tries to build your project. And it fails with the same error as above.
{% codeblock Build fails after git push %}
remote: restart_on_add=false
remote: stop
remote: Done
remote: restart_on_add=false
remote: Running .openshift/action_hooks/pre_build
remote: pre_build
remote: SBT installed
remote: Running .openshift/action_hooks/build
remote: build
remote: /var/lib/openshift/512617424382ec272a0000b1/git/yatstaging2.git
remote: /var/lib/openshift/512617424382ec272a0000b1/app-root/runtime/repo
remote: [info] Loading project definition from /var/lib/openshift/512617424382ec272a0000b1/app-root/runtime/repo/project
remote: [info] Updating {file:/var/lib/openshift/512617424382ec272a0000b1/app-root/runtime/repo/project/}default-2353b7...
        [info] Resolving org.scala-sbt#precompiled-2_10_0;0.12.2 ...
remote: [info] Done updating.
remote: [info] Set current project to YAT Server (in build file:/var/lib/openshift/512617424382ec272a0000b1/app-root/runtime/repo/)
remote: [info] Updating {file:/var/lib/openshift/512617424382ec272a0000b1/app-root/runtime/repo/}default-e6271e...
remote: [info] Resolving org.scala-lang#scala-library;2.10.0 ...
remote: [info] Done updating.
remote: [debug]
remote: [debug] Initial source changes:
remote: [debug]   removed:Set()
remote: [debug] 	added: Set(/var/lib/openshift/512617424382ec272a0000b1/app-root/runtime/repo/src/main/scala/pl/apptile/yat/YAT.scala)
remote: [debug] 	modified: Set()
remote: [debug] Removed products: Set()
remote: [debug] Modified external sources: Set()
remote: [debug] Modified binary dependencies: Set()
remote: [debug] Initial directly invalidated sources: Set(/var/lib/openshift/512617424382ec272a0000b1/app-root/runtime/repo/src/main/scala/pl/apptile/yat/YAT.scala)
remote: [debug]
remote: [debug] Sources indirectly invalidated by:
remote: [debug] 	product: Set()
remote: [debug] 	binary dep: Set()
remote: [debug] 	external source: Set()
remote: [debug] Initially invalidated: Set(/var/lib/openshift/512617424382ec272a0000b1/app-root/runtime/repo/src/main/scala/pl/apptile/yat/YAT.scala)
remote: [debug] Recompiling all 1 sources: invalidated sources (1) exceeded 50.0% of all sources
remote: [info] Compiling 1 Scala source to /var/lib/openshift/512617424382ec272a0000b1/app-root/runtime/repo/target/scala-2.10/classes...
remote: [debug] Running cached compiler 18d45f0, interfacing (CompilerInterface) with Scala compiler version 2.10.0
remote: [debug] Calling Scala compiler with arguments  (CompilerInterface):
remote: [debug] 	-d
remote: [debug] 	/var/lib/openshift/512617424382ec272a0000b1/app-root/runtime/repo/target/scala-2.10/classes
remote: [debug] 	-bootclasspath
remote: [debug] 	/usr/lib/jvm/java-1.7.0-openjdk-1.7.0.9/jre/lib/resources.jar:/usr/lib/jvm/java-1.7.0-openjdk-1.7.0.9/jre/lib/rt.jar:/usr/lib/jvm/java-1.7.0-openjdk-1.7.0.9/jre/lib/sunrsasign.jar:/usr/lib/jvm/java-1.7.0-openjdk-1.7.0.9/jre/lib/jsse.jar:/usr/lib/jvm/java-1.7.0-openjdk-1.7.0.9/jre/lib/jce.jar:/usr/lib/jvm/java-1.7.0-openjdk-1.7.0.9/jre/lib/charsets.jar:/usr/lib/jvm/java-1.7.0-openjdk-1.7.0.9/jre/lib/netx.jar:/usr/lib/jvm/java-1.7.0-openjdk-1.7.0.9/jre/lib/plugin.jar:/usr/lib/jvm/java-1.7.0-openjdk-1.7.0.9/jre/lib/rhino.jar:/usr/lib/jvm/java-1.7.0-openjdk-1.7.0.9/jre/lib/jfr.jar:/usr/lib/jvm/java-1.7.0-openjdk-1.7.0.9/jre/classes:/var/lib/openshift/512617424382ec272a0000b1/app-root/data/.sbt/boot/scala-2.10.0/lib/scala-library.jar
remote: [debug] 	-classpath
remote: [debug] 	/var/lib/openshift/512617424382ec272a0000b1/app-root/runtime/repo/target/scala-2.10/classes
remote: /var/lib/openshift/512617424382ec272a0000b1/app-root/runtime/repo/.openshift/action_hooks/build: line 21:  7042 Killed            $SBT_PATH/bin/sbt -sbt-dir $SBT_DIR -ivy $IVY_DIR start-script
remote: Running .openshift/action_hooks/deploy
remote: deploy
remote: hot_deploy_added=false
remote: start
remote: Done
remote: start: missing job name
remote: Try `start --help' for more information.
remote: Running .openshift/action_hooks/post_deploy
remote: post_deploy
{% endcodeblock %}
The process is being killed anyway. This was driving me crazy, because no matter what I did the build always crashed. Then I tried SSH and run the script manually and… bam! It worked.
{% codeblock Build successful when run manually from SSH session %}
build
/var/lib/openshift/512617424382ec272a0000b1
/var/lib/openshift/512617424382ec272a0000b1/app-root/runtime/repo
[info] Loading project definition from /var/lib/openshift/512617424382ec272a0000b1/app-root/runtime/repo/project
[info] Set current project to YAT Server (in build file:/var/lib/openshift/512617424382ec272a0000b1/app-root/runtime/repo/)
[debug]
[debug] Initial source changes:
[debug]   removed:Set()
[debug] 	added: Set(/var/lib/openshift/512617424382ec272a0000b1/app-root/runtime/repo/src/main/scala/pl/apptile/yat/YAT.scala)
[debug] 	modified: Set()
[debug] Removed products: Set()
[debug] Modified external sources: Set()
[debug] Modified binary dependencies: Set()
[debug] Initial directly invalidated sources: Set(/var/lib/openshift/512617424382ec272a0000b1/app-root/runtime/repo/src/main/scala/pl/apptile/yat/YAT.scala)
[debug]
[debug] Sources indirectly invalidated by:
[debug] 	product: Set()
[debug] 	binary dep: Set()
[debug] 	external source: Set()
[debug] Initially invalidated: Set(/var/lib/openshift/512617424382ec272a0000b1/app-root/runtime/repo/src/main/scala/pl/apptile/yat/YAT.scala)
[debug] Recompiling all 1 sources: invalidated sources (1) exceeded 50.0% of all sources
[info] Compiling 1 Scala source to /var/lib/openshift/512617424382ec272a0000b1/app-root/runtime/repo/target/scala-2.10/classes...
[debug] Running cached compiler 12a6774, interfacing (CompilerInterface) with Scala compiler version 2.10.0
[debug] Calling Scala compiler with arguments  (CompilerInterface):
[debug] 	-d
[debug] 	/var/lib/openshift/512617424382ec272a0000b1/app-root/runtime/repo/target/scala-2.10/classes
[debug] 	-bootclasspath
[debug] 	/usr/lib/jvm/java-1.7.0-openjdk-1.7.0.9/jre/lib/resources.jar:/usr/lib/jvm/java-1.7.0-openjdk-1.7.0.9/jre/lib/rt.jar:/usr/lib/jvm/java-1.7.0-openjdk-1.7.0.9/jre/lib/sunrsasign.jar:/usr/lib/jvm/java-1.7.0-openjdk-1.7.0.9/jre/lib/jsse.jar:/usr/lib/jvm/java-1.7.0-openjdk-1.7.0.9/jre/lib/jce.jar:/usr/lib/jvm/java-1.7.0-openjdk-1.7.0.9/jre/lib/charsets.jar:/usr/lib/jvm/java-1.7.0-openjdk-1.7.0.9/jre/lib/netx.jar:/usr/lib/jvm/java-1.7.0-openjdk-1.7.0.9/jre/lib/plugin.jar:/usr/lib/jvm/java-1.7.0-openjdk-1.7.0.9/jre/lib/rhino.jar:/usr/lib/jvm/java-1.7.0-openjdk-1.7.0.9/jre/lib/jfr.jar:/usr/lib/jvm/java-1.7.0-openjdk-1.7.0.9/jre/classes:/var/lib/openshift/512617424382ec272a0000b1/app-root/data/.sbt/boot/scala-2.10.0/lib/scala-library.jar
[debug] 	-classpath
[debug] 	/var/lib/openshift/512617424382ec272a0000b1/app-root/runtime/repo/target/scala-2.10/classes
[debug] Scala compilation took 34.206682197 s
[debug] Step 2 changed sources and immdediate dependencies:
[debug] 	Set(/var/lib/openshift/512617424382ec272a0000b1/app-root/runtime/repo/src/main/scala/pl/apptile/yat/YAT.scala)
[debug] Non-trivial strongly connected components:
[debug]
[debug] Step 2 invalidated sources:
[debug] 	Set(/var/lib/openshift/512617424382ec272a0000b1/app-root/runtime/repo/src/main/scala/pl/apptile/yat/YAT.scala)
[info] Wrote start script for mainClass := Some(pl.apptile.yat.YAT) to /var/lib/openshift/512617424382ec272a0000b1/app-root/runtime/repo/target/start
{% endcodeblock %}
This is quite puzzling and I can't figure out why automatic build doesn't work. What's worth mentioning though is that it does work when you change Scala to 2.9.2. But it's against requirements and hence I stopped at this point, unable to continue.

### Summary
While I was really happy that most of the things were quite easy to start with, inability to compile the sources makes it a no-no for further work, like actually starting a spray-can server. My application was a Hello World app and if this can't compile… Sorry [OpenShift](https://openshift.redhat.com/app/), for now I'm switching to [CloudFoundry](http://www.cloudfoundry.com/) to see how it fares.