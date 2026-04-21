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
