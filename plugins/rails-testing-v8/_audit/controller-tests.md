# Source traceability — controller-tests.md

Backs [`../skills/rails-testing/references/controller-tests.md`](../skills/rails-testing/references/controller-tests.md). Rails version: **8.1.3**.

| Claim | Source |
| :--- | :--- |
| `bin/rails generate scaffold_controller article`; `bin/rails generate test_unit:scaffold article` | `guides/source/testing.md` §Functional Testing for Controllers |
| Generated `ArticlesControllerTest < ActionDispatch::IntegrationTest` | `guides/source/testing.md` §Functional Testing for Controllers |
| `get`/`post`/`patch`/`put`/`delete`/`head` helpers accept `path, **args` and delegate to `process(method, path, params: nil, headers: nil, env: nil, xhr: false, as: nil)` | `actionpack/lib/action_dispatch/testing/integration.rb` lines 18, 24, 30, 36, 42, 48, 226 |
| `@controller`, `@request`, `@response` populated after a request | `actionpack/lib/action_dispatch/testing/integration.rb` lines 158, 301, 303, 308, 424–426 |
| `@controller.action_name` | `actionpack/lib/abstract_controller/base.rb` line 43 |
| `@request.media_type`, `@response.media_type` | `actionpack/lib/action_dispatch/http/request.rb` line 292; `actionpack/lib/action_dispatch/http/response.rb` line 313 |
| `cookies` helper returns the cookie jar | `actionpack/lib/action_dispatch/testing/integration.rb` line 114 |
| `flash`, `session` accessible as hashes after request | `guides/source/testing.md` §Testing Other Request Objects |
| `ActionController::HttpAuthentication::Basic.encode_credentials(user_name, password)` | `actionpack/lib/action_controller/metal/http_authentication.rb` line 134 |
| `setup do … end` / `teardown do … end` macros | `activesupport/lib/active_support/testing/setup_and_teardown.rb` lines 29, 34 |
| `assert_response`, `assert_redirected_to` (used throughout) — cross-linked | `actionpack/lib/action_dispatch/testing/assertions/response.rb` lines 33, 60; see [writing-tests.md](../skills/rails-testing/references/writing-tests.md) |
| `articles(:one)` fixture accessor — cross-linked | `activerecord/lib/active_record/test_fixtures.rb`; see [test-database.md](../skills/rails-testing/references/test-database.md) |
