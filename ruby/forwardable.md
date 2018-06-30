# Forwardable and SingleForwardable

[Forwardable](https://ruby-doc.org/stdlib-2.5.1/libdoc/forwardable/rdoc/Forwardable.html) and [SingleForwardable](http://ruby-doc.org/stdlib-2.5.1/libdoc/forwardable/rdoc/SingleForwardable.html) offer methods to forward (or delegate) method calls to a designated object.

## Forwardable

Forwardable supports three methods of delegating methods: `def_delegator`, `def_delegators` and `delegate`.

### def_delegator

`def_delegator` delegates single methods:

```ruby
class User
  extend Forwardable

  def_delegator :@address, :street
  def_delegator :@address, :city, :address_city

  # basically defines:
  #
  # def street
  #   @address.street
  # end
  #
  # def address_city
  #   @address.city
  # end
end
```

### def_delegators

`def_delegators` delegates several methods (using `def_delegator`):

```ruby
class User
  extend Forwardable

  def_delegators :@address, :street, :city

  # is basically the same as:
  #
  # def_delegator :@address, :street
  # def_delegator :@address, :city
end
```

When using `def_delegators`, the `__send__` and `__id__` methods are ignored so they are not overriden.

### delegate

`delegate` delegates methods (using `def_delegator`):

```ruby
class User
  extend Forwardable

  delegate :street => :@address
  delegate :city => :@address

  # or:
  delegate [:street, :city] => :@address

  # or:
  delegate street: :@address, city: :@address

  # is the same as:
  #
  # def_delegator :@address, :street
  # def_delegator :@address, :city
end
```

## SingleForwardable

SingleForwardable works like Forwardable but on the object level:

```ruby
user = User.new
user.extend(SingleForwardable)

user.def_delegator(:@address, :street)
user.def_delegator(:@address, :city)

# or
user.def_delegators(:@address, :street, :city)

# or
user.delegate([:street, :city] => :@address)
```
