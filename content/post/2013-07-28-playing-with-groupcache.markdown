---
layout: post
title: "Playing with groupcache"
date: 2013-07-28T14:49:00Z
comments: true
categories: ["go", "databases", "distributed computing"]
---

This week, [@bradfitz](http://twitter.com/bradfitz) (of memcached fame) released [groupcache](http://github.com/golang/groupcache) at OSCON 2013. I'm already a big fan of [memcached](http://memcached) and [camlistore](http://camlistore.org), so I couldn't wait to download it and kick the tires.

By the way, I **strongly** recommend you go through the [slides](http://talks.golang.org/2013/oscon-dl.slide#1) and [README](http://github.com/golang/groupcache) before going further.

## What groupcache isn't
After downloading it (without reading the [slides](http://talks.golang.org/2013/oscon-dl.slide#1)), I instinctively searched around for how to actually start the server(s), only to find nothing. Turns out, groupcache is more of a _library_ with a server built in, rather than a traditional standalone server. Another important consideration is that theres **no support for set/update/evict operations**, all you get is GET. Really fast, consistent, distributed GET's.

## What it is
Once you realize that groupcache is more of a **smart, distributed LRU cache**, rather than an outright memcached replacement, it all makes much more sense. Especially considering what it was built for, caching immutable file blobs for [dl.google.com](http://dl.google.com).

## How to use it
For groupcache to work, you have to give it a closure in which: given a ```key```, fill up this ```dest``` buffer with the bytes for the value of that key, from however you store them. This could be hitting a database, a network filesystem, anything. Then you create a groupcache ```group``` object, which knows the addresses of all the other groupcache instances. This is pluggable, so you can imagine rigging that up to zookeeper or the like for automatic node discovery. Finally, you start groupcache up by using go's built in ```net/http``` and a ```ServeHTP``` provided by the previously constructed ```group``` object.

## Running the demo
In order to really try out groupcache, I realized I needed to create a mini test infrastructure, consisting of a slow database, frontends, and a client. Visit the [Github Repo](http://github.com/capotej/groupcache-db-experiment) for more details. This is what the topology looks like:
![groupcache topology](https://raw.github.com/capotej/groupcache-db-experiment/master/topology.png)

#### Setup
1. ```git clone git@github.com:capotej/groupcache-db-experiment.git```
2. ```cd groupcache-db-experiment```
3. ```sh build.sh```

#### Start database server
1. ```cd dbserver && ./dbserver```

#### Start Multiple Frontends
1. ```cd frontend```
2. ```./frontend -port 8001```
3. ```./frontend -port 8002```
4. ```./frontend -port 8003```

#### Use the CLI to play around

Let's set a value into the database:

    ./cli -set -key foo -value bar

Now get it out again to make sure it's there:

    ./cli -get -key foo

You should see ```bar``` as the response, after about a noticeable, 300ms lag.

Let's ask for the same value, via cache this time:

    ./cli -cget -key foo

You should see on one of the frontend's output, the key ```foo``` was requested, and in turn requested from the database. Let's get it again:

    ./cli -cget -key foo

You should have gotten this value instantly, as it was served from groupcache.

Here's where things get interesting; Request that same key from a different frontend:

    ./cli -port 9002 -cget -key foo

You should still see ```bar``` come back instantly, even though this particular groupcache node did not have this value. This is because groupcache knew that 9001 had this key, went to that node to fetch it, then cached it itself. **This is groupcache's killer feature**, as it avoids the common thundering herd issue associated with losing cache nodes.

#### Node failure
Let's simulate single node failure, find the "owner" of key ```foo``` (this is going to be the frontend that said "asking for foo from dbserver"), and kill it with Ctrl+C. Request the value again:

    ./cli -cget -key foo

It'll most likely hit the dbserver again (unless that particular frontend happens to have it), and cache the result on one of the other remaining frontends. As more clients ask for this value, it'll spread through the caches organically. When that server comes back up, it'll start receiving other keys to share, and so on. The fan out is explained in more detail on this [slide](http://talks.golang.org/2013/oscon-dl.slide#47).

## Conclusion / Use cases
Since there is no support (by design) for eviction or updates, groupcache is a really good fit with read heavy, immutable content. Some use cases:

   * Someone like [Github](http://github.com) using it to cache blobrefs from their file servers
   * Large websites using it as a CDN (provided their assets were unique ```logo-0492830483.png```)
   * Backend for [Content-addressable storage](http://en.wikipedia.org/wiki/Content-addressable_storage)

Definitely a clever tool to have in the distributed systems toolbox.

_Shout out to professors [@jmhodges](http://twitter.com/jmhodges) and [@mrb_bk](http://twitter.com/mrb_bk) for proof reading this project and post_
