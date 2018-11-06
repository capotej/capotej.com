---
title: "Tmux Session Coloring"
date: 2014-02-24T00:00:00Z
comments: true
tags: ["shell scripting", "tmux"]
---

Recently, I've really gotten into tmux for managing all my terminal sessions/windows. Not so much for the panes, but more for keeping a highly contextual environment per project or task.

As the number of sessions grew, they became difficult to tell apart. For a few days now, I've had the idea of hashing the name of the session into a unique color, so that every session had its own `status-bg` color.

First, the `tmuxHashColor` function:

```sh
tmuxHashColor() {
  local hsh=$(echo $1 | cksum | cut -d ' ' -f 1)
  local num=$(expr $hsh % 255)
  echo "colour$num"
}
```

In our `ns` function (new session), we hash the supplied session name to a color, then use `tmux send-keys` to set its `status-bg` color to it:

```sh
ns() {
  if [ -z $1 ]; then
    1=$(basename $(pwd))
  fi
  tmux new-session -d -s $1
  local color=$(tmuxHashColor $1)
  tmux send-keys -t $1 "tmux set-option status-bg $color" C-m
  tmux send-keys -t $1 "clear" C-m
  tmux attach -t $1
}

```

Now every session has it's own distinct `status-bg` color!
