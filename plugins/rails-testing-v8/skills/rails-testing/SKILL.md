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

Placeholder — references added in subsequent phases.

## Version

This skill documents the Rails testing guide at **Rails 8.1.3**
(https://guides.rubyonrails.org/testing.html).
