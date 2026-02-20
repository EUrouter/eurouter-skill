# EUrouter API Endpoints Reference

Base URL: `https://api.eurouter.ai`

---

## POST /api/v1/chat/completions

OpenAI-compatible chat completions with streaming, tool calling, vision, audio, and reasoning support.

### Request Body

```json
{
  "model": "gpt-4o",
  "models": ["gpt-4o", "claude-3-5-sonnet"],
  "rule_id": "uuid",
  "rule_name": "my-rule",
  "messages": [
    { "role": "system", "content": "You are a helpful assistant." },
    { "role": "user", "content": "Hello!" }
  ],
  "stream": false,
  "stream_options": { "include_usage": true },
  "max_completion_tokens": 4096,
  "temperature": 1,
  "top_p": 1,
  "frequency_penalty": 0,
  "presence_penalty": 0,
  "logit_bias": {},
  "logprobs": false,
  "top_logprobs": null,
  "seed": null,
  "stop": null,
  "reasoning": { "effort": "medium", "summary": "auto" },
  "response_format": { "type": "text" },
  "tool_choice": "auto",
  "tools": [],
  "user": "user-id",
  "metadata": {},
  "provider": {}
}
```

### Field Reference

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `model` | `string` | No* | Model ID (e.g., `"gpt-4o"`). *Required unless `models`, `rule_id`, or `rule_name` provides it. |
| `models` | `string[]` | No | Fallback model list — tries each in order |
| `rule_id` | `string (uuid)` | No | Routing rule ID to apply |
| `rule_name` | `string` | No | Routing rule name to apply |
| `messages` | `Message[]` | Yes | Conversation messages (min 1) |
| `stream` | `boolean \| null` | No | Enable SSE streaming (default: false) |
| `stream_options` | `object \| null` | No | `{ include_usage: boolean }` — include usage in final stream chunk |
| `max_tokens` | `number \| null` | No | Max tokens (deprecated — use `max_completion_tokens`) |
| `max_completion_tokens` | `number \| null` | No | Maximum tokens to generate (min: 1) |
| `temperature` | `number \| null` | No | Sampling temperature 0–2 (default: 1) |
| `top_p` | `number \| null` | No | Nucleus sampling 0–1 (default: 1) |
| `frequency_penalty` | `number \| null` | No | -2 to 2 (default: 0) |
| `presence_penalty` | `number \| null` | No | -2 to 2 (default: 0) |
| `logit_bias` | `Record<string, number> \| null` | No | Token logit biases |
| `logprobs` | `boolean \| null` | No | Return log probabilities |
| `top_logprobs` | `number \| null` | No | Number of top logprobs (0–20) |
| `seed` | `number \| null` | No | Random seed for deterministic output |
| `stop` | `string \| string[] \| null` | No | Stop sequences |
| `reasoning` | `object` | No | `{ effort, summary }` — for reasoning models |
| `response_format` | `object` | No | Response format specification |
| `tool_choice` | `string \| object` | No | `"none"`, `"auto"`, `"required"`, or `{ type: "function", function: { name } }` |
| `tools` | `Tool[]` | No | Available tools for the model |
| `user` | `string` | No | End-user identifier for abuse detection |
| `metadata` | `Record<string, string>` | No | Custom metadata |
| `provider` | `ProviderPreferences` | No | Provider routing preferences (see provider-preferences.md) |

### Message Types

**System message:**
```json
{ "role": "system", "content": "You are a helpful assistant.", "name": "optional-name" }
```

**User message (text):**
```json
{ "role": "user", "content": "What is GDPR?" }
```

**User message (vision — image URL):**
```json
{
  "role": "user",
  "content": [
    { "type": "text", "text": "What's in this image?" },
    { "type": "image_url", "image_url": { "url": "https://example.com/image.png", "detail": "auto" } }
  ]
}
```

**User message (audio):**
```json
{
  "role": "user",
  "content": [
    { "type": "input_audio", "input_audio": { "data": "<base64>", "format": "wav" } }
  ]
}
```
Supported audio formats: `wav`, `mp3`, `flac`, `m4a`, `ogg`, `pcm16`, `pcm24`

**Developer message:**
```json
{ "role": "developer", "content": "Always respond in French." }
```

**Assistant message (with tool calls):**
```json
{
  "role": "assistant",
  "content": null,
  "tool_calls": [
    {
      "id": "call_abc123",
      "type": "function",
      "function": { "name": "get_weather", "arguments": "{\"location\": \"Paris\"}" }
    }
  ]
}
```

