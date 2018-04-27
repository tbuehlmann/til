# Pass By

What's the difference between Pass By Value and Pass By Reference and what does Ruby use?

## Pass By Value

Pass By Value means that when we're passing anything into a method, a mere copy of its value is used inside the method. If Ruby were strictly Pass By Value, this would happen:

```ruby
def change(just_a_copy)
  just_a_copy[:foo] = 'bar'
end

some_hash = {}
change(some_hash)

some_hash # => {}
```

## What is Pass By Reference?

Pass By Reference means that when we're passing anything into a method, the same reference will be used inside the method. If Ruby were Pass By Reference, this would happen:

```ruby
def change(same_variable)
  same_variable = 'something else'
end

variable = 'something'
change(variable)

variable # => 'something else'
```

## What is Ruby?

Ruby is Pass By Value, but the copied values are actually Object References. That's why changing an object in a method changes it for all variables referencing that Object.

See apeiros' [Gist](https://gist.github.com/apeiros/bf3ddb3d332dc01ac43840e0d08b382d).
