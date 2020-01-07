---
title: Mastering IRB
prev: "/developing/libraries.html"
next: "/developing/documenting.html"
---

## Mastering IRB[](#mastering-irb)



### Command line options[](#command-line-options)


```
Usage:  irb.rb [options] [programfile] [arguments]
  -f                Suppress read of ~/.irbrc
  -d                Set $DEBUG to true (same as `ruby -d`)
  -r load-module    Same as `ruby -r`
  -I path           Specify $LOAD_PATH directory
  -U                Same as `ruby -U`
  -E enc            Same as `ruby -E`
  -w                Same as `ruby -w`
  -W[level=2]       Same as `ruby -W`
  --inspect         Use `inspect` for output (default except for bc mode)
  --noinspect       Don't use inspect for output
  --multiline       Use multiline editor module
  --nomultiline     Don't use multiline editor module
  --singleline      Use singleline editor module
  --nosingleline    Don't use singleline editor module
  --colorize        Use colorization
  --nocolorize      Don't use colorization
  --prompt prompt-mode
  --prompt-mode prompt-mode
                    Switch prompt mode. Pre-defined prompt modes are
                    `default`, `simple`, `xmp` and `inf-ruby`
  --inf-ruby-mode   Use prompt appropriate for inf-ruby-mode on emacs.
                    Suppresses --multiline and --singleline.
  --simple-prompt   Simple prompt mode
  --noprompt        No prompt mode
  --tracer          Display trace for each execution of commands.
  --back-trace-limit n
                    Display backtrace top n and tail n. The default
                    value is 16.
  -v, --version     Print the version of irb
```

### Configuration[](#configuration)

IRB reads from `~/.irbrc` when it's invoked.

If `~/.irbrc` doesn't exist, `irb` will try to read in the following
order:

* `.irbrc`
* `irb.rc`
* `_irbrc`
* `$irbrc`

The following are alternatives to the command line options. To use them
type as follows in an `irb` session:


```
IRB.conf[:IRB_NAME]="irb"
IRB.conf[:INSPECT_MODE]=nil
IRB.conf[:IRB_RC] = nil
IRB.conf[:BACK_TRACE_LIMIT]=16
IRB.conf[:USE_LOADER] = false
IRB.conf[:USE_MULTILINE] = nil
IRB.conf[:USE_SINGLELINE] = nil
IRB.conf[:USE_COLORIZE] = true
IRB.conf[:USE_TRACER] = false
IRB.conf[:IGNORE_SIGINT] = true
IRB.conf[:IGNORE_EOF] = false
IRB.conf[:PROMPT_MODE] = :DEFAULT
IRB.conf[:PROMPT] = {...}
```

#### Auto indentation[](#auto-indentation)

To disable auto-indent mode in irb, add the following to your `.irbrc`: 

```ruby
IRB.conf[:AUTO_INDENT] = false
```

#### Autocompletion[](#autocompletion)

To enable autocompletion for irb, add the following to your `.irbrc`: 

```ruby
require 'irb/completion'
```

#### History[](#history)

By default, irb will store the last 1000 commands you used in
`IRB.conf[:HISTORY_FILE]` (`~/.irb_history` by default).

If you want to disable history, add the following to your `.irbrc`: 

```ruby
IRB.conf[:SAVE_HISTORY] = nil
```

See `IRB::Context#save_history=` for more information.

The history of *results* of commands evaluated is not stored by default,
but can be turned on to be stored with this `.irbrc` setting:


```
IRB.conf[:EVAL_HISTORY] = <number>
```

See `IRB::Context#eval_history=` and History class. The history of
command results is not permanently saved in any file.

### Customizing the IRB Prompt[](#customizing-the-irb-prompt)

In order to customize the prompt, you can change the following Hash:


```ruby
IRB.conf[:PROMPT]
```

This example can be used in your `.irbrc`


