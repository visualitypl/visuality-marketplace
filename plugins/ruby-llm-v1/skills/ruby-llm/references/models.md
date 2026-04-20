# Model Registry

RubyLLM ships a registry (`lib/ruby_llm/models.json`) with metadata for every known model: provider, type, capabilities, context window, pricing.

## Finding a model

```ruby
info = RubyLLM.models.find('claude-sonnet-4-6')

info.name               # => "Claude Sonnet 4.6"
info.provider           # => "anthropic"   (String, not Symbol)
info.context_window     # => 200000
info.supports_vision?   # => true
info.supports_functions? # => true
info.input_price_per_million   # USD
info.output_price_per_million
```

Raises `RubyLLM::ModelNotFoundError` for unknown IDs — wrap in `rescue` or pre-check with `RubyLLM.models.any? { |m| m.id == id }`. The two-arg form `find(model_id, :provider)` scopes the lookup and also resolves aliases (e.g., Bedrock region prefixes).

## Listing / filtering

```ruby
RubyLLM.models.all                    # everything
RubyLLM.models.chat_models            # only chat
RubyLLM.models.embedding_models
RubyLLM.models.by_provider(:openai)
RubyLLM.models.by_family('claude3_sonnet')

# Chained filters
vision_models = RubyLLM.models
  .by_provider(:openai)
  .select(&:supports_vision?)
```

## Model metadata fields

| Field | Notes |
| :--- | :--- |
| `id` | Provider's own identifier (pass this to `RubyLLM.chat(model:)`) |
| `provider` | String: `"openai"`, `"anthropic"`, `"gemini"`, ... |
| `type` | String: `"chat"`, `"embedding"`, `"image"`, `"audio"`, `"video"`, `"moderation"` (derived from `modalities.output`) |
| `supports_vision?` | True if `modalities.input` includes `'image'` |
| `supports_video?` | True if `modalities.input` includes `'video'` |
| `supports_functions?` | Alias for `function_calling?` |
| `function_calling?`, `structured_output?`, `batch?`, `reasoning?`, `citations?`, `streaming?` | Capability booleans |
| `supports?("name")` | Generic capability check against the `capabilities` array |
| `modalities.input`, `modalities.output` | Arrays of strings like `["text","image","audio"]` — use these for audio/pdf/video checks |
| `context_window` | Max input + output tokens |
| `max_output_tokens` (alias: `max_tokens`) | Max tokens per response |
| `input_price_per_million` | USD per 1M input tokens (= `pricing.text_tokens.input`) |
| `output_price_per_million` | USD per 1M output tokens |
| `knowledge_cutoff` | `Date` (may be nil) |
| `family` | Model family string, e.g. `"claude3_sonnet"` |
| `created_at` | `Time` (UTC) if provider reports a release date |
| `metadata` | Hash with provider-specific extras (`source`, `open_weights`, `status`, ...) |
| `display_name` / `label` | Human-friendly name; `label` prefixes with the provider |

## Refreshing the registry

Pull the latest metadata from provider APIs + models.dev:

```ruby
RubyLLM.models.refresh!                    # hits remote + local providers (default)
RubyLLM.models.refresh!(remote_only: true) # skip Ollama/GPUStack etc.

# v1.9+: persist the refreshed data to disk
RubyLLM.models.save_to_json                          # → default: config.model_registry_file
RubyLLM.models.save_to_json('config/models.json')    # custom path
RubyLLM.models.load_from_json!                       # reload from disk without re-fetching
```

Do this periodically (e.g., a weekly rake task) to stay current with new model releases.

Rails install also creates a rake task that loads `models.json` into the `Model` table (requires `acts_as_model`):
```bash
bin/rails ruby_llm:load_models
```

With `acts_as_model` included, the registry is backed by the database — `RubyLLM.models.all` reads from the `Model` table when it has rows, falling back to the JSON registry otherwise.

## Models not in the registry — `assume_model_exists`

For private deployments, fine-tuned models, or local runners (Ollama, llama.cpp) where the registry has no entry:

```ruby
chat = RubyLLM.chat(
  model:               'custom-deployment-name',
  provider:            :openai,   # REQUIRED when bypassing the registry
  assume_model_exists: true
)
```

Combine with a custom endpoint if needed — two equivalent shapes:

```ruby
# (a) Via OpenAI-compatible routing — works with LM Studio, llama.cpp, LocalAI, etc.
RubyLLM.configure do |config|
  config.openai_api_base = "http://localhost:1234/v1"
end
chat = RubyLLM.chat(
  model: 'local-model',
  provider: :openai,
  assume_model_exists: true
)

# (b) Via the native :ollama / :gpustack providers — `assume_model_exists:` is implicit
RubyLLM.configure do |config|
  config.ollama_api_base = "http://localhost:11434/v1"
end
chat = RubyLLM.chat(model: 'llama3.2', provider: :ollama)
```

Providers flagged `local?` (Ollama, GPUStack) set `assume_model_exists: true` automatically, so the model ID is accepted without a registry entry. Azure is similar via `assume_models_exist?` — Azure deployments are not models in the registry sense.

## Picking a model — decision heuristic

1. **Default** to the current `config.default_model` unless you have a reason to override.
2. For **cheap + fast** classification / routing: small models (Haiku, GPT-5 mini, Gemini Flash).
3. For **complex reasoning / code**: flagship (Opus, GPT-5, Gemini Pro).
4. For **vision**: check `supports_vision?`. Not every model in a family supports images.
5. For **tools**: check `supports_functions?` (or `function_calling?`). Some smaller / older models don't.
6. For **audio**: check `info.modalities.input.include?('audio')` — there is no `supports_audio?` helper.
7. For **long context**: compare `context_window`. Some models beat each other by 5-10x here.

## Other filters worth knowing

```ruby
RubyLLM.models.audio_models    # models that output audio
RubyLLM.models.image_models    # models that output images
RubyLLM.models.find('claude-sonnet-4-6', :anthropic)  # two-arg form scopes to provider
```

Use `RubyLLM.models.find(...)` to verify capabilities before committing to a model in your code.
