# Active Record Connection Pool

```
ActiveRecord::ConnectionTimeoutError: could not obtain a connection from the pool within 5.000 seconds (waited 5.002 seconds); all pooled connections were in use
```

Encountering this? Seems like your Puma/Sidekiq or Rails configuration isn't quite right. This post is about correctly configuring them. There's also a [TL;DR](#tldr).

## The Connection Pool

In order to understand why the error above is happening, you need to understand Active Record's connection pool. The connection pool is a pool of database connections. The connection pool's maximum number of connections (or size) is defined by the `pool` setting of your config/database.yml file. The default is 5, which means there won't be more than 5 connections in the pool. The connections are lazily created, which means the pool starts with 0 connections and only ever creates one when needed. When a connection is idling in the pool for 300 seconds, it will be disconnected and removed.

Whenever Active Record is used and the database is to be queried, Rails will checkout a connection from the pool and use it. The connection will only be returned back to the pool if the thread that is using the connection dies or the thread returns it manually. This is a per-thread behaviour, so each thread will checkout a single connection. And that's basically the problem we've seen at the top: When all connections from the pool are in use and a thread tries to checkout a connection, it tries so for 5 seconds and then raises the mentioned exception.

The defaults are described at [api.rubyonrails.org](https://api.rubyonrails.org/classes/ActiveRecord/ConnectionAdapters/ConnectionPool.html#class-ActiveRecord::ConnectionAdapters::ConnectionPool-label-Options).

It's worth mentioning that each Rails process has its own independent connection pool. When having a Puma server running with 2 workers (= 2 processes), each worker would have its own connection pool.

## Approaching the Issue

The error above occurs when a thread tries to checkout a connection from the pool and all connections are in use for more than 5 seconds. This happens when the connection pool's size is smaller than the concurrency configuration of your application (may it be Puma, Sidekiq or whatever). Having less connections than your concurrency configuration doesn't mean there will be errors, though, but if the error raises, that's the reason.

The solution is simple, do either of these:

1. Increase/decrease the connection pool's size to match the application's concurrency configuration
2. Increase/decrease the application's concurrency configuration to match the connection pool's size

## Puma

Puma allows multi-process and multi-thread request processing. For Puma, "workers" are processes and "threads" are threads. To avoid `ActiveRecord::ConnectionTimeoutError` errors, make sure your Puma thread configuration is ≤ your connection pool's size.

When used in Rails 6.0.3.4, the default Puma thread configuration from config/puma.rb is `ENV.fetch("RAILS_MAX_THREADS") { 5 }`. So if `ENV["RAILS_MAX_THREADS"]` is present, that is used, if not, use 5 threads. Fortunately, the exact same applies for the connection pool's size. So when using the environment variable, adjusting `ENV["RAILS_MAX_THREADS"]` will just work and you shouldn't experience a `ActiveRecord::ConnectionTimeoutError`. When not using `ENV["RAILS_MAX_THREADS"]`, there's no problem as well as both the Puma threads and the connection pool's size will be 5.

## Sidekiq

Sidekiq allows multi-thread job processing. For Sidekiq, the concurrency configuration defines how many "workers" (= threads) will perform jobs. To avoid `ActiveRecord::ConnectionTimeoutError` errors, make sure your Sidekiq concurrency is ≤ your connection pool's size.

In Sidekiq 6.1.2, the concurrency is defined by `ENV["RAILS_MAX_THREADS"]` when present. If it's not present, the fallback is 10 threads (!). This can lead to errors as the connection pool's default size is 5. When using `ENV["RAILS_MAX_THREADS"]`, you shouldn't experience a `ActiveRecord::ConnectionTimeoutError`. So make sure to either have `ENV["RAILS_MAX_THREADS"]` present, use the Sidekiq CLI options (like `bundle exec sidekiq -c 5)`) or have a config/sidekiq.yml with `concurrency: 5`.

When multiple concurrency settings are present, the most left value has priority:

```
sidekiq.yml > CLI Option > ENV["RAILS_MAX_THREADS"] > 10
```

Note: Sidekiq 5.2.2 and earlier have a default value of 25.

## TL;DR

Make sure your concurrency configuration is ≤ your connection pool's size:

- Puma thread configuration ≤ connection pool size
- Sidekiq concurrency ≤ connection pool size
- If possible, use `ENV["RAILS_MAX_THREADS"]`
