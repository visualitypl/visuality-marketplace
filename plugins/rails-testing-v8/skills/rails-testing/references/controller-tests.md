# Functional Testing for Controllers

Covers how to write functional controller tests: simulating requests to controller actions, inspecting the response, and asserting on side effects like redirects, flash messages, and database changes. Mirrors the **Functional Testing for Controllers** section of the Rails 8.1.3 testing guide (`guides/source/testing.md` §877–1278).

When writing functional tests, you are focusing on testing how controller actions handle the requests and the expected result or response. Functional controller tests are used to test controllers and other behavior, like API responses.

## What to Include in Your Functional Tests

You could test for things such as:

* was the web request successful?
* was the user redirected to the right page?
* was the user successfully authenticated?
* was the correct information displayed in the response?

The easiest way to see functional tests in action is to generate a controller using the scaffold generator:

```bash
$ bin/rails generate scaffold_controller article
...
create  app/controllers/articles_controller.rb
...
invoke  test_unit
create    test/controllers/articles_controller_test.rb
...
```

This will generate the controller code and tests for an `Article` resource.

If you already have a controller and just want to generate the test scaffold code for each of the seven default actions, use:

```bash
$ bin/rails generate test_unit:scaffold article
...
invoke  test_unit
create    test/controllers/articles_controller_test.rb
...
```

> **NOTE:** If you are generating test scaffold code, you will see an `@article` value set and used throughout the test file. This instance of `article` uses the attributes nested within a `:one` key in the `test/fixtures/articles.yml` file. Make sure you have set the key and related values in this file before you try to run the tests.

Here is the `test_should_get_index` test from `articles_controller_test.rb`:

```ruby
# test/controllers/articles_controller_test.rb
class ArticlesControllerTest < ActionDispatch::IntegrationTest
  test "should get index" do
    get articles_url
    assert_response :success
  end
end
```

Rails simulates a request on the `index` action, making sure the request was successful, and also ensuring that the right response body has been generated.

The `get` method kicks off the web request and populates the results into `@response`. It can accept up to 6 arguments:

| Argument | Purpose |
| :--- | :--- |
| URI (positional) | The URI of the controller action you are requesting. A string or a route helper (e.g. `articles_url`). |
| `params:` | A hash of request parameters to pass into the action (e.g. query string parameters or article attributes). |
| `headers:` | Headers to be passed with the request. |
| `env:` | Customizes the request environment. |
| `xhr:` | Whether the request is an AJAX request. Set to `true` to mark the request as AJAX. |
| `as:` | Encodes the request with a different content type (e.g. `:json`). |

All keyword arguments are optional.

Example: Calling the `:show` action (via a `get` request) for the first `Article`, passing in an `HTTP_REFERER` header:

```ruby
get article_url(Article.first), headers: { "HTTP_REFERER" => "http://example.com/home" }
```

Another example: Calling the `:update` action (via a `patch` request) for the last `Article`, passing in new text for the `title` in `params`, as an AJAX request:

```ruby
patch article_url(Article.last), params: { article: { title: "updated" } }, xhr: true
```

One more example: Calling the `:create` action (via a `post` request) to create a new article, passing in text for the `title` in `params`, as JSON request:

```ruby
post articles_url, params: { article: { title: "Ahoy!" } }, as: :json
```

> **NOTE:** If you try running `test_should_create_article` from `articles_controller_test.rb` it will (correctly) fail due to the newly added model-level validation.

To make `test_should_create_article` pass:

```ruby
# test/controllers/articles_controller_test.rb
test "should create article" do
  assert_difference("Article.count") do
    post articles_url, params: { article: { body: "Rails is awesome!", title: "Hello Rails" } }
  end

  assert_redirected_to article_path(Article.last)
end
```

See [assertions.md](assertions.md) for the full signatures of `assert_difference` and `assert_redirected_to`.

> **NOTE:** If you followed the Basic Authentication section of the Getting Started guide, you'll need to add an authorization header to every request:

```ruby
post articles_url,
  params: { article: { body: "Rails is awesome!", title: "Hello Rails" } },
  headers: { Authorization: ActionController::HttpAuthentication::Basic.encode_credentials("dhh", "secret") }
```

## HTTP Request Types for Functional Tests

There are 6 request types supported in Rails functional tests:

* `get`
* `post`
* `patch`
* `put`
* `head`
* `delete`

All of the request types have equivalent methods that you can use. In a typical CRUD application you'll use `post`, `get`, `put`, and `delete` most often.

> **NOTE:** Functional tests do not verify whether the specified request type is accepted by the action; instead, they focus on the result. For testing the request type, request tests are available, making your tests more purposeful.

## Testing XHR (AJAX) Requests

An AJAX request is a technique where content is fetched from the server using asynchronous HTTP requests and the relevant parts of the page are updated without requiring a full page load.

To test AJAX requests, specify the `xhr: true` option to `get`, `post`, `patch`, `put`, and `delete`:

```ruby
# test/controllers/articles_controller_test.rb
test "AJAX request" do
  article = articles(:one)
  get article_url(article), xhr: true

  assert_equal "hello world", @response.body
  assert_equal "text/javascript", @response.media_type
end
```

## Testing Other Request Objects

After any request has been made and processed, you will have 3 Hash objects ready for use:

| Object | Holds |
| :--- | :--- |
| `cookies` | Any cookies that are set. |
| `flash` | Any objects living in the flash. |
| `session` | Any object living in session variables. |

