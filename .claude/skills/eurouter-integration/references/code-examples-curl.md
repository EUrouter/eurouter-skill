# EUrouter curl Examples

Base URL: `https://api.eurouter.ai`

Replace `eur_YOUR_KEY_HERE` with your actual API key.

---

## 1. Check Credits (Auth Test)

```bash
curl https://api.eurouter.ai/api/v1/credits \
  -H "x-api-key: eur_YOUR_KEY_HERE"
```

Response:
```json
{ "data": { "total_credits": 100.0, "total_usage": 45.32 } }
```

---

## 2. Basic Chat Completion

```bash
curl https://api.eurouter.ai/api/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "x-api-key: eur_YOUR_KEY_HERE" \
  -d '{
    "model": "openai/gpt-4o",
    "messages": [
      {"role": "user", "content": "What is GDPR?"}
    ]
  }'
```

---

## 3. EU-Restricted Chat Completion

```bash
curl https://api.eurouter.ai/api/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "x-api-key: eur_YOUR_KEY_HERE" \
  -d '{
    "model": "openai/gpt-4o",
    "messages": [
      {"role": "system", "content": "You are a GDPR compliance assistant."},
      {"role": "user", "content": "Explain data subject rights."}
    ],
    "max_completion_tokens": 1024,
    "temperature": 0.7,
    "provider": {
      "data_residency": "eu",
      "eu_owned": true,
      "data_collection": "deny",
      "max_retention_days": 0
    }
  }'
```

---

## 4. Streaming Chat Completion

```bash
curl -N https://api.eurouter.ai/api/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "x-api-key: eur_YOUR_KEY_HERE" \
  -d '{
    "model": "anthropic/claude-3-5-sonnet",
    "messages": [
      {"role": "user", "content": "Write a haiku about the EU."}
    ],
    "stream": true,
    "stream_options": {"include_usage": true},
    "provider": {
      "data_residency": "eu"
    }
  }'
```

The `-N` flag disables output buffering so you see chunks as they arrive.

---

## 5. Embeddings

```bash
curl https://api.eurouter.ai/api/v1/embeddings \
  -H "Content-Type: application/json" \
  -H "x-api-key: eur_YOUR_KEY_HERE" \
  -d '{
    "model": "openai/text-embedding-3-small",
    "input": ["GDPR compliance is important for EU businesses."],
    "provider": {
      "data_residency": "eu"
    }
  }'
```

---

## 6. List Models (Filtered by Provider)

```bash
# No auth required for model listing
curl "https://api.eurouter.ai/api/v1/models?provider=scaleway"
```

Filter by supported parameters:
```bash
curl "https://api.eurouter.ai/api/v1/models?supported_parameters=tools,vision"
```

---

## 7. Create a Routing Rule

```bash
curl https://api.eurouter.ai/api/v1/routing-rules \
  -H "Content-Type: application/json" \
  -H "x-api-key: eur_YOUR_KEY_HERE" \
  -d '{
    "name": "gdpr-strict",
    "description": "Strict GDPR: EU owned, no training, zero retention",
    "provider": {
      "eu_owned": true,
      "data_collection": "deny",
      "max_retention_days": 0,
      "data_residency": "eu"
    }
  }'
```

Then use it in requests:
```bash
curl https://api.eurouter.ai/api/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "x-api-key: eur_YOUR_KEY_HERE" \
  -d '{
    "model": "openai/gpt-4o",
    "rule_name": "gdpr-strict",
    "messages": [{"role": "user", "content": "Hello!"}]
  }'
```

---

## 8. Dry-Run Test

Preview which providers satisfy a routing configuration:

```bash
curl https://api.eurouter.ai/api/v1/routing-rules/dry-run \
  -H "Content-Type: application/json" \
  -H "x-api-key: eur_YOUR_KEY_HERE" \
  -d '{
    "models": ["openai/gpt-4o", "anthropic/claude-3-5-sonnet"],
    "preference_overrides": {
      "data_residency": "eu",
      "eu_owned": true
    }
  }'
```

---

## 9. List Providers

```bash
# No auth required
curl https://api.eurouter.ai/api/v1/providers
```

---

## Response Headers

Every generation request returns routing transparency headers. View them with `-i`:

```bash
curl -i https://api.eurouter.ai/api/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "x-api-key: eur_YOUR_KEY_HERE" \
  -d '{
    "model": "openai/gpt-4o",
    "messages": [{"role": "user", "content": "Hello"}],
    "provider": {"data_residency": "eu"}
  }'
```

Look for these headers:
```
x-provider-slug: scaleway
x-model-used: openai/gpt-4o
x-routing-strategy: lowest-cost
x-fallback-count: 0
```
