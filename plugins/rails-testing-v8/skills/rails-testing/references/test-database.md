# The Test Database

Covers how Rails prepares, populates, and isolates test data. Mirrors the **The Test Database** section of the Rails 8.1.3 testing guide (`guides/source/testing.md` §591–843). Three concerns:

1. **Schema** — getting the test database migrated.
2. **Fixtures** — declarative YAML test data loaded into the test database.
3. **Transactions** — automatic per-test rollback that keeps tests isolated.

Every Rails app has three environments — development, test, production — each with its own database configured in `config/database.yml`. A dedicated test database lets your tests set up data in isolation.

## Maintaining the test database schema

Before running tests, the test database must have the current schema. The test helper checks for pending migrations and attempts to load `db/schema.rb` or `db/structure.sql` into the test DB. If migrations are still pending, it raises an error.

| Situation | Command |
| :--- | :--- |
| Schema is out of date (normal case) | `bin/rails db:migrate RAILS_ENV=test` |
| Existing migrations were modified — DB needs rebuild | `bin/rails test:db` |

## Fixtures

_Fixtures_ are a consistent set of test data, written in YAML, loaded into the test database before your tests run. One file per model, stored in `test/fixtures/`.

> **When to use fixtures:** the guide's own note — "Fixtures are not designed to create every object that your tests need, and are best managed when only used for default data that can be applied to the common case."

The authoritative API reference is the [`ActiveRecord::FixtureSet`](https://api.rubyonrails.org/classes/ActiveRecord/FixtureSet.html) documentation.

### YAML syntax

A fixture file is a flat map from fixture name → attribute hash:

```yaml
# test/fixtures/users.yml
# lo & behold! I am a YAML comment!
david:
  name: David Heinemeier Hansson
  birthday: 1979-10-15
  profession: Systems development

steve:
  name: Steve Ross Kellock
  birthday: 1974-09-27
  profession: guy with keyboard
```

- Each fixture has a **name** (`david`, `steve`) followed by an indented hash of `key: value` pairs.
- Records are typically separated by a blank line.
- Comments start with `#` in the first column.

### Associations — reference by fixture name

For `belongs_to` / `has_many` associations, refer to the target fixture **by its name** instead of setting a foreign-key id. Rails auto-assigns primary keys consistently across runs, so named references work:

```yaml
# test/fixtures/categories.yml
web_frameworks:
  name: Web Frameworks
```

```yaml
# test/fixtures/articles.yml
first:
  title: Welcome to Rails!
  category: web_frameworks      # → references categories(:web_frameworks)
```

For **polymorphic** associations (e.g. Action Text `rich_texts`), the reference takes the form `fixture_name (ClassName)`:

```yaml
# test/fixtures/action_text/rich_texts.yml
first_content:
  record: first (Article)       # polymorphic: fixture `first` from Article
  name: content
  body: <div>Hello, from <strong>a fixture</strong></div>
```

### File attachment fixtures (Active Storage)

Active Storage `Blob` and `Attachment` records are regular Active Record rows, so they can be populated via fixtures. Use the `ActiveStorage::FixtureSet.blob` helper in ERB.

Given a model with an attachment:

```ruby
class Article < ApplicationRecord
  has_one_attached :thumbnail
end
```

and an image placed at `test/fixtures/files/first.png`, wire it up with two fixture files:

```yaml
# test/fixtures/active_storage/blobs.yml
first_thumbnail_blob: <%= ActiveStorage::FixtureSet.blob filename: "first.png" %>
```

```yaml
# test/fixtures/active_storage/attachments.yml
first_thumbnail_attachment:
  name: thumbnail               # matches has_one_attached :thumbnail
  record: first (Article)       # owning record (polymorphic form)
  blob: first_thumbnail_blob    # references the blob fixture above
```

Signature (verified against `activestorage/lib/active_storage/fixture_set.rb`):

```ruby
ActiveStorage::FixtureSet.blob(filename:, **attributes)
# returns a YAML string to be interpolated into the .yml fixture
```

Passing extra attributes is supported, e.g. `ActiveStorage::FixtureSet.blob(filename: "x.png", content_type: "image/png")`.

### Embedding Ruby (ERB) in fixtures

Fixture YAML is pre-processed with ERB before YAML parsing, so you can generate data programmatically:

```erb
<% 1000.times do |n| %>
  user_<%= n %>:
    username: <%= "user#{n}" %>
    email: <%= "user#{n}@example.com" %>
<% end %>
```

### When and how fixtures load

Rails loads all fixtures from `test/fixtures/` automatically. Loading happens in three steps:

1. **Remove** any existing rows from the table that the fixture file targets.
2. **Insert** the fixture rows.
3. **Dump** the fixture data into a method on the test case (see next section).

> **Permission note:** to remove existing rows cleanly, Rails tries to disable referential-integrity triggers (foreign keys, check constraints). In PostgreSQL only superusers can disable all triggers — grant your test DB user the required permissions, or you'll hit errors. See [Postgres `ALTER TABLE` docs](https://www.postgresql.org/docs/current/sql-altertable.html).

### Accessing fixtures — they are Active Record objects

Each fixture set gives your tests a method named after the fixture file. Calling `<fixture_set>(:name)` returns the **Active Record instance**:

```ruby
# this will return the User object for the fixture named david
users(:david)
# => #<User id: 1, name: "David Heinemeier Hansson", ...>

# attribute access — normal AR
users(:david).id
users(:david).name

# full AR methods available
david = users(:david)
david.call(david.partner)
```

Pass multiple names to get an array:

```ruby
users(:david, :steve)
# => [#<User ...>, #<User ...>]
```

The accessor is scoped to the test case — it's generated by the fixtures machinery, not a global method.

## Transactional tests

By default, Rails wraps each test in a database transaction that is rolled back at the end. This keeps tests independent: no test can see another test's writes.

```ruby
class MyTest < ActiveSupport::TestCase
  test "newly created users are active by default" do
    # The implicit transaction around the test means the user created here
    # won't be visible to other tests — it's rolled back.
    assert User.create.active?
  end
end
```

Application-level calls to `ActiveRecord::Base.current_transaction` still report as expected — the implicit test-wrapper transaction does not interfere with application semantics:

```ruby
class MyTest < ActiveSupport::TestCase
  test "current_transaction reads application-level state" do
    assert User.current_transaction.blank?
  end
end
```

See the [API docs for `current_transaction`](https://api.rubyonrails.org/classes/ActiveRecord/Transactions/ClassMethods.html#method-i-current_transaction).

With [multiple writing databases](https://guides.rubyonrails.org/active_record_multiple_databases.html), tests are wrapped in one transaction per writing database — all are rolled back.

### Opting out

To disable the implicit transaction wrap for a whole test case, set the class attribute to `false`:

```ruby
class MyTest < ActiveSupport::TestCase
  # No implicit database transaction wraps tests in this class.
  self.use_transactional_tests = false
end
```
