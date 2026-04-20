# Tools

Tools let the model call your Ruby code. Inherit from `RubyLLM::Tool`, declare params, implement `execute`.

## Minimal tool (v1.9+ `params` DSL)

```ruby
class Weather < RubyLLM::Tool
  description "Gets current weather for a location"

  params do
    string :latitude,  description: "Latitude (e.g., 52.5200)"
    string :longitude, description: "Longitude (e.g., 13.4050)"
  end

  def execute(latitude:, longitude:)
    url = "https://api.open-meteo.com/v1/forecast" \
          "?latitude=#{latitude}&longitude=#{longitude}" \
          "&current=temperature_2m,wind_speed_10m"
    response = Faraday.get(url)
    JSON.parse(response.body)
  rescue => e
    { error: e.message }
  end
end
```

Three required pieces:
1. `description` — what the tool does; the model reads this to decide when to call.
2. `params` — typed inputs the model must provide.
3. `execute(**args)` — your implementation; receives keyword args matching params.

## Tool name auto-derivation

The name the model sees is derived from your class name: camelCase is snake_cased, a trailing `_tool` is stripped. So:

| Class | Tool name (and `choice:` symbol) |
| :--- | :--- |
| `Weather` | `:weather` |
| `WeatherTool` | `:weather` (the `_tool` suffix is dropped) |
| `DocumentSearch` | `:document_search` |
| `My::NestedTool::SaveNote` | `:save_note` |

Override by defining an instance method `name` on the subclass if the derived value isn't what you want.

If the model invents a tool name that isn't registered, RubyLLM doesn't crash — it feeds the model an `error:` payload listing the available tools and lets the conversation continue so the model can self-correct.

## Attaching tools to a chat

```ruby
chat = RubyLLM.chat(model: 'gpt-4o')
chat.with_tool(Weather.new)

response = chat.ask("What's the weather in Berlin? (52.52, 13.40)")
puts response.content
```

Multiple tools:
```ruby
chat.with_tools(Weather, Calculator)
# instances and classes both work

chat.with_tools(OnlyThisTool, replace: true)  # drop previously attached tools
chat.with_tools(replace: true)                # no positional args = clear all tools
```

## `params` DSL — all supported shapes

```ruby
class MeetingSearch < RubyLLM::Tool
  description "Search available meeting slots"

  params do
    object :window, description: "Time window to reserve" do
      string :start,  description: "ISO8601 start time"
      string :finish, description: "ISO8601 end time"
    end

    array :participants, of: :string, description: "Email addresses"

    any_of :format, description: "Optional meeting format" do
      string enum: %w[virtual in_person]
      null
    end
  end

  def execute(window:, participants:, format: nil)
    # ...
  end
end
```