**Tool response:**
```json
{ "role": "tool", "tool_call_id": "call_abc123", "content": "{\"temperature\": 18}" }
```

### Tool Definition

```json
{
  "type": "function",
  "function": {
    "name": "get_weather",
    "description": "Get the current weather for a location",
    "parameters": {
      "type": "object",
      "properties": {
        "location": { "type": "string", "description": "City name" }
      },
      "required": ["location"]
    },
    "strict": true
  }
}
```

### Reasoning Parameters

For reasoning models (e.g., `o1`, `claude-3-5-sonnet` with extended thinking):

```json
{
  "reasoning": {
    "effort": "medium",
    "summary": "auto"
  }
}
```

- `effort`: `"none"`, `"minimal"`, `"low"`, `"medium"`, `"high"`
- `summary`: `"auto"`, `"concise"`, `"detailed"`

### Response Format Options

```json
// Plain text (default)
{ "type": "text" }

// JSON object
{ "type": "json_object" }

// Structured JSON with schema
{
  "type": "json_schema",
  "json_schema": {
    "name": "my_schema",
    "description": "Description",
    "schema": { "type": "object", "properties": { ... } },
    "strict": true
  }
}

// GBNF grammar
{ "type": "grammar", "grammar": "root ::= ..." }

// Python code
{ "type": "python" }
```

### Response Shape

```json
{
  "id": "chatcmpl-abc123",
  "object": "chat.completion",
  "created": 1700000000,
  "model": "gpt-4o",
  "system_fingerprint": "fp_abc123",
  "choices": [
    {
      "index": 0,
      "message": {
        "role": "assistant",
        "content": "Hello! How can I help you today?"
      },
      "finish_reason": "stop",
      "logprobs": null
    }
  ],
  "usage": {
    "prompt_tokens": 10,
    "completion_tokens": 15,
    "total_tokens": 25,
    "completion_tokens_details": {
      "reasoning_tokens": 0,
      "audio_tokens": 0
    },
    "prompt_tokens_details": {
      "cached_tokens": 0,
      "audio_tokens": 0
    }
  }
}
```

Finish reasons: `"stop"`, `"length"`, `"tool_calls"`, `"content_filter"`, `"error"`

### Streaming Chunk Shape

```json
{
  "id": "chatcmpl-abc123",
  "object": "chat.completion.chunk",
  "created": 1700000000,
  "model": "gpt-4o",
  "choices": [
    {
      "index": 0,
      "delta": { "role": "assistant", "content": "Hello" },
      "finish_reason": null
    }
  ]
}
```

The final chunk (with `stream_options.include_usage: true`) includes a `usage` object.

---

## POST /api/v1/completions

Legacy text completion endpoint (prompt in, text out).

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `model` | `string` | Yes | Model ID (e.g., `"gpt-3.5-turbo-instruct"`) |
| `prompt` | `string` | Yes | The text prompt |
| `models` | `string[]` | No | Fallback model list |
| `provider` | `ProviderPreferences` | No | Provider routing preferences |
| `stream` | `boolean \| null` | No | Enable streaming |
| `max_tokens` | `number \| null` | No | Max tokens (min: 1) |
| `temperature` | `number \| null` | No | 0–2 (default: 1) |
| `top_p` | `number \| null` | No | 0–1 (default: 1) |
| `top_k` | `number \| null` | No | Top-k sampling (0 = disabled) |
| `frequency_penalty` | `number \| null` | No | -2 to 2 |
| `presence_penalty` | `number \| null` | No | -2 to 2 |
| `repetition_penalty` | `number \| null` | No | Repetition penalty multiplier |
| `stop` | `string \| string[] \| null` | No | Stop sequences |
| `seed` | `number \| null` | No | Random seed |
| `min_p` | `number \| null` | No | Min probability threshold 0–1 |
| `top_a` | `number \| null` | No | Top-a sampling 0–1 |

### Response Shape

```json
{
  "id": "cmpl-abc123",
  "object": "text_completion",
  "created": 1700000000,
  "model": "gpt-3.5-turbo-instruct",
  "choices": [
    {
      "text": "Generated text here...",
      "index": 0,
      "finish_reason": "stop"
    }
  ],
  "usage": {
    "prompt_tokens": 5,
    "completion_tokens": 20,
    "total_tokens": 25
  }
}
```

---

## POST /api/v1/embeddings

