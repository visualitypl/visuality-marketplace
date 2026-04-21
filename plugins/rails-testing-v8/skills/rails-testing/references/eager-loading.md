# Testing Eager Loading

Covers how to verify that every file in your project loads cleanly, so you catch load-time errors before production. Mirrors the **Testing Eager Loading** section of the Rails 8.1.3 testing guide (`guides/source/testing.md` §2759–2796).

Normally, applications do not eager load in the `development` or `test` environments to speed things up. But they do in the `production` environment.

If some file in the project cannot be loaded for whatever reason, it is important to detect it before deploying to production.

## Continuous Integration

If your project has CI in place, eager loading in CI is an easy way to ensure the application eager loads.

CIs typically set an environment variable to indicate the test suite is running there. For example, it could be `CI`:

```ruby
# config/environments/test.rb
config.eager_load = ENV["CI"].present?
```

Starting with Rails 7, newly generated applications are configured that way by default.

If your project does not have continuous integration, you can still eager load in the test suite by calling `Rails.application.eager_load!`:

```ruby
# test/zeitwerk_compliance_test.rb
require "test_helper"

class ZeitwerkComplianceTest < ActiveSupport::TestCase
  test "eager loads all files without errors" do
    assert_nothing_raised { Rails.application.eager_load! }
  end
end
```

## Source traceability

| Claim | Source |
| :--- | :--- |
| Dev/test do not eager load; production does; eager-loading in CI catches errors pre-deploy | `guides/source/testing.md` lines 2762–2766 |
| `config.eager_load = ENV["CI"].present?` in `config/environments/test.rb`; default since Rails 7 | `guides/source/testing.md` lines 2770–2782 |
| `Rails.application.eager_load!` in a compliance test for projects without CI | `guides/source/testing.md` lines 2784–2796 |

See also: [`ci.md`](ci.md) for the single-command CI test invocation.

Rails version: **8.1.3**.
