---
title: 'etc: Access to Unix /etc'
prev: "/stdlib/cli.html"
next: "/stdlib/cli/fcntl.html"
---


```ruby
require 'etc'
```

## Etc[](#etc)

The Etc module provides access to information typically stored in files in the /etc directory on Unix systems.

The information accessible consists of the information found in the /etc/passwd and /etc/group files, plus information about the system's temporary directory (/tmp) and configuration directory (/etc).

The Etc module provides a more reliable way to access information about the logged in user than environment variables such as +$USER+.

### Example:[](#example)


```ruby
require 'etc'

login = Etc.getlogin
info = Etc.getpwnam(login)
username = info.gecos.split(/,/).first
puts "Hello #{username}, I see your login name is #{login}"
```

Note that the methods provided by this module are not always secure. It should be used for informational purposes, and not for security.

All operations defined in this module are class methods, so that you can include the Etc module into your class.

<a href='https://ruby-doc.org/stdlib-2.7.0/libdoc/etc/rdoc/Etc.html' class='ruby-doc remote' target='_blank'>Etc Reference</a>

