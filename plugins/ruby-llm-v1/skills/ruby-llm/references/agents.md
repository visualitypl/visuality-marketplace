# Agents & Workflows

An agent is a class with a pre-configured chat setup (model + instructions + optional tools/schema/temperature/thinking). A workflow is plain orchestration code that coordinates one or more agents.

## Defining an agent

```ruby
class ResearchAgent < RubyLLM::Agent
  model "gemini-3.1-pro-preview"
  instructions "Given a topic, return concise, reliable key facts."
end

# Use
facts = ResearchAgent.new.ask("quantum tunneling").content
```

`Agent.new` creates an internal `RubyLLM.chat` and delegates `.ask`/`.say`, `.with_tool(s)`, `.with_model`, `.with_temperature`, `.with_thinking`, `.with_context`, `.with_params`, `.with_headers`, `.with_schema`, `.on_*`, `.add_message`, `.complete`, `.reset_messages!`, and `.each` to it — so the agent instance behaves like a Chat. **Not delegated:** `with_instructions` (class-level DSL only) and `with_runtime_instructions` (use `SupportAgent.find` / `sync_instructions!` on Rails-persisted agents).

## Full class-level DSL

```ruby
class ResearchAgent < RubyLLM::Agent
  model "claude-sonnet-4-6"

  # Tools — splat of classes, splat of instances, or a block for dynamic lists
  tools WebSearch, ReadUrl
  tools { [WebSearch.new(api_key: ENV['SERP_KEY']), ReadUrl.new] }

  # Instructions — string, a block (evaluated with input access), or omit for the default prompt file
  instructions "Return concise, sourced facts. Cite URLs."

  # Parameters that get passed into instructions/tools at runtime
  inputs :topic, :depth

  temperature 0.3
  thinking effort: :low      # or budget: 5000
  schema ResearchSchema      # class, hash, or `schema do ... end` block
  params top_p: 0.9          # provider-specific; also accepts a block for runtime values
  headers 'anthropic-beta' => 'prompt-caching-2024-07-31'
  context RubyLLM.context { |c| c.request_timeout = 60 }
end

agent = ResearchAgent.new(topic: "quantum tunneling", depth: "deep")
agent.ask("Start research")
```

`schema`, `tools`, `params`, and `headers` also accept a block that's evaluated in the runtime context — useful when the value depends on an input:

```ruby
inputs :strict
schema do
  strict ? StrictResultSchema : LooseResultSchema
end
```

### ERB prompt files

If you call `instructions` with no args (or never declare it), RubyLLM looks for `app/prompts/<agent_name>/instructions.txt.erb` (snake-cased class name). The file is missing → `RubyLLM::PromptNotFoundError`. Inputs declared via `inputs :topic, :depth` are passed to ERB as locals:

```erb
<%# app/prompts/research_agent/instructions.txt.erb %>
You are researching <%= topic %> at <%= depth %> depth.
Return 5 concise facts with sources.
```

To use a **different** prompt file, render it from within a block — the block body runs in a runtime context that exposes `prompt(name, **locals)`, `chat`, and each declared input as a method:

```ruby
class ResearchAgent < RubyLLM::Agent
  model "claude-sonnet-4-6"
  inputs :topic

  instructions { prompt('summarize', focus_on: topic) }
  # reads app/prompts/research_agent/summarize.txt.erb with `focus_on` as a local
end
```

Same trick for string interpolation: `instructions { "Research #{topic} carefully." }`. The block is re-evaluated per `.chat` / `.new` call, so runtime inputs are honored.

## Rails-persisted agents

If the agent declares a `chat_model`, you can create/find persisted chat records — the agent's config is applied automatically on create and on find (though `find` uses runtime-only instructions so re-loading doesn't duplicate system rows):

