# Chat

## Basic exchange

```ruby
chat = RubyLLM.chat
response = chat.ask("Explain 'Convention over Configuration' in Rails.")
puts response.content
```

`chat` keeps history automatically — subsequent `.ask` calls on the same object extend the conversation.

```ruby
chat.ask("Give a specific example.")
chat.messages.each { |m| puts "[#{m.role.upcase}] #{m.content}" }
```

## System prompts / instructions

```ruby
chat.with_instructions("You are a helpful assistant that explains Ruby concepts simply.")
chat.ask("What is a variable?")

# Replace the system prompt
chat.with_instructions("Always end responses with 'Got it?'")

# Append another system message
chat.with_instructions("Use one short paragraph.", append: true)

# Legacy flag: `replace: false` is equivalent to `append: true`
chat.with_instructions("Also cite sources.", replace: false)
```

`with_instructions` adds a persistent system-role message applied to every turn in this chat. System messages are stored in `chat.messages` with `role: :system` and live at the front of the array regardless of when you add them.

## Picking a model

```ruby
chat = RubyLLM.chat(model: 'claude-sonnet-4-6')
# or switch mid-conversation:
chat.with_model('claude-opus-4-7')
```

No provider argument needed — RubyLLM infers it from the model ID via the registry (`models.json`). Override only for unregistered models via `provider:` + `assume_model_exists: true`.

## Multimodal input (images, audio, PDFs, text files)

Pass files via `with:`. Auto-detection handles mime type.

```ruby
# Image
chat = RubyLLM.chat(model: 'gpt-4o')
chat.ask("Describe this logo.", with: "path/to/logo.png")

# Multiple images
chat.ask("Compare these UIs.", with: ["v1.png", "v2.png"])

# Audio
chat = RubyLLM.chat(model: 'gpt-4o-audio-preview')
chat.ask("Transcribe this meeting.", with: "meeting.mp3")

# PDFs / text
chat.ask("Summarize this.", with: "paper.pdf")
chat.ask("Explain this code.", with: "app/models/user.rb")

# Mixed auto-detection
chat.ask("Analyze these files", with: [
  "diagram.png", "report.pdf", "notes.txt", "recording.mp3"
])
```

The model used must support the modality — check [models.md](models.md) for capability flags.

## Temperature

```ruby
factual  = RubyLLM.chat.with_temperature(0.2)
creative = RubyLLM.chat.with_temperature(0.9)
```

## Provider-specific parameters and headers

Escape hatches for settings not mapped to the unified API:

```ruby
chat = RubyLLM.chat.with_params(response_format: { type: 'json_object' })
chat = RubyLLM.chat.with_headers('anthropic-beta' => 'prompt-caching-2024-07-31')
```

Prefer [structured-output.md](structured-output.md) `with_schema` over `with_params` when you want typed JSON — it's portable across providers.

## Extended thinking (Claude, reasoning models)

```ruby
chat = RubyLLM.chat(model: 'claude-opus-4-7')
chat.with_thinking(effort: :high)    # :low / :medium / :high
chat.with_thinking(budget: 10_000)   # explicit token budget
chat.with_thinking(effort: :none)    # disable on models that think by default (e.g. Qwen3)
```

Requires a model that supports `reasoning?`. Check with `RubyLLM.models.find(id).reasoning?` (see [models.md](models.md)). Passing neither `:effort` nor `:budget` raises `ArgumentError` — RubyLLM forwards the values verbatim, so consult your provider's docs for accepted ranges.

`response.thinking` returns a `RubyLLM::Thinking` (or nil) with `.text` and `.signature` — not a string:

```ruby
puts response.thinking&.text       # the model's reasoning
puts response.thinking_tokens      # alias: reasoning_tokens
```

## Clearing history, aliases

```ruby
chat.reset_messages!   # drop ALL messages (including system instructions). Tools stay attached.
chat.say("hi")         # alias for .ask
```

To wipe history but keep a system prompt, call `with_instructions` again after `reset_messages!`. `Chat` includes `Enumerable` (delegating to `messages.each`), so `chat.map(&:role)`, `chat.select(&:tool_call?)`, etc. work.

## Streaming

Pass a block to `.ask` to stream chunks. Block receives `RubyLLM::Chunk` objects; the method still returns the accumulated `Message` after the stream ends.

```ruby
final = chat.ask("Tell me a story.") do |chunk|
  print chunk.content
end
puts "\n--- Full response ---"
puts final.content
```

See [streaming.md](streaming.md) for chunk structure and framework integration.

## Raw provider payloads (`Content::Raw`)

When you need to send a provider-shaped payload untouched (e.g., Anthropic prompt-caching blocks), wrap it:

```ruby
raw = RubyLLM::Content::Raw.new([
  { type: 'text', text: large_context, cache_control: { type: 'ephemeral' } },
  { type: 'text', text: "Today's request: #{summary}" }
])

chat.add_message(role: :system, content: raw)
chat.ask(raw)
```

`ask`, `add_message`, tool return values, and the streaming accumulator all understand `Content::Raw`. Anthropic has a convenience builder: `RubyLLM::Providers::Anthropic::Content.new(text, cache: true)` returns a properly shaped `Content::Raw`.

## Raw HTTP response

```ruby
response.raw          # Faraday::Response — status, headers, body
response.raw.headers  # e.g. rate-limit headers
```

Use it when you need provider specifics that the unified interface doesn't expose. For errors, the same object lives on `RubyLLM::Error#response` — see [errors.md](errors.md).

## Token accounting

```ruby
response = chat.ask("Explain the Ruby GIL.")
puts "Input: #{response.input_tokens}"
puts "Output: #{response.output_tokens}"
puts "Cached: #{response.cached_tokens}"
puts "Cache writes: #{response.cache_creation_tokens}"
```

Fields may be nil for providers that don't report them. During streaming, token counts are typically populated only on the final chunk.

## Event callbacks

```ruby
chat.on_new_message  { print "Assistant > " }
chat.on_end_message  { |msg| puts "\n[Complete]" }
chat.on_tool_call    { |call| puts "Calling: #{call.name} #{call.arguments.inspect}" }
chat.on_tool_result  { |result| puts "Got: #{result}" }

chat.ask("What's the weather in Berlin?")
```

Useful for logging, progress indicators, and debugging tool-using conversations.

## Tools (brief)

```ruby
chat.with_tool(Weather.new)
chat.with_tool(Weather.new, choice: :required, calls: :one)   # single-tool form accepts the same flags

# Multiple, with fresh set (replace: true drops previously attached tools):
chat.with_tools(Weather, Calculator, replace: true, choice: :auto, calls: :many)
```

`choice:` and `calls:` default to `nil` (= whatever the provider's default is). Valid values are listed in [tools.md](tools.md#tool-call-controls-v113).
