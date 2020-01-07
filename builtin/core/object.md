---
title: Object
prev: builtin/core/kernel.html
next: builtin/core/module-class.html
---

## BasicObject and Object[](#basicobject-and-object)



### BasicObject[](#basicobject)

BasicObject is the parent class of all classes in Ruby. It's an explicit
blank class.

BasicObject can be used for creating object hierarchies independent of
Ruby's object hierarchy, proxy objects like the Delegator class, or
other uses where namespace pollution from Ruby's methods and classes
must be avoided.

To avoid polluting BasicObject for other users an appropriately named
subclass of BasicObject should be created instead of directly modifying
BasicObject:


```ruby
class MyObjectSystem < BasicObject
end
```

BasicObject does not include Kernel (for methods like `puts`) and
BasicObject is outside of the namespace of the standard library so
common classes will not be found without using a full class path.

A variety of strategies can be used to provide useful portions of the
standard library to subclasses of BasicObject. A subclass could `include
Kernel` to obtain `puts`, `exit`, etc. A custom Kernel-like module could
be created and included or delegation can be used via
`#method_missing`: 

```ruby
class MyObjectSystem < BasicObject
  DELEGATE = [:puts, :p]

  def method_missing(name, *args, &block)
    return super unless DELEGATE.include? name
    ::Kernel.send(name, *args, &block)
  end

  def respond_to_missing?(name, include_private = false)
    DELEGATE.include?(name) or super
  end
end
```

Access to classes and modules from the Ruby standard library can be
obtained in a BasicObject subclass by referencing the desired constant
from the root like `::File` or `::Enumerator`. Like `#method_missing`,
`#const_missing` can be used to delegate constant lookup to `Object`: 

```ruby
class MyObjectSystem < BasicObject
  def self.const_missing(name)
    ::Object.const_get(name)
  end
end
```

<a href='https://ruby-doc.org/core-2.7.0/BasicObject.html'
class='ruby-doc remote' target='_blank'>BasicObject Reference</a>



### Object[](#object)

Object is the default root of all Ruby objects. Object inherits from
BasicObject which allows creating alternate object hierarchies. Methods
on Object are available to all classes unless explicitly overridden.

Object mixes in the Kernel module, making the built-in kernel functions
globally accessible. Although the instance methods of Object are defined
by the Kernel module, we have chosen to document them here for clarity.

When referencing constants in classes inheriting from Object you do not
need to use the full namespace. For example, referencing `File` inside
`YourClass` will find the top-level File class.

In the descriptions of Object's methods, the parameter *symbol* refers
to a symbol, which is either a quoted string or a Symbol (such as
`:name`).

<a href='https://ruby-doc.org/core-2.7.0/Object.html' class='ruby-doc
remote' target='_blank'>Object Reference</a>

