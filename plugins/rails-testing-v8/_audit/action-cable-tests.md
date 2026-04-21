# Source traceability — action-cable-tests.md

Backs [`../skills/rails-testing/references/action-cable-tests.md`](../skills/rails-testing/references/action-cable-tests.md). Rails version: **8.1.3**.

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
