# withoverlay

A bash wrapper to run singularity commands and capture filesystem information in an overlay

This is withoverlay 1.0 by Pierre Rioux

Usage: withoverlay [-v] overlaypath[=overlaysize] containerimage command [args]

This program will run command (and args) from within a singularity
container with a writable overlay to capture results. Files created
while the command run will only end up in the overlay if they are not in
one of the pre-mounted singularity paths (typically: the HOME directory
and /tmp). Overlays cover the entire file system down from the root, /,
but typically the only writable place is directlry under "/home", so
programs should store persistent data in "/home/choosethis".

## Requirements:

Singularity 3.2 or better is absolutely necessary.

## Options:

* -v

Make this program a bit more verbose. Normally the program is
silent about all its activities and will only print whatever output
the command produces. Note that the initial creation of the overlay
can take some time, so it recommanded to use the verbose option
with it. That way the user can see the progress of this phase.

## Arguments:

### overlaypath
 
The path to an overlay file; if the file doesn't exist it will
be created, provided a suffix "=overlaysize" is provided. If the
overlap file exists, the suffix will be ignored. The value is a
size in gigabytes, e.g hello.img=20 means use the file 'hello.img'
 and create it with a size of 20 gigabytes if needed.

### containerimage

The path to a singularity container image to use to run your
command.

### command [args]

A command or program with arguments. Note that the command will be
passed as-is to the 'singularity exec' command, and hanlding of
single quoted / double quoted arguments are tricky. We recommand
wrapping any complex set of commands within a shell wrapper and
executing that wrapper as a simple command here instead.

Two special commands are supported:

#### getlog

This will print a log of the activity (mostly commands)
that were used through this program (maintained internally
in the overlay in path $INTERNAL_LOG).

#### addlog words words words

This will add a log message with words words words.

## Examples:

```bash
  # Just create a 1G overlay, don't run anything in it yet:
  withoverlay -v ~/base.img=1  centos.img true
  withoverlay    ~/base.img=1  centos.img getlog # the '=1' will be ignored here
```

```bash
  # Run some commands and store data in the overlay:
  withoverlay -v ~/data.img=10 centos.img bash -c 'mkdir /home/hello;uptime >/home/hello/uptime'
  withoverlay    ~/data.img    centos.img cat /home/hello/uptime
  withoverlay    ~/data.img    centos.img addlog this is great
  withoverlay    ~/data.img    centos.img getlog
```

```bash
  # Another abstract example
  withoverlay hello.img=5 planet.img planet_maker --name pluto --distance 5.5h
  withoverlay hello.img=5 planet.img moon_maker   --name charon --parent pluto
  withoverlay hello.img=5 planet.img getlog
```

## Notes

Do not run this on MacOS; before the script is even able to crash
when it attempts to run singularity, it will have created the empty
placeholder file for the overlay filesystem, and since MacOS doesn't
support sparse files (or at least, the commands here do not create
a file with holes), you'll end up with a huge chunk of your disk space
missing.

## Author

Pierre Rioux
July 2019
