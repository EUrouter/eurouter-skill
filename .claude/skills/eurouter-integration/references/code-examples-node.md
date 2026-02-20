# EUrouter Node.js / TypeScript Code Examples

All examples use the official `openai` npm package. Install with `npm install openai`.

---

## 1. Basic Setup (Drop-in Migration)

```typescript
import OpenAI from "openai";

// Only baseURL and apiKey change from standard OpenAI usage
const client = new OpenAI({
  baseURL: "https://api.eurouter.ai/api/v1",
  apiKey: "eur_YOUR_KEY_HERE",
});

const response = await client.chat.completions.create({
  model: "gpt-4o",  messages: [{ role: "user", content: "What is GDPR?" }],
});

console.log(response.choices[0].message.content);
```

---

## 2. EU-Compliant Chat Completion

```typescript
import OpenAI from "openai";

const client = new OpenAI({
  baseURL: "https://api.eurouter.ai/api/v1",
  apiKey: "eur_YOUR_KEY_HERE",
});

const response = await client.chat.completions.create({
  model: "gpt-4o",
  messages: [
    { role: "system", content: "You are a GDPR compliance assistant." },
    { role: "user", content: "Explain data subject rights under GDPR." },
  ],
  max_completion_tokens: 1024,
  temperature: 0.7,
  // @ts-expect-error EUrouter-specific field
  provider: {
    data_residency: "eu",
    eu_owned: true,
    data_collection: "deny",
    max_retention_days: 0,
  },
});

console.log(response.choices[0].message.content);
```

> **Note:** The `provider` field is EUrouter-specific and not in the OpenAI SDK types. Use `// @ts-expect-error` or cast to `any` when using TypeScript.

---

## 3. Streaming with EU Compliance

```typescript
import OpenAI from "openai";

const client = new OpenAI({
  baseURL: "https://api.eurouter.ai/api/v1",
  apiKey: "eur_YOUR_KEY_HERE",
});

const stream = await client.chat.completions.create({
  model: "claude-3-5-sonnet",
  messages: [{ role: "user", content: "Write a poem about Europe." }],
  stream: true,
  stream_options: { include_usage: true },
  // @ts-expect-error EUrouter-specific field
  provider: {
    data_residency: "eu",
  },
});

for await (const chunk of stream) {
  const content = chunk.choices[0]?.delta?.content;
  if (content) {
    process.stdout.write(content);
  }

  // Usage is in the final chunk
  if (chunk.usage) {
    console.log(`\n\nTokens used: ${chunk.usage.total_tokens}`);
  }
}
```

---

## 4. Tool Calling

```typescript
import OpenAI from "openai";

const client = new OpenAI({
  baseURL: "https://api.eurouter.ai/api/v1",
  apiKey: "eur_YOUR_KEY_HERE",
});

const tools: OpenAI.ChatCompletionTool[] = [
  {
    type: "function",
    function: {
      name: "get_weather",
      description: "Get the current weather for a location",
      parameters: {
        type: "object",
        properties: {
          location: {
            type: "string",
            description: "City name, e.g. 'Berlin'",
          },
        },
        required: ["location"],
      },
    },
  },
];

// First call: model decides to use the tool
const response = await client.chat.completions.create({
  model: "gpt-4o",
  messages: [{ role: "user", content: "What's the weather in Berlin?" }],
  tools,
  tool_choice: "auto",
  // @ts-expect-error EUrouter-specific field
  provider: { data_residency: "eu" },
});

const message = response.choices[0].message;

if (message.tool_calls) {
  const toolCall = message.tool_calls[0];
  const args = JSON.parse(toolCall.function.arguments);
  console.log(`Model wants to call: ${toolCall.function.name}(${JSON.stringify(args)})`);

  // Simulate tool response
  const weatherData = { temperature: 18, condition: "partly cloudy" };

  // Second call: send tool result back
  const followUp = await client.chat.completions.create({
    model: "gpt-4o",
    messages: [
      { role: "user", content: "What's the weather in Berlin?" },
      message,
      {
        role: "tool",
        tool_call_id: toolCall.id,
        content: JSON.stringify(weatherData),
      },
    ],
    tools,
    // @ts-expect-error EUrouter-specific field
    provider: { data_residency: "eu" },
  });

  console.log(followUp.choices[0].message.content);
}
```

