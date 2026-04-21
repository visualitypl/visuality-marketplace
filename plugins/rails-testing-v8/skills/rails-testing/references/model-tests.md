# Testing Models

Covers model tests: where they live, the generator that creates them, and the superclass they inherit from. Mirrors the **Testing Models** section of the Rails 8.1.3 testing guide (`guides/source/testing.md` §845–876).

Model tests exercise the models of your application and their associated logic. You test that logic using the assertions and fixtures explored elsewhere in this skill:

- Assertions — both the Minitest set and the Rails-specific extensions: [`writing-tests.md`](writing-tests.md).
- Fixtures — YAML test data, associations, ERB, and the `<fixture_set>(:name)` accessor (e.g. `users(:david)`) that returns an Active Record instance: [`test-database.md`](test-database.md).

## Location

Model tests are stored under `test/models`.

## Generator

Rails provides a generator to create a model test skeleton:

```bash
$ bin/rails generate test_unit:model article
create  test/models/article_test.rb
```

The generated file:

```ruby
# test/models/article_test.rb
require "test_helper"

class ArticleTest < ActiveSupport::TestCase
  # test "the truth" do
  #   assert true
  # end
end
```

## Superclass

Model tests don't have their own superclass like `ActionMailer::TestCase`. They inherit directly from [`ActiveSupport::TestCase`](https://api.rubyonrails.org/classes/ActiveSupport/TestCase.html).

## Source traceability

| Claim | Source |
| :--- | :--- |
| Model tests live under `test/models` | `guides/source/testing.md` §Testing Models |
| `bin/rails generate test_unit:model <name>` creates `test/models/<name>_test.rb` | `railties/lib/rails/generators/test_unit/model/model_generator.rb` line 16 |
| Generated `ArticleTest` stub contents | `railties/lib/rails/generators/test_unit/model/templates/unit_test.rb.tt` |
| Model tests inherit from `ActiveSupport::TestCase` (no dedicated `Model::TestCase`) | `activesupport/lib/active_support/test_case.rb` line 23; `guides/source/testing.md` §Testing Models |

Rails version: **8.1.3**.
