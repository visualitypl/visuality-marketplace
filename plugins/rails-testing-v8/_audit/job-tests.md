# Source traceability — job-tests.md

Backs [`../skills/rails-testing/references/job-tests.md`](../skills/rails-testing/references/job-tests.md). Rails version: **8.1.3**.

| Claim | Source |
| :--- | :--- |
| Intro: jobs tested in isolation vs. in context | `guides/source/testing.md` lines 2365–2370 |
| Generated test file lives in `test/jobs`; `BillingJobTest < ActiveJob::TestCase`; `perform_enqueued_jobs do BillingJob.perform_later(...)` pattern | `guides/source/testing.md` lines 2371–2389; `activejob/lib/active_job/test_case.rb` lines 5–10 (`ActiveJob::TestCase < ActiveSupport::TestCase`, includes `ActiveJob::TestHelper`) |
| Default queue adapter defers execution until `perform_enqueued_jobs`; clears all jobs before each test | `guides/source/testing.md` lines 2391–2393 |
| Rationale for `perform_enqueued_jobs` + `perform_later` over `perform_now` (retries are surfaced) | `guides/source/testing.md` lines 2395–2397 |
| `perform_enqueued_jobs(only:, except:, queue:, at:, &block)` signature | `activejob/lib/active_job/test_helper.rb` line 620 |
| In-context intro; `ActiveJob::TestHelper` module; `assert_enqueued_with` | `guides/source/testing.md` lines 2406–2410 |
| `AccountTest < ActiveSupport::TestCase` with `include ActiveJob::TestHelper`, `assert_enqueued_with(job: BillingJob) do … end`, trailing `perform_enqueued_jobs` | `guides/source/testing.md` lines 2412–2432 |
| `assert_enqueued_with(job:, args:, at:, queue:, priority:, &block)` signature | `activejob/lib/active_job/test_helper.rb` line 406 |
| Exceptions intro: `perform_enqueued_jobs` fails on raised exceptions — call `perform` directly | `guides/source/testing.md` lines 2439–2444 |
| `assert_raises(InsufficientFundsError) do BillingJob.new(...).perform` example | `guides/source/testing.md` lines 2446–2457 |
| Caveat: direct `perform` circumvents argument serialization etc. | `guides/source/testing.md` lines 2459–2460 |
