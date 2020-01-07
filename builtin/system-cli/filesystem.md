---
title: Filesystem
prev: "/builtin/system-cli/io.html"
next: "/builtin/system-cli/args.html"
---

## Dir[](#dir)

Objects of class Dir are directory streams representing directories in
the underlying file system. They provide a variety of ways to list
directories and their contents. See also File.

<a href='https://ruby-doc.org/core-2.7.0/Dir.html' class='ruby-doc
remote' target='_blank'>Dir Reference</a>



### File[](#file)

A File is an abstraction of any file object accessible by the program
and is closely associated with class IO. File includes the methods of
module FileTest as class methods, allowing you to write (for example)
`File.exist?("foo")`.

In the description of File methods, *permission bits* are a
platform-specific set of bits that indicate permissions of a file. On
Unix-based systems, permissions are viewed as a set of three octets, for
the owner, the group, and the rest of the world. For each of these
entities, permissions may be set to read, write, or execute the file:

The permission bits `0644` (in octal) would thus be interpreted as
read/write for owner, and read-only for group and other. Higher-order
bits may also be used to indicate the type of file (plain, directory,
pipe, socket, and so on) and various other special features. If the
permissions are for a directory, the meaning of the execute bit changes;
when set the directory can be searched.

On non-Posix operating systems, there may be only the ability to make a
file read-only or read-write. In this case, the remaining permission
bits will be synthesized to resemble typical values. For instance, on
Windows NT the default permission bits are `0644`, which means
read/write for owner, read-only for all others. The only change that can
be made is to make the file read-only, which is reported as `0444`.

Various constants for the methods in File can be found in
File::Constants.

<a href='https://ruby-doc.org/core-2.7.0/File.html' class='ruby-doc
remote' target='_blank'>File Reference</a>



### File::Constants[](#fileconstants)

File::Constants provides file-related constants. All possible file
constants are listed in the documentation but they may not all be
present on your platform.

If the underlying platform doesn't define a constant the corresponding
Ruby constant is not defined.

Your platform documentations (e.g. man open(2)) may describe more
detailed information.

<a href='https://ruby-doc.org/core-2.7.0/File/Constants.html'
class='ruby-doc remote' target='_blank'>File::Constants Reference</a>



### File::Stat[](#filestat)

Objects of class File::Stat encapsulate common status information for
File objects. The information is recorded at the moment the File::Stat
object is created; changes made to the file after that point will not be
reflected. File::Stat objects are returned by `IO#stat`, File::stat,
`File#lstat`, and File::lstat. Many of these methods return
platform-specific values, and not all values are meaningful on all
systems. See also `Kernel#test`.

<a href='https://ruby-doc.org/core-2.7.0/File/Stat.html' class='ruby-doc
remote' target='_blank'>File::Stat Reference</a>



### FileTest[](#filetest)

FileTest implements file test operations similar to those used in
File::Stat. It exists as a standalone module, and its methods are also
insinuated into the File class. (Note that this is not done by
inclusion: the interpreter cheats).

<a href='https://ruby-doc.org/core-2.7.0/FileTest.html' class='ruby-doc
remote' target='_blank'>FileTest Reference</a>



### Pathname[](#pathname)

*Part of standard library. You need to `require 'pathname'` before
using.*

Pathname represents the name of a file or directory on the filesystem,
but not the file itself.

The pathname depends on the Operating System: Unix, Windows, etc. This
library works with pathnames of local OS, however non-Unix pathnames are
supported experimentally.

A Pathname can be relative or absolute. It's not until you try to
reference the file that it even matters whether the file exists or not.

Pathname is immutable. It has no method for destructive update.

The goal of this class is to manipulate file path information in a
neater way than standard Ruby provides. The examples below demonstrate
the difference.

**All** functionality from File, FileTest, and some from Dir and
FileUtils is included, in an unsurprising way. It is essentially a
facade for all of these, and more.


```ruby
require 'pathname'
pn = Pathname.new("/usr/bin/ruby")
size = pn.size              # 27662
isdir = pn.directory?       # false
dir  = pn.dirname           # Pathname:/usr/bin
base = pn.basename          # Pathname:ruby
dir, base = pn.split        # [Pathname:/usr/bin, Pathname:ruby]
data = pn.read
pn.open { |f| _ }
pn.each_line { |line| _ }
```

<a
href='https://ruby-doc.org/stdlib-2.7.0/libdoc/pathname/rdoc/Pathname.html'
class='ruby-doc remote' target='_blank'>Pathname Reference</a>



### Tempfile[](#tempfile)

*Part of standard library. You need to `require 'tempfile'` before
using.*

