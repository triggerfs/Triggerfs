# TriggerFS

A virtual FS using Go and Fuse native binding.

Execute configurable commands on read calls of files or patterns of filenames.

The Idea is to have a virtual readonly filesystem populated by files defined in a config and generate the contents on read accesses by running configurable commands and returning their output as content.


# Config

create a config.conf file, the structure of the contents should look like this:
```
# set the title of the running triggerFS instance
title = "foofs"

# cache the filesize after executing command
size_cache = true

# directory definitions
dir {
# create a directory by defining a new section with the path as name
# leading and trailing slashes will be cut at runtime, so they don't need to be given necessarily
# attributes will be inherited by parent directories if not defined otherwise
	/foo/bar/ {
		permission = "0775"
		size	=	4096
		ctime	=	1524234000
		atime	=	1524234000
		mtime	=	1524234000
		}
}


# file definitions
file {
# create a file by defining a new section with the path to the file as name
	/testfile {
# set the fileattributes and the command that will define the content
# attributes will be inherited by parent directories if not defined otherwise
	
# this string will be executed with /bin/sh -c '<exec>' each time the file is read
		exec	=	"echo foobar"
		
# most programs are strict about the size given by getattr() and won't process larger contents
# depending on the nature of the given exec command the global "size_cache" option might be of use
# to get the correct size after running once. set size_cache = true at the top to enable it.
# in doubt, set the size higher than the expected size of the command output
		size	=	50

# set the permission of the file
		permission = "0644"

# supported time attributes: ctime,mtime,atime
		ctime	=	1524234000
		mtime	=	1524234000
	}
}


# pattern definitions
pattern {
# create a filepattern by defining a new section with the path and the pattern as name
# basename(<name>) will be used as pattern

# here we create a pattern for all *.txt files
	/testdir/.*.txt	{
# attributes will be inherited by parent directories if not defined otherwise
		
# this string will be executed with /bin/sh -c '<exec>' each time a file matching the pattern is read
		exec	=	"echo %FILE%"
		
# set the permission of the file
		permission	=	"0644"
		
# most programs are strict about the size given by getattr() and won't process larger contents
# depending on the nature of the given exec command the global "size_cache" option might be of use
# to get the correct size after running once. set size_cache = true at the top to enable it.
# in doubt, set the size higher than the expected size of the command output
		size	=	100
		
# supported time attributes: ctime,mtime,atime
		ctime	=	1524234000
		mtime	=	1524234000
		atime	=	1524234000	
	}
}
```
You can use %FILE% or %PATH% in your exec command wich will get replaced with the filename or the full path.



# Usage

```
go get
go build
mkdir mountpoint
./triggerfs -c config.conf mountpoint/ 
ls mountpoint
cat mountpoint/testfile

```

# Clean up
```
fusermount -u mountpoint
```