```ruby
class SupportAgent < RubyLLM::Agent
  model "claude-sonnet-4-6"
  chat_model Chat                          # class or "Chat" string
  inputs :customer_name                    # applied only when you pass them in
  instructions { "You help #{customer_name}." }
end

record = SupportAgent.create!(user: current_user, customer_name: "Alice")
record.ask("my order is late")             # persists user + assistant rows

existing = SupportAgent.find(record.id, customer_name: "Alice")
SupportAgent.sync_instructions!(existing, customer_name: "Alice")  # rewrite system message
```

- `kwargs` are split: anything declared via `inputs` is an input value; everything else flows into the Rails `create!`/`create` call (e.g., `user: current_user`).
- `Agent.chat(**inputs)` returns a plain `RubyLLM::Chat` (no DB record) — use this when you just want an ephemeral, pre-configured chat object.

## Workflow patterns

### Sequential — output of A feeds B

```ruby
class ResearchWriterWorkflow
  def create_article(topic)
    research = ResearchAgent.new.ask(topic).content
    WriterAgent.new.ask(research).content
  end
end
```

### Routing — pick the specialist

```ruby
def agent_for(query)
  case classify(query)
  when :code     then CodeAgent
  when :creative then CreativeAgent
  when :factual  then FactualAgent
  else FactualAgent
  end
end

reply = agent_for(user_query).new.ask(user_query).content
```

### Parallel — independent analyses via `Async`

Requires the `async` gem. RubyLLM uses `Net::HTTP` under the hood, which cooperates with Ruby's Fiber scheduler — inside an `Async` block LLM calls automatically yield instead of blocking, so no explicit configuration is needed.

```ruby
require 'async'

results = Async do |task|
  sentiment = task.async { SentimentAgent.new.ask(text).content }
  summary   = task.async { SummaryAgent.new.ask(text).content }
  keywords  = task.async { KeywordAgent.new.ask(text).content }

  {
    sentiment: sentiment.wait,
    summary:   summary.wait,
    keywords:  keywords.wait
  }
end.wait
```

For Rails, wrap the `Async` block in a background job (Sidekiq/GoodJob/Solid Queue) so the web thread isn't blocked.

### Evaluation loop — critic refines the draft

```ruby
MAX_ROUNDS = 3

draft = WriterAgent.new.ask(task).content

MAX_ROUNDS.times do
  verdict, feedback = review(task: task, draft: draft)
  break if verdict == "pass"
  draft = revise(task: task, draft: draft, feedback: feedback)
end

draft
```

`review` and `revise` are methods that call CriticAgent and WriterAgent respectively. Cap the rounds — otherwise a stubborn critic can loop forever.

## Agents with tools

```ruby
class DBAnalystAgent < RubyLLM::Agent
  model "claude-sonnet-4-6"
  instructions <<~PROMPT
    You are a data analyst. Use the query tool to fetch data,
    then answer the user's question in one paragraph.
  PROMPT
end

agent = DBAnalystAgent.new
agent.with_tools(QueryDatabase.new(AnalyticsDB))
agent.ask("How many users signed up last week?")
```

Tool API is identical to regular chat — see [tools.md](tools.md).

## RAG (retrieval-augmented generation)

No built-in RAG, but the pattern is short:

```ruby
class RagAgent < RubyLLM::Agent
  model "claude-sonnet-4-6"
  instructions "Answer using only the provided context. If the context doesn't contain the answer, say so."

  def answer(question)
    query_embedding = RubyLLM.embed(question).vectors
    chunks = Document
      .nearest_neighbors(:embedding, query_embedding, distance: "cosine")
      .limit(5)

    context = chunks.map(&:content).join("\n\n---\n\n")
    ask("Context:\n#{context}\n\nQuestion: #{question}")
  end
end
```

Uses `pgvector` on the `Document` model. See [other-capabilities.md](other-capabilities.md) for the embedding side.

## When a workflow is overkill

If you need one prompt → one response, use `RubyLLM.chat` directly. Agents and workflows earn their complexity when:

- You have distinct responsibilities that need separate system prompts.
- You want to route different inputs to different models (cheap classifier + expensive specialist).
- You need a critic loop, a fan-out, or RAG.
