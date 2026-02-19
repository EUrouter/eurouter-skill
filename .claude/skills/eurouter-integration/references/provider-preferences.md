# EUrouter Provider Preferences Reference

The `provider` object can be included in any generation request (`/chat/completions`, `/completions`, `/embeddings`, `/responses`) or stored in a routing rule.

---

## All Fields

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `order` | `string[]` | — | Provider priority — try these first, in order |
| `ignore` | `string[]` | — | Exclude these providers from routing |
| `only` | `string[]` | — | Exclusive allowlist — use ONLY these providers |
| `allow_fallbacks` | `boolean` | `true` | Allow fallback to other providers on failure. Set `false` to error instead. |
| `sort` | `string` | — | Sort strategy: `"price"` (cheapest), `"latency"` (fastest), `"throughput"` (highest tokens/s) |
| `require_parameters` | `boolean` | — | Only use providers that support ALL parameters in the request |
| `quantizations` | `string[]` | — | Filter by model quantization levels |
| `data_collection` | `string` | — | `"allow"` or `"deny"` — filter by training data policy |
| `data_residency` | `string` | — | Region code for data processing location |
| `max_retention_days` | `number` | — | Max data retention in days. `0` = zero-data-retention only. |
| `eu_owned` | `boolean` | — | Only EU-headquartered providers |

---

## Provider Slugs

Valid values for `order`, `ignore`, and `only` arrays:

| Slug | Provider | HQ |
|------|----------|-----|
| `openai` | OpenAI | US |
| `anthropic` | Anthropic | US |
| `mistral` | Mistral | FR |
| `scaleway` | Scaleway | FR |
| `greenpt` | GreenPT | PT |
| `microsoft-foundry` | Microsoft Foundry | US |
| `nebius` | Nebius | NL |
| `ionos` | IONOS | DE |
| `ovhcloud` | OVHcloud | FR |
| `aws-bedrock` | AWS Bedrock | US |

---

## Quantization Values

Valid values for the `quantizations` array:

`int4`, `int8`, `fp4`, `fp6`, `fp8`, `fp16`, `bf16`, `fp32`

---

## Data Residency Codes

The `data_residency` field accepts region codes like:
- `"eu"` — European Union
- `"us"` — United States
- `"de"` — Germany
- `"fr"` — France
- Country-level ISO codes

Use `GET /api/v1/routing-rules/eu-residency` to get the full list of EU, EEA, and EU-adequate countries.

---

## GDPR Compliance Profile Templates

### Strict GDPR

EU-owned providers, no training data collection, zero data retention, EU-hosted endpoints:

```json
{
  "provider": {
    "eu_owned": true,
    "data_collection": "deny",
    "max_retention_days": 0,
    "data_residency": "eu"
  }
}
```

### EU Hosted, Cheapest Price

Route to EU endpoints, sorted by lowest cost:

```json
{
  "provider": {
    "data_residency": "eu",
    "sort": "price"
  }
}
```

### EU Hosted, Lowest Latency

Route to EU endpoints, sorted by fastest response:

```json
{
  "provider": {
    "data_residency": "eu",
    "sort": "latency"
  }
}
```

### Specific EU Providers Only

Restrict to known EU-headquartered providers:

```json
{
  "provider": {
    "only": ["scaleway", "ionos", "ovhcloud"]
  }
}
```

### EU Hosted with Fallback Control

EU hosted, but fail instead of falling back to non-EU providers:

```json
{
  "provider": {
    "data_residency": "eu",
    "allow_fallbacks": false
  }
}
```

### Prefer Specific Provider, Allow Fallback

Try Scaleway first, fall back to other EU providers:

```json
{
  "provider": {
    "order": ["scaleway", "ovhcloud", "ionos"],
    "data_residency": "eu"
  }
}
```

### Exclude Specific Providers

Use any provider except OpenAI and AWS:

```json
{
  "provider": {
    "ignore": ["openai", "aws-bedrock"]
  }
}
```

---

## Combining Fields

Fields are combined with AND logic. For example:

```json
{
  "provider": {
    "data_residency": "eu",
    "eu_owned": true,
    "data_collection": "deny",
    "sort": "price"
  }
}
```

This means: only endpoints that are (in the EU) AND (EU-owned provider) AND (no training data), sorted by cheapest price.

If no providers match all criteria, the request returns a `503 SERVICE_UNAVAILABLE` error (unless `allow_fallbacks` is true and fallback providers exist).
