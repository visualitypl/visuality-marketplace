# The Rails Test Runner

Covers `bin/rails test` invocation patterns — running the full suite, a single file, a specific test method, a line number or range, a directory — and the runner's minitest option flags. Mirrors the **The Rails Test Runner** section of the Rails 8.1.3 testing guide (`guides/source/testing.md` §512–590).

## Running a test suite

Run all tests at once with the `bin/rails test` command, or a single test file by appending the filename:

```bash
$ bin/rails test test/models/article_test.rb
Running 1 tests in a single process (parallelization threshold is 50)
Run options: --seed 1559

# Running:

..

Finished in 0.027034s, 73.9810 runs/s, 110.9715 assertions/s.

2 runs, 3 assertions, 0 failures, 0 errors, 0 skips
```

This will run all test methods from the test case. The output format above (seed, run count, Finished-in line, tallies) is what every invocation below produces — only the command changes.

Run a particular test method by name with `-n` / `--name`:

```bash
$ bin/rails test test/models/article_test.rb -n test_the_truth
```

Run a test at a specific line by appending the line number:

```bash
$ bin/rails test test/models/article_test.rb:6
```

Run a range of tests by providing the line range:

```bash
$ bin/rails test test/models/article_test.rb:6-20
```

Run an entire directory of tests by providing the path to the directory:

```bash
$ bin/rails test test/controllers
```

The test runner also provides other features like failing fast, showing verbose progress, and so on. Inspect the full help with:

```bash
$ bin/rails test -h
```

`bin/rails test` runs all tests **except system tests** — system tests are invoked separately via `bin/rails test:system`.

## Runner flags

The runner's minitest options include:

| Option | Purpose |
| :--- | :--- |
| `-h`, `--help` | Display help. |
| `--no-plugins` | Bypass minitest plugin auto-loading (or set `$MT_NO_PLUGINS`). |
| `-s`, `--seed SEED` | Sets random seed. Also via env. Eg: `SEED=n rake`. |
| `-v`, `--verbose` | Verbose. Show progress processing files. |
| `--show-skips` | Show skipped at the end of run. |
| `-n`, `--name PATTERN` | Filter run on `/regexp/` or string. |
| `--exclude PATTERN` | Exclude `/regexp/` or string from run. |
| `-S`, `--skip CODES` | Skip reporting of certain types of results (eg `E`). |
| `-w`, `--warnings` | Run with Ruby warnings enabled. |
| `-e`, `--environment ENV` | Run tests in the ENV environment. |
| `-b`, `--backtrace` | Show the complete backtrace. |
| `-d`, `--defer-output` | Output test failures and errors after the test run. |
| `-f`, `--fail-fast` | Abort test run on first failure or error. |
| `-c`, `--[no-]color` | Enable color in the output. |
| `--profile [COUNT]` | Enable profiling of tests and list the slowest test cases (default: 10). |
| `-p`, `--pride` | Pride. Show your testing pride! |

Known extensions: `rails`, `pride`.
