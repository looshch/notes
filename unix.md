# Notes On Unix
* [Commands](#commands)
  * [Text](#text)
  * [File](#file)
  * [Process](#process)
  * [Network](#network)
  * [User](#user)
  * [Miscellaneous](#miscellaneous)
* [Files](#files)
  * [File Structure](#file-structure)
* [Processes](#processes)

## Commands
### Text
#### `head`
first lines

#### `tail`
last lines

#### `cat`
con**cat**enate files

#### `grep`
search file contents

#### `awk`
pattern-directed scanning and processing language

#### `diff`
show files differences

#### `comm`
select or reject lines common to two files

#### `uniq`
report or filter out repeated lines

#### `banner`
print large banner

#### `fold`
fold lines

#### `more`
#### `less`

#### `rev`
reverse lines

#### `sort`
sort or merge lines

#### `wc`
word, line, character, and byte count

#### `split`
split files into pieces

### File
#### `pwd`
get working directory name. Displays `PWD` value

#### `cd`
change working directory. Sets `PWD` value

#### `ls`
list directory contents

#### `touch`
change file access and modification time; create a new one

#### `chmod`
change file modes or Access Control Lists

#### `chown`
change file owner and group

#### `chgrp`
change file group ownership

#### `rm`
delete directory contents

#### `mkdir`
create directories

#### `rmdir`
delete directories

#### `cp`
**c**o**p**y files

#### `mv`
**m**o**v**e or rename files

#### `locate` (macOS: `mdfind`)
locate files

#### `find`
locate files and directories

#### `df`
disk free space

#### `du`
disk usage

#### `dd`
convert and copy a file

#### `tar`
archive files

#### `basename`, `dirname`
get filename or directory portion of pathname

#### `cmp`
compare two files byte by byte

### Process
#### `top`
display processes

#### `jobs`
display jobs

#### `batch`, `at`, `atq`, `atrm`
queue, examine, or delete jobs for later execution

#### `cron`
execute scheduled commands

#### `kill`
terminate processes

#### `nice`
execute utilities with an altered scheduling priority

#### `tee`
duplicate standard input

### Network
#### `uname`
system information

#### `hostname`
set or get name of current host

#### `ping`
send ECHO_REQUESTs to hosts

#### `netstat`
show network status

#### `ftp`
file transfer program

#### `ssh`
#### `traceroute`
print route packets take to network host

#### `dig`
DNS lookup

### User
#### `passwd`
modify user’s password

#### `finger`
user information lookup program

### Miscellaneous
#### `history`
entered commands

#### `man`
commands manuals

#### `echo`
print strings

#### `date`
print or set date and time

#### `alias`
create commands for arbitrary shell command input

#### `ncal`, `cal`
display calendar

#### `units`
units conversion

#### `uptime`
how long system has been running

#### `say`
convert text to audible speech

#### `tab2space`
expand tabs

#### `which`
#### `whereis`

## Files
### File Structure
* `/`
  * `/bin`

  * `/boot`

  * `/dev`
    devices
    * `/dev/null`\
      empty data source and destination
  * `/etc`
    * `/etc/opt`
    * `/etc/sgml`
    * `/etc/X11`
    * `/etc/xml`

  * `/home`

  * `/lib`

  * `/media`

  * `/mnt`

  * `/opt`

  * `/proc`

  * `/root`

  * `/run`

  * `/sbin`

  * `/srv`

  * `/tmp`\
    for temporary files

  * `/usr`
    * `/usr/bin`
    * `/usr/include`
    * `/usr/lib`
    * `/usr/local`
    * `/usr/sbin`
    * `/usr/share`
    * `/usr/src`

  * `/var`
    * `/var/cache`
    * `/var/lib`
    * `/var/lock`
    * `/var/log`
    * `/var/mail`
    * `/var/opt`
    * `/var/run`
    * `/var/spool`
    * `/var/tmp`

an open file is represented by a file descriptor which is an index into an array
of open files kept by the kernel and can be retreived by an `open` call. The
file descriptor is the gateway into the kernel's abstractions of underlying
hardware. File can represent, for example, a hardware device, a file system.
The operating system requires a device driver to be able to communicate with a
hardware device. To provide the abstraction to user-space, the kernel provides
a file-interface via a device layer. Every running program starts with three
files already opened: stdin, -out and -err, numbered 0, 1 and 2 correspondingly

### Redirects
* a > b — overwrite b with a’s stdout
  * a >> b — append a’s stdout to b

* a < b — copy b’s data to a’s stdin

* a | b — move a’s stdout to b’s stdin\
  \
  the pipe is one of the fundamental forms of inter-process communication.
  Besides a data transfer, it can act as a signaling channel. If a process
  reads an empty pipe, it will by default block or be put into hibernation
  until there is some data available. Two processes may use a pipe to
  communicate that some action has been taken just by writing a byte of data.

## Processes
a process consists of a process ID, memory with code, data, stack and heap;
files, registers and kernel state
