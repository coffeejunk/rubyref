---
title: 'open3: Flexible child processes'
prev: stdlib/cli/io.html
next: stdlib/cli/optparse.html
---


```ruby
require 'open3'
```

## Open3[](#open3)

Open3 grants you access to stdin, stdout, stderr and a thread to wait
for the child process when running another program. You can specify
various attributes, redirections, current directory, etc., of the
program in the same way as for Process.spawn.

* Open3.popen3 : pipes for stdin, stdout, stderr
* Open3.popen2 : pipes for stdin, stdout
* Open3.popen2e : pipes for stdin, merged stdout and stderr
* Open3.capture3 : give a string for stdin; get strings for stdout,
  stderr
* Open3.capture2 : give a string for stdin; get a string for stdout
* Open3.capture2e : give a string for stdin; get a string for merged
  stdout and stderr

* Open3.pipeline\_rw : pipes for first stdin and last stdout of a
  pipeline
* Open3.pipeline\_r : pipe for last stdout of a pipeline
* Open3.pipeline\_w : pipe for first stdin of a pipeline
* Open3.pipeline\_start : run a pipeline without waiting
* Open3.pipeline : run a pipeline and wait for its completion

<a href='https://ruby-doc.org/stdlib-2.7.0/libdoc/open3/rdoc/Open3.html'
class='ruby-doc remote' target='_blank'>Open3 Reference</a>

