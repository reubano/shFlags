# shFlags

## Introduction

Shell Flags (shFlags) is a library written to greatly simplify the handling of command-line flags in Bourne based Unix shell scripts.

shFlags is a port of the google-gflags C++/Python library and has been tested on the following OSes:
* [Linux (Ubuntu)](http://ubuntu.com/)
* [Mac OS X 10.5](http://www.apple.com/macosx/)
* [Solaris](http://www.sun.com/software/solaris/) 9, 10 

shFlags has been tested on the following shells
* [Bourne Shell](http://en.wikipedia.org/wiki/Bourne_shell) (sh)
* [GNU Bourne Again SHell](http://www.gnu.org/software/bash/) (bash)
* [DASH](http://gondor.apana.org.au/~herbert/dash/) (dash)
* [Korn Shell](http://www.kornshell.com/) (ksh)
* [the Public Domain Korn Shell](http://web.cs.mun.ca/%7Emichael/pdksh/) (pdksh)
* [Zsh](http://www.zsh.org) (zsh)

## Requirements

* [GNU getopt(1)](http://linux.die.net/man/1/getopt)

## Mac OS X Preparation

* Install [Macports](https://trac.macports.org/wiki/InstallingMacPorts) 

* Check that Macports is installed

```
port help
```

* Install getopt using [Macports](http://www.macports.org/) 

```
sudo port install getopt
```

* Add macports to your path 

```
echo "export PATH=/opt/local/bin:/opt/local/sbin:$PATH" >> ~/.profile
```

* Update the shell 

```
source ~/.profile
```

## Installation

* Download shflags

```
cd ~/Downloads
curl -fsSL https://github.com/reubano/shFlags/zipball/master | tar -xf -
```

* Copy shflags to `/usr/lib`

```
sudo cp shFlags/src/shflags /usr/lib/shflags
```

* Make example scripts executable and copy to `~/bin`

```
sudo chmod +x shflags/examples/*.sh
sudo cp shflags/examples/*.sh ~/bin
```

## Quick Start

Getting started with shFlags is quite easy. We'll start with the proverbial "Hello, world" example `~/bin/hello_world.sh`.

```ruby
#!/usr/bin/env bash
	
# source shflags
. /usr/lib/shflags

# define a 'name' command-line string flag
DEFINE_string 'name' 'world' 'name to say hello to' 'n'

# parse the command-line
FLAGS "$@" || exit 1
eval set -- "${FLAGS_ARGV}"

# say Hello!
echo "Hello, ${FLAGS_name}!"
```

Go ahead and give the script a run.

	hello_world.sh

Output

	Hello, world!

So what just happened? Let's go through the script line-by-line.

```ruby
# source shflags
. /usr/lib/shflags
```

This line sources the shflags library into the current shell environment. It brings in several functions and sets several variables, but you won't actually see output from this line.

```ruby
# define a 'name' command-line string flag
DEFINE_string 'name' 'world' 'name to say hello to' 'n'
```

Here we have defined a 'string' flag called name, we gave it a default value of 'world', we described the variable with 'name to say hello to', and we defined a short flag called n. This function has just told shFlags everything it needs to know about the name flag. It now knows:

* how to handle the short command-line option `-n`
* how to handle the long command-line option `--name Kate` (not supported on all systems)
* how to describe the flag if `-h` (or `--help`) is given as an option
* to accept and validate any input as a string value
* to store any input (or the default value) in the `FLAGS_name` variable 

Wow. That's a lot. Let's keep going.

```ruby
# parse the command-line
FLAGS "$@" || exit 1
eval set -- "${FLAGS_ARGV}"
```

Here is where all the magic happens. The call to `FLAGS` with the passing of `"$@"` sends all command-line arguments into the shFlags library for processing. If everything is successful, a return value of `0 (${FLAGS_TRUE})` is returned an things continue. If a false value of `1 (${FLAGS_FALSE})` or an error value of `2 (${FLAGS_ERROR})` is returned, the processing is halted with `exit 1`. If there are any command-line arguments left over that shFlags didn't know how to process, they get shifted off the stack with `shift ${FLAGS_ARGC}` so the script can handle them itself.

There is actually a lot that happened with this `FLAGS` call, but the basic action you will be interested in right now is that a `FLAGS_name` variable was defined. As we didn't pass the `-n` flag (or `--name`) on the command-line, the value placed in the variable is `world`, the default value specified when we defined the flag.

```ruby
# say Hello!
echo "Hello, ${FLAGS_name}!"
```

Now that we have accepted parsed all our flags, we can make use of the results. This snippet prints out Hello to the screen (`Hello, world!` in this case) as the `FLAGS_name` variable contained `world`.

Let's give the script another go, this time passing `-n Kate` on the command-line.
	
	hello_world.sh -n Kate
	
Output
	
	Hello, Kate!

This time around, the `FLAGS_name` variable was defined with the value `Kate` as we passed the `-n` flag, and `Hello, Kate!` was output. I bet you never thought accepting command-line arguments could be so easy!

What about spaces in the flag value? Give it a try! (**Note: this requires the enhanced getopt referenced [above][Requirements]**)
	
	hello_world.sh --name 'Kate Ward'
	
Output

	Hello, Kate Ward!

It works!

What happens if you can't remember the command-line flags you have defined, and what to find that out? Try passing the `-h` flag (or `--help`).

	hello_world.sh -h
	
Output

	USAGE: hello_world.sh [flags] args
	flags:
	  -h  show this help
	  -n  name to say hello to

There you have it. A list of all the command line options supported in a nice clean format. What could be easier? 

### Usage

#### Understanding flag parsing

The actual flag parsing step looks deceptively simple, but at the same time confusing.

```ruby
FLAGS "$@" || exit 1
eval set -- "${FLAGS_ARGV}"
```

The `FLAGS "$@" || exit 1` takes the command-line arguments `$@` in their unparsed form and passes them to the `FLAGS` function for processing. If `$*` had been used, any spaces in string arguments that were passed would be lost. The double-quotes surrounding `$@` are also required to maintain the spaces in string arguments. If `FLAGS` returns with a failure (represented by `${FLAGS_ERROR}`) the script is exited with an exit code of 1.

`eval set -- "${FLAGS_ARGV}` replaces the `$1`, `$2`, `$@`, etc. variables with the non-flag values so that your script can process them normally. This is typically useful when you want to pass some flags to your script to alter processing, but you still want to allow non-flag type options to be passed as well.

#### Supported flag types

shFlags accepts the following flag types (types not natively supported by your shell, e.g., float) will be treated as string.

##### boolean

Boolean values are true/false values or conditions. A typical use might be to have support for some special functionality to be disabled, but a boolean flag could enable that functionality (e.g. enabling debug mode output).

Boolean flags do not take any arguments, and their value is either 1 (false) or 0 (true). For long flags, the false value is specified on the command line by prepending the word 'no'. With short flags, the presense of the flag toggles the current value between true and false. Specifying a short boolean flag twice on the command results in returning the value back to the default value.

In your script, you can use the `${FLAGS_TRUE}` or `${FLAGS_FALSE}` constants to compare the value of your flag variable against the "official" true/false values. Don't forget that in shell, true is 0 and false is 1 (quite the opposite of normal programming languages where true is 1 and false is 0).

A default value is required for boolean flags. For example, lets say a Boolean flag was created whose long name was 'update' and whose short name was `x`, and the default value was `false`. This flag could be explicitly set to `true` with `--update` or by `-x`, and it could be explicitly set to `false` with `--noupdate`.

##### float

Float values are signed numbers that may or not contain a decimal point. As shell does not support float values natively, they are represented internally as strings. As such, any comparisons must be done as string comparisons using the = and != operators – eq, ge, gt, le, lt, and ne will not work.

##### integer

Float values are signed numbers that do not contain a decimal point. Shell supports integer values natively, so standard integer operators (eq, ge, gt, ...) may be used for comparisons.

##### string

Strings are, well, strings. Treat them like you would any other string.

#### Automated help

Unlike getopt, shFlags provides an automated means of generating help for the user. It has its limitations, but it is definitely better than nothing. To get access to help, simply add the `-h` (or `--help`) flag to the command line, and you will get help for all of the defined flags. If you want to make your usage line a bit more informative, perhaps even with a multiline string of text, simply define the `FLAGS_HELP` variable and shFlags will make use of it.

## Examples

#### Debug mode output

The debug example `~/bin/debug_output.sh` sets up a debug flag that when specified will enable debug output to STDERR. If the flag is not specified (the default behavior), debug output is not enabled.

```ruby
#!/usr/bin/env bash

# source shflags
. /usr/lib/shflags

debug() { [ ${FLAGS_debug} -eq ${FLAGS_TRUE} ] && echo "DEBUG: $@" >&2; }

# define flags
DEFINE_boolean 'debug' false 'enable debug mode' 'd'

# parse the command-line
FLAGS "$@" || exit 1
eval set -- "${FLAGS_ARGV}"

debug 'debug mode enabled'
echo 'something interesting'
```

#### Accept flags and options

The write date example `~/bin/write_date.sh` will try to write the current date to a filename given on the command-line. If the file already exists, the script will fail, but if a `-f` (or `--force`) flag is given, the existing file will be overwritten.

```ruby
#!/usr/bin/env bash

# source shflags
. /usr/lib/shflags

write_date() { date >"$1"; }

# configure shflags
DEFINE_boolean 'force' false 'force overwriting' 'f'
FLAGS_HELP="USAGE: $0 [flags] filename"

# parse the command-line
FLAGS "$@" || exit 1
eval set -- "${FLAGS_ARGV}"

# check for filename
if [ $## -eq 0 ]; then
	echo 'error: filename missing' >&2
	flags_help
	exit 1
fi

filename=$1

if [ ! -f "${filename}" ]; then
	write_date "${filename}"
else
	if [ ${FLAGS_force} -eq ${FLAGS_TRUE} ]; then
		write_date "${filename}"
	else
		echo 'warning: filename exists; not overwriting' >&2
		exit 2
	fi
fi
```

Let's try calling this without a filename.

	write_date.sh
	
It fails (notice the custom usage definition line)

	error: filename missing
	USAGE: write_date.sh [flags] filename
	flags:
		-h	show this help
		-f	force overwriting

Let's take a look at the exit code

	echo $?

Output

	1

This should create the file just fine.

	write_date.sh junk.dat
	cat junk.dat

Output

	Thu Jun 26 23:06:34 IST 2008

This should fail with an error.

	write_date.sh junk.dat

Output

	warning: filename exists; not overwriting

Now let's take a look at this exit code

	echo $?

Output

	2

This succeeds because we pass the `-f` flag.

	write_date.sh -f junk.dat
	cat junk.dat

Output

	Thu Jun 26 23:10:02 IST 2008

## Known Issues

The getopt provided by default on Mac OS X and Solaris is the standard (non-enhanced) version.
