# EUrouter Python Code Examples

All examples use the official `openai` Python SDK. Install with `pip install openai`.

---

## 1. Basic Setup (Drop-in Migration)

```python
from openai import OpenAI

# Only base_url and api_key change from standard OpenAI usage
client = OpenAI(
    base_url="https://api.eurouter.ai/api/v1",
    api_key="eur_YOUR_KEY_HERE",
)

response = client.chat.completions.create(
    model="openai/gpt-4o",  # Models use provider/model-name format
    messages=[{"role": "user", "content": "What is GDPR?"}],
)

print(response.choices[0].message.content)
```

---

## 2. EU-Compliant Chat Completion

```python
from openai import OpenAI

client = OpenAI(
    base_url="https://api.eurouter.ai/api/v1",
    api_key="eur_YOUR_KEY_HERE",
)

response = client.chat.completions.create(
    model="openai/gpt-4o",
    messages=[
        {"role": "system", "content": "You are a GDPR compliance assistant."},
        {"role": "user", "content": "Explain data subject rights under GDPR."},
    ],
    max_completion_tokens=1024,
    temperature=0.7,
    # EUrouter-specific: enforce EU data residency
    extra_body={
        "provider": {
            "data_residency": "eu",
            "eu_owned": True,
            "data_collection": "deny",
            "max_retention_days": 0,
        }
    },
)

print(response.choices[0].message.content)

# Check which provider handled the request
print(f"Provider: {response.headers.get('x-provider-slug')}")
print(f"Model used: {response.headers.get('x-model-used')}")
```

---

## 3. Streaming with EU Compliance

```python
from openai import OpenAI

client = OpenAI(
    base_url="https://api.eurouter.ai/api/v1",
    api_key="eur_YOUR_KEY_HERE",
)

stream = client.chat.completions.create(
    model="anthropic/claude-3-5-sonnet",
    messages=[{"role": "user", "content": "Write a poem about Europe."}],
    stream=True,
    stream_options={"include_usage": True},
    extra_body={
        "provider": {
            "data_residency": "eu",
        }
    },
)

for chunk in stream:
    if chunk.choices and chunk.choices[0].delta.content:
        print(chunk.choices[0].delta.content, end="", flush=True)

    # Usage is in the final chunk
    if chunk.usage:
        print(f"\n\nTokens used: {chunk.usage.total_tokens}")
```

---

## 4. Tool Calling

```python
import json
from openai import OpenAI

client = OpenAI(
    base_url="https://api.eurouter.ai/api/v1",
    api_key="eur_YOUR_KEY_HERE",
)

tools = [
    {
        "type": "function",
        "function": {
            "name": "get_weather",
            "description": "Get the current weather for a location",
            "parameters": {
                "type": "object",
                "properties": {
                    "location": {
                        "type": "string",
                        "description": "City name, e.g. 'Berlin'",
                    }
                },
                "required": ["location"],
            },
        },
    }
]

# First call: model decides to use the tool
response = client.chat.completions.create(
    model="openai/gpt-4o",
    messages=[{"role": "user", "content": "What's the weather in Berlin?"}],
    tools=tools,
    tool_choice="auto",
    extra_body={"provider": {"data_residency": "eu"}},
)

message = response.choices[0].message

if message.tool_calls:
    tool_call = message.tool_calls[0]
    args = json.loads(tool_call.function.arguments)
    print(f"Model wants to call: {tool_call.function.name}({args})")

    # Simulate tool response
    weather_data = {"temperature": 18, "condition": "partly cloudy"}

    # Second call: send tool result back
    follow_up = client.chat.completions.create(
        model="openai/gpt-4o",
        messages=[
            {"role": "user", "content": "What's the weather in Berlin?"},
            message.model_dump(),
            {
                "role": "tool",
                "tool_call_id": tool_call.id,
                "content": json.dumps(weather_data),
            },
        ],
        tools=tools,
        extra_body={"provider": {"data_residency": "eu"}},
    )
    print(follow_up.choices[0].message.content)
```

---

## 5. Vision (Image Input)

