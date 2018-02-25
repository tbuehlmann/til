# GDBM

[GNU dbm](https://www.gnu.org.ua/software/gdbm/) (GDBM) is a simple database for storing key value pairs in a file. Keys and values are strings and a database can only be accessed by either many readers or exactly one writer at a time.

Ruby's standard library includes a [gdbm](https://github.com/ruby/gdbm) C extension that wraps around the library and let's us use it like a Hash.

## Writing

Writing key value pairs:

```ruby
require 'gdbm'

gdbm = GDBM.new('database.db')
gdbm['foo'] = 'bar'
gdbm.close
```

## Reading

```ruby
gdbm = GDBM.new('database.db')
gdbm['foo'] # => 'bar'
gdbm.close
```

As `GDBM` includes the `Enumerable` module, all its goodies are available.

## Block Shortcut

Instead of `.new` we can use `.open` which will open a database file, yield the `GDBM` object and close the database when it's done:

```ruby
GDBM.open('database.db') do |gdbm|
  gdbm['foo'] # => 'bar'
end
```

## Read-Only Access

The default access is `GDBM::WRITER` which can read and write data. If we only want to read data, we can open a database as a `GDBM::READER`:

```ruby
read_only_gdbm = GDBM.new('database.db', mode = 0666, GDBM::READER)
read_only_gdbm['foo'] # => 'bar'
```

… which will raise when trying to write:

```
read_only_gdbm['foo'] = 'baz' # => GDBMError: Reader can't store
```

## Concurrent Access

As stated, only many readers or exactly one writer can access a database at a time. But it seems the wrapper is clever for a single process, as:

```ruby
gdbm_writer = GDBM.new('database.db', mode = 0666, GDBM::WRITER)
gdbm_reader = GDBM.new('database.db', mode = 0666, GDBM::READER)
```

… works, but this doesn't:

```ruby
gdbm_writer = GDBM.new('database.db', mode = 0666, GDBM::WRITER)

fork do
  gdbm_reader = GDBM.new('database.db', mode = 0666, GDBM::READER) # raises Errno::EAGAIN: Resource temporarily unavailable - database.db
end
```