```ruby
IRB.conf[:PROMPT][:MY_PROMPT] = { # name of prompt mode
  :AUTO_INDENT => false,          # disables auto-indent mode
  :PROMPT_I =>  ">> ",            # simple prompt
  :PROMPT_S => nil,               # prompt for continuated strings
  :PROMPT_C => nil,               # prompt for continuated statement
  :RETURN => "    ==>%s\n"        # format to return value
}

IRB.conf[:PROMPT_MODE] = :MY_PROMPT
```

Or, invoke irb with the above prompt mode by:


```
irb --prompt my-prompt
```

Constants `PROMPT_I`, `PROMPT_S` and `PROMPT_C` specify the format. In
the prompt specification, some special strings are available:


```
%N    # command name which is running
%m    # to_s of main object (self)
%M    # inspect of main object (self)
%l    # type of string(", ', /, ]), `]' is inner %w[...]
%NNi  # indent level. NN is digits and means as same as printf("%NNd").
      # It can be omitted
%NNn  # line number.
%%    # %
```

For instance, the default prompt mode is defined as follows:


```ruby
IRB.conf[:PROMPT_MODE][:DEFAULT] = {
  :PROMPT_I => "%N(%m):%03n:%i> ",
  :PROMPT_N => "%N(%m):%03n:%i> ",
  :PROMPT_S => "%N(%m):%03n:%i%l ",
  :PROMPT_C => "%N(%m):%03n:%i* ",
  :RETURN => "%s\n" # used to printf
}
```

irb comes with a number of available modes:


```ruby
# :NULL:
#   :PROMPT_I:
#   :PROMPT_N:
#   :PROMPT_S:
#   :PROMPT_C:
#   :RETURN: |
#     %s
# :DEFAULT:
#   :PROMPT_I: ! '%N(%m):%03n:%i> '
#   :PROMPT_N: ! '%N(%m):%03n:%i> '
#   :PROMPT_S: ! '%N(%m):%03n:%i%l '
#   :PROMPT_C: ! '%N(%m):%03n:%i* '
#   :RETURN: |
#     => %s
# :CLASSIC:
#   :PROMPT_I: ! '%N(%m):%03n:%i> '
#   :PROMPT_N: ! '%N(%m):%03n:%i> '
#   :PROMPT_S: ! '%N(%m):%03n:%i%l '
#   :PROMPT_C: ! '%N(%m):%03n:%i* '
#   :RETURN: |
#     %s
# :SIMPLE:
#   :PROMPT_I: ! '>> '
#   :PROMPT_N: ! '>> '
#   :PROMPT_S:
#   :PROMPT_C: ! '?> '
#   :RETURN: |
#     => %s
# :INF_RUBY:
#   :PROMPT_I: ! '%N(%m):%03n:%i> '
#   :PROMPT_N:
#   :PROMPT_S:
#   :PROMPT_C:
#   :RETURN: |
#     %s
#   :AUTO_INDENT: true
# :XMP:
#   :PROMPT_I:
#   :PROMPT_N:
#   :PROMPT_S:
#   :PROMPT_C:
#   :RETURN: |2
#         ==>%s
```

### Restrictions[](#restrictions)

Because irb evaluates input immediately after it is syntactically
complete, the results may be slightly different than directly using
Ruby.

### IRB Sessions[](#irb-sessions)

IRB has a special feature, that allows you to manage many sessions at
once.

You can create new sessions with Irb.irb, and get a list of current
sessions with the `jobs` command in the prompt.

#### Commands[](#commands)

JobManager provides commands to handle the current sessions:


```ruby
jobs    # List of current sessions
fg      # Switches to the session of the given number
kill    # Kills the session with the given number
```

The `exit` command, or ::irb\_exit, will quit the current session and
call any exit hooks with IRB.irb\_at\_exit.

A few commands for loading files within the session are also available:

* `source`: Loads a given file in the current session and displays the
  source lines, see IrbLoader#source\_file

* `irb_load`: Loads the given file similarly to `Kernel#load`, see
  `IrbLoader#irb_load`
