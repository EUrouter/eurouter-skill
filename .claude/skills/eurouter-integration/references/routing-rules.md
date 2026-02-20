# EUrouter Routing Rules API Reference

Routing rules are named, reusable presets for model and provider routing. Create them once and reference by name or UUID in any request.

**Base URL:** `https://api.eurouter.ai`

---

## Permissions Required

| Action | Permission |
|--------|-----------|
| List / Get / Dry-run | `routing:read` |
| Create / Update | `routing:write` |
| Delete | `routing:delete` |

---

## POST /api/v1/routing-rules

Create a routing rule.

### Request Body

```json
{
  "name": "gdpr-strict",
  "description": "Strict GDPR compliance for production",
  "scope": "org",
  "org_id": "uuid",
  "model": "gpt-4o",
  "models": ["gpt-4o", "claude-3-5-sonnet"],
  "provider": {
    "eu_owned": true,
    "data_collection": "deny",
    "max_retention_days": 0,
    "data_residency": "eu"
  },
  "enabled": true
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | `string` | Yes | Display name (1–255 chars) |
| `description` | `string \| null` | No | Description (max 1000 chars) |
| `scope` | `string` | No | `"user"` (default), `"org"`, or `"api_key"` |
| `org_id` | `string (uuid)` | No | Required if scope is `"org"` |
| `api_key_id` | `string (uuid)` | No | Required if scope is `"api_key"` |
| `model` | `string` | No | Primary model |
| `models` | `string[]` | No | Fallback model chain |
| `provider` | `ProviderPreferences` | No | Provider routing preferences |
| `enabled` | `boolean` | No | Active status (default: `true`) |

### Response

```json
{
  "data": {
    "id": "uuid",
    "name": "gdpr-strict",
    "description": "Strict GDPR compliance for production",
    "user_id": "user-id",
    "org_id": "uuid",
    "api_key_id": null,
    "model": "gpt-4o",
    "models": ["gpt-4o", "claude-3-5-sonnet"],
    "provider": { ... },
    "enabled": true,
    "created_at": "2025-01-15T10:30:00.000Z",
    "updated_at": "2025-01-15T10:30:00.000Z"
  }
}
```

---

## GET /api/v1/routing-rules

List routing rules.

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `include_disabled` | `boolean` | Include disabled rules (default: `false`) |
| `scope` | `string` | Filter: `"all"`, `"user"`, `"org"`, `"api_key"` |

### Response

```json
{
  "data": [
    { "id": "uuid", "name": "gdpr-strict", ... },
    { "id": "uuid", "name": "eu-cheap", ... }
  ]
}
```

---

## GET /api/v1/routing-rules/:id

Get a single routing rule by UUID.

---

## PATCH /api/v1/routing-rules/:id

Update a routing rule.

### Request Body

All fields optional:

```json
{
  "name": "new-name",
  "description": "Updated description",
  "model": "claude-3-5-sonnet",
  "models": null,
  "provider": { "data_residency": "eu" },
  "enabled": false
}
```

Set `model`, `models`, `provider`, or `description` to `null` to remove them.

---

## DELETE /api/v1/routing-rules/:id

Delete a routing rule.

### Response

```json
{ "data": { "success": true } }
```

---

## POST /api/v1/routing-rules/dry-run

Test a routing configuration without making a real request. See which providers would be available for your models with the given preferences.

### Request Body

```json
{
  "rule_id": "uuid",
  "rule_name": "gdpr-strict",
  "model": "gpt-4o",
  "models": ["gpt-4o", "claude-3-5-sonnet"],
  "preference_overrides": {
    "data_residency": "eu",
    "eu_owned": true
  }
}
```

At least one of `rule_id`, `rule_name`, `preference_overrides`, `model`, or `models` must be provided.

| Field | Type | Description |
|-------|------|-------------|
| `rule_id` | `string (uuid)` | Existing rule to test |
| `rule_name` | `string` | Existing rule to test (by name) |
| `model` | `string` | Single model to evaluate |
| `models` | `string[]` | Multiple models to evaluate |
| `preference_overrides` | `ProviderPreferences` | Override rule preferences for what-if testing |

**Behavior:**
- If `model` is provided: evaluates for that single model
- If `models` is provided: evaluates each model in the fallback chain
- If neither: evaluates across ALL available models

### Response

```json
{
  "data": {
    "rule_applied": {
      "id": "uuid",
      "name": "gdpr-strict"
    },
    "effective_preferences": {
      "eu_owned": true,
      "data_residency": "eu"
    },
    "results": [
      {
        "model": "gpt-4o",
        "provider_count": 2,
        "providers": ["scaleway", "ovhcloud"]
      },
      {
        "model": "claude-3-5-sonnet",
        "provider_count": 1,
        "providers": ["scaleway"]
      }
    ],
    "summary": {
      "total_models": 2,
      "models_with_providers": 2
    }
  }
}
```

---

## GET /api/v1/routing-rules/eu-residency

Returns lists of EU, EEA, and EU-adequate countries for reference.

### Response

```json
{
  "data": {
    "eu_countries": [
      { "code": "DE", "name": "Germany" },
      { "code": "FR", "name": "France" },
      ...
    ],
    "eea_countries": [
      { "code": "NO", "name": "Norway" },
      ...
    ],
    "eu_adequate_countries": [
      { "code": "JP", "name": "Japan" },
      ...
    ]
  }
}
```

---

## Using Rules in Requests

Reference a rule in any generation request:

```json
// By name
{
  "rule_name": "gdpr-strict",
  "messages": [{ "role": "user", "content": "Hello" }]
}

// By UUID
{
  "rule_id": "550e8400-e29b-41d4-a716-446655440000",
  "messages": [{ "role": "user", "content": "Hello" }]
}
```

**Merging behavior:** If a request includes both a rule reference and inline `model`/`models`/`provider` fields, the inline values take precedence over the rule's values. This allows per-request overrides on top of a base rule.

If both `rule_id` and `rule_name` are provided, `rule_id` takes precedence.

---

## Scoping

Rules can be scoped to different levels:

| Scope | Description |
|-------|-------------|
| `user` | Owned by and available to a single user (default) |
| `org` | Available to all members of an organization |
| `api_key` | Bound to a specific API key |
