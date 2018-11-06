---
layout: post
title: "Using Rack applications inside GWT Hosted mode"
date: 2009-07-19T18:51:00Z
comments: false
permalink: /post/145035194/using-rack-applications-inside-gwt-hosted-mode
tags:
---



This guide will show you how you can use JRuby to run any Rack application inside Google Web Toolkit’s (GWT) hosted mode server so your interface and your backend are of the Same Origin.

BackgroundGWT has two ways of interacting with a server: GWT Remote Procedure Call (RPC) and plain HTTP (XHR). GWT-RPC is a high level library designed for interacting with server-side Java code. GWT-RPC implements the GWT Remote Service interface allowing you to call those methods from the user interface. Essentially, GWT handles the dirty work for you. However, it only works on Java backends that can implement that interface. Since most of my backends are Sinatra/Rack applications, I’ll be using the plain HTTP library.

The problemDue to the restriction of the [Same Origin](http://en.wikipedia.org/wiki/Same_origin_policy) policy, the interface served out of GWT’s development, or Hosted Mode server can only make requests back to itself. If you were using real servlets or GWT’s RemoteService this wouldn’t be an issue; but since Rack applications listen on their own port, you cannot make requests from GWT to our application without resorting to something like JSONP or server-side proxying. This leaves you having to compile our interface to HTML/JS/CSS, which is lengthy process, and serve it from the origin of the Rack application to see our changes.

The solutionSince I wanted to develop using GWT’s development environment with a Rack backend, I devised a way to use jruby-rack to load arbitrary Rack applications alongside our interface.

First let’s setup our environment:

###Download and unpack the latest GWT for your platform (mine’s being linux)

```sh
wget http://google-web-toolkit.googlecode.com/files/gwt-linux-1.7.0.tar.bz2
tar -xvjpf gwt-linux-1.7.0.tar.bz2cd gwt-linux-1.7.0
```

###Download the latest jruby-complete.jar

```sh
wget http://repository.codehaus.org/org/jruby/jruby-complete/1.3.1/jruby-complete-1.3.1.jar
mv jruby-complete-1.3.1.jar jruby-complete.jar
```

###Download the latest jruby-rack.jar

```sh
wget http://repository.codehaus.org/org/jruby/rack/jruby-rack/0.9.4/jruby-rack-0.9.4.jar
mv jruby-rack-0.9.4.jar jruby-rack.jar
```

###Create an app with webAppCreator
```sh
./webAppCreator -out MySinatra com.example.MySinatra
cd MySinatra
```


###Package gem dependencies
In order for this to work you have to package any gem dependencies your backend needs (sinatra, in our case) as jars within your application. For Sinatra it looks like this:

```sh
java -jar jruby-complete.jar -S gem install -i ./sinatra sinatra --no-rdoc --no-ri
jar cf sinatra.jar -C sinatra .
```

###Add jruby-complete.jar, jruby-rack.jar, sinatra.jar (and any other jars you’ve created) to the libs target of your build.xml

```xml
<target name="libs" description="Copy libs to WEB-INF/lib">
  <mkdir dir="war/WEB-INF/lib" />
  <copy todir="war/WEB-INF/lib" file="${gwt.sdk}/gwt-servlet.jar" />
  <!-- Add any additional server libs that need to be copied -->
  <copy todir="war/WEB-INF/lib" file="${gwt.sdk}/jruby-complete.jar" />
  <copy todir="war/WEB-INF/lib" file="${gwt.sdk}/jruby-rack.jar" />
  <copy todir="war/WEB-INF/lib" file="${gwt.sdk}/sinatra.jar" />
</target>
```

###Add these lines right after <web-app> in war/WEB-INF/web.xml
```xml
<context-param>
  <param-name>rackup</param-name>
  <param-value>
    require 'rubygems'
    require './lib/sinatra_app'
    map '/api' do
      run MyApp 
    end
  </param-value>
</context-param>
<filter>
  <filter-name>RackFilter</filter-name>
  <filter-class>org.jruby.rack.RackFilter</filter-class>
</filter>
<filter-mapping>
  <filter-name>RackFilter</filter-name>
  <url-pattern>/api/*</url-pattern>
</filter-mapping>
<listener>
  <listener-class>org.jruby.rack.RackServletContextListener</listener-class>
</listener>
```

Note: All you’re doing here is passing the contents of a config.ru file into the `<param-value>` element for the `<context-param>` (make sure this is HTML encoded!). This states that any request to /api is to be handled by your Sinatra application and not GWT’s Hosted mode servlet.

###Create your Sinatra backend and place it in war/WEB-INF/lib/sinatra_app.rb

```ruby
require 'sinatra'
require 'open-uri'
class MyApp < Sinatra::Base
  get '/showpage' do
    open('http://www.yahoo.com').read
  end

  get '/helloworld' do
    'hello world'
  end
end
```

###Run your new awesome setup
`ant hosted`

Now when navigate to [http://localhost:8888/api/helloworld](http://localhost:8888/api/helloworld) or [http://localhost:8888/api/showpage](http://localhost:8888/api/showpage) you should see the Sinatra application being served via GWT.
