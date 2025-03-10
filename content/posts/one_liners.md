+++
title = 'Bash One Liners'
date = 2024-12-06T14:28:45+11:00
draft = true
tags = ["bash"]
+++

*A simple reference for useful commands that might not be used often enough to remember fully.*


#### File Statistics

##### Show the sizes of everything in the current directory in human readable format:
`du -ah -d 1`





#### Docker
*The more I learn about docker the less useful these appear*.

##### Find a particular Instance Id:
`docker ps | cut -d ' ' -f 1 | grep <instance name>`

##### Copy something to a location in a running instance:
`docker cp /path/to/something <instance id>:/path/in/instance;`





#### File Watching

##### Execute command on change continuously:
`while inotifywait -e modify <file to watch>; do <some command>; done`
You can monitor directories recursively, e.g. to run test suite on 
any change to src or tests: 
`while inotifywait -r lib test; do clear && mix test; done`


**Edit** There is a tool called `entr` that probably does this better:
`find files/to/watch | entr -s "command to run"`



#### *I Dont't Care* Mode
To run a program silently and invisibly, redirect stdout and strerr to 
/dev/null and launch in the background:
`yourcommand > /dev/null 2>&1 &`.


#### Navigation

##### Back Button
`cd -` will take you to the previous directory. Useful after you `cd` to the 
home dir and change your mind.
