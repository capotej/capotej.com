---
layout: post
title: "an embedded key / value store for shell scripts"
date: 2012-10-07T10:06:00Z
comments: true
tags: ['shell scripting', 'databases']
---

UPDATE: this is now available as a [sub](http://github.com/37signals/sub) command, here: [kiev](http://github.com/capotej/kiev)

Cooked this up last night when I needed a simple key/value store for use in a shell script:

<!--more-->

```sh db.sh
#!/bin/sh

DBFILE=example.db

put(){
  echo "export kv_$1=$2" >> $DBFILE
}

del(){
  echo "unset kv_$1" >> $DBFILE
}

get(){
  source $DBFILE
  eval r=\$$(echo "kv_$1")
  echo $r
}

list(){
  source $DBFILE
  for i in $(env | grep "kv_" | cut -d= -f1 ); do
    eval r=\$$i; echo $(echo $i | sed -e 's/kv_//') $r;
  done
}

## cmd dispatch

if [ ${1:-0} == "set" ]; then
  put $2 $3
elif [ ${1:-0} == "get" ] ; then
  get $2
elif [ ${1:-0} == "list" ] ; then
  list
elif [ ${1:-0} == "del" ] ; then
  del $2
else
  echo "unknown cmd"
fi
```

Use it like so:


`$ ./db.sh set foo bar`

`$ ./db.sh get foo`

`$ ./db.sh set foo baz`

`$ ./db.sh get foo`

`$ ./db.sh del foo`

`$ ./db.sh list`


## How it works

Every time you update/set/delete a value, it writes a shell expression to an append-only log,
exporting a shell variable (key) with that value. By sourcing the file every time we read a value, we
replay the log, bringing our environment to a consistent state. Then, reading the value is just looking
up that dynamic variable (key) in our shell environment.
