# Testing Routes

Covers where route tests live and the authoritative reference for the routing-assertion helpers. Mirrors the **Testing Routes** section of the Rails 8.1.3 testing guide (`guides/source/testing.md` §1836–1847).

Like everything else in your Rails application, you can test your routes. Route tests are stored in `test/controllers/` or are part of controller tests. If your application has complex routes, Rails provides a number of useful helpers to test them.

For more information on routing assertions available in Rails, see the API documentation for [`ActionDispatch::Assertions::RoutingAssertions`](https://api.rubyonrails.org/classes/ActionDispatch/Assertions/RoutingAssertions.html).

The three main routing assertions — `assert_recognizes`, `assert_generates`, and `assert_routing` — are covered in the Rails-Specific Assertions table: see [`writing-tests.md`](writing-tests.md).

## Source traceability

| Claim | Source |
| :--- | :--- |
| Route tests live in `test/controllers/` or inside controller tests | `guides/source/testing.md` §Testing Routes (line 1840) |
| Routing assertions are documented in `ActionDispatch::Assertions::RoutingAssertions` | `actionpack/lib/action_dispatch/testing/assertions/routing.rb`; `guides/source/testing.md` line 1846 |

Rails version: **8.1.3**.
