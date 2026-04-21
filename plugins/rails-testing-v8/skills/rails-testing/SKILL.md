---
name: rails-testing
description: >-
  Rails 8 testing with Minitest and fixtures, projecting
  https://guides.rubyonrails.org/testing.html (Rails 8.1.3). Covers the test
  database, fixtures (including ERB-embedded fixtures and file attachment
  fixtures), model tests, functional controller tests, integration tests,
  system tests (Capybara), test helpers, route tests, view tests, mailer
  tests, job tests, Action Cable tests, parallel testing, CI, and
  eager-loading tests.
  TRIGGER when ANY of: Gemfile has `gem 'rails'` AND file path matches
  `test/**/*_test.rb`; file path matches `test/fixtures/**/*.yml` or
  `test/fixtures/files/**`; file contains `ActiveSupport::TestCase`,
  `ActionDispatch::IntegrationTest`, `ActionDispatch::SystemTestCase`,
  `ActionController::TestCase`, `ActionMailer::TestCase`,
  `ActiveJob::TestCase`, `ActionCable::Channel::TestCase`,
  `ActionCable::Connection::TestCase`, or `ApplicationSystemTestCase`; user
  runs `bin/rails test`, `bin/rails test:system`, or
  `bin/rails db:test:prepare`; user mentions `assert_difference`,
  `assert_changes`, `assert_enqueued_with`, `assert_broadcast_on`,
  `perform_enqueued_jobs`, `fixture_file_upload`, `parallelize(workers:`, or
  `driven_by`; user asks how to write tests in a Rails app with Minitest
  or fixtures.
  SKIP when: non-Rails Ruby code; Rails file edits outside `test/` that
  don't reference a Rails test class.
rails_version: "8.1.3"
---

# Rails Testing

Rails 8 testing with Minitest and fixtures. Projects the official [Rails 8.1.3 testing guide](https://guides.rubyonrails.org/testing.html) into on-demand references. Every claim is verified against `rails/rails@v8.1.3` source.

## When to consult which reference

Claude Code loads this SKILL.md into context and pulls individual references only when a topic is relevant. Open the matching file before writing test code — do not rely on prior knowledge of the API.

| Scenario | File |
| :--- | :--- |
| Test layout, writing your first test, Minitest + Rails-specific assertions, the `bin/rails test` runner | [references/writing-tests.md](references/writing-tests.md) |
| Test database setup, fixtures (YAML, associations, Active Storage, ERB), accessing fixtures as AR objects, transactional tests | [references/test-database.md](references/test-database.md) |
| Model tests: location, generator, superclass | [references/model-tests.md](references/model-tests.md) |
| Controller functional tests: HTTP helpers (`get`/`post`/...), XHR, request/response objects, headers, flash, show/update/delete patterns | [references/controller-tests.md](references/controller-tests.md) |
| Integration tests: `ActionDispatch::IntegrationTest`, generator, multi-request workflows, `follow_redirect!`, `host!`, `https!`, `open_session` | [references/integration-tests.md](references/integration-tests.md) |
| System tests (Capybara): when to use, generator, `driven_by`, screen sizes, Capybara assertions, screenshot helper | [references/system-tests.md](references/system-tests.md) |
| Test helpers: extracting shared helpers into modules, eagerly requiring helpers | [references/test-helpers.md](references/test-helpers.md) |
| Route tests: where they live, pointer to routing assertions | [references/route-tests.md](references/route-tests.md) |
| View tests: `assert_dom`, `document_root_element`, `rendered` parsers, partial tests, helper tests | [references/view-tests.md](references/view-tests.md) |
| Mailer tests: `ActionMailer::TestCase`, fixtures in `test/fixtures/<mailer>/`, `assert_emails`, `assert_enqueued_email_with`, integration + system patterns | [references/mailer-tests.md](references/mailer-tests.md) |
| Job tests: `ActiveJob::TestCase`, testing jobs in isolation vs in context, `assert_enqueued_with`, `perform_enqueued_jobs`, testing raised exceptions | [references/job-tests.md](references/job-tests.md) |

Other topics (Action Cable tests, parallel testing, CI, eager-loading) will land in later reference files as this skill is built out.

## Core principles

1. **Fixtures are for default data, not per-test setup.** The guide is explicit: "Fixtures are not designed to create every object that your tests need, and are best managed when only used for default data that can be applied to the common case."
2. **Tests are transactional by default.** Each test runs inside a transaction rolled back at the end — so writes from one test never leak to another. Disable with `self.use_transactional_tests = false` only when the behavior under test needs real commits.
3. **Fixture names are primary-key-stable.** Rails derives deterministic primary keys from fixture names (via `ActiveRecord::FixtureSet.identify`), so `users(:david).id` is the same every run. This is why associations can reference fixtures by name.

## Version

This skill documents the Rails testing guide at **Rails 8.1.3** (https://guides.rubyonrails.org/testing.html). The rails source is pinned to the `v8.1.3` tag — see each reference file's "Source traceability" table for citations.
