# Source traceability — integration-tests.md

Backs [`../skills/rails-testing/references/integration-tests.md`](../skills/rails-testing/references/integration-tests.md). Rails version: **8.1.3**.

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
| `get`/`post` request helpers — cross-linked | `actionpack/lib/action_dispatch/testing/integration.rb`; see [controller-tests.md](../skills/rails-testing/references/controller-tests.md) |
| `assert_response`, `assert_redirected_to` — cross-linked | `actionpack/lib/action_dispatch/testing/assertions/response.rb`; see [assertions.md](../skills/rails-testing/references/assertions.md) |
