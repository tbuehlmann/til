# RSpec::Rails Types

Controller-related RSpec::Rails Spec Types are:


| Tests      | Rails Name       | Uses                            | RSpec Name      | RSpec Type    |
|:----------:|:----------------:|:-------------------------------:|:---------------:|:-------------:|
| Model      | Model Test       | ActiveSupport::TestCase         | Model Spec      | `:model`      |
| Controller | Controller Test  | ActionController::TestCase      | Controller Spec | `:controller` |
| Controller | Integration Test | ActionDispatch::IntegrationTest | Request Spec    | `:request`    |
| System     | System Test      | ActionDispatch::SystemTestCase  | System Spec     | `:system`     |
| System     |                  |                                 | Feature Spec    | `:feature`    |

# Controller Spec

Controller Specs don't use Rails' middleware stack (and therefore no routing). They internally run `ProjectsController.new(…).dispatch!` in the same thread.

Example:

```ruby
RSpec.describe ProjectsController, type: :controller do
  describe 'GET index' do
    it 'responds with OK' do
      get :index
      expect(response).to have_http_status(:ok)
    end
  end
end
```

# Request Spec

Request Specs use Rails's middleware stack (therefore using routing), effectively using `Rails.application`. They internally create an `ActionDispatch::Integration::Session` (which uses `Rack::Test::Session.new(Rack::MockSession.new(@app, host))`) in the same thread.

Example:

```ruby
RSpec.describe 'Projects', type: :request do
  describe 'GET /projects' do
    it 'responds with OK' do
      get projects_path
      expect(response).to have_http_status(:ok)
    end
  end
end
```

# System Spec

System Specs run an application server (like Puma) in a different thread using Capybara. Calling `visit` sends a request to that server.

Example:

```ruby
RSpec.describe 'Projects', type: :system do
  describe 'GET /projects' do
    it 'displays a title' do
      visit projects_url
      expect(page).to have_text('Projects')
    end
  end
end
```

# Feature Spec

Feature Specs work like System Specs, they simply existed before Rails added System Tests. If possible, use System Specs over Feature Specs.
