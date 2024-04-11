# Enqueue after Transaction Commit

https://github.com/rails/rails/pull/51426 ([this week in rails](https://rubyonrails.org/2024/4/5/this-week-in-rails)) introduced a feature that, when inside a transaction, will enqueue Active Jobs only after the transaction commits:

```ruby
Topic.transaction do
  topic = Topic.create
  NewTopicNotificationJob.perform_later(topic)
end
```

In this example, `NewTopicNotificationJob` will not be enqueued inside the transaction, it enqueues when the transactions commits. This feature is enabled by default for Job Backends that don't share the database with Active Record (Async, Sidekiq, SuckerPunch, the Test Adapter, …).
