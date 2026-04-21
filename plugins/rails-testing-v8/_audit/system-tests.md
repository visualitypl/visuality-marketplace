# Source traceability — system-tests.md

Backs [`../skills/rails-testing/references/system-tests.md`](../skills/rails-testing/references/system-tests.md). Rails version: **8.1.3**.

| Claim | Source |
| :--- | :--- |
| Intro: system testing runs in a real or headless browser; Capybara under the hood | `guides/source/testing.md` lines 1420–1424 |
| "When to Use System Tests" trade-offs and guidance | `guides/source/testing.md` lines 1428–1447 |
| Scaffold opt-in via `--system-tests=true`; independent `bin/rails generate system_test <name>` | `guides/source/testing.md` lines 1451–1465; `railties/lib/rails/generators/test_unit/system/system_generator.rb` lines 10–16 |
| `system_test` generator creates `test/application_system_test_case.rb` and `test/system/<name>_test.rb` | `railties/lib/rails/generators/test_unit/system/system_generator.rb` lines 11–15; templates `application_system_test_case.rb.tt`, `system_test.rb.tt` |
| Generated skeleton inherits from `ApplicationSystemTestCase`, which inherits from `ActionDispatch::SystemTestCase` | `railties/lib/rails/generators/test_unit/system/templates/system_test.rb.tt` line 3; `railties/lib/rails/generators/test_unit/system/templates/application_system_test_case.rb.tt` line 3 |
| Default driver is Selenium + Chrome + `[1400, 1400]` screen size | `guides/source/testing.md` lines 1491–1493; `actionpack/lib/action_dispatch/system_test_case.rb` line 158 (`driven_by(driver, using: :chrome, screen_size: [1400, 1400], options: {}, &capabilities)`) |
| `driven_by` required/optional arguments (`:using`, `:screen_size`, `:options`) | `guides/source/testing.md` lines 1519–1523; `actionpack/lib/action_dispatch/system_test_case.rb` line 158 |
| Cuprite / Firefox / `:headless_chrome` / `:headless_firefox` configuration examples | `guides/source/testing.md` lines 1510–1543 |
| Remote browser via `options: { browser: :remote, url: url }` and `SELENIUM_REMOTE_URL` | `guides/source/testing.md` lines 1545–1567 |
| Remote server via `Capybara.server_host` / `Capybara.app_host` in `setup` | `guides/source/testing.md` lines 1569–1586 |
| `ArticlesTest` example, `visit`/`assert_selector`/`click_on`/`fill_in`/`assert_text` usage | `guides/source/testing.md` lines 1620–1674 (Capybara DSL is provided via `include Capybara::DSL` — `actionpack/lib/action_dispatch/system_test_case.rb` line 115) |
| `bin/rails test:system` runs system tests; `bin/rails test` does not; `bin/rails test:all` runs all | `guides/source/testing.md` lines 1635–1641 |
| `MobileSystemTestCase` with `screen_size: [375, 667]` and `PostsTest` inheriting from it | `guides/source/testing.md` lines 1684–1704 |
| Capybara assertions table (`assert_button`, `assert_current_path`, `assert_field`, `assert_link`, `assert_selector`, `assert_table`, `assert_text`) | `guides/source/testing.md` lines 1713–1721 (Capybara gem — link above) |
| `ScreenshotHelper#take_screenshot` and `take_failed_screenshot`; failed screenshot runs in `before_teardown` | `guides/source/testing.md` lines 1726–1737; `actionpack/lib/action_dispatch/system_testing/test_helpers/screenshot_helper.rb` lines 41, 53 |
| "Taking It Further" — system testing vs integration testing framing | `guides/source/testing.md` lines 1741–1745 |