A utility class for managing temporary files. When you create a Tempfile
object, it will create a temporary file with a unique filename. A
Tempfile objects behaves just like a File object, and you can perform
all the usual file operations on it: reading data, writing data,
changing its permissions, etc. So although this class does not
explicitly document all instance methods supported by File, you can in
fact call any File instance method on a Tempfile object.


```ruby
require 'tempfile'

file = Tempfile.new('foo')
file.path      # => A unique filename in the OS's temp directory,
               #    e.g.: "/tmp/foo.24722.0"
               #    This filename contains 'foo' in its basename.
file.write("hello world")
file.rewind
file.read      # => "hello world"
file.close
file.unlink    # deletes the temp file
```

<a
href='https://ruby-doc.org/stdlib-2.7.0/libdoc/tempfile/rdoc/Tempfile.html'
class='ruby-doc remote' target='_blank'>Tempfile Reference</a>



### FileUtils[](#fileutils)

*Part of standard library. You need to `require 'fileutils'` before
using.*



Namespace for several file utility methods for copying, moving,
removing, etc.


```ruby
require 'fileutils'

FileUtils.cd(dir, **options)
FileUtils.cd(dir, **options) {|dir| block }
FileUtils.pwd()
FileUtils.mkdir(dir, **options)
FileUtils.mkdir(list, **options)
FileUtils.mkdir_p(dir, **options)
FileUtils.mkdir_p(list, **options)
FileUtils.rmdir(dir, **options)
FileUtils.rmdir(list, **options)
FileUtils.ln(target, link, **options)
FileUtils.ln(targets, dir, **options)
FileUtils.ln_s(target, link, **options)
FileUtils.ln_s(targets, dir, **options)
FileUtils.ln_sf(target, link, **options)
FileUtils.cp(src, dest, **options)
FileUtils.cp(list, dir, **options)
FileUtils.cp_r(src, dest, **options)
FileUtils.cp_r(list, dir, **options)
FileUtils.mv(src, dest, **options)
FileUtils.mv(list, dir, **options)
FileUtils.rm(list, **options)
FileUtils.rm_r(list, **options)
FileUtils.rm_rf(list, **options)
FileUtils.install(src, dest, **options)
FileUtils.chmod(mode, list, **options)
FileUtils.chmod_R(mode, list, **options)
FileUtils.chown(user, group, list, **options)
FileUtils.chown_R(user, group, list, **options)
FileUtils.touch(list, **options)
```

Possible `options` are:

* `:force`: forced operation (rewrite files if exist, remove
  directories if not empty, etc.);

* `:verbose`: print command to be run, in bash syntax, before
  performing it;
* `:preserve`: preserve object's group, user and modification time on
  copying;
* `:noop`: no changes are made (usable in combination with `:verbose`
  which will print the command to run)

Each method documents the options that it honours. See also ::commands,
::options and ::options\_of methods to introspect which command have
which options.

All methods that have the concept of a "source" file or directory can
take either one file or a list of files in that argument. See the method
documentation for examples.

There are some `low level` methods, which do not accept keyword
arguments:


```ruby
FileUtils.copy_entry(src, dest, preserve = false, dereference_root = false, remove_destination = false)
FileUtils.copy_file(src, dest, preserve = false, dereference = true)
FileUtils.copy_stream(srcstream, deststream)
FileUtils.remove_entry(path, force = false)
FileUtils.remove_entry_secure(path, force = false)
FileUtils.remove_file(path, force = false)
FileUtils.compare_file(path_a, path_b)
FileUtils.compare_stream(stream_a, stream_b)
FileUtils.uptodate?(file, cmp_list)
```

<a
href='https://ruby-doc.org/stdlib-2.7.0/libdoc/fileutils/rdoc/FileUtils.html'
class='ruby-doc remote' target='_blank'>FileUtils Reference</a>



### Find[](#find)

*Part of standard library. You need to `require 'find'` before using.*

The `Find` module supports the top-down traversal of a set of file
paths.

For example, to total the size of all files under your home directory,
ignoring anything in a "dot" directory (e.g. $HOME/.ssh):


```ruby
require 'find'

total_size = 0

Find.find(ENV["HOME"]) do |path|
  if FileTest.directory?(path)
    if File.basename(path).start_with?('.')
      Find.prune       # Don't look any further into this directory.
    else
      next
    end
  else
    total_size += FileTest.size(path)
  end
end
```

<a href='https://ruby-doc.org/stdlib-2.7.0/libdoc/find/rdoc/Find.html'
class='ruby-doc remote' target='_blank'>Find Reference</a>