* `irb_require`: Loads the given file similarly to `Kernel#require`

#### Configuration[](#configuration-1)

The command line options, or IRB.conf, specify the default behavior of
Irb.irb.

On the other hand, each conf in IRB@Command+line+options is used to
individually configure IRB.irb.

If a proc is set for `IRB.conf[:IRB_RC]`, its will be invoked after
execution of that proc with the context of the current session as its
argument. Each session can be configured using this mechanism.

#### Session variables[](#session-variables)

There are a few variables in every Irb session that can come in handy:

* `_`: The value command executed, as a local variable
* `__`: The history of evaluated commands. Available only if
  `IRB.conf[:EVAL_HISTORY]` is not `nil` (which is the default). See
  also IRB::Context#eval\_history= and IRB::History.

* `__[line_no]`: Returns the evaluation value at the given line number,
  `line_no`. If `line_no` is a negative, the return value `line_no` many
  lines before the most recent return value.

#### Example using IRB Sessions[](#example-using-irb-sessions)


```irb
# invoke a new session
irb(main):001:0> irb
# list open sessions
irb.1(main):001:0> jobs
  #0->irb on main (#<Thread:0x400fb7e4> : stop)
  #1->irb#1 on main (#<Thread:0x40125d64> : running)

# change the active session
irb.1(main):002:0> fg 0
# define class Foo in top-level session
irb(main):002:0> class Foo;end
# invoke a new session with the context of Foo
irb(main):003:0> irb Foo
# define Foo#foo
irb.2(Foo):001:0> def foo
irb.2(Foo):002:1>   print 1
irb.2(Foo):003:1> end

# change the active session
irb.2(Foo):004:0> fg 0
# list open sessions
irb(main):004:0> jobs
  #0->irb on main (#<Thread:0x400fb7e4> : running)
  #1->irb#1 on main (#<Thread:0x40125d64> : stop)
  #2->irb#2 on Foo (#<Thread:0x4011d54c> : stop)
# check if Foo#foo is available
irb(main):005:0> Foo.instance_methods #=> [:foo, ...]

# change the active session
irb(main):006:0> fg 2
# define Foo#bar in the context of Foo
irb.2(Foo):005:0> def bar
irb.2(Foo):006:1>  print "bar"
irb.2(Foo):007:1> end
irb.2(Foo):010:0>  Foo.instance_methods #=> [:bar, :foo, ...]

# change the active session
irb.2(Foo):011:0> fg 0
irb(main):007:0> f = Foo.new  #=> #<Foo:0x4010af3c>
# invoke a new session with the context of f (instance of Foo)
irb(main):008:0> irb f
# list open sessions
irb.3(<Foo:0x4010af3c>):001:0> jobs
  #0->irb on main (#<Thread:0x400fb7e4> : stop)
  #1->irb#1 on main (#<Thread:0x40125d64> : stop)
  #2->irb#2 on Foo (#<Thread:0x4011d54c> : stop)
  #3->irb#3 on #<Foo:0x4010af3c> (#<Thread:0x4010a1e0> : running)
# evaluate f.foo
irb.3(<Foo:0x4010af3c>):002:0> foo #=> 1 => nil
# evaluate f.bar
irb.3(<Foo:0x4010af3c>):003:0> bar #=> bar => nil
# kill jobs 1, 2, and 3
irb.3(<Foo:0x4010af3c>):004:0> kill 1, 2, 3
# list open sessions, should only include main session
irb(main):009:0> jobs
  #0->irb on main (#<Thread:0x400fb7e4> : running)
# quit irb
irb(main):010:0> exit
```

<a href='https://ruby-doc.org/stdlib-2.7.0/libdoc/irb/rdoc/IRB.html'
class='ruby-doc remote' target='_blank'>IRB Reference</a>

