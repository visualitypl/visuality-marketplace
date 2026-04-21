# Test Helpers

Covers how to organize reusable test code — custom helper modules mixed into test cases. Mirrors the **Test Helpers** section of the Rails 8.1.3 testing guide (`guides/source/testing.md` §1747–1835).

To avoid code duplication, you can add your own test helpers. Here is an example for signing in:

```ruby
# test/test_helper.rb

module SignInHelper
  def sign_in_as(user)
    post sign_in_url(email: user.email, password: user.password)
  end
end

class ActionDispatch::IntegrationTest
  include SignInHelper
end
```

```ruby
# test/controllers/profile_controller_test.rb
require "test_helper"

class ProfileControllerTest < ActionDispatch::IntegrationTest
  test "should show profile" do
    # helper is now reusable from any controller test case
    sign_in_as users(:david)

    get profile_url
    assert_response :success
  end
end
```

## Using Separate Files

If you find your helpers are cluttering `test_helper.rb`, you can extract them into separate files. A good place to store them is `test/lib` or `test/test_helpers`.

```ruby
# test/test_helpers/multiple_assertions.rb
module MultipleAssertions
  def assert_multiple_of_forty_two(number)
    assert (number % 42 == 0), "expected #{number} to be a multiple of 42"
  end
end
```

These helpers can then be explicitly required and included as needed:

```ruby
# test/models/number_test.rb
require "test_helper"
require "test_helpers/multiple_assertions"

class NumberTest < ActiveSupport::TestCase
  include MultipleAssertions

  test "420 is a multiple of 42" do
    assert_multiple_of_forty_two 420
  end
end
```

They can also continue to be included directly into the relevant parent classes:

```ruby
# test/test_helper.rb
require "test_helpers/sign_in_helper"

class ActionDispatch::IntegrationTest
  include SignInHelper
end
```

## Eagerly Requiring Helpers

You may find it convenient to eagerly require helpers in `test_helper.rb` so your test files have implicit access to them. This can be accomplished using globbing, as follows

```ruby
# test/test_helper.rb
Dir[Rails.root.join("test", "test_helpers", "**", "*.rb")].each { |file| require file }
```

This has the downside of increasing the boot-up time, as opposed to manually requiring only the necessary files in your individual tests.

## Source traceability

| Claim | Source |
| :--- | :--- |
| `SignInHelper` module defined in `test/test_helper.rb` and mixed into `ActionDispatch::IntegrationTest`; `ProfileControllerTest` usage | `guides/source/testing.md` lines 1750–1778 |
| Extract helpers into `test/lib` or `test/test_helpers`; `MultipleAssertions` module example | `guides/source/testing.md` lines 1783–1793 |
| Explicit `require "test_helpers/multiple_assertions"` + `include` in `NumberTest` | `guides/source/testing.md` lines 1796–1808 |
| Alternative: `require` the helper in `test_helper.rb` and `include` into the parent test class | `guides/source/testing.md` lines 1811–1819 |
| Eager `Dir[Rails.root.join("test", "test_helpers", "**", "*.rb")].each { |file| require file }` pattern and its boot-up-time trade-off | `guides/source/testing.md` lines 1822–1834 |

Rails version: **8.1.3**.
