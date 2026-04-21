# Changelog

All notable changes to plugins in this marketplace.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/). Release tags follow the pattern `v{plugin-version}-{lib}-{lib-version}` (e.g. `v1.1.0-rails-8.1.3`) so a marketplace tag identifies both the plugin release and the target-library version it projects.

Entries are grouped by plugin; within a plugin, newest first.

## rails-testing-v8

Projects the [Rails 8.1.3 testing guide](https://guides.rubyonrails.org/testing.html) with every claim verified against `rails/rails@v8.1.3` source.

### [1.1.0] — 2026-04-21

Reference-tree restructuring for token efficiency. No change in target-library coverage.

#### Changed
- Split `references/writing-tests.md` (457 lines) into three focused files: `writing-tests.md` (first-test tutorial + TDD walkthrough only), `assertions.md` (Minitest non-obvious entries + Rails-specific assertions + test-case superclasses), and `test-runner.md` (`bin/rails test` invocations and flags). Agents now load only the topic they need.
- Trimmed the Minitest assertions table to non-obvious entries (`assert_same`, `assert_empty`, `assert_in_delta`, `assert_in_epsilon`, `assert_throws`, `assert_operator`, `assert_predicate`, and their `_not_` counterparts); the well-known assertions (`assert`, `assert_equal`, `assert_nil`, `assert_raises`, `assert_includes`, etc.) link out to the minitest API docs instead of duplicating them.
- Collapsed four near-duplicate `assert_enqueued_email_with` examples in `mailer-tests.md` to two: the positional-args form (with an inline comment covering the keyword-arg variant) and the `params:` + `args:` parameterized form (with a trailing NOTE covering the `UserMailer.with(to: …)` alternative).
- Trimmed duplicate `bin/rails test` session-output blocks in `writing-tests.md`, `test-runner.md`, and `controller-tests.md` — one fail-run + one pass-run per teaching moment; subsequent confirmations use prose.

#### Removed
- `## Source traceability` sections from every `references/*.md` file. Re-audits regenerate citations against the pinned `rails/rails@v8.1.3` source rather than relying on stored tables; prior tables remain available in git history if needed as a starting point.
- `Rails version: **8.1.3**.` footer from every `references/*.md` file (version remains in `SKILL.md` frontmatter as `rails_version: "8.1.3"`).

### [1.0.0] — initial release

Initial release. Projects the official Rails 8.1.3 testing guide as an auto-invoked Claude Code skill, with every claim verified against `rails/rails@v8.1.3` source.

#### Added
- `SKILL.md` with TRIGGER/SKIP auto-invocation rules and a scenario-to-reference routing table.
- 16 on-demand references covering: writing tests, test database + fixtures, model tests, controller tests, integration tests, system tests (Capybara), test helpers, route tests, view tests, mailer tests, job tests, Action Cable tests, CI, parallel testing, eager-loading tests, and error-rescue + time-dependent testing.
- Per-reference source-traceability tables citing Rails source at the `v8.1.3` tag (extracted in 1.1.0; available in git history).