Primitives: `string`, `integer`, `number`, `boolean`, `null`. Composites: `object { ... }`, `array of: :type` (or with a block for array-of-object), `any_of :name do ... end`, `one_of :name do ... end` (exactly one matches), `optional :name do ... end` (shortcut for `any_of` + `null`). Options: `enum:`, `required: false`, plus per-type constraints (`min_length`/`max_length`/`pattern`/`format` on strings, `minimum`/`maximum`/`multiple_of` on numbers, `min_items`/`max_items` on arrays) — see [structured-output.md](structured-output.md#schema-primitives).

## `param` helper (simpler, flatter)

```ruby
class Translate < RubyLLM::Tool
  description "Translate text between languages"

  param :text,        desc: "Text to translate"
  param :target_lang, desc: "Target language code (e.g., 'fr')"
  param :units,       type: :string, desc: "Unit system", required: false

  def execute(text:, target_lang:, units: nil)
    # ...
  end
end
```

## Manual JSON schema (last resort)

```ruby
class Lookup < RubyLLM::Tool
  description "Look up product info"

  params type: "object",
    properties: {
      sku:    { type: "string", description: "Product SKU" },
      locale: { type: "string", description: "Country code", default: "US" }
    },
    required: %w[sku]

  def execute(sku:, locale: "US")
    # ...
  end
end
```

## Custom initializer (dependency injection)

```ruby
class DocumentSearch < RubyLLM::Tool
  description "Search the documentation index"
  params do
    string  :query,  description: "Search query"
    integer :limit,  description: "Max results", required: false
  end

  def initialize(database)
    @database = database
  end

  def execute(query:, limit: 5)
    @database.search(query, limit: limit)
  end
end

chat.with_tool(DocumentSearch.new(MyDatabase))
```

## Tool call controls (v1.13+)

```ruby
# Let the model choose whether to call (default)
chat.with_tools(Weather, Calculator, choice: :auto)

# Require a tool call in the next response
chat.with_tools(Weather, Calculator, choice: :required)

# Disable all tools without removing them
chat.with_tools(Weather, Calculator, choice: :none)

# Force a specific tool (symbol or class)
chat.with_tools(Weather, Calculator, choice: :weather)
chat.with_tools(Weather, Calculator, choice: Weather)

# Limit calls per turn
chat.with_tools(Weather, Calculator, calls: :many)  # default
chat.with_tools(Weather, Calculator, calls: :one)   # single call — `calls: 1` also accepted
```

After a forced `choice:` (anything other than `:auto` / `:none`) executes, RubyLLM automatically resets the choice to `nil` so the next turn is back to provider default — otherwise the model would be stuck calling the same tool forever.

## Returning rich content

Return `RubyLLM::Content` to include file attachments (e.g., generated charts):

```ruby
def execute(query:)
  chart_path = generate_chart(query)
  RubyLLM::Content.new(
    "Analysis complete for: #{query}",
    [chart_path]
  )
end
```

## Provider-specific tool metadata (`with_params` at class level)

Some providers accept extra metadata per tool (Anthropic's `cache_control`, for example). Declare once on the class — RubyLLM merges the hash into the provider payload; providers that don't recognize the keys ignore them.

```ruby
class ChangelogTool < RubyLLM::Tool
  description "Formats commits into changelog entries."
  params { array :commits, of: :string }

  with_params cache_control: { type: 'ephemeral' }   # class-level

  def execute(commits:)
    # ...
  end
end
```

This is distinct from `chat.with_params(...)` (instance-level, applies to the whole request).

## `halt` — skip model follow-up

By default, after a tool executes, the result goes back to the model for a natural-language response. `halt` returns the tool's output directly as the final answer:

```ruby
def execute(path:, content:)
  File.write(path, content)
  halt "Saved to #{path}"
end
```

Use when the tool's output IS the answer and further LLM commentary adds cost/latency without value.

## Error handling

**Recoverable** (model can adjust): return an error hash.
```ruby
def execute(location:)
  return { error: "Location must be at least 3 chars" } if location.length < 3
  # ...
rescue Faraday::ConnectionFailed
  { error: "Weather service unavailable" }
end
```

**Unrecoverable** (application bug): raise. Wrap the full `chat.ask` call in your own `rescue`.

## Security (critical)

Tool arguments come from the LLM — treat them like raw user input from the internet.

- **Never** `eval`, `system`, `send`, `public_send` on strings containing params.
- **Never** interpolate params into SQL. Use parameterized queries.
- **Always** validate types, ranges, and formats.
- **Sanitize** strings before filesystem / shell / URL use.
- **Apply least privilege** — if the tool only needs read access, give it read access.

Example — a file-writing tool that is safe:

```ruby
class SaveNote < RubyLLM::Tool
  description "Save a note under notes/"
  params do
    string :slug,    description: "Lowercase filename, no slashes"
    string :content, description: "Markdown body"
  end

  def execute(slug:, content:)
    unless slug.match?(/\A[a-z0-9\-]+\z/)
      return { error: "slug must be [a-z0-9-] only" }
    end

    path = Rails.root.join("notes", "#{slug}.md")
    File.write(path, content)
    halt "Saved: #{path}"
  end
end
```
