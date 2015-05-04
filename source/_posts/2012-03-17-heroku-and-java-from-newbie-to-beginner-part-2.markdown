---
layout: post
title: "Heroku and Java - from newbie to beginner, part 2"
date: 2012-03-17 1:13
comments: true
categories: 
---
### The problem

So after a few days I could get back to my little Recaps project. I started with checking logs and found something like this:

{% codeblock %}
2012-03-04T01:52:51+00:00 heroku[web.1]: Idling
2012-03-04T01:52:53+00:00 heroku[web.1]: Stopping process with SIGTERM
2012-03-04T01:53:03+00:00 heroku[web.1]: Error R12 (Exit timeout) -> Process failed to exit within 10 seconds of SIGTERM
2012-03-04T01:53:03+00:00 heroku[web.1]: Stopping process with SIGKILL
{% endcodeblock %}

I don't know what you think but whenever I see 'Error' in my logs I get concerned. So I decided to remove that nasty line. What transpired was not pleasant.

### The solution

This seems like a simple problem. I started up with finding what this SIGTERM is all about. I knew it was Linux signal, I just wanted to know what Heroku is actually doing. So basically sometimes Heroku just sends SIGTERM to your process so that it's allowed to gracefully shut down. This is very simple.

As I mentioned in my last post, I decided to use Jetty instead of Grizzly. At first I decided to use jetty-runner to run my web application and it worked fine, resources were scanned as Jersey servlet started up. Deploying to Heroku was also easy and with modified Procfile the application started up.

Nevertheless the application did not react correctly to SIGTERM, so without delving deeper in jetty-runner configuration, I decided to just use the embedded Jetty server.
It's very simple and running foreman start made the application actually start. So without further consideration I just deployed the changed application to Heroku. To check whether the error appears again, after first startup I just did heroku restart and connected to logs in another terminal. But the exit timeout error message was still there. My mistake there - I did not test the application whether it will exit properly when using foreman. So again, foreman start and then ctrl+c just to see what happens (later I tried kill -s TERM procand got similar output):

{% codeblock %}
pbu@pbudesk ~/recaps $ foreman start
21:57:27 web.1     | started with pid 9603
21:57:27 web.1     | 0    [main] INFO  org.eclipse.jetty.server.Server  - jetty-8.1.1.v20120215
21:57:27 web.1     | 110  [main] INFO  org.eclipse.jetty.webapp.StandardDescriptorProcessor  - NO JSP Support for /, did not find org.apache.jasper.servlet.JspServlet
21:57:27 web.1     | 132  [main] INFO  org.eclipse.jetty.server.handler.ContextHandler  - started o.e.j.w.WebAppContext{/,file:/home/pbu/Devel/IdeaProjects/recaps/webmodule/src/main/webapp/},webmodule/src/main/webapp
21:57:27 web.1     | 133  [main] INFO  org.eclipse.jetty.server.handler.ContextHandler  - started o.e.j.w.WebAppContext{/,file:/home/pbu/Devel/IdeaProjects/recaps/webmodule/src/main/webapp/},webmodule/src/main/webapp
21:57:27 web.1     | 183  [main] INFO  org.eclipse.jetty.server.AbstractConnector  - Started SelectChannelConnector@0.0.0.0:5000
^CSIGINT received
21:57:57 system    | sending SIGTERM to all processes
21:57:57 system    | sending SIGTERM to pid 9603
21:57:57 web.1     | process terminated
pbu@pbudesk ~/recaps $
{% endcodeblock %}

OK, so when foreman receives SIGINT it sends SIGTERM to all processes, cool - probably Heroku dynos behave the same. Still, it wasn't a graceful shutdown, but Jetty has a [graceful shutdown section](http://docs.codehaus.org/display/JETTY/How+to+gracefully+shutdown) that mentions two nice properties: gracefulShutdown and stopAtShutdown. The modified class looks like this:

