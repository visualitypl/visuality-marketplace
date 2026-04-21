# Source traceability — test-runner.md

Backs [`../skills/rails-testing/references/test-runner.md`](../skills/rails-testing/references/test-runner.md). Rails version: **8.1.3**.

| Claim | Source |
| :--- | :--- |
| `bin/rails test` runs all tests; appending a filename runs that file | `guides/source/testing.md` §The Rails Test Runner |
| `-n` / `--name` runs a specific method; line numbers and line ranges (`path:6`, `path:6-20`) target specific tests; passing a directory runs all tests beneath it | `guides/source/testing.md` §The Rails Test Runner |
| `bin/rails test -h` prints full help; `bin/rails test` skips system tests (run those via `bin/rails test:system`) | `guides/source/testing.md` §The Rails Test Runner |
| Runner flag catalog (`-h`, `--no-plugins`, `-s`, `-v`, `--show-skips`, `-n`, `--exclude`, `-S`, `-w`, `-e`, `-b`, `-d`, `-f`, `-c`, `--profile`, `-p`) | `guides/source/testing.md` §The Rails Test Runner |
