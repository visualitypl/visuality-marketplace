# Error Handling

## Hierarchy

**API errors** (inherit from `RubyLLM::Error`):

| Class | Status |
| :--- | :--- |
| `BadRequestError` | 400 |
| `UnauthorizedError` | 401 |
| `PaymentRequiredError` | 402 |
| `ForbiddenError` | 403 |
| `RateLimitError` | 429 |
| `ServerError` | 500 |
| `ServiceUnavailableError` | 502 / 503 / 504 |
| `OverloadedError` | 529 |
| `ContextLengthExceededError` | — |

**Non-API errors** (inherit from `StandardError`):

- `ConfigurationError` — missing/invalid config.
- `ModelNotFoundError` — model ID not in registry and no `assume_model_exists`.
- `InvalidRoleError` — tried to add a message with an unknown role.
- `InvalidToolChoiceError` — bad `choice:` value passed to `with_tools`.
- `PromptNotFoundError` — Agent tried to render an ERB prompt file that doesn't exist.
- `UnsupportedAttachmentError` — file type can't be sent to the chosen model.

## Basic handling

```ruby
begin
  response = chat.ask("Translate 'hello' to French.")
  puts response.content
rescue RubyLLM::Error => e
  puts "API error: #{e.message}"
rescue RubyLLM::ConfigurationError => e
  puts "Config missing: #{e.message}"
end
```

## Specific recovery

```ruby
begin
  chat.ask(long_prompt)
rescue RubyLLM::ContextLengthExceededError
  # summarize or split
rescue RubyLLM::RateLimitError => e
  # RubyLLM already retries 429s internally (see below). If you still get
  # this, the retry budget was exhausted — back off further or surface it.
  retry_after = e.response&.headers&.[]("retry-after")&.to_i
  sleep(retry_after || 10)
  retry
rescue RubyLLM::UnauthorizedError
  raise "API key is invalid — check ENV['ANTHROPIC_API_KEY']"
end
```

`RubyLLM::Error#response` is the underlying `Faraday::Response` — use it for headers and body details. There is no `retry_after` method on the error itself.

## Drilling into the response

Every API error exposes the underlying `Faraday::Response`:

```ruby
rescue RubyLLM::ForbiddenError => e
  puts "Status: #{e.response&.status}"
  puts "Body:   #{e.response&.body}"
  if e.response&.body.to_s.include?('invalid_organization')
    Rails.logger.warn "OpenAI org ID misconfigured"
  end
end
```

## Automatic retries

RubyLLM retries these automatically:
- Timeouts (`Faraday::TimeoutError`)
- Connection failures
- Rate limits (`429`)
- Server errors (`5xx`)

Configure:

```ruby
RubyLLM.configure do |config|
  config.max_retries           = 5     # default: 3
  config.retry_interval        = 0.5   # default: 0.1 seconds
  config.retry_backoff_factor  = 2     # exponential
end
```

**Not retried:** `ContextLengthExceededError` (no point — retry won't help).

## Streaming errors

When `chat.ask` is given a block and fails mid-stream:

- The error raises **after** the block finishes (the remaining chunks the provider already sent may still have yielded).
- Partial content is still accessible via your accumulator or `response.content` if `ask` returned.

```ruby
accumulated = +""
begin
  chat.ask(prompt) { |c| accumulated << c.content.to_s }
rescue RubyLLM::Error => e
  Rails.logger.error("Stream failed after #{accumulated.bytesize}B: #{e.message}")
  # decide: retry, show partial, or abort
end
```

## Debug mode

```bash
RUBYLLM_DEBUG=true bundle exec rails server
```

Or:

```ruby
config.logger          = Logger.new($stdout)
config.log_level       = :debug
config.log_stream_debug = true
```

## Tool errors (brief)

Inside a tool's `execute`, return an error hash if the LLM could recover (bad input), raise if the caller should handle it. See [tools.md](tools.md#error-handling).