```python
from openai import OpenAI

client = OpenAI(
    base_url="https://api.eurouter.ai/api/v1",
    api_key="eur_YOUR_KEY_HERE",
)

response = client.chat.completions.create(
    model="openai/gpt-4o",
    messages=[
        {
            "role": "user",
            "content": [
                {"type": "text", "text": "What's in this image?"},
                {
                    "type": "image_url",
                    "image_url": {
                        "url": "https://upload.wikimedia.org/wikipedia/commons/thumb/b/b7/Flag_of_Europe.svg/1200px-Flag_of_Europe.svg.png",
                        "detail": "auto",
                    },
                },
            ],
        }
    ],
    max_completion_tokens=512,
    extra_body={"provider": {"data_residency": "eu"}},
)

print(response.choices[0].message.content)
```

---

## 6. Embeddings

```python
from openai import OpenAI

client = OpenAI(
    base_url="https://api.eurouter.ai/api/v1",
    api_key="eur_YOUR_KEY_HERE",
)

response = client.embeddings.create(
    model="openai/text-embedding-3-small",
    input=["GDPR compliance is important for EU businesses."],
    # Note: model fallback (models array) is NOT supported for embeddings
    extra_body={
        "provider": {
            "data_residency": "eu",
        }
    },
)

embedding = response.data[0].embedding
print(f"Embedding dimensions: {len(embedding)}")
print(f"First 5 values: {embedding[:5]}")
```

---

## 7. Multi-Model Fallback

```python
from openai import OpenAI

client = OpenAI(
    base_url="https://api.eurouter.ai/api/v1",
    api_key="eur_YOUR_KEY_HERE",
)

# If the first model's providers all fail, EUrouter tries the next model
response = client.chat.completions.create(
    model="anthropic/claude-3-5-sonnet",  # Primary model
    messages=[{"role": "user", "content": "Explain quantum computing simply."}],
    extra_body={
        "models": [
            "anthropic/claude-3-5-sonnet",
            "openai/gpt-4o",
            "mistral/mistral-large",
        ],
        "provider": {
            "data_residency": "eu",
        },
    },
)

print(response.choices[0].message.content)
# Check if fallback occurred
print(f"Model used: {response.headers.get('x-model-used')}")
fallback_count = response.headers.get("x-model-fallback-count")
if fallback_count:
    print(f"Model fallbacks: {fallback_count}")
```

---

## 8. Using a Routing Rule by Name

```python
from openai import OpenAI

client = OpenAI(
    base_url="https://api.eurouter.ai/api/v1",
    api_key="eur_YOUR_KEY_HERE",
)

# Reference a pre-configured routing rule by name
# (Create routing rules via POST /api/v1/routing-rules)
response = client.chat.completions.create(
    model="openai/gpt-4o",
    messages=[{"role": "user", "content": "Hello!"}],
    extra_body={
        "rule_name": "gdpr-strict",  # Your routing rule name
    },
)

print(response.choices[0].message.content)
```

---

## 9. Listing Models Filtered by Provider

```python
import httpx

# Model listing is public — no auth required
response = httpx.get(
    "https://api.eurouter.ai/api/v1/models",
    params={"provider": "scaleway"},
)
response.raise_for_status()

models = response.json()["data"]
for model in models:
    print(f"{model['id']} — {model['name']} (context: {model['context_length']})")
```

---

## 10. Creating a Routing Rule

```python
import httpx

response = httpx.post(
    "https://api.eurouter.ai/api/v1/routing-rules",
    headers={"x-api-key": "eur_YOUR_KEY_HERE"},
    json={
        "name": "production-eu",
        "description": "Production routing: EU only, cheapest price",
        "model": "openai/gpt-4o",
        "models": ["openai/gpt-4o", "anthropic/claude-3-5-sonnet"],
        "provider": {
            "data_residency": "eu",
            "eu_owned": True,
            "data_collection": "deny",
            "sort": "price",
        },
    },
)
response.raise_for_status()

rule = response.json()["data"]
print(f"Created rule: {rule['name']} (id: {rule['id']})")
print(f"Use in requests with: rule_name=\"{rule['name']}\"")
```
