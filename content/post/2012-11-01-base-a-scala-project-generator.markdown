---
layout: post
title: "base: a scala project generator"
date: 2012-11-01T14:39:00Z
comments: true
tags: ["efficiency", "scala", "shell scripting"]
---

Finally got tired of copy pasting other projects and gutting them to make new ones, so I created [base](http://github.com/capotej/base), a shell command that creates new scala projects.

<!--more-->

Creating the project:

```sh
$ base new com.capotej.newproj
creating project: newproj
  creating App.scala
  creating AppSpec.scala
  creating pom.xml
  creating .gitignore
  creating .travis.yml
  creating LICENSE
  creating README.markdown
Done! run mvn scala:run to run your projec
```

Based on the package name, it infered that the project name is ```newproj``` and created the project under that folder. Let's build and run it:

```sh
$ cd newproj
$ mvn compile scala:run
(... maven output ...)
hello world
```

This uses the new incremental compiler for maven, [zinc](http://github.com/typesafehub/zinc), which dramatically speeds up compile times (except for the first time you run it). It also sets you up with the latest scalatest maven plugin, which gives you sweet looking test output, like so:

![](https://i.imgur.com/qyyem.png)

See the base [README](http://github.com/capotej/base#readme) for installation instructions.