As is the case with normal Hash objects, you can access the values by string key. You can also reference them by symbol. For example:

```ruby
flash["gordon"]               # or flash[:gordon]
session["shmession"]          # or session[:shmession]
cookies["are_good_for_u"]     # or cookies[:are_good_for_u]
```

## Instance Variables

You also have access to three instance variables in your functional tests after a request is made:

| Variable | Holds |
| :--- | :--- |
| `@controller` | The controller processing the request. |
| `@request` | The request object. |
| `@response` | The response object. |

```ruby
# test/controllers/articles_controller_test.rb
class ArticlesControllerTest < ActionDispatch::IntegrationTest
  test "should get index" do
    get articles_url

    assert_equal "index", @controller.action_name
    assert_equal "application/x-www-form-urlencoded", @request.media_type
    assert_match "Articles", @response.body
  end
end
```

## Setting Headers and CGI Variables

HTTP headers are pieces of information sent along with HTTP requests to provide important metadata. CGI variables are environment variables used to exchange information between the web server and the application.

HTTP headers and CGI variables can both be passed as `headers:`:

```ruby
# setting an HTTP Header
get articles_url, headers: { "Content-Type": "text/plain" } # simulate the request with custom header

# setting a CGI variable
get articles_url, headers: { "HTTP_REFERER": "http://example.com/home" } # simulate the request with custom env variable
```

## Testing `flash` Notices

One of the three hash objects available in tests is `flash`. This section outlines how to test the appearance of a `flash` message whenever someone successfully creates a new article.

First, add an assertion to `test_should_create_article`:

```ruby
# test/controllers/articles_controller_test.rb
test "should create article" do
  assert_difference("Article.count") do
    post articles_url, params: { article: { title: "Some title" } }
  end

  assert_redirected_to article_path(Article.last)
  assert_equal "Article was successfully created.", flash[:notice]
end
```

If the test is run now, it should fail:

```bash
$ bin/rails test test/controllers/articles_controller_test.rb -n test_should_create_article
Running 1 tests in a single process (parallelization threshold is 50)
Run options: -n test_should_create_article --seed 32266

# Running:

F

Finished in 0.114870s, 8.7055 runs/s, 34.8220 assertions/s.

  1) Failure:
ArticlesControllerTest#test_should_create_article [/test/controllers/articles_controller_test.rb:16]:
--- expected
+++ actual
@@ -1 +1 @@
-"Article was successfully created."
+nil

1 runs, 4 assertions, 1 failures, 0 errors, 0 skips
```

Now implement the flash message in the controller:

```ruby
# app/controllers/articles_controller.rb
def create
  @article = Article.new(article_params)

  if @article.save
    flash[:notice] = "Article was successfully created."
    redirect_to @article
  else
    render "new"
  end
end
```

Now the tests should pass:

```bash
$ bin/rails test test/controllers/articles_controller_test.rb -n test_should_create_article
Running 1 tests in a single process (parallelization threshold is 50)
Run options: -n test_should_create_article --seed 18981

# Running:

.

Finished in 0.081972s, 12.1993 runs/s, 48.7972 assertions/s.

1 runs, 4 assertions, 0 failures, 0 errors, 0 skips
```

> **NOTE:** If you generated your controller using the scaffold generator, the flash message will already be implemented in your `create` action.

## Tests for `show`, `update`, and `delete` Actions

You can write a test for `:show` as follows:

```ruby
# test/controllers/articles_controller_test.rb
test "should show article" do
  article = articles(:one)
  get article_url(article)
  assert_response :success
end
```

The `articles()` accessor returns the Active Record instance for a fixture — see [test-database.md](test-database.md) for how fixture accessors work.

Deleting an existing article:

```ruby
# test/controllers/articles_controller_test.rb
test "should delete article" do
  article = articles(:one)
  assert_difference("Article.count", -1) do
    delete article_url(article)
  end

  assert_redirected_to articles_path
end
```

Updating an existing article:

```ruby
# test/controllers/articles_controller_test.rb
test "should update article" do
  article = articles(:one)

  patch article_url(article), params: { article: { title: "updated" } }

  assert_redirected_to article_path(article)
  # Reload article to refresh data and assert that title is updated.
  article.reload
  assert_equal "updated", article.title
end
```

There is some duplication in these three tests — they all access the same article fixture data. It is possible to DRY the implementation by using the `setup` and `teardown` methods provided by `ActiveSupport::Callbacks`:

```ruby
# test/controllers/articles_controller_test.rb
require "test_helper"

class ArticlesControllerTest < ActionDispatch::IntegrationTest
  # called before every single test
  setup do
    @article = articles(:one)
  end

  # called after every single test
  teardown do
    # when controller is using cache it may be a good idea to reset it afterwards
    Rails.cache.clear
  end

  test "should show article" do
    # Reuse the @article instance variable from setup
    get article_url(@article)
    assert_response :success
  end

  test "should destroy article" do
    assert_difference("Article.count", -1) do
      delete article_url(@article)
    end

    assert_redirected_to articles_path
  end

  test "should update article" do
    patch article_url(@article), params: { article: { title: "updated" } }

    assert_redirected_to article_path(@article)
    # Reload association to fetch updated data and assert that title is updated.
    @article.reload
    assert_equal "updated", @article.title
  end
end
```

> **NOTE:** Similar to other callbacks in Rails, the `setup` and `teardown` methods can also accept a block, lambda, or a method name as a symbol to be called.
