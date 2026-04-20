# Rails Integration

## Install

```bash
bin/rails generate ruby_llm:install
bin/rails db:migrate
bin/rails ruby_llm:load_models
```

Creates migrations + models for `Chat`, `Message`, `ToolCall`, `Model`. Writes `config/initializers/ruby_llm.rb` with `use_new_acts_as = true`. Creates empty convention folders: `app/agents/`, `app/tools/`, `app/schemas/`, `app/prompts/`. Installs ActiveStorage for file attachments.

### Generator options

Custom model names from the start (otherwise the ones above are used):

```bash
bin/rails generate ruby_llm:install chat:Conversation message:ChatMessage tool_call:Invocation
```

Skip ActiveStorage (attachments won't work):

```bash
bin/rails generate ruby_llm:install --skip-active-storage
```

### Other generators

| Generator | What it makes |
| :--- | :--- |
| `ruby_llm:install` | Migrations, models, initializer, ActiveStorage, convention folders |
| `ruby_llm:chat_ui` | Controllers, views, streaming job, routes, `broadcast_append_chunk` |
| `ruby_llm:agent NAME` | `app/agents/<name>_agent.rb` + `app/prompts/<name>_agent/instructions.txt.erb` |
| `ruby_llm:tool NAME` | `app/tools/<name>_tool.rb` |
| `ruby_llm:schema NAME` | `app/schemas/<name>_schema.rb` |
| `ruby_llm:upgrade_to_v1_14` | Migrates DB columns to the 1.14 shape |
| `ruby_llm:upgrade_to_v1_10` | Earlier upgrade migration |
| `ruby_llm:upgrade_to_v1_9` | Earlier upgrade migration |
| `ruby_llm:upgrade_to_v1_7` | Flips initializer to `use_new_acts_as = true` and rewrites models |

### Chat UI scaffold (Turbo streaming, Tailwind)

```bash
bin/rails generate ruby_llm:chat_ui
# --ui=scaffold  (plain Rails scaffold look)
# --ui=tailwind  (tailwindcss-rails look, default when Tailwind is detected)
# --ui=auto      (pick automatically; default)
```

Adds controllers, views, a background job for streaming, and the `broadcasts_to` + `broadcast_append_chunk` code to your Message model. Requires `turbo-rails`.

## Core models

```ruby
# app/models/chat.rb
class Chat < ApplicationRecord
  acts_as_chat
  belongs_to :user, optional: true
end

# app/models/message.rb
class Message < ApplicationRecord
  acts_as_message
  has_many_attached :attachments
end

# app/models/tool_call.rb
class ToolCall < ApplicationRecord
  acts_as_tool_call
end

# app/models/model.rb
class Model < ApplicationRecord
  acts_as_model
end
```

## Usage

```ruby
chat = Chat.create!(model: 'claude-sonnet-4-6', user: current_user)

response = chat.ask("What is the capital of France?")
# user + assistant messages are persisted automatically

chat.messages.last.content
# => "Paris is the capital of France..."

chat.ask("Tell me more about it.")
```

## Persistence flow

When you call `chat.ask(...)`:

1. User message is saved immediately.
2. An empty assistant message is created (before streaming begins).
3. On success, the assistant message is updated with content and metadata.
4. On API failure, the empty assistant message is automatically destroyed.

This design lets Turbo Streams broadcast the assistant row into the DOM immediately, then update its content as chunks arrive — without leaving orphan records if the API call fails.

## Critical caveat — do NOT add content presence validation

Because step 2 creates an empty message, this will break:

```ruby
# DO NOT DO THIS
class Message < ApplicationRecord
  acts_as_message
  validates :content, presence: true   # will reject the placeholder row
end
```

If you need validation, override `persist_new_message` and `persist_message_completion` on the model. Validate at completion time, not at creation.

## File attachments (ActiveStorage)

```ruby
chat.ask("What's in this file?", with: "path/to/document.pdf")
chat.ask("Analyze these.",       with: ["file1.pdf", "file2.jpg"])
chat.ask("Review this.",         with: params[:uploaded_file])  # ActionDispatch::Http::UploadedFile
```

Attachments are stored via `has_many_attached :attachments` on the Message model.

## Streaming with Turbo

`broadcasts_to` is **not** provided by RubyLLM — it comes from the `turbo-rails` gem. Add it yourself, or run `ruby_llm:chat_ui` which wires it up for you:

```ruby
# app/models/message.rb
class Message < ApplicationRecord
  acts_as_message
  has_many_attached :attachments

  # from turbo-rails:
  broadcasts_to ->(msg) { "chat_#{msg.chat_id}" }, inserts_by: :append

  # added by ruby_llm:chat_ui — streaming chunks append to the assistant row:
  def broadcast_append_chunk(content)
    broadcast_append_to "chat_#{chat_id}",
      target: "message_#{id}_content",
      content: ERB::Util.html_escape(content.to_s)
  end
end
```

Then kick off streaming from a background job:

```ruby
# app/jobs/chat_response_job.rb
class ChatResponseJob < ApplicationJob
  def perform(chat_id, user_content)
    chat = Chat.find(chat_id)
    chat.ask(user_content) do |chunk|
      next if chunk.content.nil? || chunk.content.empty?
      chat.messages.last.broadcast_append_chunk(chunk.content)
    end
  end
end
```

Use `chat.ask(content) do |chunk| ... end` — not `chat.complete` — so the user message gets added and persisted before streaming begins.

See [streaming.md](streaming.md) for chunk details and the out-of-order delivery fix.

## Runtime-only instructions (not persisted)

```ruby
chat.with_runtime_instructions("For this request only: answer in JSON.")
chat.ask("list the primes under 20")
```

`with_runtime_instructions` applies the system message to the underlying `RubyLLM::Chat` and remembers it across `to_llm` rebuilds, but **does not insert a row** in the messages table. Use this for per-request instructions you don't want to persist as part of the conversation (e.g., policy boilerplate, tenant scoping). `with_instructions` remains the persistent path.

## Custom model names

If your domain uses `Conversation` / `ChatMessage` instead of `Chat` / `Message`:

```ruby
class Conversation < ApplicationRecord
  acts_as_chat messages:      :chat_messages,
               message_class: 'ChatMessage'
end

class ChatMessage < ApplicationRecord
  acts_as_message chat:       :conversation,
                  chat_class: 'Conversation'
end
```

## Combining with tools

```ruby
class WeatherTool < RubyLLM::Tool
  description "Current weather"
  params { string :city }
  def execute(city:)
    WeatherClient.fetch(city)
  end
end

chat = Chat.create!(model: 'claude-sonnet-4-6')
chat.with_tools(WeatherTool)
chat.ask("What's the weather in Warsaw?")

# chat.messages now includes ToolCall rows for the model's calls
```

See [tools.md](tools.md) for the full tool API.

## `use_new_acts_as` flag

Fresh projects don't need to touch this — the `ruby_llm:install` generator writes `config.use_new_acts_as = true` into `config/initializers/ruby_llm.rb` automatically.

Legacy projects (acts_as API before 1.7) still load the old code path. To migrate:

```bash
bin/rails generate ruby_llm:upgrade_to_v1_7
```

This edits your initializer to set `use_new_acts_as = true` and updates your models. Railtie reads the flag inside `ActiveSupport.on_load(:active_record)`, so the initializer fires early enough.
