# Embeddings & Image Generation

Non-chat capabilities. Shorter than chat because the APIs are slim.

---

## Embeddings

### Single text

```ruby
embedding = RubyLLM.embed("Ruby is a programmer's best friend")
vector = embedding.vectors            # => [Float, Float, ...]
dim    = vector.length                # => e.g. 1536
```

### Batch (preferred for multiple texts)

```ruby
texts = ["Ruby", "Python", "JavaScript"]
embeddings = RubyLLM.embed(texts)
embeddings.vectors.length    # => 3
```

Batching is cheaper and faster than sequential single calls.

### Model & dimensions

```ruby
# Pick a specific model
RubyLLM.embed("sentence", model: "text-embedding-3-large")

# Request reduced dimensions (where supported)
RubyLLM.embed("sentence", model: "text-embedding-3-small", dimensions: 512)
```

Set defaults globally:
```ruby
config.default_embedding_model = 'text-embedding-3-large'
```

### Cosine similarity

```ruby
require 'matrix'

v1 = Vector.elements(embedding1.vectors)
v2 = Vector.elements(embedding2.vectors)

similarity = v1.inner_product(v2) / (v1.norm * v2.norm)
```

### Rails + pgvector

```ruby
# Gemfile: gem 'neighbor'
class Document < ApplicationRecord
  has_neighbors :embedding

  before_save :generate_embedding, if: :content_changed?

  private

  def generate_embedding
    self.embedding = RubyLLM.embed(content).vectors
  end
end

# Nearest neighbour query
Document.nearest_neighbors(:embedding, query_vector, distance: "cosine").limit(5)
```

---

## Image generation

### Basic call

```ruby
image = RubyLLM.paint("A photorealistic red panda coding Ruby")

image.url         # URL (OpenAI / DALL-E) — nil for base64 providers
image.data        # Base64 string (Google Imagen) — nil for URL providers
image.base64?     # which path is this?
image.save("panda.png")
image.to_blob     # raw binary string
```

### Model & size

```ruby
image = RubyLLM.paint("prompt", model: "dall-e-3", size: "1024x1024")
image = RubyLLM.paint("prompt", model: "gpt-image-1.5", size: "1792x1024")  # landscape
image = RubyLLM.paint("prompt", model: "gpt-image-1.5", size: "1024x1792")  # portrait
```

Set default:
```ruby
config.default_image_model = 'dall-e-3'
```

### Image object attributes

```ruby
image.url             # URL string (OpenAI / DALL-E)
image.data            # Base64 string (if provider returns inline)
image.mime_type       # e.g. "image/png"
image.revised_prompt  # provider-revised prompt (OpenAI)
image.model_id
```

Image editing is **not supported** by `RubyLLM.paint` — its keyword list is `prompt, model:, provider:, assume_model_exists:, size:, context:`, and it only calls the text-to-image endpoint. For OpenAI's `images/edits` endpoint, use Faraday directly or a dedicated gem.

### Rails + ActiveStorage

```ruby
io = StringIO.new(image.to_blob)
product.generated_image.attach(io: io, filename: "hero.png", content_type: "image/png")
```

### Performance

Image generation typically takes 5–20 seconds. Do not do it inline on a web request — push to a background job (Solid Queue, Sidekiq, GoodJob).

```ruby
class GenerateHeroImageJob < ApplicationJob
  def perform(product_id, prompt)
    product = Product.find(product_id)
    image   = RubyLLM.paint(prompt)
    io      = StringIO.new(image.to_blob)
    product.generated_image.attach(io: io, filename: "hero.png")
  end
end
```

---

## Moderation and transcription

Two more capabilities that don't need a full chat: content moderation and audio transcription.

### Moderation

```ruby
result = RubyLLM.moderate("some user-submitted text")
result.id                  # OpenAI moderation id
result.model               # => e.g. "omni-moderation-latest"
result.results             # Array of Hashes — each has 'flagged' (bool), 'categories' (Hash), 'category_scores' (Hash)
result.flagged?            # => true if any entry is flagged
result.flagged_categories  # => e.g. ["harassment", "violence"]
result.category_scores     # first entry's score hash
result.categories          # first entry's category flags
```

Default model: `config.default_moderation_model` (falls back to `'omni-moderation-latest'`). OpenAI-compatible providers.

### Transcription

```ruby
transcription = RubyLLM.transcribe("meeting.mp3")
transcription.text          # => "So the Q3 roadmap..."
transcription.language      # language code if provider returned one
transcription.duration      # seconds
transcription.segments      # provider-specific segments array
transcription.input_tokens  # usage (nil unless the provider reports it)
transcription.output_tokens
```

Default model: `config.default_transcription_model` (falls back to `'whisper-1'`). Supported for providers with an audio endpoint (OpenAI, VertexAI, Gemini).

```ruby
RubyLLM.transcribe("fr_audio.mp3", language: "fr", model: "whisper-1")
RubyLLM.transcribe("clip.wav", provider: :vertexai, assume_model_exists: true)
```

Any extra kwargs beyond `model:`/`provider:`/`language:`/`assume_model_exists:`/`context:` pass through to the provider's transcribe call.
