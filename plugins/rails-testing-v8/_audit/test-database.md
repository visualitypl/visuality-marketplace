# Source traceability — test-database.md

Backs [`../skills/rails-testing/references/test-database.md`](../skills/rails-testing/references/test-database.md). Rails version: **8.1.3**.

| Claim | Source file |
| :--- | :--- |
| Schema load + `bin/rails test:db` | `guides/source/testing.md` §Maintaining the Test Database Schema |
| Fixture YAML / associations / polymorphic `record:` | `guides/source/testing.md` §Fixtures |
| `ActiveStorage::FixtureSet.blob(filename:, **attributes)` | `activestorage/lib/active_storage/fixture_set.rb` |
| `fixtures(:name)` accessor returns AR instance | `activerecord/lib/active_record/test_fixtures.rb` |
| `use_transactional_tests` class attribute, default `true` | `activerecord/lib/active_record/test_fixtures.rb` |
| `ActiveRecord::Base.current_transaction` unaffected by test wrapper | `guides/source/testing.md` §Transactions |
