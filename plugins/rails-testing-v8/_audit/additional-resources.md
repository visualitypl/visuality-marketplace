# Source traceability — additional-resources.md

Backs [`../skills/rails-testing/references/additional-resources.md`](../skills/rails-testing/references/additional-resources.md). Rails version: **8.1.3**.

| Claim | Source |
| :--- | :--- |
| Rails rescues errors in system/integration/functional tests and renders HTML error pages; controlled by `config.action_dispatch.show_exceptions` | `guides/source/testing.md` lines 2802–2806 |
| `travel_to(date_or_time, &block)` stubs `Date.current` inside the block | `guides/source/testing.md` lines 2813–2828; `activesupport/lib/active_support/testing/time_helpers.rb` line 133 (`def travel_to(date_or_time, with_usec: false)`) |
| Full catalog of time helpers in `ActiveSupport::Testing::TimeHelpers` API docs | `guides/source/testing.md` lines 2830–2836 |
