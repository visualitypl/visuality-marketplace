# Source traceability — model-tests.md

Backs [`../skills/rails-testing/references/model-tests.md`](../skills/rails-testing/references/model-tests.md). Rails version: **8.1.3**.

| Claim | Source |
| :--- | :--- |
| Model tests live under `test/models` | `guides/source/testing.md` §Testing Models |
| `bin/rails generate test_unit:model <name>` creates `test/models/<name>_test.rb` | `railties/lib/rails/generators/test_unit/model/model_generator.rb` line 16 |
| Generated `ArticleTest` stub contents | `railties/lib/rails/generators/test_unit/model/templates/unit_test.rb.tt` |
| Model tests inherit from `ActiveSupport::TestCase` (no dedicated `Model::TestCase`) | `activesupport/lib/active_support/test_case.rb` line 23; `guides/source/testing.md` §Testing Models |
