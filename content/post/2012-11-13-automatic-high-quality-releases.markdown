---
layout: post
title: "automatic high quality releases"
date: 2012-11-13T21:31:00Z
comments: true
categories: ["shell scripting", "finatra"]
---

Recently, I invested some time into automating some of the work that goes into a [Finatra](http://github.com/capotej/finatra#readme) release.

The work consists of updating:

* The version in the XML fragment of the main [README.markdown](https://github.com/finatra/https://github.com/capotej/finatra/blob/master/README.markdown)

* The version in the [pom.xml](https://github.com/capotej/finatra-example/blob/master/pom.xml) of the [example app](http://github.com/capotej/finatra-example)

* Any API changes in the [example app](http://github.com/capotej/finatra-example)

* The version in the template pom.xml of the app generator

* The generated unit test of the app generator that demonstrates testing any new API features

* Any API changes inside the app template of the app generator

Using [sub](https://github.com/37signals/sub#readme), I was able to create a `finatra` [command](https://github.com/capotej/finatra/tree/master/script/finatra) that automated all of the above based on a single template, which also happens to be the [main unit test](https://github.com/capotej/finatra/blob/master/src/test/scala/com/twitter/finatra/ExampleSpec.scala). This ensures that the README, the example app, and the app generator never fall out of sync with the frameworks API.


Last week we released [1.1.0](https://github.com/capotej/finatra/commit/37c81957271dde77d4c3f6361bbae705a5142c89), and the README was [completely generated](https://github.com/capotej/finatra/commit/913d0ed5bfa18c903feb5779d4d8b9d87703b6c5), as was the [example app](https://github.com/capotej/finatra-example/commit/dbc82908360f3cb4cfc4388c28f593f17258fab2). Not to mention, all generated apps would also contain the latest templates and examples!

![](https://i0.kym-cdn.com/photos/images/original/000/021/073/1254172884282.jpg?1254173845)

Let's dive into how it all works:

## The source of truth

I annotated our main unit test with special tokens, like so:

```scala ExampleAppSpec.scala
class ExampleSpec extends SpecHelper {

  /* ###BEGIN_APP### */

  class ExampleApp extends Controller {

    /**
     * Basic Example
     *
     * curl http://localhost:7070/hello => "hello world"
     */
    get("/") { request =>
      render.plain("hello world").toFuture
    }

  }

  val app = new ExampleApp

  /* ###END_APP### */


  /* ###BEGIN_SPEC### */

  "GET /hello" should "respond with hello world" in {
    get("/")
    response.body should equal ("hello world")
  }

  /* ###END_SPEC### */
}
```

Using the special `/* ### */` comments, the main app and its test can be extracted from the code of our test.

## The app generator

Now that we have our "template", we can build our app generator to use it. I customized [base](http://capotej.com/blog/2012/11/01/base-a-scala-project-generator/) and ended up with: [script/finatra/libexec/finatra-new](https://github.com/capotej/finatra/blob/master/script/finatra/libexec/finatra-new)

You can then run:

```sh
$ ./finatra new com.example.myapp
```

and it will generate ```myapp/``` based on the tested example code from the test suite above.

## The example app

The [example app](https://github.com/capotej/finatra-example#readme) is just a generated app using the latest app generator:

```sh

#!/bin/bash
# Usage: finatra update-example
# Summary: generates the example app from the template

set -e

source $_FINATRA_ROOT/lib/base.sh

tmpdir=$(mktemp -d /tmp/finatra_example.XXX)

$_FINATRA_ROOT/bin/finatra new com.twitter.finatra_example $tmpdir

cp -Rv $tmpdir/finatra_example/ $EXAMPLE_REPO

rm -rf $tmpdir

cd $EXAMPLE_REPO && mvn test

```

This also tests the app generator and the generated app!

## Updating the README

Lastly, there's a [command](https://github.com/capotej/finatra/blob/master/script/finatra/libexec/finatra-update-readme) for updating the README with the new example and version number.