{% codeblock lang:java %}
public class Serve {
    public static void main(String[] args) throws Exception {
        int port = Integer.valueOf(System.getenv("PORT"));

        Server jetty = new Server(port);

        WebAppContext context = new WebAppContext();
        context.setContextPath("/");
        String webapp = "webmodule/src/main/webapp";
        context.setWar(webapp);
        context.setResourceBase(webapp);

        jetty.setHandler(context);

        jetty.setGracefulShutdown(1000);
        jetty.setStopAtShutdown(true);

        jetty.start();
        jetty.join();
    }
}
{% endcodeblock %}

Running foreman again and using ctrl+c proves this to be working! Great!

{% codeblock %}
pbu@pbudesk ~/recaps $ foreman start
22:11:47 web.1     | started with pid 9863
22:11:47 web.1     | 0    [main] INFO  org.eclipse.jetty.server.Server  - jetty-8.1.1.v20120215
22:11:47 web.1     | 110  [main] INFO  org.eclipse.jetty.webapp.StandardDescriptorProcessor  - NO JSP Support for /, did not find org.apache.jasper.servlet.JspServlet
22:11:47 web.1     | 131  [main] INFO  org.eclipse.jetty.server.handler.ContextHandler  - started o.e.j.w.WebAppContext{/,file:/home/pbu/Devel/IdeaProjects/recaps/webmodule/src/main/webapp/},webmodule/src/main/webapp
22:11:47 web.1     | 132  [main] INFO  org.eclipse.jetty.server.handler.ContextHandler  - started o.e.j.w.WebAppContext{/,file:/home/pbu/Devel/IdeaProjects/recaps/webmodule/src/main/webapp/},webmodule/src/main/webapp
22:11:48 web.1     | 183  [main] INFO  org.eclipse.jetty.server.AbstractConnector  - Started SelectChannelConnector@0.0.0.0:5000
^C22:11:49 web.1     | 1969 [Thread-1] INFO  org.eclipse.jetty.server.Server  - Graceful shutdown SelectChannelConnector@0.0.0.0:5000
22:11:49 web.1     | 1970 [Thread-1] INFO  org.eclipse.jetty.server.Server  - Graceful shutdown o.e.j.w.WebAppContext{/,file:/home/pbu/Devel/IdeaProjects/recaps/webmodule/src/main/webapp/},webmodule/src/main/webapp
SIGINT received
22:11:49 system    | sending SIGTERM to all processes
22:11:49 system    | sending SIGTERM to pid 9863
22:11:50 web.1     | 2982 [Thread-1] INFO  org.eclipse.jetty.server.handler.ContextHandler  - stopped o.e.j.w.WebAppContext{/,file:/home/pbu/Devel/IdeaProjects/recaps/webmodule/src/main/webapp/},webmodule/src/main/webapp
22:11:50 web.1     | process terminated
pbu@pbudesk ~/recaps $
So off to deploy it to the cloud! Again, deploy, heroku restart and watch logs... but it doesn't work.
{% endcodeblock %}

### Different way

After initial failure I've tried a different approach. I found out that you can register shutdown hooks - very easy thing. To do that, just register a new thread with Runtime.getRuntime().addShutdownHook(Thread) method:

{% codeblock lang:java %}
public class Serve {
    public static void main(String[] args) throws Exception {
        Runtime.getRuntime().addShutdownHook(new Thread() {
            @Override
            public void run() {
                System.out.println("Shutting down by shutdown hook");
            }
        });
        int port = Integer.valueOf(System.getenv("PORT"));

        Server jetty = new Server(port);

        WebAppContext context = new WebAppContext();
        context.setContextPath("/");
        String webapp = "webmodule/src/main/webapp";
        context.setWar(webapp);
        context.setResourceBase(webapp);

        jetty.setHandler(context);

        jetty.setGracefulShutdown(1000);
        jetty.setStopAtShutdown(true);

        jetty.start();
        jetty.join();
    }
}
{% endcodeblock %}

Final test with foreman proves it to be working too, however once again it doesn't work on Heroku.

At this point, I have no idea how to get rid of that timeout. It is not very important, I just wanted to check whether I can somehow react to it, but to no avail. For now I guess I'll just contact Heroku, maybe they'll help. Another option could be trying embedded Tomcat, but maybe at a later time. For now, I have other things to do, like checking out [Jelastic](http://jelastic.com/).