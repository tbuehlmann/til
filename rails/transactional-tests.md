# Transactional Tests

Transactional Tests (known as Transactional Fixtures in Rails < 5.0.0) is a configuration option that, when enabled, wraps each test in a database transaction which is being rolled back in a teardown.

The configuration is enabled per [default](https://github.com/rails/rails/blob/v5.2.0/activerecord/lib/active_record/fixtures.rb#L873).

## Problem

A recurring problem with integration tests was that the application server used by Capybara was running in a different thread than the test and therefore used a different database connection. Since a database transaction was wrapping the test, the application server wouldn't see any changes to the database.

This lead to disabling Transactional Tests/Fixtures and using a database cleaner library to clear the database state before/after each test like [DatabaseRewinder](https://github.com/amatsuda/database_rewinder) or [DatabaseCleaner](https://github.com/DatabaseCleaner/database_cleaner).

This behaviour changed with [rails!28083](https://github.com/rails/rails/pull/28083). From Rails 5.1.0 onwards, the test and application server would be locked to use the same connection, effectively fixing the problem and removing the need for a database cleaner.

This new behaviour looks to be present for all kinds of tests but [is assumed](https://github.com/rails/rails/pull/28083#issuecomment-313493049) to only work for System Tests.

## RSpec::Rails

The same behaviour is available for RSpec::Rails when enabling the config:

```ruby
RSpec.configure do |config|
  config.use_transactional_fixtures = true
end
```

The config's name might change in the future, but now (RSpec::Rails 3.7.2) it's `use_transactional_fixtures`.
