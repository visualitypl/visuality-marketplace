# Streaming

Block-based API. Pass a block to `chat.ask` and you get `RubyLLM::Chunk` objects as they arrive. `ask` still returns the accumulated `Message` after the stream ends.

## Basic streaming

```ruby
final_message = chat.ask("Tell me a story.") do |chunk|
  print chunk.content
  $stdout.flush
end

puts "\n\nFull text:"
puts final_message.content
```

## Chunk structure

Each yielded `RubyLLM::Chunk` has:

| Field | Notes |
| :--- | :--- |
| `content` | Text fragment. May be nil for metadata-only chunks. |
| `role` | Always `:assistant` for responses. |
| `model_id` | Model that produced the chunk. |
| `tool_calls` | Partial / complete tool invocation data (Hash). |
| `input_tokens` / `output_tokens` | Usually only final chunk has accurate counts. |
| `thinking` | Optional extended-thinking stream (Claude, etc.). |

Do not depend on token counts being present or accurate in intermediate chunks.

## Rails / Turbo Streams

Run the streaming in a background job, broadcast chunks to connected clients:

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

This mirrors what `bin/rails generate ruby_llm:chat_ui` writes. `chat.ask` (not `chat.complete`) adds the user message and streams the reply; `chat.messages.last` is the empty assistant row created before the first chunk arrives, so `broadcast_append_chunk` can target it. See [rails.md](rails.md) for the `broadcast_append_chunk` definition and `broadcasts_to`.

### Message ordering gotcha

Action Cable's concurrent processing can deliver broadcasts out of order. Reorder client-side by `created_at` in Stimulus:

```javascript
// app/javascript/controllers/messages_controller.js
import { Controller } from "@hotwired/stimulus"

export default class extends Controller {
  connect() { this.#reorder() }
  messageAdded() { this.#reorder() }

  #reorder() {
    const items = Array.from(this.element.children)
    items.sort((a, b) => a.dataset.createdAt.localeCompare(b.dataset.createdAt))
    items.forEach(el => this.element.appendChild(el))
  }
}
```

## Sinatra / Server-Sent Events

```ruby
get '/stream' do
  content_type 'text/event-stream'
  stream do |out|
    chat = RubyLLM.chat
    chat.ask(params[:q]) do |chunk|
      out << "data: #{chunk.content.to_json}\n\n"
    end
    out << "event: done\ndata: {}\n\n"
  rescue RubyLLM::Error => e
    out << "event: error\ndata: #{e.message.to_json}\n\n"
  end
end
```

## Tool calls during streaming

Streaming with tools goes through distinct phases:
1. Initial response generation (partial tokens streamed).
2. Tool call detection — arguments arrive incrementally.
3. Execution pause while the tool runs.
4. Resumed streaming with the tool result incorporated.

Handle both text and `tool_calls` chunks in your block if you want visibility into each phase.

## Errors during streaming

Errors raise **after** the block completes. Partial content accumulates before the error — check `response.content` (or your accumulator) even on failure. See [errors.md](errors.md).
