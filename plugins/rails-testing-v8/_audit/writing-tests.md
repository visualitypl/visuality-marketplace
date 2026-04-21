# Source traceability — writing-tests.md

Backs [`../skills/rails-testing/references/writing-tests.md`](../skills/rails-testing/references/writing-tests.md). Rails version: **8.1.3**.

| Claim | Source |
| :--- | :--- |
| `test` directory layout (`controllers/`, `fixtures/`, `helpers/`, `integration/`, `mailers/`, `models/`, `test_helper.rb`) | `guides/source/testing.md` §Test Setup / Test Directories |
| System tests inherit from Capybara; `system/` + `application_system_test_case.rb` generated on first system test | `guides/source/testing.md` §Test Directories |
| Three environments (development/test/production); `RAILS_ENV=test` set automatically | `guides/source/testing.md` §The Test Environment |
| Scaffolded `ArticleTest` stub; `test` macro equivalent to `def test_*` | `guides/source/testing.md` §Writing Your First Test |
| `config.active_support.test_order` configures test order | `guides/source/testing.md` §Reporting Errors |
| `-b` / `--backtrace` shows full backtrace | `guides/source/testing.md` §Reporting Errors |
| Minitest assertion signatures (`assert`, `assert_equal`, `assert_raises`, etc.) | `guides/source/testing.md` §Minitest Assertions |
| `assert_difference`, `assert_no_difference`, `assert_changes`, `assert_no_changes`, `assert_nothing_raised` signatures | `activesupport/lib/active_support/testing/assertions.rb` lines 48, 105, 162, 212, 281 |
| `assert_recognizes`, `assert_generates`, `assert_routing` signatures | `actionpack/lib/action_dispatch/testing/assertions/routing.rb` lines 176, 216, 260 |
| `assert_response`, `assert_redirected_to` signatures | `actionpack/lib/action_dispatch/testing/assertions/response.rb` lines 33, 60 |
| `assert_queries_count`, `assert_no_queries`, `assert_queries_match`, `assert_no_queries_match` signatures | `activerecord/lib/active_record/testing/query_assertions.rb` lines 24, 48, 65, 93 |
| `assert_error_reported`, `assert_no_error_reported` signatures | `activesupport/lib/active_support/testing/error_reporter_assertions.rb` lines 62, 88 |
| Test case superclasses (`ActiveSupport::TestCase`, `ActionMailer::TestCase`, etc.) | `guides/source/testing.md` §Assertions in Test Cases |
| `bin/rails test` runner flags (`-n`, line numbers, line ranges, directories, `-h`, `-b`, `-f`, `--profile`, etc.) | `guides/source/testing.md` §The Rails Test Runner |
