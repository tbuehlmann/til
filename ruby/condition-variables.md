# Condition Variables

Condition variables can put threads to sleep and wake them later on when a certain condition is met. This is useful for implementing blocking method calls without having to poll for other objects or using expensive loops. They have to be used in a `Mutex#synchronize` call.

When we want to put a thread to sleep, we can call `condition_variable.wait(some_mutex)`. This will put the calling thread to sleep and unlock the mutex. This is not limited to a single thread but several threads can wait on a single condition variable. In order to wake exactly one thread up again, we can call `condition_variable.signal`. This will wake up the first sleeping/waiting thread, locking the mutex again.

We could also wake up all sleeping threads by calling `condition_variable.broadcast`.

## Example

Below is an example of a Queue class with a blocking `shift` method. When the queue is empty and a thread is trying to shift an item off of it, the condition variable will put the thread to sleep, effectively blocking the method call. Once there is an item added to the queue, the condition variable will signal and wake up the thread.

```ruby
class BlockingQueue
  def initialize
    @items = []
    @mutex = Mutex.new
    @condition_variable = ConditionVariable.new
  end

  def <<(item)
    @mutex.synchronize do
      @items << item
      @condition_variable.signal
    end
  end

  def shift
    @mutex.synchronize do
      while @items.empty?
        @condition_variable.wait(@mutex)
      end

      @items.shift
    end
  end
end

queue = BlockingQueue.new

consumer = Thread.new do
  puts 'Consumer: Waiting for an Item'
  item = queue.shift
  puts 'Consumer: Shifted an Item'
end

producer = Thread.new do
  sleep 2
  puts 'Producer: Adding an Item'
  queue << 'Some Item'
end

[consumer, producer].each(&:join)
```

… will output:

```
Consumer: Waiting for an Item
Producer: Adding an Item
Consumer: Shifted an Item
```

## Spurious Wakeups

It's possible for threads to spuriously wakeup from a condition variable induced sleep without receiving a signal or broadcast. This has to do with speeding up condition variable operations and can be read up [here](https://stackoverflow.com/questions/8594591/why-does-pthread-cond-wait-have-spurious-wakeups/8594644#8594644). That's also a reason for always using a `while` loop around a `condition_variable.wait(mutex)`, so a spurios wakeup won't do any harm.

## Credits

Credits go to Tom Van Eyck for his excellent blog post on [Ruby concurrency: in praise of condition variables](https://vaneyckt.io/posts/ruby_concurrency_in_praise_of_condition_variables/).
