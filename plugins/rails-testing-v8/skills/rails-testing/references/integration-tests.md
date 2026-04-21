# Integration Testing

Covers how to write integration tests: simulating a sequence of requests that span multiple controllers to verify application workflows end-to-end. Mirrors the **Integration Testing** section of the Rails 8.1.3 testing guide (`guides/source/testing.md` §1279–1416).

Integration tests take functional controller tests one step further — they focus on testing how several parts of an application interact, and are generally used to test important workflows.

Rails integration tests are stored in the `test/integration` directory.

Rails provides a generator to create an integration test skeleton:

```bash
$ bin/rails generate integration_test user_flows
      invoke  test_unit
      create  test/integration/user_flows_test.rb
```

Here's what a freshly generated integration test looks like:

```ruby
# test/integration/user_flows_test.rb
require "test_helper"

class UserFlowsTest < ActionDispatch::IntegrationTest
  # test "the truth" do
  #   assert true
  # end
end
```

Here the test is inheriting from [`ActionDispatch::IntegrationTest`](https://api.rubyonrails.org/classes/ActionDispatch/IntegrationTest.html). This makes some additional helpers available for integration tests (see [Helpers Available for Integration Tests](#helpers-available-for-integration-tests)) alongside the standard testing helpers.

## Implementing an Integration Test

Let's add an integration test to our blog application, by starting with a basic workflow of creating a new blog article to verify that everything is working properly.

Start by generating the integration test skeleton:

```bash
$ bin/rails generate integration_test blog_flow
```

It should have created a test file placeholder. With the output of the previous command you should see:

```
      invoke  test_unit
      create    test/integration/blog_flow_test.rb
```

Now open that file and write the first assertion:

```ruby
# test/integration/blog_flow_test.rb
require "test_helper"

class BlogFlowTest < ActionDispatch::IntegrationTest
  test "can see the welcome page" do
    get "/"
    assert_dom "h1", "Welcome#index"
  end
end
```

If you visit the root path, you should see `welcome/index.html.erb` rendered for the view. So this assertion should pass.

> **NOTE:** The assertion `assert_dom` (aliased to `assert_select`) is available in integration tests to check the presence of key HTML elements and their content.

#### Creating Articles Integration

To test the ability to create a new article in our blog and display the resulting article, see the example below:

```ruby
# test/integration/blog_flow_test.rb
test "can create an article" do
  get "/articles/new"
  assert_response :success

  post "/articles",
    params: { article: { title: "can create", body: "article successfully." } }
  assert_response :redirect
  follow_redirect!
  assert_response :success
  assert_dom "p", "Title:\n  can create"
end
```

The `:new` action of our Articles controller is called first, and the response should be successful.

Next, a `post` request is made to the `:create` action of the Articles controller:

```ruby
post "/articles",
  params: { article: { title: "can create", body: "article successfully." } }
assert_response :redirect
follow_redirect!
```

The two lines following the request are to handle the redirect setup when creating a new article.

> **NOTE:** Don't forget to call `follow_redirect!` if you plan to make subsequent requests after a redirect is made.

Finally it can be asserted that the response was successful and the newly-created article is readable on the page.

A very small workflow for visiting our blog and creating a new article was successfully tested above. To extend this, additional tests could be added for features like adding comments, editing comments or removing articles. Integration tests are a great place to experiment with all kinds of use cases for our applications.

## Helpers Available for Integration Tests

There are numerous helpers to choose from for use in integration tests. Some include:

| Helper module | Purpose |
| :--- | :--- |
| [`ActionDispatch::Integration::Runner`](https://api.rubyonrails.org/classes/ActionDispatch/Integration/Runner.html) | Helpers relating to the integration test runner, including creating a new session. |
| [`ActionDispatch::Integration::RequestHelpers`](https://api.rubyonrails.org/classes/ActionDispatch/Integration/RequestHelpers.html) | Performing requests. |
| [`ActionDispatch::TestProcess::FixtureFile`](https://api.rubyonrails.org/classes/ActionDispatch/TestProcess/FixtureFile.html) | Uploading files. |
| [`ActionDispatch::Integration::Session`](https://api.rubyonrails.org/classes/ActionDispatch/Integration/Session.html) | Modifying sessions or the state of the integration tests. |

## Source traceability

| Claim | Source |
| :--- | :--- |
| `bin/rails generate integration_test <name>` creates `test/integration/<name>_test.rb` | `railties/lib/rails/generators/test_unit/integration/integration_generator.rb` line 11; `railties/lib/rails/generators/test_unit/integration/templates/integration_test.rb.tt` |
| Generated skeleton inherits from `ActionDispatch::IntegrationTest` | `railties/lib/rails/generators/test_unit/integration/templates/integration_test.rb.tt` line 4 |
| `ActionDispatch::IntegrationTest < ActiveSupport::TestCase` | `actionpack/lib/action_dispatch/testing/integration.rb` line 650 |
| `follow_redirect!(headers: {}, **args)` re-uses the original verb for 307/308, otherwise GET | `actionpack/lib/action_dispatch/testing/integration.rb` lines 65–81 |
| `host!` (alias for `host=`) sets the hostname used in the next request | `actionpack/lib/action_dispatch/testing/integration.rb` line 316 |
| `https!(flag = true)` mimics a secure HTTPS request | `actionpack/lib/action_dispatch/testing/integration.rb` line 180 |
| `open_session` duplicates the current session so parallel sessions can be tested side-by-side | `actionpack/lib/action_dispatch/testing/integration.rb` lines 405–411 |
| `BlogFlowTest` / `CreatingArticlesTest` examples and `assert_dom`/`assert_select` alias note | `guides/source/testing.md` §Integration Testing |
| `get`/`post` request helpers — cross-linked | `actionpack/lib/action_dispatch/testing/integration.rb`; see [controller-tests.md](controller-tests.md) |
| `assert_response`, `assert_redirected_to` — cross-linked | `actionpack/lib/action_dispatch/testing/assertions/response.rb`; see [writing-tests.md](writing-tests.md) |

Rails version: **8.1.3**.
