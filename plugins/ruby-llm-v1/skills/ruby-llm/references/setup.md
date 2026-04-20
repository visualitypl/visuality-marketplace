# Setup & Configuration

## Install

```ruby
# Gemfile
gem 'ruby_llm'
```

```bash
bundle install
# or
bundle add ruby_llm
```

## Minimal configuration

```ruby
# config/initializers/ruby_llm.rb  (Rails)
# or any setup file in plain Ruby
RubyLLM.configure do |config|
  config.openai_api_key    = ENV['OPENAI_API_KEY']
  config.anthropic_api_key = ENV['ANTHROPIC_API_KEY']
end
```

Configure only the providers you actually use.

## Supported providers — full key list

Every key name is the exact attribute on the `Configuration` object (verified against each provider's `configuration_options` in `lib/ruby_llm/providers/*.rb`).

| Provider | Config keys | Required |
| :--- | :--- | :--- |
| Anthropic | `anthropic_api_key`, `anthropic_api_base` | `api_key` |
| OpenAI | `openai_api_key`, `openai_api_base`, `openai_organization_id`, `openai_project_id`, `openai_use_system_role` | `api_key` |
| Gemini | `gemini_api_key`, `gemini_api_base` | `api_key` |
| Azure | `azure_api_base`, `azure_api_key`, `azure_ai_auth_token` | base + (key or token) |
| Bedrock | `bedrock_api_key`, `bedrock_secret_key`, `bedrock_region`, `bedrock_session_token` | key + secret + region |
| Vertex AI | `vertexai_project_id`, `vertexai_location`, `vertexai_service_account_key` | `project_id` + `location` (key is optional — falls back to Application Default Credentials) |
| DeepSeek | `deepseek_api_key`, `deepseek_api_base` | `api_key` |
| Mistral | `mistral_api_key` | yes |
| Perplexity | `perplexity_api_key` | yes |
| xAI | `xai_api_key` | yes |
| OpenRouter | `openrouter_api_key`, `openrouter_api_base` | `api_key` |
| Ollama | `ollama_api_base`, `ollama_api_key` | depends on server |
| GPUStack | `gpustack_api_base`, `gpustack_api_key` | depends on server |

## Defaults

```ruby
RubyLLM.configure do |config|
  config.default_model              = 'claude-sonnet-4-6'
  config.default_embedding_model    = 'text-embedding-3-large'
  config.default_image_model        = 'dall-e-3'
  config.default_moderation_model   = 'omni-moderation-latest'
  config.default_transcription_model = 'whisper-1'
end
```

After this, `RubyLLM.chat` / `RubyLLM.embed` / `RubyLLM.paint` / `RubyLLM.moderate` / `RubyLLM.transcribe` use those defaults. The gem's own defaults (without your override) are `'gpt-5.4'`, `'text-embedding-3-small'`, `'gpt-image-1.5'`, `'omni-moderation-latest'`, `'whisper-1'`.

## Connection settings

```ruby
config.request_timeout          = 300    # default: 300 seconds
config.max_retries              = 3      # default: 3
config.retry_interval           = 0.1    # default: 0.1
config.retry_backoff_factor     = 2      # default: 2
config.retry_interval_randomness = 0.5   # default: 0.5 (jitter fraction)
config.http_proxy               = "http://user:pass@proxy.company.com:8080"
```

Retries apply to transient errors (timeouts, connection failures, rate limits, 5xx). `ContextLengthExceededError` is not retried.

## Logging

```ruby
config.logger             = Rails.logger
config.log_file           = $stdout     # default: $stdout
config.log_level          = :debug      # default: INFO (DEBUG if RUBYLLM_DEBUG env set)
config.log_stream_debug   = true        # default: ENV['RUBYLLM_STREAM_DEBUG'] == 'true'
config.log_regexp_timeout = 1.5
```

Quick debug toggle without editing config:
```bash
export RUBYLLM_DEBUG=true
export RUBYLLM_STREAM_DEBUG=true   # also dumps raw stream events
```

## Custom OpenAI-compatible endpoint

For Azure, proxies, local servers (llama.cpp, LocalAI, etc.):

```ruby
RubyLLM.configure do |config|
  config.openai_api_base         = "http://localhost:8080/v1"
  config.openai_use_system_role  = true   # for servers that need 'system' role
end

chat = RubyLLM.chat(model: 'custom-model', assume_model_exists: true)
```

`assume_model_exists: true` bypasses registry validation. Required when the model is not in `models.json`.

## Isolated contexts (multi-tenancy, per-request keys)

```ruby
ctx = RubyLLM.context do |config|
  config.openai_api_key  = tenant.openai_key
  config.request_timeout = 180
end

response = ctx.chat(model: 'gpt-4o').ask("Hello")
```

The block yields a **duplicated** `Configuration` — you edit the copy, the global singleton (`RubyLLM.config`) stays untouched. The returned `Context` exposes three entry points: `ctx.chat`, `ctx.embed`, `ctx.paint`. For moderation/transcription, call `RubyLLM.moderate(..., context: ctx)` / `RubyLLM.transcribe(..., context: ctx)` directly. Switch a mid-life chat onto a context via `chat.with_context(ctx)`.

## Rails — initializer location

All RubyLLM config goes in `config/initializers/ruby_llm.rb`. The `ruby_llm:install` generator writes this file for you (including `use_new_acts_as = true`). The Railtie reads settings inside `ActiveSupport.on_load(:active_record)`, which fires after initializers, so the initializer location works for every option including `use_new_acts_as`.

See [rails.md](rails.md) for the generator details.
