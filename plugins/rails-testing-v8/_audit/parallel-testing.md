# Source traceability — parallel-testing.md

Backs [`../skills/rails-testing/references/parallel-testing.md`](../skills/rails-testing/references/parallel-testing.md). Rails version: **8.1.3**.

| Claim | Source |
| :--- | :--- |
| Intro: parallel testing reduces suite time; forking is default, threading is supported | `guides/source/testing.md` lines 2629–2634 |
| Default parallelization forks processes via DRb; default worker count is the machine core count, overridable via `parallelize` | `guides/source/testing.md` lines 2636–2641 |
| `class ActiveSupport::TestCase; parallelize(workers: 2); end` in `test_helper.rb` | `guides/source/testing.md` lines 2643–2649 |
| `parallelize(workers: :number_of_processors, with: :processes, threshold: ActiveSupport.test_parallelization_threshold, parallelize_databases: ActiveSupport.parallelize_test_databases)` signature | `activesupport/lib/active_support/test_case.rb` line 113 |
| `PARALLEL_WORKERS=15 bin/rails test` environment variable usage | `guides/source/testing.md` lines 2651–2658 |
| `ENV["PARALLEL_WORKERS"]` overrides the `workers` argument when set | `activesupport/lib/active_support/test_case.rb` lines 115–116 |
| Per-worker test databases suffixed with worker number (`test-database-0`, `test-database-1`); 1 or fewer workers skips forking and uses `test-database` | `guides/source/testing.md` lines 2660–2668 |
| `parallelize_setup` / `parallelize_teardown` hooks run right after fork / right before close; not available with threaded parallelization | `guides/source/testing.md` lines 2670–2693 |
| `parallelize_setup(&block)` delegates to `ActiveSupport::Testing::Parallelization.after_fork_hook`; `parallelize_teardown(&block)` delegates to `run_cleanup_hook` | `activesupport/lib/active_support/test_case.rb` lines 155–157, 172–174 |
| Threaded parallelizer is backed by minitest's `Parallel::Executor`; switch with `parallelize(workers: :number_of_processors, with: :threads)` | `guides/source/testing.md` lines 2695–2708 |
| JRuby / TruffleRuby-generated apps include `with: :threads` automatically | `guides/source/testing.md` lines 2710–2711 |
| `PARALLEL_WORKERS` also applies to threaded parallelization | `guides/source/testing.md` lines 2713–2715 |
| Parallel transactions in threads block each other under the implicit test transaction; workaround is `self.use_transactional_tests = false` | `guides/source/testing.md` lines 2717–2724 |
| `WorkerTest < ActiveSupport::TestCase` with `self.use_transactional_tests = false` example | `guides/source/testing.md` lines 2726–2734 |
| With transactional tests disabled, data is not rolled back automatically | `guides/source/testing.md` lines 2736–2737 |
| Parallelization threshold default is 50 tests; configurable via `config.active_support.test_parallelization_threshold` | `guides/source/testing.md` lines 2739–2749 |
| `ActiveSupport.test_parallelization_threshold` default `50` | `activesupport/lib/active_support.rb` line 105 |
| `parallelize threshold: 100` at the test case level | `guides/source/testing.md` lines 2751–2756 |
