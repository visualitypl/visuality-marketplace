# Additional Testing Resources

Covers two topics the guide groups under its closing **Additional Testing Resources** H2: how Rails rescues errors during integration/system/controller tests, and how to test time-dependent code with `travel_to`. Mirrors `guides/source/testing.md` §2797–2836.

## Errors

In system tests, integration tests and functional controller tests, Rails will attempt to rescue from errors raised and respond with HTML error pages by default. This behavior can be controlled by the [`config.action_dispatch.show_exceptions`](/configuring.html#config-action-dispatch-show-exceptions) configuration.

## Testing Time-Dependent Code

Rails provides built-in helper methods that enable you to assert that your time-sensitive code works as expected.

The following example uses the [`travel_to`](https://api.rubyonrails.org/classes/ActiveSupport/Testing/TimeHelpers.html#method-i-travel_to) helper:

```ruby
# Given a user is eligible for gifting a month after they register.
user = User.create(name: "Gaurish", activation_date: Date.new(2004, 10, 24))
assert_not user.applicable_for_gifting?

travel_to Date.new(2004, 11, 24) do
  # Inside the `travel_to` block `Date.current` is stubbed
  assert_equal Date.new(2004, 10, 24), user.activation_date
  assert user.applicable_for_gifting?
end

# The change was visible only inside the `travel_to` block.
assert_equal Date.new(2004, 10, 24), user.activation_date
```

Please see [`ActiveSupport::Testing::TimeHelpers`](https://api.rubyonrails.org/classes/ActiveSupport/Testing/TimeHelpers.html) API reference for more information about the available time helpers.
