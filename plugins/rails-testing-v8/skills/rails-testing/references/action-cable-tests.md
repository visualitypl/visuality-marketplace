# Testing Action Cable

Covers how to test Action Cable connections, channels, and broadcasts triggered from other components in a Rails app. Mirrors the **Testing Action Cable** section of the Rails 8.1.3 testing guide (`guides/source/testing.md` §2462–2610).

Since Action Cable is used at different levels inside your application, you'll need to test both the channels, connection classes themselves, and that other entities broadcast correct messages.

## Connection Test Case

By default, when you generate a new Rails application with Action Cable, a test for the base connection class (`ApplicationCable::Connection`) is generated as well under `test/channels/application_cable` directory.

Connection tests aim to check whether a connection's identifiers get assigned properly or that any improper connection requests are rejected. Here is an example:

```ruby
# test/channels/application_cable/connection_test.rb
class ApplicationCable::ConnectionTest < ActionCable::Connection::TestCase
  test "connects with params" do
    # Simulate a connection opening by calling the `connect` method
    connect params: { user_id: 42 }

    # You can access the Connection object via `connection` in tests
    assert_equal connection.user_id, "42"
  end

  test "rejects connection without params" do
    # Use `assert_reject_connection` matcher to verify that
    # connection is rejected
    assert_reject_connection { connect }
  end
end
```

You can also specify request cookies the same way you do in integration tests:

```ruby
# test/channels/application_cable/connection_test.rb
test "connects with cookies" do
  cookies.signed[:user_id] = "42"

  connect

  assert_equal connection.user_id, "42"
end
```