OpenAI-compatible embeddings endpoint.

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `model` | `string` | Yes | Model ID (e.g., `"text-embedding-3-small"`) |
| `input` | `string \| string[]` | Yes | Text to embed (single string or array) |
| `encoding_format` | `"float" \| "base64"` | No | Output format (default: `"float"`) |
| `dimensions` | `number` | No | Output dimensions (only some models support this) |
| `user` | `string` | No | End-user identifier |
| `provider` | `ProviderPreferences` | No | Provider routing preferences |

**Note:** Model fallback via `models` array is NOT supported for embeddings (vector dimensions differ between models). Only provider-level fallback (same model, different provider) is supported.

### Response Shape

```json
{
  "object": "list",
  "data": [
    {
      "object": "embedding",
      "embedding": [0.0023, -0.0091, 0.0151, ...],
      "index": 0
    }
  ],
  "model": "text-embedding-3-small",
  "usage": {
    "prompt_tokens": 8,
    "total_tokens": 8
  }
}
```

---

## POST /api/v1/responses

OpenAI Responses API compatible endpoint. Uses `input` instead of `messages` and supports additional tool types.

Key differences from chat completions:
- Uses `input` (string or array of items) instead of `messages`
- Uses `instructions` instead of system messages
- Supports `previous_response_id` for conversation continuity
- Additional tool types: `web_search_preview`, `file_search`, `code_interpreter`, `computer_use_preview`
- Streaming emits typed events (`response.created`, `response.output_text.delta`, `response.completed`, etc.)

Supports the same `provider`, `models`, `rule_id`, and `rule_name` fields as chat completions.

---

## GET /api/v1/models

List available models. **No authentication required.**

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `category` | `string` | Filter by model category |
| `supported_parameters` | `string` | Filter by supported params (comma-separated, e.g., `"tools,vision"`) |
| `provider` | `string` | Filter by provider slug (e.g., `"scaleway"`) |

### Response Shape

```json
{
  "data": [
    {
      "id": "gpt-4o",
      "name": "GPT-4o",
      "description": "...",
      "context_length": 128000,
      "pricing": {
        "prompt": "0.0000025",
        "completion": "0.00001",
        "request": "0",
        "image": "0",
        "currency": "USD",
        "discount": 0
      },
      "architecture": {
        "modality": "multimodal",
        "input_modalities": ["text", "image", "audio"],
        "output_modalities": ["text"],
        "tokenizer": "cl100k_base",
        "instruct_type": null
      },
      "top_provider": {
        "context_length": 128000,
        "max_completion_tokens": 16384,
        "is_moderated": true
      },
      "supported_parameters": ["temperature", "top_p", "tools", "stream", ...],
      "supported_api_endpoints": ["chat.completions"],
      "providers": [
        { "name": "OpenAI", "slug": "openai" },
        { "name": "Scaleway", "slug": "scaleway" }
      ]
    }
  ]
}
```

**Note:** Prices are strings (not floats) to avoid floating-point precision issues.

---

## GET /api/v1/credits

Check credit balance. Requires authentication.

### Response Shape

```json
{
  "data": {
    "total_credits": 100.0,
    "total_usage": 45.32
  }
}
```

Credits are in **EUR**.

---

## API Key Management

### GET /api/v1/keys

List API keys. Requires `keys:read` permission.

Query params: `offset` (pagination), `include_disabled` (boolean).

### POST /api/v1/keys

Create an API key. Requires `keys:write` permission.

```json
{
  "name": "Production Server",
  "limit": 100.0,
  "limit_reset": "monthly"
}
```

Response includes a `key` field with the full API key — **shown only once**.

### GET /api/v1/keys/:hash

Get a key by its public identifier. Requires `keys:read`.

### PATCH /api/v1/keys/:hash

Update a key. Requires `keys:write`.

```json
{
  "name": "New Name",
  "disabled": false,
  "limit": 200.0,
  "limit_reset": "weekly"
}
```

### DELETE /api/v1/keys/:hash

Delete a key. Requires `keys:write`.

### Key Object Shape

```json
{
  "name": "Production Server",
  "label": "ABC123456789",
  "hash": "ABC123456789",
  "limit": 100.0,
  "limit_remaining": 54.68,
  "limit_reset": "monthly",
  "usage": 45.32,
  "usage_daily": 0,
  "usage_weekly": 0,
  "usage_monthly": 45.32,
  "disabled": false,
  "created_at": "2025-01-15T10:30:00.000Z",
  "updated_at": "2025-01-15T10:30:00.000Z",
  "last_used": "2025-02-01T14:00:00.000Z"
}
```

`limit_reset` values: `"daily"`, `"weekly"`, `"monthly"`, or `null` (no reset).
