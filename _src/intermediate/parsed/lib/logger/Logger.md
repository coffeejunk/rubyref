# Logger

## Description

The Logger class provides a simple but sophisticated logging utility that you
can use to output messages.

The messages have associated levels, such as `INFO` or `ERROR` that indicate
their importance.  You can then give the Logger a level, and only messages at
that level or higher will be printed.

The levels are:

`UNKNOWN`
:   An unknown message that should always be logged.
`FATAL`
:   An unhandleable error that results in a program crash.
`ERROR`
:   A handleable error condition.
`WARN`
:   A warning.
`INFO`
:   Generic (useful) information about system operation.
`DEBUG`
:   Low-level information for developers.


For instance, in a production system, you may have your Logger set to `INFO`
or even `WARN`. When you are developing the system, however, you probably want
to know about the program's internal state, and would set the Logger to
`DEBUG`.

**Note**: Logger does not escape or sanitize any messages passed to it.
Developers should be aware of when potentially malicious data (user-input) is
passed to Logger, and manually escape the untrusted data:

    logger.info("User-input: #{input.dump}")
    logger.info("User-input: %p" % input)

You can use #formatter= for escaping all data.

    original_formatter = Logger::Formatter.new
    logger.formatter = proc { |severity, datetime, progname, msg|
      original_formatter.call(severity, datetime, progname, msg.dump)
    }
    logger.info(input)

### Example

This creates a Logger that outputs to the standard output stream, with a level
of `WARN`:

    require 'logger'

    logger = Logger.new(STDOUT)
    logger.level = Logger::WARN

    logger.debug("Created logger")
    logger.info("Program started")
    logger.warn("Nothing to do!")

    path = "a_non_existent_file"

    begin
      File.foreach(path) do |line|
        unless line =~ /^(\w+) = (.*)$/
          logger.error("Line in wrong format: #{line.chomp}")
        end
      end
    rescue => err
      logger.fatal("Caught exception; exiting")
      logger.fatal(err)
    end

Because the Logger's level is set to `WARN`, only the warning, error, and
fatal messages are recorded.  The debug and info messages are silently
discarded.

### Features

There are several interesting features that Logger provides, like auto-rolling
of log files, setting the format of log messages, and specifying a program
name in conjunction with the message.  The next section shows you how to
achieve these things.

## HOWTOs

### How to create a logger

The options below give you various choices, in more or less increasing
complexity.

1.  Create a logger which logs messages to STDERR/STDOUT.

        logger = Logger.new(STDERR)
        logger = Logger.new(STDOUT)

2.  Create a logger for the file which has the specified name.

        logger = Logger.new('logfile.log')

3.  Create a logger for the specified file.

        file = File.open('foo.log', File::WRONLY | File::APPEND)
        # To create new logfile, add File::CREAT like:
        # file = File.open('foo.log', File::WRONLY | File::APPEND | File::CREAT)
        logger = Logger.new(file)

4.  Create a logger which ages the logfile once it reaches a certain size.
    Leave 10 "old" log files where each file is about 1,024,000 bytes.

        logger = Logger.new('foo.log', 10, 1024000)

5.  Create a logger which ages the logfile daily/weekly/monthly.

        logger = Logger.new('foo.log', 'daily')
        logger = Logger.new('foo.log', 'weekly')
        logger = Logger.new('foo.log', 'monthly')


### How to log a message

Notice the different methods (`fatal`, `error`, `info`) being used to log
messages of various levels?  Other methods in this family are `warn` and
`debug`.  `add` is used below to log a message of an arbitrary (perhaps
dynamic) level.

1.  Message in a block.

        logger.fatal { "Argument 'foo' not given." }

2.  Message as a string.

        logger.error "Argument #{@foo} mismatch."

3.  With progname.

        logger.info('initialize') { "Initializing..." }

4.  With severity.

        logger.add(Logger::FATAL) { 'Fatal error!' }


The block form allows you to create potentially complex log messages, but to
delay their evaluation until and unless the message is logged.  For example,
if we have the following:

    logger.debug { "This is a " + potentially + " expensive operation" }

If the logger's level is `INFO` or higher, no debug messages will be logged,
and the entire block will not even be evaluated.  Compare to this:

    logger.debug("This is a " + potentially + " expensive operation")

Here, the string concatenation is done every time, even if the log level is
not set to show the debug message.

### How to close a logger

    logger.close

### Setting severity threshold

1.  Original interface.

        logger.sev_threshold = Logger::WARN

2.  Log4r (somewhat) compatible interface.

        logger.level = Logger::INFO

        # DEBUG < INFO < WARN < ERROR < FATAL < UNKNOWN

3.  Symbol or String (case insensitive)

        logger.level = :info
        logger.level = 'INFO'

        # :debug < :info < :warn < :error < :fatal < :unknown

4.  Constructor

        Logger.new(logdev, level: Logger::INFO)
        Logger.new(logdev, level: :info)
        Logger.new(logdev, level: 'INFO')


## Format

Log messages are rendered in the output stream in a certain format by default.
 The default format and a sample are shown below:

Log format:
    SeverityID, [DateTime #pid] SeverityLabel -- ProgName: message

Log sample:
    I, [1999-03-03T02:34:24.895701 #19074]  INFO -- Main: info.

You may change the date and time format via #datetime_format=.

    logger.datetime_format = '%Y-%m-%d %H:%M:%S'
          # e.g. "2004-01-03 00:54:26"

or via the constructor.

    Logger.new(logdev, datetime_format: '%Y-%m-%d %H:%M:%S')

Or, you may change the overall format via the #formatter= method.

    logger.formatter = proc do |severity, datetime, progname, msg|
      "#{datetime}: #{msg}\n"
    end
    # e.g. "2005-09-22 08:51:08 +0900: hello world"

or via the constructor.

    Logger.new(logdev, formatter: proc {|severity, datetime, progname, msg|
      "#{datetime}: #{msg}\n"
    })

[Logger Reference](https://ruby-doc.org/stdlib-2.7.0/libdoc/logger/rdoc/Logger.html)
