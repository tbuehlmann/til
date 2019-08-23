# Headless System Tests

System Tests internally use Capybara. The default driver used by Capybara is `:selenium`. This driver can use different browsers to request pages, where Google Chrome is the default.

A regular System Test setup looks like this:

```ruby
# test/system/projects_test.rb

require 'application_system_test_case'

class ProjectsTest < ApplicationSystemTestCase
  test 'visiting the index' do
    visit projects_url
    assert_selector 'h1', text: 'Projects'
  end
end
```

where `ApplicationSystemTestCase` is defined as:

```ruby
# test/application_system_test_case.rb

require 'test_helper'

class ApplicationSystemTestCase < ActionDispatch::SystemTestCase
  driven_by :selenium, using: :chrome, screen_size: [1400, 1400]
end
```

Running this test will visually open up Google Chrome, visit the page and run the assertion.

## Beheading the Browser

If we don't want to open up Google Chrome (or if we can't do so, think of a CI pipeline), we can configure the System Test to use a headless Google Chrome as its browser:

```ruby
class ApplicationSystemTestCase < ActionDispatch::SystemTestCase
  driven_by :selenium, using: :headless_chrome, screen_size: [1400, 1400]
end
```

This will not open up Google Chrome visually, but instead headless, not visible at all.

## System Specs in RSpec

System Specs in RSpec don't use the `ApplicationSystemTestCase`, but we can use `driven_by` to override the default behaviour in our spec:

```ruby
# spec/system/projects_spec.rb
require 'rails_helper'

RSpec.describe 'Project management', type: :system do
  before do
    driven_by :selenium, using: :headless_chrome, screen_size: [1400, 1400]
  end

  it 'displays projects' do
    visit projects_url
    expect(page).to have_text('Projects')
  end
end
```

## Using Docker

When using Docker (or rather: using root), Google Chrome won't work as expected and needs the `--no-sandbox` option to properly function:

```ruby
before do
  driven_by :selenium, using: :headless_chrome, screen_size: [1400, 1400] do |driver_option|
    driver_option.add_argument('--no-sandbox')
  end
end
```
