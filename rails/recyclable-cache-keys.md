# Recyclable Cache Keys

In earlier versions, Rails would generate cache keys for Active Record objects using the model name, the id and the updated_at timestamp:

```ruby
Project.first.cache_key # => 'projects/1-20180522081941758726000'
```

When an object changed a lot, we would have a lot of trash cache keys in the cache store, waiting for expiration.

Since Rails 5.2, cache keys for Active Record objects are stable and generated without a timestamp:

```ruby
Project.first.cache_key # => 'projects/1'
```

Having this, we don't trash our cache store with cache keys that are stale, but instead override the same cache key over and over. Therefore the term "recyclable cache keys".

## Internals

Internally, when caching an object, Rails will not cache that object directly. Instead, it will cache an `ActiveSupport::Cache::Entry` which wraps the object and adds some information: The creation date of that entry and the expiration time in seconds (for cache stores that can't expire on their own) and a version. The version part is new and consists of the updated_at timestamp for ActiveRecord object cache keys.

When using `Rails.cache.fetch`, the actual version is compared to the cached version and if there's a mismatch it is considered a cache miss.

## Opting out

When we want to use old-fashioned cache keys, we can use the `#cache_key_with_version` method:

```ruby
Project.first.cache_key_with_version # => 'projects/1-20180522081941758726000'
```

… or disable cache versioning completely using:

```ruby
config.active_record.cache_versioning = false
```
