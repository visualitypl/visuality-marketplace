# Testing Mailers

Covers how to test mailer classes in a Rails app. Mirrors the **Testing Mailers** section of the Rails 8.1.3 testing guide (`guides/source/testing.md` §2129–2364).

Your mailer classes - like every other part of your Rails application - should be tested to ensure that they are working as expected.

The goals of testing your mailer classes are to ensure that:

* emails are being processed (created and sent)
* the email content is correct (subject, sender, body, etc)
* the right emails are being sent at the right times

There are two aspects of testing your mailer, the unit tests and the functional tests. In the unit tests, you run the mailer in isolation with tightly controlled inputs and compare the output to a known value (a [fixture](#fixtures)). In the functional tests you don't so much test the details produced by the mailer; instead, you test that the controllers and models are using the mailer in the right way. You test to prove that the right email was sent at the right time.

## Unit Testing

In order to test that your mailer is working as expected, you can use unit tests to compare the actual results of the mailer with pre-written examples of what should be produced.

### Mailer Fixtures

For the purposes of unit testing a mailer, fixtures are used to provide an example of how the output _should_ look. Because these are example emails, and not Active Record data like the other fixtures, they are kept in their own subdirectory apart from the other fixtures. The name of the directory within `test/fixtures` directly corresponds to the name of the mailer. So, for a mailer named `UserMailer`, the fixtures should reside in `test/fixtures/user_mailer` directory.

If you generated your mailer, the generator does not create stub fixtures for the mailers actions. You'll have to create those files yourself as described above.

### The Basic Test Case

Here's a unit test to test a mailer named `UserMailer` whose action `invite` is used to send an invitation to a friend:

```ruby
# test/mailers/user_mailer_test.rb
require "test_helper"

class UserMailerTest < ActionMailer::TestCase
  test "invite" do
    # Create the email and store it for further assertions
    email = UserMailer.create_invite("me@example.com",
                                     "friend@example.com", Time.now)

    # Send the email, then test that it got queued
    assert_emails 1 do
      email.deliver_now
    end

    # Test the body of the sent email contains what we expect it to
    assert_equal ["me@example.com"], email.from
    assert_equal ["friend@example.com"], email.to
    assert_equal "You have been invited by me@example.com", email.subject
    assert_equal read_fixture("invite").join, email.body.to_s
  end
end
```

In the test the email is created and the returned object is stored in the `email` variable. The first assert checks it was sent, then, in the second batch of assertions, the email contents are checked. The helper `read_fixture` is used to read in the content from this file.

> **NOTE:** `email.body.to_s` is present when there's only one (HTML or text) part present. If the mailer provides both, you can test your fixture against specific parts with `email.text_part.body.to_s` or `email.html_part.body.to_s`.

Here's the content of the `invite` fixture:

```
# test/fixtures/user_mailer/invite
Hi friend@example.com,

You have been invited.

Cheers!
```

### Configuring the Delivery Method for Test

The line `ActionMailer::Base.delivery_method = :test` in `config/environments/test.rb` sets the delivery method to test mode so that the email will not actually be delivered (useful to avoid spamming your users while testing). Instead, the email will be appended to an array (`ActionMailer::Base.deliveries`).

> **NOTE:** The `ActionMailer::Base.deliveries` array is only reset automatically in `ActionMailer::TestCase` and `ActionDispatch::IntegrationTest` tests. If you want to have a clean slate outside these test cases, you can reset it manually with: `ActionMailer::Base.deliveries.clear`

### Testing Enqueued Emails

You can use the `assert_enqueued_email_with` assertion to confirm that the email has been enqueued with all of the expected mailer method arguments and/or parameterized mailer parameters. This matches emails enqueued with `deliver_later`.

Basic form — mailer class, action, and positional `args:`:

```ruby
# test/mailers/user_mailer_test.rb
require "test_helper"

class UserMailerTest < ActionMailer::TestCase
  test "invite" do
    email = UserMailer.create_invite("me@example.com", "friend@example.com")

    # args: accepts positional args; pass [{ from:, to: }] to match keyword-style mailer args
    assert_enqueued_email_with UserMailer, :create_invite, args: ["me@example.com", "friend@example.com"] do
      email.deliver_later
    end
  end
end
```

Parameterized mailer — mailer parameters go in `params:`, mailer-method arguments in `args:`:

```ruby
# test/mailers/user_mailer_test.rb
require "test_helper"

class UserMailerTest < ActionMailer::TestCase
  test "invite" do
    email = UserMailer.with(all: "good").create_invite("me@example.com", "friend@example.com")

    assert_enqueued_email_with UserMailer, :create_invite,
      params: { all: "good" }, args: ["me@example.com", "friend@example.com"] do
      email.deliver_later
    end
  end
end
```

> **NOTE:** Alternatively, pass a parameterized mailer directly as the first argument: `assert_enqueued_email_with UserMailer.with(to: "friend@example.com"), :create_invite do … end`. Same effect, terser when you only need the `params:` half.

## Functional and System Testing

Unit testing allows us to test the attributes of the email while functional and system testing allows us to test whether user interactions appropriately trigger the email to be delivered. For example, you can check that the invite friend operation is sending an email appropriately:

```ruby
# Integration Test
# test/controllers/users_controller_test.rb
require "test_helper"

class UsersControllerTest < ActionDispatch::IntegrationTest
  test "invite friend" do
    # Asserts the difference in the ActionMailer::Base.deliveries
    assert_emails 1 do
      post invite_friend_url, params: { email: "friend@example.com" }
    end
  end
end
```

```ruby
# System Test
# test/system/users_test.rb
require "test_helper"

class UsersTest < ActionDispatch::SystemTestCase
  driven_by :selenium, using: :headless_chrome

  test "inviting a friend" do
    visit invite_users_url
    fill_in "Email", with: "friend@example.com"
    assert_emails 1 do
      click_on "Invite"
    end
  end
end
```

> **NOTE:** The `assert_emails` method is not tied to a particular deliver method and will work with emails delivered with either the `deliver_now` or `deliver_later` method. If we explicitly want to assert that the email has been enqueued we can use the `assert_enqueued_email_with` ([examples above](#testing-enqueued-emails)) or `assert_enqueued_emails` methods. More information can be found in the [documentation](https://api.rubyonrails.org/classes/ActionMailer/TestHelper.html).

See also: [`assertions.md`](assertions.md) for Rails-specific assertions; [`test-database.md`](test-database.md) for general fixture conventions.
