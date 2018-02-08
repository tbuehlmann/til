# Prepending Modules

When calling `SomeClass.new.some_method`, Ruby first looks for the method `some_method` in `SomeClass`, then `Object`, then `Kernel` and then `BasicObject`. That's the class's ancestor chain:

```ruby
SomeClass = Class.new
SomeClass.ancestors # => [SomeClass, Object, Kernel, BasicObject]
```

When including a module into a class, that module comes right after the class itself:

```ruby
SomeModule = Module.new

class SomeClass
  include SomeModule
end

SomeClass.ancestors # => [SomeClass, SomeModule, Object, Kernel, BasicObject]
```

Since Ruby 2.0.0 we can prepend a module to a class, which will place it to the ancestor chain's first position:

```ruby
SomeModule = Module.new

class SomeClass
  prepend SomeModule
end

SomeClass.ancestors # => [SomeModule, SomeClass, Object, Kernel, BasicObject]
```

Note: The actual method lookup described here is simplified and does not mention singleton classes and other internals.
