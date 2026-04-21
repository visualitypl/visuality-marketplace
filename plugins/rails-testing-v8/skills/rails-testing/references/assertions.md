# Assertions

Covers the assertion helpers available in Rails tests: Minitest's non-obvious entries, the full set of Rails-specific assertions, and which test-case classes expose them. Mirrors the **Minitest Assertions**, **Rails-Specific Assertions**, and **Assertions in Test Cases** sections of the Rails 8.1.3 testing guide (`guides/source/testing.md` §304–510).

## Minitest Assertions — non-obvious entries

Standard Minitest assertions (`assert`, `assert_not`, `assert_equal`, `assert_not_equal`, `assert_nil`, `assert_not_nil`, `assert_match`, `assert_no_match`, `assert_includes`, `assert_not_includes`, `assert_raises`, `assert_instance_of`, `assert_not_instance_of`, `assert_kind_of`, `assert_not_kind_of`, `assert_respond_to`, `assert_not_respond_to`, `flunk`) work as documented in the [minitest API docs](http://docs.seattlerb.org/minitest/Minitest/Assertions.html). The entries below are the ones with subtle semantics — identity vs. equality, numeric tolerance, operator/predicate sugar, throw-vs-raise.

| Assertion | Purpose |
| :--- | :--- |
| `assert_same(expected, actual, [msg])` | Ensures that `expected.equal?(actual)` is true — object **identity**, not equality. |
| `assert_not_same(expected, actual, [msg])` | Ensures that `expected.equal?(actual)` is false. |
| `assert_empty(obj, [msg])` | Ensures that `obj.empty?` is true. |
| `assert_not_empty(obj, [msg])` | Ensures that `obj.empty?` is false. |
| `assert_in_delta(expected, actual, [delta], [msg])` | Ensures that the numbers `expected` and `actual` are within `delta` of each other (absolute-error tolerance for floats). |
| `assert_not_in_delta(expected, actual, [delta], [msg])` | Ensures that the numbers `expected` and `actual` are not within `delta` of each other. |
| `assert_in_epsilon(expected, actual, [epsilon], [msg])` | Ensures that the numbers `expected` and `actual` have a relative error less than `epsilon`. |
| `assert_not_in_epsilon(expected, actual, [epsilon], [msg])` | Ensures that the numbers `expected` and `actual` have a relative error not less than `epsilon`. |
| `assert_throws(symbol, [msg]) { block }` | Ensures the block throws the symbol via Ruby's `throw` — distinct from `assert_raises`, which matches exceptions. |
| `assert_operator(obj1, operator, [obj2], [msg])` | Ensures that `obj1.send(operator, obj2)` is true. |
| `assert_not_operator(obj1, operator, [obj2], [msg])` | Ensures that `obj1.send(operator, obj2)` is false. |
| `assert_predicate(obj, predicate, [msg])` | Ensures that `obj.send(predicate)` is true, e.g. `assert_predicate str, :empty?`. |
| `assert_not_predicate(obj, predicate, [msg])` | Ensures that `obj.send(predicate)` is false, e.g. `assert_not_predicate str, :empty?`. |

With minitest you can add your own assertions. In fact, that's exactly what Rails does — it includes some specialized assertions to make your life easier.

> **NOTE:** Creating your own assertions is a topic that this guide does not cover in depth.

## Rails-Specific Assertions

Rails adds some custom assertions of its own to the `minitest` framework:

| Assertion | Purpose |
| :--- | :--- |
| `assert_difference(expressions, difference = 1, message = nil) {...}` | Test numeric difference between the return value of an expression as a result of what is evaluated in the yielded block. |
| `assert_no_difference(expressions, message = nil, &block)` | Asserts that the numeric result of evaluating an expression is not changed before and after invoking the passed in block. |
| `assert_changes(expressions, message = nil, from:, to:, &block)` | Test that the result of evaluating an expression is changed after invoking the passed in block. |
| `assert_no_changes(expressions, message = nil, &block)` | Test the result of evaluating an expression is not changed after invoking the passed in block. |
| `assert_nothing_raised { block }` | Ensures that the given block doesn't raise any exceptions. |
| `assert_recognizes(expected_options, path, extras = {}, message = nil)` | Asserts that the routing of the given path was handled correctly and that the parsed options (given in the `expected_options` hash) match path. Basically, it asserts that Rails recognizes the route given by `expected_options`. |
| `assert_generates(expected_path, options, defaults = {}, extras = {}, message = nil)` | Asserts that the provided options can be used to generate the provided path. This is the inverse of `assert_recognizes`. The `extras` parameter is used to tell the request the names and values of additional request parameters that would be in a query string. The `message` parameter allows you to specify a custom error message for assertion failures. |
| `assert_routing(expected_path, options, defaults = {}, extras = {}, message = nil)` | Asserts that `path` and `options` match both ways; in other words, it verifies that `path` generates `options` and then that `options` generates `path`. This essentially combines `assert_recognizes` and `assert_generates` into one step. The `extras` hash allows you to specify options that would normally be provided as a query string to the action. The `message` parameter allows you to specify a custom error message to display upon failure. |
| `assert_response(type, message = nil)` | Asserts that the response comes with a specific status code. You can specify `:success` to indicate 200-299, `:redirect` to indicate 300-399, `:missing` to indicate 404, or `:error` to match the 500-599 range. You can also pass an explicit status number or its symbolic equivalent. |
| `assert_redirected_to(options = {}, message = nil)` | Asserts that the response is a redirect to a URL matching the given options. You can also pass named routes such as `assert_redirected_to root_path` and Active Record objects such as `assert_redirected_to @article`. |
| `assert_queries_count(count = nil, include_schema: false, &block)` | Asserts that `&block` generates an `int` number of SQL queries. |
| `assert_no_queries(include_schema: false, &block)` | Asserts that `&block` generates no SQL queries. |
| `assert_queries_match(pattern, count: nil, include_schema: false, &block)` | Asserts that `&block` generates SQL queries that match the pattern. |
| `assert_no_queries_match(pattern, &block)` | Asserts that `&block` generates no SQL queries that match the pattern. |
| `assert_error_reported(class) { block }` | Asserts that the error class has been reported, e.g. `assert_error_reported IOError { Rails.error.report(IOError.new("Oops")) }`. |
| `assert_no_error_reported { block }` | Asserts that no errors have been reported, e.g. `assert_no_error_reported { perform_service }`. |

## Assertions in Test Cases

All the basic assertions such as `assert_equal` defined in `Minitest::Assertions` are also available in the classes we use in our own test cases. In fact, Rails provides the following classes for you to inherit from:

* [`ActiveSupport::TestCase`](https://api.rubyonrails.org/classes/ActiveSupport/TestCase.html)
* [`ActionMailer::TestCase`](https://api.rubyonrails.org/classes/ActionMailer/TestCase.html)
* [`ActionView::TestCase`](https://api.rubyonrails.org/classes/ActionView/TestCase.html)
* [`ActiveJob::TestCase`](https://api.rubyonrails.org/classes/ActiveJob/TestCase.html)
* [`ActionDispatch::Integration::Session`](https://api.rubyonrails.org/classes/ActionDispatch/Integration/Session.html)
* [`ActionDispatch::SystemTestCase`](https://api.rubyonrails.org/classes/ActionDispatch/SystemTestCase.html)
* [`Rails::Generators::TestCase`](https://api.rubyonrails.org/classes/Rails/Generators/TestCase.html)

Each of these classes includes `Minitest::Assertions`, allowing you to use all of the basic assertions in your tests.

> **TIP:** For more information on `minitest`, refer to the [minitest documentation](http://docs.seattlerb.org/minitest).
