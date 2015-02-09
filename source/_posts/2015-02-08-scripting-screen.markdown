---
layout: post
title: "Scripting Screen"
date: 2015-02-08 21:11:19 -0800
comments: true
categories:
---

Recently i was presented with the challenge of executing commands against a running program. This program was already running inside screen so it seemed natural to me to do some screen scripting. To my surprise i failed to find anything straight forward that explained how this could work. So here is my take on it in hopes that it helps someone else or myself if I forget it lol.

The simplest form is:

```bash
screen -S test -X stuff "ls^M"
```

This works for lots of things, here is a more complex call:

```bash
screen -S test -X stuff "irb^M2+5^M^D"
```

This opens irb, types 2+5, executes it and exits irb.

Lets break its parts down:

- ```screen``` duh
- ```-S test``` use a running screen session named test
- ```-X stuff "irb^M2+5^M^D"``` push the string argument as input to screen
- **^M** new line
- **^D** control + d

Using this method you can script anything that you had to do manually before.

Let me know if you know of another/better way.

## Play Code

Create a new directory, cd into it and create a new executable file with the following content.

Gotcha: To automate everyting I had to sleep a few times. Not sure how to get around that yet.

```bash
#!/bin/bash

SCREEN_NAME=`date "+%Y%m%d%H%M%s%N"`
SCREEN_NAME="session_${SCREEN_NAME}"

get_pid() {
	echo $(screen -ls | awk '/\.'"${SCREEN_NAME}"'\t/ {print strtonum($1)}')
}

clean() {
	rm -f screenlog.0
	rm -f 1
}

open() {
	clean
	screen -L -dmS "${SCREEN_NAME}"
	sleep 0.2
}

script_screen() {
	script_screen_raw "${1}^M"
}

script_screen_raw() {
	screen -S "${SCREEN_NAME}" -p 0 -X stuff "${1}"
	sleep 0.2
}

close() {
	kill `get_pid`
}

exit_irb() {
	script_screen_raw "^D"
}


open

script_screen "touch 1"

script_screen "irb"

script_screen "File.open('1', 'w') { |file| file.puts('your text') }"

exit_irb

script_screen "echo hi"

close

echo -e "\n=============="
cat screenlog.0
echo -e "\n=============="

```