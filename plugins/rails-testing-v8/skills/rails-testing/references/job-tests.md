# Testing Jobs

Covers how to test Active Job classes in a Rails app. Mirrors the **Testing Jobs** section of the Rails 8.1.3 testing guide (`guides/source/testing.md` §2365–2461).

Jobs can be tested in isolation (focusing on the job's behavior) and in context (focusing on the calling code's behavior).

## Testing Jobs in Isolation

When you generate a job, an associated test file will also be generated in the `test/jobs` directory.

Here is a test you could write for a billing job:

```ruby
# test/jobs/billing_job_test.rb
require "test_helper"

class BillingJobTest < ActiveJob::TestCase
  test "account is charged" do
    perform_enqueued_jobs do
      BillingJob.perform_later(account, product)
    end
    assert account.reload.charged_for?(product)
  end
end
```

The default queue adapter for tests will not perform jobs until [`perform_enqueued_jobs`](https://api.rubyonrails.org/classes/ActiveJob/TestHelper.html#method-i-perform_enqueued_jobs) is called. Additionally, it will clear all jobs before each test is run so that tests do not interfere with each other.

The test uses `perform_enqueued_jobs` and [`perform_later`](https://api.rubyonrails.org/classes/ActiveJob/Enqueuing/ClassMethods.html#method-i-perform_later) instead of [`perform_now`](https://api.rubyonrails.org/classes/ActiveJob/Execution/ClassMethods.html#method-i-perform_now) so that if retries are configured, retry failures are caught by the test instead of being re-enqueued and ignored.

## Testing Jobs in Context

It's good practice to test that jobs are correctly enqueued, for example, by a controller action. The [`ActiveJob::TestHelper`](https://api.rubyonrails.org/classes/ActiveJob/TestHelper.html) module provides several methods that can help with this, such as [`assert_enqueued_with`](https://api.rubyonrails.org/classes/ActiveJob/TestHelper.html#method-i-assert_enqueued_with).

Here is an example that tests an account model method:

```ruby
# test/models/account_test.rb
require "test_helper"

class AccountTest < ActiveSupport::TestCase
  include ActiveJob::TestHelper

  test "#charge_for enqueues billing job" do
    assert_enqueued_with(job: BillingJob) do
      account.charge_for(product)
    end

    assert_not account.reload.charged_for?(product)

    perform_enqueued_jobs

    assert account.reload.charged_for?(product)
  end
end
```

## Testing that Exceptions are Raised

Testing that your job raises an exception in certain cases can be tricky, especially when you have retries configured. The `perform_enqueued_jobs` helper fails any test where a job raises an exception, so to have the test succeed when the exception is raised you have to call the job's `perform` method directly.

```ruby
# test/jobs/billing_job_test.rb
require "test_helper"

class BillingJobTest < ActiveJob::TestCase
  test "does not charge accounts with insufficient funds" do
    assert_raises(InsufficientFundsError) do
      BillingJob.new(empty_account, product).perform
    end
    assert_not account.reload.charged_for?(product)
  end
end
```

This method is not recommended in general, as it circumvents some parts of the framework, such as argument serialization.

See also: [`writing-tests.md`](writing-tests.md) for Rails-specific assertions; [`mailer-tests.md`](mailer-tests.md) for `assert_enqueued_email_with` (mailer-specific wrapper).
