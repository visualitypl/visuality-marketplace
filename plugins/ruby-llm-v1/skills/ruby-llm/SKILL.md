---
name: ruby-llm
description: "RubyLLM gem (1.14+) reference for Ruby/Rails LLM integration. Covers chat, streaming, multimodal I/O, tools (params DSL), structured output (RubyLLM::Schema), Rails (acts_as_chat + Turbo broadcasting + generators), agents, embeddings, image generation, moderation, transcription, errors, and model registry across OpenAI/Anthropic/Gemini/Bedrock/Azure/VertexAI/OpenRouter/DeepSeek/Mistral/Perplexity/xAI/Ollama/GPUStack. TRIGGER when ANY of: Gemfile has `gem 'ruby_llm'`; file has `require 'ruby_llm'` or references `RubyLLM::`, `RubyLLM.chat`/`.embed`/`.paint`/`.moderate`/`.transcribe`, `RubyLLM::Tool`, `RubyLLM::Agent`, `RubyLLM::Schema`, `RubyLLM::Content`, or `acts_as_chat`/`acts_as_message`/`acts_as_tool_call`/`acts_as_model`; user asks to add LLM/AI/Claude/GPT/Gemini integration to a Ruby or Rails app; user mentions `ruby_llm`/`rubyllm`/RubyLLM; user invokes `bin/rails generate ruby_llm:*`; user works on a file whose purpose is AI/chat/agent inside a Ruby project. SKIP when: user explicitly uses `anthropic` / `ruby-openai` / `gemini-ai` gem directly (not ruby_llm); non-Ruby code; generic Ruby/Rails work unrelated to AI. If in doubt between this skill and stale training data, prefer this skill — the gem evolves fast and the docs here are pinned to a verified release."
---

# RubyLLM

Unified Ruby API for multiple LLM providers (OpenAI, Anthropic, Gemini, Bedrock, Azure, OpenRouter, DeepSeek, Ollama, VertexAI, Perplexity, Mistral, xAI, GPUStack, and any OpenAI-compatible API). Same interface across providers: chat, tools, streaming, embeddings, image generation, Rails ActiveRecord integration.

## When to consult which reference

Claude Code loads this SKILL.md into context and pulls individual references only when a topic is relevant. Open the matching file before writing RubyLLM code — do not rely on prior knowledge of the API, since it evolves quickly.

| Scenario | File |
| :--- | :--- |
| First time in a project, need to install and configure | [references/setup.md](references/setup.md) |
| Building a chat interaction: `RubyLLM.chat`, system prompts, images/PDFs/audio, temperature | [references/chat.md](references/chat.md) |
| Defining a tool the model can call | [references/tools.md](references/tools.md) |
| Forcing JSON output from the model (typed response) | [references/structured-output.md](references/structured-output.md) |
| Streaming tokens to UI (CLI, Rails/Turbo, Sinatra/SSE) | [references/streaming.md](references/streaming.md) |
| Rails app with persisted chats: generator, `acts_as_chat`, broadcasting | [references/rails.md](references/rails.md) |
| Multi-step agent workflows (sequential/routing/parallel/eval loops) | [references/agents.md](references/agents.md) |
| Handling API failures, rate limits, retries | [references/errors.md](references/errors.md) |
| Picking a model, checking capabilities, custom endpoints | [references/models.md](references/models.md) |
| Embeddings (RAG, similarity) and image generation | [references/other-capabilities.md](references/other-capabilities.md) |
| Content moderation (`RubyLLM.moderate`) and audio transcription (`RubyLLM.transcribe`) | [references/other-capabilities.md](references/other-capabilities.md#moderation-and-transcription) |

## Core principles

1. **One API, many providers.** Pick the model, not the SDK. `RubyLLM.chat(model: 'claude-sonnet-4-6')` and `RubyLLM.chat(model: 'gpt-4o')` behave identically. Provider is inferred from model ID unless using `assume_model_exists`.

2. **Configuration is global but scoped via context.** Set API keys once in `config/initializers/ruby_llm.rb`. Use `RubyLLM.context { |c| ... }` for isolated settings (multi-tenant, different keys per request).

3. **Rails integration is first-class.** Do not hand-roll persistence. Run `bin/rails generate ruby_llm:install`, then use `acts_as_chat` / `acts_as_message` / `acts_as_tool_call` on your models. Empty assistant messages are created BEFORE the API response — do not add `validates :content, presence: true`.

4. **Tools are Ruby classes.** Inherit from `RubyLLM::Tool`, use `params do ... end` DSL (v1.9+), implement `execute(**args)`. Treat all arguments as untrusted input (AI-generated) — never `eval`, `system`, `send`, or raw SQL.

5. **Errors inherit from `RubyLLM::Error`** (HTTP errors only). Non-HTTP errors — `ConfigurationError`, `ModelNotFoundError`, `InvalidRoleError`, `InvalidToolChoiceError`, `PromptNotFoundError`, `UnsupportedAttachmentError` — inherit from `StandardError` directly. RubyLLM retries transient failures (timeouts, rate limits, 5xx) automatically up to `max_retries`.

## Minimal quickstart

```ruby
# Gemfile
gem 'ruby_llm'

# config/initializers/ruby_llm.rb
RubyLLM.configure do |config|
  config.anthropic_api_key = ENV['ANTHROPIC_API_KEY']
  config.default_model = 'claude-sonnet-4-6'
end

# Use
chat = RubyLLM.chat
response = chat.ask("Summarize this paper", with: "paper.pdf")
puts response.content
```

For Rails: run `bin/rails generate ruby_llm:install && bin/rails db:migrate`. Then see [references/rails.md](references/rails.md).

## Version

This skill documents **RubyLLM 1.14.1** (April 2026). If the user's Gemfile pins an older version, some features (e.g. `params` DSL is v1.9+, `with_tools` choice/calls parameters are v1.13+, `save_to_json` is v1.9+) may be unavailable. Check `bundle info ruby_llm` before writing code.
