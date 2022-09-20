# Deleting Files after Sending

From a controller action, [`send_file`](https://github.com/rails/rails/blob/v7.0.4/actionpack/lib/action_controller/metal/data_streaming.rb#L69-L78) lets you send a file:

```ruby
class ReportsController < ApplicationController
  def show
    file = Report.new.generate_csv
    send_file file, filename: 'report.csv'
  end
end
```

Now, `send_file` doesn't send the file immediately, but rather saves the path in a variable and uses it for delivery later on. That's why you shouldn't delete the file right after calling `send_file`. This will not work:

```ruby
def show
  file = Report.new.generate_csv
  send_file file, filename: 'report.csv'

  File.delete(file)
  File.unlink(file)
end
```

Using an `after_action` callback will behave the same, so that won't work either. If you're only using `delete` and not `unlink`, you might be lucky and it might work, but don't count on it.

A proper way would be to keep track of files and use a Rack middleware that deletes them after the response body was closed. Something like this:

```ruby
class FileReaper
  def initialize(app)
    @app = app
  end

  def call(env)
    env['app.files_to_reap'] ||= []
    status, headers, body = @app.call(env)

    body_proxy = Rack::BodyProxy.new(body) do
      env['app.files_to_reap'].each do |file|
        File.delete(file) if File.file?(file)
      end
    end

    [status, headers, body_proxy]
  end
end


# config/application.rb
config.middleware.use(FileReaper)

# app/controllers/reports_controller.rb
def show
  file = Report.new.generate_csv
  send_file file, filename: 'report.csv'

  request.env['app.files_to_reap'] << file
end
```

If you're dealing with `Tempfile` objects, there's already a `Rack::TempfileReaper` middleware for that that is enabled in Rails per default:

```ruby
def show
  tempfile = Report.new.generate_csv_tempfile
  send_file tempfile, filename: 'report.csv'

  request.env['rack.tempfiles'] << tempfile
end
```
