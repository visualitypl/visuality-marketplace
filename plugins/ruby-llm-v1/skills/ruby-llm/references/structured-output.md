# Structured Output

Force the model to return JSON matching a schema. Two routes: `RubyLLM::Schema` DSL (recommended) or manual JSON schema hash.

## RubyLLM::Schema DSL

```ruby
class PersonSchema < RubyLLM::Schema
  string  :name, description: "Person's full name"
  integer :age,  description: "Person's age in years"
  string  :city, required: false
end

chat = RubyLLM.chat
response = chat.with_schema(PersonSchema).ask("Generate a person named Alice")

puts response.content
# => {"name" => "Alice", "age" => 30}
```

`response.content` is a parsed `Hash`, not a JSON string.

## Schema primitives

```ruby
class TicketSchema < RubyLLM::Schema
  string  :title,       description: "Short headline", max_length: 120
  string  :description, description: "Full body"
  string  :email,       format: "email"
  integer :priority,    description: "1 (low) to 5 (urgent)", minimum: 1, maximum: 5
  number  :score,       multiple_of: 0.5
  boolean :billable
  null    :placeholder   # standalone null type, rarely used outside any_of

  array :tags, of: :string, min_items: 1, max_items: 20

  object :assignee do
    string  :email
    integer :employee_id
  end

  # Union types — lets a field accept multiple shapes (or be nullable):
  any_of :extra do
    string
    null
  end
end
```

Primitives: `string`, `integer`, `number`, `boolean`, `null`. Composites: `array of: :type` (or `do ... end` for array-of-objects), `object do ... end`, `any_of :name do ... end`, `one_of :name do ... end` (exactly one), `optional :name do ... end` (shortcut for `any_of` + `null`). Options: `enum:`, `required: false` (default `true`).

Per-type constraints (pass as kwargs):

| Type | Constraints |
| :--- | :--- |
| `string` | `min_length`, `max_length`, `pattern` (regex string), `format` (`"email"`, `"uri"`, `"date"`, `"date-time"`, `"uuid"`), `enum` |
| `integer` / `number` | `minimum`, `maximum`, `multiple_of` |
| `array` | `min_items`, `max_items`, `of:` |
| `any_of` / `one_of` | block with one sub-schema call per alternative |

## Composition and reuse

```ruby
class AddressSchema < RubyLLM::Schema
  string :street
  string :city
end

class CompanySchema < RubyLLM::Schema
  object :ceo, of: PersonSchema            # embed another schema
  array  :employees, of: PersonSchema
  object :address, of: AddressSchema
end
```

Inline reusable fragment via `define` (emits a `$defs` entry; reference via `of: :<name>`):

```ruby
class OrderSchema < RubyLLM::Schema
  define :location do
    string :latitude
    string :longitude
  end

  array  :shipping_stops, of: :location
  object :origin,         of: :location
end
```

`of: :root` references the enclosing schema itself (useful for recursive trees). `object :x, reference: :y` still works but emits a deprecation warning — use `of:`.

### Schema-level options

Set on the class with bare DSL calls:

```ruby
class TicketSchema < RubyLLM::Schema
  name "support_ticket"                 # overrides the auto-derived class name
  description "A user-submitted ticket" # becomes the schema description
  strict false                          # default: true (OpenAI strict mode)
  additional_properties true            # default: false

  string :title
end

TicketSchema.valid?    # run the internal validator
TicketSchema.validate! # raise on problems
```

## Factory form (no class)

```ruby
PersonSchema = RubyLLM::Schema.create do
  string :name
  integer :age
end
```

## Removing the schema mid-conversation

Pass `nil` to drop the active schema and return to free-form responses:

```ruby
chat.with_schema(PersonSchema).ask("Generate a person")
chat.with_schema(nil).ask("Now describe their career in prose")
```

## Manual JSON schema (when you need shapes the DSL doesn't cover)

```ruby
person_schema = {
  type: 'object',
  properties: {
    name:    { type: 'string' },
    age:     { type: 'integer' },
    hobbies: { type: 'array', items: { type: 'string' } }
  },
  required: ['name', 'age', 'hobbies'],
  additionalProperties: false
}

response = chat.with_schema(person_schema).ask("Generate a person")
```

## Tips

- **Front-load descriptions.** Each field's `description:` becomes part of the prompt. Vague descriptions → vague fills.
- **Use enums for controlled vocabulary.** `enum: %w[low medium high]` is more reliable than "pick one of: low, medium, high" in a description.
- **Avoid deep nesting.** Two levels at most if you can. Providers differ in how well they handle nested schemas.
- **`additionalProperties: false`** (manual schema) stops the model from inventing fields — useful when you're feeding the output into a strict consumer.

## When to use schemas vs. tools vs. `with_params`

| Need | Use |
| :--- | :--- |
| Typed JSON answer once | `with_schema` |
| Model calls your code mid-conversation | tool (see [tools.md](tools.md)) |
| OpenAI-specific `response_format` flag | `with_params(response_format: { type: 'json_object' })` — not portable |
