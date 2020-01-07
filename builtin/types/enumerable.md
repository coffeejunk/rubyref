---
title: Enumerable
prev: "/builtin/types/times-dates.html"
next: "/builtin/types/array.html"
---

## Enumerable[](#enumerable)

The Enumerable mixin provides collection classes with several traversal and searching methods, and with the ability to sort. The class must provide a method `#each`, which yields successive members of the collection. If Enumerable#max, `#min`, or `#sort` is used, the objects in the collection must also implement a meaningful `<=>` operator, as these methods rely on an ordering between members of the collection.

**`Enumerable` is a very important module.** It is Ruby's way for performing almost any cycle. The module is included in collections, like `Array` and `Hash` (see next chapters), and some other classes (like `Range`).


```ruby
numbers = [1, 2, 8, 9, 18, 7]

numbers.each { |n| puts n }       # prints each number
numbers.map { |n| n**2 }          #=> [1, 4, 64, 81, 324, 49]
numbers.select { |n| n.odd? }     #=> [1, 9, 7]
numbers.reject { |n| n.odd? }     #=> [2, 8, 18]
numbers.partition { |n| n.odd? }  #=> [[1, 9, 7], [2, 8, 18]]
numbers.sort                      #=> [1, 2, 7, 8, 9, 18]
numbers.take_while { |n| n < 9 }  #=> [1, 2, 8]
numbers.drop_while { |n| n < 9 }  #=> [9, 18, 7]
# ...and so on

# Range is Enumerable, too
(1..10).select { |n| n.odd? }   #=> [1, 3, 5, 7, 9]
```

Also, many Ruby classes that are not `Enumerable` by themselves (like `String`) provide methods which return `Enumerator` (see below), which is also `Enumerable`, and can be processed in the same manner:


```ruby
"test".each_char                          #=> #<Enumerator: "test":each_char>
"test".each_char.select { |c| c < 't' }   #=> ["e", "s"]
"test".each_char.sort                     #=> ["e", "s", "t", "t"]
```

<a href='https://ruby-doc.org/core-2.7.0/Enumerable.html' class='ruby-doc remote' target='_blank'>Enumerable Reference</a>



### Enumerator[](#enumerator)

A class which allows both internal and external iteration.

An Enumerator can be created by the following methods.

* `Object#to_enum`
* `Object#enum_for`
* Enumerator.new

Most methods have two forms: a block form where the contents are evaluated for each item in the enumeration, and a non-block form which returns a new Enumerator wrapping the iteration.


```ruby
enumerator = %w(one two three).each
puts enumerator.class # => Enumerator

enumerator.each_with_object("foo") do |item, obj|
  puts "#{obj}: #{item}"
end

# foo: one
# foo: two
# foo: three

enum_with_obj = enumerator.each_with_object("foo")
puts enum_with_obj.class # => Enumerator

enum_with_obj.each do |item, obj|
  puts "#{obj}: #{item}"
end

# foo: one
# foo: two
# foo: three
```

This allows you to chain Enumerators together. For example, you can map a list's elements to strings containing the index and the element as a string via:


```ruby
puts %w[foo bar baz].map.with_index { |w, i| "#{i}:#{w}" }
# => ["0:foo", "1:bar", "2:baz"]
```

An Enumerator can also be used as an external iterator. For example, Enumerator#next returns the next value of the iterator or raises StopIteration if the Enumerator is at the end.


```ruby
e = [1,2,3].each   # returns an enumerator object.
puts e.next   # => 1
puts e.next   # => 2
puts e.next   # => 3
puts e.next   # raises StopIteration
```

You can use this to implement an internal iterator as follows:


```ruby
def ext_each(e)
  while true
    begin
      vs = e.next_values
    rescue StopIteration
      return $!.result
    end
    y = yield(*vs)
    e.feed y
  end
end

o = Object.new

def o.each
  puts yield
  puts yield(1)
  puts yield(1, 2)
  3
end

# use o.each as an internal iterator directly.
puts o.each {|*x| puts x; [:b, *x] }
# => [], [:b], [1], [:b, 1], [1, 2], [:b, 1, 2], 3

# convert o.each to an external iterator for
# implementing an internal iterator.
puts ext_each(o.to_enum) {|*x| puts x; [:b, *x] }
# => [], [:b], [1], [:b, 1], [1, 2], [:b, 1, 2], 3
```

<a href='https://ruby-doc.org/core-2.7.0/Enumerator.html' class='ruby-doc remote' target='_blank'>Enumerator Reference</a>



### Enumerator::Lazy[](#enumeratorlazy)

`Enumerator::Lazy` is a special type of `Enumerator`, with enumerating methods (like `#map`, `#select`, `#grep` and so on) redefined the way they are not processing values immediately, but gather a list of operations, which would be performed on subsequent `#force` or `#first`.

This allows idiomatic calculations on long or infinite sequence, as well as chaining of calculations without constructing intermediate arrays.

`Enumerator::Lazy` can be constructed from any `Enumerable` `#lazy` method.

Example:


```ruby
lazy = (1..Float::INFINITY).lazy.select(&:odd?).drop(10).take_while { |i| i < 30 }
# => #<Enumerator::Lazy: #<Enumerator::Lazy: #<Enumerator::Lazy: #<Enumerator::Lazy: 1..Infinity>:select>:drop(10)>:take_while>

lazy.force
#=> [21, 23, 25, 27, 29]

# or
lazy.first(2)
#=> [21, 23]
```



<a href='https://ruby-doc.org/core-2.7.0/Enumerator/Lazy.html' class='ruby-doc remote' target='_blank'>Enumerator::Lazy Reference</a>



### Enumerator::ArithmeticSequence[](#enumeratorarithmeticsequence)

<div class="since-version">Since Ruby 2.6</div>

Enumerator::ArithmeticSequence is a subclass of Enumerator, that is a representation of sequences of numbers with common difference. Instances of this class can be generated by the `Range#step` and `Numeric#step` methods.

<a href='https://ruby-doc.org/core-2.7.0/Enumerator/ArithmeticSequence.html' class='ruby-doc remote' target='_blank'>Enumerator::ArithmeticSequence Reference</a>



### Enumerator::Chain[](#enumeratorchain)

<div class="since-version">Since Ruby 2.6</div>

Enumerator::Chain is a subclass of Enumerator, which represents a chain of enumerables that works as a single enumerator.

This type of objects can be created by `Enumerable#chain` and Enumerator#+.

<a href='https://ruby-doc.org/core-2.7.0/Enumerator/Chain.html' class='ruby-doc remote' target='_blank'>Enumerator::Chain Reference</a>







