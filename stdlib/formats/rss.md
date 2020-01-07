---
title: rss
prev: stdlib/formats/rexml.html
next: stdlib/formats/zlib.html
---


```ruby
require 'rss'
```

## RSS[](#rss)

## RSS reading and writing[](#rss-reading-and-writing)

Really Simple Syndication (RSS) is a family of formats that describe
'feeds,' specially constructed XML documents that allow an interested
person to subscribe and receive updates from a particular web service.
This portion of the standard library provides tooling to read and create
these feeds.

The standard library supports RSS 0.91, 1.0, 2.0, and Atom, a related
format. Here are some links to the standards documents for these
formats:

* RSS
  * <a href='http://www.rssboard.org/rss-0-9-1-netscape' class='remote'
    target='_blank'>0.9.1</a>
  * <a href='http://web.resource.org/rss/1.0/' class='remote'
    target='_blank'>1.0</a>
  * <a href='http://www.rssboard.org/rss-specification' class='remote'
    target='_blank'>2.0</a>

* <a href='http://tools.ietf.org/html/rfc4287' class='remote'
  target='_blank'>Atom</a>

### Consuming RSS[](#consuming-rss)

If you'd like to read someone's RSS feed with your Ruby code, you've
come to the right place. It's really easy to do this, but we'll need the
help of open-uri:


```ruby
require 'rss'
require 'open-uri'

url = 'http://www.ruby-lang.org/en/feeds/news.rss'
open(url) do |rss|
  feed = RSS::Parser.parse(rss)
  puts "Title: #{feed.channel.title}"
  feed.items.each do |item|
    puts "Item: #{item.title}"
  end
end
```

As you can see, the workhorse is `RSS::Parser#parse`, which takes the
source of the feed and a parameter that performs validation on the feed.
We get back an object that has all of the data from our feed, accessible
through methods. This example shows getting the title out of the channel
element, and looping through the list of items.

### Producing RSS[](#producing-rss)

Producing our own RSS feeds is easy as well. Let's make a very basic
feed:


```ruby
require "rss"

rss = RSS::Maker.make("atom") do |maker|
  maker.channel.author = "matz"
  maker.channel.updated = Time.now.to_s
  maker.channel.about = "http://www.ruby-lang.org/en/feeds/news.rss"
  maker.channel.title = "Example Feed"

  maker.items.new_item do |item|
    item.link = "http://www.ruby-lang.org/en/news/2010/12/25/ruby-1-9-2-p136-is-released/"
    item.title = "Ruby 1.9.2-p136 is released"
    item.updated = Time.now.to_s
  end
end

puts rss
```

As you can see, this is a very Builder-like DSL. This code will spit out
an Atom feed with one item. If we needed a second item, we'd make
another block with maker.items.new\_item and build a second one.

<a href='https://ruby-doc.org/stdlib-2.7.0/libdoc/rss/rdoc/RSS.html'
class='ruby-doc remote' target='_blank'>RSS Reference</a>

