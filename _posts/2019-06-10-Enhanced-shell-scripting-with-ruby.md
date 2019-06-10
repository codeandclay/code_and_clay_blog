---
title: Enhance shell scripting with Ruby
layout: post
---

From [Dev_Dungeon](https://www.devdungeon.com/content/enhanced-shell-scripting-ruby):

> Ruby is a better Perl and in my opinion is an essential language for system administrators. If you are still writing scripts in Bash, I hope this inspires you to start integrating Ruby in to your shell scripts. I will show you how you can ease in to it and make the transition very smoothly.

I've learnt a few things from the article:

> Debug output, and anything that does not belong in the output of the application should go to STDERR and only data that should be piped to anothe application or stored should go to STDOUT

There's `ARGF` in addition to `ARGV`.

I can ask the user for a password without the password being echoed back to the terminal:

```ruby
#!/usr/bin/ruby
require 'io/console'

# The prompt is optional
password = IO::console.getpass "Enter Password: "
puts "Your password was #{password.length} characters long."
```

I can check for return codes and errors via the `$?` object.

I can trap interrupt signals with `Signal.trap()`.

```ruby
#!/usr/bin/ruby

Signal.trap("SIGINT") {
    puts 'Caught a CTRL-C / SIGINT. Shutting down cleanly.'
    exit(true)
}

puts 'Running forever until CTRL-C / SIGINT signal is recieved.'
while true do end
```
