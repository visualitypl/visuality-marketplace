# System Testing

Covers how to write system tests: full-browser tests that exercise the application the way a user would. Mirrors the **System Testing** section of the Rails 8.1.3 testing guide (`guides/source/testing.md` §1417–1746).

Similarly to integration testing, system testing allows you to test how the components of your app work together, but from the point of view of a user. It does this by running tests in either a real or a headless browser (a browser which runs in the background without opening a visible window). System tests use [Capybara](https://www.rubydoc.info/github/jnicklas/capybara) under the hood.

## When to Use System Tests

System tests provide the most realistic testing experience as they test your application from a user's perspective. However, they come with important trade-offs:

* **They are significantly slower** than unit and integration tests
* **They can be brittle** and prone to failures from timing issues or UI changes
* **They require more maintenance** as your UI evolves

Given these trade-offs, **system tests should be reserved for critical user paths** rather than being created for every feature. Consider writing system tests for:

* **Core business workflows** (e.g., user registration, checkout process, payment flows)
* **Critical user interactions** that integrate multiple components
* **Complex JavaScript interactions** that can't be tested at lower levels

For most features, integration tests provide a better balance of coverage and maintainability. Save system tests for scenarios where you need to verify the complete user experience.

## Generating System Tests

Rails no longer generates system tests by default when using scaffolds. This change reflects the best practice of using system tests sparingly. You can generate system tests in two ways:

1. **When scaffolding**, explicitly enable system tests:

    ```bash
    $ bin/rails generate scaffold Article title:string body:text --system-tests=true
    ```

2. **Generate system tests independently** for critical features:

    ```bash
    $ bin/rails generate system_test articles
    ```

Rails system tests are stored in the `test/system` directory in your application. To generate a system test skeleton, run the following command:

```bash
$ bin/rails generate system_test users
      invoke  test_unit
      create    test/application_system_test_case.rb
      create    test/system/users_test.rb
```

Here's what a freshly generated system test looks like:

```ruby
# test/system/users_test.rb
require "application_system_test_case"

class UsersTest < ApplicationSystemTestCase
  # test "visiting the index" do
  #   visit users_url
  #
  #   assert_dom "h1", text: "Users"
  # end
end
```

By default, system tests are run with the Selenium driver, using the Chrome browser, and a screen size of 1400x1400. The next section explains how to change the default settings.

## Changing the Default Settings

Rails makes changing the default settings for system tests very simple. All the setup is abstracted away so you can focus on writing your tests.

When you generate system tests, an `application_system_test_case.rb` file is created in the test directory. This is where all the configuration for your system tests should live.

If you want to change the default settings, you can change what the system tests are "driven by". If you want to change the driver from Selenium to Cuprite, you'd add the [`cuprite`](https://github.com/rubycdp/cuprite) gem to your `Gemfile`. Then in your `application_system_test_case.rb` file you'd do the following:

```ruby
# test/application_system_test_case.rb
require "test_helper"
require "capybara/cuprite"

class ApplicationSystemTestCase < ActionDispatch::SystemTestCase
  driven_by :cuprite
end
```

The driver name is a required argument for `driven_by`. The optional arguments that can be passed to `driven_by` are `:using` for the browser (this will only be used by Selenium), `:screen_size` to change the size of the screen for screenshots, and `:options` which can be used to set options supported by the driver.

```ruby
# test/application_system_test_case.rb
require "test_helper"

class ApplicationSystemTestCase < ActionDispatch::SystemTestCase
  driven_by :selenium, using: :firefox
end
```

If you want to use a headless browser, you could use Headless Chrome or Headless Firefox by adding `headless_chrome` or `headless_firefox` in the `:using` argument.

```ruby
# test/application_system_test_case.rb
require "test_helper"

class ApplicationSystemTestCase < ActionDispatch::SystemTestCase
  driven_by :selenium, using: :headless_chrome
end
```

If you want to use a remote browser, e.g. [Headless Chrome in Docker](https://github.com/SeleniumHQ/docker-selenium), you have to add a remote `url` and set `browser` as remote through `options`.

```ruby
# test/application_system_test_case.rb
require "test_helper"

class ApplicationSystemTestCase < ActionDispatch::SystemTestCase
  url = ENV.fetch("SELENIUM_REMOTE_URL", nil)
  options = if url
    { browser: :remote, url: url }
  else
    { browser: :chrome }
  end
  driven_by :selenium, using: :headless_chrome, options: options
end
```

Now you should get a connection to the remote browser.

```bash
$ SELENIUM_REMOTE_URL=http://localhost:4444/wd/hub bin/rails test:system
```

If your application is remote, e.g. within a Docker container, Capybara needs more input about how to [call remote servers](https://github.com/teamcapybara/capybara#calling-remote-servers).

```ruby
# test/application_system_test_case.rb
require "test_helper"

class ApplicationSystemTestCase < ActionDispatch::SystemTestCase
  setup do
      Capybara.server_host = "0.0.0.0" # bind to all interfaces
      Capybara.app_host = "http://#{IPSocket.getaddress(Socket.gethostname)}" if ENV["SELENIUM_REMOTE_URL"].present?
    end
  # ...
end
```

Now you should get a connection to a remote browser and server, regardless if it is running in a Docker container or CI.

If your Capybara configuration requires more setup than provided by Rails, this additional configuration can be added into the `application_system_test_case.rb` file.

Please see [Capybara's documentation](https://github.com/teamcapybara/capybara#setup) for additional settings.

## Implementing a System Test

This section will demonstrate how to add a system test to your application, which tests a visit to the index page to create a new blog article.

> **NOTE:** The scaffold generator no longer creates system tests by default. To include system tests when scaffolding, use the `--system-tests=true` option. Otherwise, create system tests manually for your critical user paths.

```bash
$ bin/rails generate system_test articles
```

It should have created a test file placeholder. With the output of the previous command you should see:

```
      invoke  test_unit
      create    test/application_system_test_case.rb
      create    test/system/articles_test.rb
```

Now, let's open that file and write the first assertion:

```ruby
# test/system/articles_test.rb
require "application_system_test_case"

class ArticlesTest < ApplicationSystemTestCase
  test "viewing the index" do
    visit articles_path
    assert_selector "h1", text: "Articles"
  end
end
```

The test should see that there is an `h1` on the articles index page and pass.

Run the system tests.

```bash
$ bin/rails test:system
```

> **NOTE:** By default, running `bin/rails test` won't run your system tests. Make sure to run `bin/rails test:system` to actually run them. You can also run `bin/rails test:all` to run all tests, including system tests.

### Creating Articles System Test

Now you can test the flow for creating a new article.

```ruby
# test/system/articles_test.rb
test "should create Article" do
  visit articles_path

  click_on "New Article"

  fill_in "Title", with: "Creating an Article"
  fill_in "Body", with: "Created this article successfully!"

  click_on "Create Article"

  assert_text "Creating an Article"
end
```

The first step is to call `visit articles_path`. This will take the test to the articles index page.

Then the `click_on "New Article"` will find the "New Article" button on the index page. This will redirect the browser to `/articles/new`.

Then the test will fill in the title and body of the article with the specified text. Once the fields are filled in, "Create Article" is clicked on which will send a POST request to `/articles/create`.

This redirects the user back to the articles index page, and there it is asserted that the text from the new article's title is on the articles index page.

### Testing for Multiple Screen Sizes

If you want to test for mobile sizes in addition to testing for desktop, you can create another class that inherits from `ActionDispatch::SystemTestCase` and use it in your test suite. In this example, a file called `mobile_system_test_case.rb` is created in the `/test` directory with the following configuration.

```ruby
# test/mobile_system_test_case.rb
require "test_helper"

class MobileSystemTestCase < ActionDispatch::SystemTestCase
  driven_by :selenium, using: :chrome, screen_size: [375, 667]
end
```

To use this configuration, create a test inside `test/system` that inherits from `MobileSystemTestCase`. Now you can test your app using multiple different configurations.

```ruby
# test/system/posts_test.rb
require "mobile_system_test_case"

class PostsTest < MobileSystemTestCase
  test "visiting the index" do
    visit posts_url
    assert_selector "h1", text: "Posts"
  end
end
```

### Capybara Assertions

Here's an extract of the assertions provided by [`Capybara`](https://rubydoc.info/github/teamcapybara/capybara/master/Capybara/Minitest/Assertions) which can be used in system tests.

| Assertion | Purpose |
| :--- | :--- |
| `assert_button(locator = nil, **options, &optional_filter_block)` | Checks if the page has a button with the given text, value or id. |
| `assert_current_path(string, **options)` | Asserts that the page has the given path. |
| `assert_field(locator = nil, **options, &optional_filter_block)` | Checks if the page has a form field with the given label, name or id. |
| `assert_link(locator = nil, **options, &optional_filter_block)` | Checks if the page has a link with the given text or id. |
| `assert_selector(*args, &optional_filter_block)` | Asserts that a given selector is on the page. |
| `assert_table(locator = nil, **options, &optional_filter_block` | Checks if the page has a table with the given id or caption. |
| `assert_text(type, text, **options)` | Asserts that the page has the given text content. |

### Screenshot Helper

The [`ScreenshotHelper`](https://api.rubyonrails.org/classes/ActionDispatch/SystemTesting/TestHelpers/ScreenshotHelper.html) is a helper designed to capture screenshots of your tests. This can be helpful for viewing the browser at the point a test failed, or to view screenshots later for debugging.

Two methods are provided: `take_screenshot` and `take_failed_screenshot`. `take_failed_screenshot` is automatically included in `before_teardown` inside Rails.

The `take_screenshot` helper method can be included anywhere in your tests to take a screenshot of the browser.

### Taking It Further

System testing is similar to [integration testing](integration-tests.md) in that it tests the user's interaction with your controller, model, and view, but system testing tests your application as if a real user were using it. With system tests, you can test anything that a user would do in your application such as commenting, deleting articles, publishing draft articles, etc.
