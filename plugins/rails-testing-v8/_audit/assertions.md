# Source traceability — assertions.md

Backs [`../skills/rails-testing/references/assertions.md`](../skills/rails-testing/references/assertions.md). Rails version: **8.1.3**.

| Claim | Source |
| :--- | :--- |
| Minitest assertion signatures (`assert`, `assert_equal`, `assert_raises`, `assert_same`, `assert_empty`, `assert_in_delta`, `assert_in_epsilon`, `assert_throws`, `assert_operator`, `assert_predicate`, etc.) | `guides/source/testing.md` §Minitest Assertions; [minitest API docs](http://docs.seattlerb.org/minitest/Minitest/Assertions.html) |
| `assert_difference`, `assert_no_difference`, `assert_changes`, `assert_no_changes`, `assert_nothing_raised` signatures | `activesupport/lib/active_support/testing/assertions.rb` lines 48, 105, 162, 212, 281 |
| `assert_recognizes`, `assert_generates`, `assert_routing` signatures | `actionpack/lib/action_dispatch/testing/assertions/routing.rb` lines 176, 216, 260 |
| `assert_response`, `assert_redirected_to` signatures | `actionpack/lib/action_dispatch/testing/assertions/response.rb` lines 33, 60 |
| `assert_queries_count`, `assert_no_queries`, `assert_queries_match`, `assert_no_queries_match` signatures | `activerecord/lib/active_record/testing/query_assertions.rb` lines 24, 48, 65, 93 |
| `assert_error_reported`, `assert_no_error_reported` signatures | `activesupport/lib/active_support/testing/error_reporter_assertions.rb` lines 62, 88 |
| Test case superclasses (`ActiveSupport::TestCase`, `ActionMailer::TestCase`, `ActionView::TestCase`, `ActiveJob::TestCase`, `ActionDispatch::Integration::Session`, `ActionDispatch::SystemTestCase`, `Rails::Generators::TestCase`) | `guides/source/testing.md` §Assertions in Test Cases |