See the API documentation for [`ActionCable::Connection::TestCase`](https://api.rubyonrails.org/classes/ActionCable/Connection/TestCase.html) for more information.

## Channel Test Case

By default, when you generate a channel, an associated test will be generated as well under the `test/channels` directory. Here's an example test with a chat channel:

```ruby
# test/channels/chat_channel_test.rb
require "test_helper"

class ChatChannelTest < ActionCable::Channel::TestCase
  test "subscribes and stream for room" do
    # Simulate a subscription creation by calling `subscribe`
    subscribe room: "15"

    # You can access the Channel object via `subscription` in tests
    assert subscription.confirmed?
    assert_has_stream "chat_15"
  end
end
```

This test is pretty simple and only asserts that the channel subscribes the connection to a particular stream.

You can also specify the underlying connection identifiers. Here's an example test with a web notifications channel:

```ruby
# test/channels/web_notifications_channel_test.rb
require "test_helper"

class WebNotificationsChannelTest < ActionCable::Channel::TestCase
  test "subscribes and stream for user" do
    stub_connection current_user: users(:john)

    subscribe

    assert_has_stream_for users(:john)
  end
end
```

See the API documentation for [`ActionCable::Channel::TestCase`](https://api.rubyonrails.org/classes/ActionCable/Channel/TestCase.html) for more information.

## Custom Assertions And Testing Broadcasts Inside Other Components

Action Cable ships with a bunch of custom assertions that can be used to lessen the verbosity of tests. For a full list of available assertions, see the API documentation for [`ActionCable::TestHelper`](https://api.rubyonrails.org/classes/ActionCable/TestHelper.html).

It's a good practice to ensure that the correct message has been broadcasted inside other components (e.g. inside your controllers). This is precisely where the custom assertions provided by Action Cable are pretty useful. For instance, within a model:

```ruby
# test/models/product_test.rb
require "test_helper"

class ProductTest < ActionCable::TestCase
  test "broadcast status after charge" do
    assert_broadcast_on("products:#{product.id}", type: "charged") do
      product.charge(account)
    end
  end
end
```

If you want to test the broadcasting made with `Channel.broadcast_to`, you should use `Channel.broadcasting_for` to generate an underlying stream name:

```ruby
# app/jobs/chat_relay_job.rb
class ChatRelayJob < ApplicationJob
  def perform(room, message)
    ChatChannel.broadcast_to room, text: message
  end
end
```

```ruby
# test/jobs/chat_relay_job_test.rb
require "test_helper"

class ChatRelayJobTest < ActiveJob::TestCase
  include ActionCable::TestHelper

  test "broadcast message to room" do
    room = rooms(:all)

    assert_broadcast_on(ChatChannel.broadcasting_for(room), text: "Hi!") do
      ChatRelayJob.perform_now(room, "Hi!")
    end
  end
end
```

## Source traceability

| Claim | Source |
| :--- | :--- |
| Intro: test channels, connections, and that other entities broadcast correct messages | `guides/source/testing.md` lines 2462–2467 |
| Generated `ApplicationCable::Connection` test under `test/channels/application_cable`; connection tests verify identifier assignment and rejection | `guides/source/testing.md` lines 2469–2477 |
| `ApplicationCable::ConnectionTest < ActionCable::Connection::TestCase`; `connect params: { user_id: 42 }`; `connection.user_id`; `assert_reject_connection { connect }` example | `guides/source/testing.md` lines 2479–2495 |
| `ActionCable::Connection::TestCase < ActiveSupport::TestCase`; `connect(path = ActionCable.server.config.mount_path, **request_params)`; `disconnect`; `cookies` | `actioncable/lib/action_cable/connection/test_case.rb` lines 138, 192, 205, 212 |
| `assert_reject_connection(&block)` | `actioncable/lib/action_cable/connection/test_case.rb` line 28 |
| Request cookies example (`cookies.signed[:user_id] = "42"`) | `guides/source/testing.md` lines 2497–2507 |
| API doc link to `ActionCable::Connection::TestCase` | `guides/source/testing.md` lines 2509–2511 |
| Channel tests generated under `test/channels`; `ChatChannelTest < ActionCable::Channel::TestCase`; `subscribe room: "15"`; `subscription.confirmed?`; `assert_has_stream "chat_15"` | `guides/source/testing.md` lines 2513–2535 |
| `ActionCable::Channel::TestCase < ActiveSupport::TestCase`; `attr_reader :connection, :subscription`; `subscribe(params = {})`; `perform(action, data = {})`; `stub_connection(identifiers = {})`; `assert_has_stream(stream)` | `actioncable/lib/action_cable/channel/test_case.rb` lines 190, 202, 249, 266, 243, 304 |
| `stub_connection current_user: users(:john)`; `assert_has_stream_for users(:john)` example | `guides/source/testing.md` lines 2537–2552 |
| `assert_has_stream_for(object)` delegates via `broadcasting_for(object)` | `actioncable/lib/action_cable/channel/test_case.rb` lines 315–316 |
| API doc link to `ActionCable::Channel::TestCase` | `guides/source/testing.md` lines 2554–2556 |
| Custom assertions + API doc link to `ActionCable::TestHelper` | `guides/source/testing.md` lines 2558–2563 |
| `ProductTest < ActionCable::TestCase`; `assert_broadcast_on("products:#{product.id}", type: "charged") do … end` model example | `guides/source/testing.md` lines 2565–2580 |
| `ActionCable::TestCase < ActiveSupport::TestCase` includes `ActionCable::TestHelper` | `actioncable/lib/action_cable/test_case.rb` lines 8–9 |
| `assert_broadcast_on(stream, data, &block)` | `actioncable/lib/action_cable/test_helper.rb` line 116 |
| `Channel.broadcast_to` / `Channel.broadcasting_for` guidance; `ChatRelayJob` + `ChatRelayJobTest` example with `include ActionCable::TestHelper` and `assert_broadcast_on(ChatChannel.broadcasting_for(room), ...)` | `guides/source/testing.md` lines 2582–2609 |
| `Channel.broadcasting_for(broadcastables)` class method | `actioncable/lib/action_cable/channel/broadcasting.rb` line 24 |

See also: [`writing-tests.md`](writing-tests.md) for Rails-specific assertions; [`job-tests.md`](job-tests.md) for `ActiveJob::TestCase` used by the `ChatRelayJobTest` example.

Rails version: **8.1.3**.