---

## 5. Vision (Image Input)

```typescript
import OpenAI from "openai";

const client = new OpenAI({
  baseURL: "https://api.eurouter.ai/api/v1",
  apiKey: "eur_YOUR_KEY_HERE",
});

const response = await client.chat.completions.create({
  model: "gpt-4o",
  messages: [
    {
      role: "user",
      content: [
        { type: "text", text: "What's in this image?" },
        {
          type: "image_url",
          image_url: {
            url: "https://upload.wikimedia.org/wikipedia/commons/thumb/b/b7/Flag_of_Europe.svg/1200px-Flag_of_Europe.svg.png",
            detail: "auto",
          },
        },
      ],
    },
  ],
  max_completion_tokens: 512,
  // @ts-expect-error EUrouter-specific field
  provider: { data_residency: "eu" },
});

console.log(response.choices[0].message.content);
```

---

## 6. Embeddings

```typescript
import OpenAI from "openai";

const client = new OpenAI({
  baseURL: "https://api.eurouter.ai/api/v1",
  apiKey: "eur_YOUR_KEY_HERE",
});

const response = await client.embeddings.create({
  model: "text-embedding-3-small",
  input: ["GDPR compliance is important for EU businesses."],
  // Note: model fallback (models array) is NOT supported for embeddings
});

const embedding = response.data[0].embedding;
console.log(`Embedding dimensions: ${embedding.length}`);
console.log(`First 5 values: ${embedding.slice(0, 5)}`);
```

---

## 7. Multi-Model Fallback

```typescript
import OpenAI from "openai";

const client = new OpenAI({
  baseURL: "https://api.eurouter.ai/api/v1",
  apiKey: "eur_YOUR_KEY_HERE",
});

// If the first model's providers all fail, EUrouter tries the next model
const response = await client.chat.completions.create({
  model: "claude-3-5-sonnet", // Primary model
  messages: [{ role: "user", content: "Explain quantum computing simply." }],
  // @ts-expect-error EUrouter-specific fields
  models: [
    "claude-3-5-sonnet",
    "gpt-4o",
    "mistral-large",
  ],
  provider: {
    data_residency: "eu",
  },
});

console.log(response.choices[0].message.content);
```

---

## 8. Using a Routing Rule by Name

```typescript
import OpenAI from "openai";

const client = new OpenAI({
  baseURL: "https://api.eurouter.ai/api/v1",
  apiKey: "eur_YOUR_KEY_HERE",
});

// Reference a pre-configured routing rule by name
// (Create routing rules via POST /api/v1/routing-rules)
const response = await client.chat.completions.create({
  model: "gpt-4o",
  messages: [{ role: "user", content: "Hello!" }],
  // @ts-expect-error EUrouter-specific field
  rule_name: "gdpr-strict",
});

console.log(response.choices[0].message.content);
```

---

## 9. Listing Models Filtered by Provider

```typescript
// Model listing is public — no auth required
const response = await fetch(
  "https://api.eurouter.ai/api/v1/models?provider=scaleway"
);
const { data: models } = await response.json();

for (const model of models) {
  console.log(`${model.id} — ${model.name} (context: ${model.context_length})`);
}
```

---

## 10. Creating a Routing Rule

```typescript
const response = await fetch("https://api.eurouter.ai/api/v1/routing-rules", {
  method: "POST",
  headers: {
    "Content-Type": "application/json",
    "x-api-key": "eur_YOUR_KEY_HERE",
  },
  body: JSON.stringify({
    name: "production-eu",
    description: "Production routing: EU only, cheapest price",
    model: "gpt-4o",
    models: ["gpt-4o", "claude-3-5-sonnet"],
    provider: {
      data_residency: "eu",
      eu_owned: true,
      data_collection: "deny",
      sort: "price",
    },
  }),
});

const { data: rule } = await response.json();
console.log(`Created rule: ${rule.name} (id: ${rule.id})`);
console.log(`Use in requests with: rule_name="${rule.name}"`);
```
