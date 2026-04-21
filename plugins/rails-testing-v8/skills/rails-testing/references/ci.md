# Running tests in Continuous Integration (CI)

Covers the single command needed to run the full Rails test suite in a CI pipeline. Mirrors the **Running tests in Continuous Integration (CI)** section of the Rails 8.1.3 testing guide (`guides/source/testing.md` §2611–2628).

Continuous Integration (CI) is a development practice where changes are frequently integrated into the main codebase, and as such, are automatically tested before merge.

To run all tests in a CI environment, there's just one command you need:

```bash
$ bin/rails test
```

If you are using [System Tests](system-tests.md), `bin/rails test` will not run them, since they can be slow. To also run them, add another CI step that runs `bin/rails test:system`, or change your first step to `bin/rails test:all`, which runs all tests including system tests.

## Source traceability

| Claim | Source |
| :--- | :--- |
| `bin/rails test` is the single command to run all tests in CI | `guides/source/testing.md` lines 2614–2622 |
| `bin/rails test` skips system tests; use `bin/rails test:system` or `bin/rails test:all` to include them | `guides/source/testing.md` lines 2624–2627 |

See also: [`writing-tests.md`](writing-tests.md) for the full `bin/rails test` runner options; [`system-tests.md`](system-tests.md) for system-test setup.

Rails version: **8.1.3**.
