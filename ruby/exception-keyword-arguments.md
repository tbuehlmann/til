# Exception Keyword Argument

Ruby 2.6 introduces a new `exception` Keyword Argument for certain Kernel methods which changes their behaviour:

```ruby
Integer('not-a-number') # => ArgumentError (invalid value for Integer(): "not-a-number")
Integer('not-a-number', exception: false) # nil

Float('not-a-number') # => ArgumentError (invalid value for Float(): "not-a-number")
Float('not-a-number', exception: false) # => nil

Complex('not-a-number') # => ArgumentError (invalid value for convert(): "not-a-number")
Complex('not-a-number', exception: false) # => nil

Rational('not-a-number') # => ArgumentError (invalid value for convert(): "not-a-number")
Rational('not-a-number', exception: false) # => nil

system('not-a-command') # => nil
system('not-a-command', exception: true) # => Errno::ENOENT (No such file or directory - not-a-command)
```

For `Kernel#system`, `exception: true` will trigger if execution succeeds but the exit status is non-zero:

```ruby
system('exit 1') # => false
system('exit 2', exception: true) # => RuntimeError (Command failed with exit 1: exit 1)
```

Props to Shannon Skipper for his [article](https://medium.com/square-corner-blog/rubys-new-exception-keyword-arguments-4d5bbb504d37) on the topic.
