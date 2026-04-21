# Source traceability — test-helpers.md

Backs [`../skills/rails-testing/references/test-helpers.md`](../skills/rails-testing/references/test-helpers.md). Rails version: **8.1.3**.

| Claim | Source |
| :--- | :--- |
| `SignInHelper` module defined in `test/test_helper.rb` and mixed into `ActionDispatch::IntegrationTest`; `ProfileControllerTest` usage | `guides/source/testing.md` lines 1750–1778 |
| Extract helpers into `test/lib` or `test/test_helpers`; `MultipleAssertions` module example | `guides/source/testing.md` lines 1783–1793 |
| Explicit `require "test_helpers/multiple_assertions"` + `include` in `NumberTest` | `guides/source/testing.md` lines 1796–1808 |
| Alternative: `require` the helper in `test_helper.rb` and `include` into the parent test class | `guides/source/testing.md` lines 1811–1819 |
| Eager `Dir[Rails.root.join("test", "test_helpers", "**", "*.rb")].each { |file| require file }` pattern and its boot-up-time trade-off | `guides/source/testing.md` lines 1822–1834 |
