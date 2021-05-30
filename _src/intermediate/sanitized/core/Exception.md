# Exception

Class Exception and its subclasses are used to communicate between
`Kernel#raise` and `rescue` statements in `begin ... end` blocks.

An Exception object carries information about an exception:

*   Its type (the exception's class).
*   An optional descriptive message.
*   Optional backtrace information.


Some built-in subclasses of Exception have additional methods: e.g.,
`NameError#name`.

## Defaults

Two Ruby statements have default exception classes:

*   `raise`: defaults to RuntimeError.
*   `rescue`: defaults to StandardError.


## Global Variables

When an exception has been raised but not yet handled (in `rescue`, `ensure`,
`at_exit` and `END` blocks), two global variables are set:

*   `$!` contains the current exception.
*   `$@` contains its backtrace.


## Custom Exceptions

To provide additional or alternate information, a program may create custom
exception classes that derive from the built-in exception classes.

A good practice is for a library to create a single "generic" exception class
(typically a subclass of StandardError or RuntimeError) and have its other
exception classes derive from that class. This allows the user to rescue the
generic exception, thus catching all exceptions the library may raise even if
future versions of the library add new exception subclasses.

For example:

    class MyLibrary
      class Error < ::StandardError
      end

      class WidgetError < Error
      end

      class FrobError < Error
      end

    end

To handle both MyLibrary::WidgetError and MyLibrary::FrobError the library
user can rescue MyLibrary::Error.

## Built-In Exception Classes

The built-in subclasses of Exception are:

*   NoMemoryError
*   ScriptError
    *   LoadError
    *   NotImplementedError
    *   SyntaxError

*   SecurityError
*   SignalException
    *   Interrupt

*   StandardError
    *   ArgumentError
        *   UncaughtThrowError

    *   EncodingError
    *   FiberError
    *   IOError
        *   EOFError

    *   IndexError
        *   KeyError
        *   StopIteration
            *   ClosedQueueError


    *   LocalJumpError
    *   NameError
        *   NoMethodError

    *   RangeError
        *   FloatDomainError

    *   RegexpError
    *   RuntimeError
        *   FrozenError

    *   SystemCallError
        *   Errno::*

    *   ThreadError
    *   TypeError
    *   ZeroDivisionError

*   SystemExit
*   SystemStackError
*   fatal


[Exception Reference](https://ruby-doc.org/core-2.7.0/Exception.html)