# EUrouter Integration Skill

An AI coding assistant skill that helps developers integrate [EUrouter](https://eurouter.ai) into their applications.

EUrouter is an OpenAI-compatible AI gateway built for EU customers who need GDPR compliance. It routes requests to 10+ providers through a single API, with built-in EU data residency, provider filtering, and zero-data-retention controls.

## Install

```bash
npx skills add eurouter/eurouter-skill
```

This installs the skill into your AI coding assistant (Claude Code, etc.), so it can guide you through EUrouter integration whenever you need help.

## What the Skill Covers

- **Drop-in migration** from OpenAI or OpenRouter (just change `base_url` and `api_key`)
- **EU/GDPR compliance** configuration (`data_residency`, `eu_owned`, `data_collection`, `max_retention_days`)
- **All API endpoints**: chat completions, text completions, embeddings, Responses API, model catalog
- **Streaming**, tool calling, vision, audio input, reasoning models
- **Multi-model fallback** for resilience
- **Routing rules** for reusable compliance policies
- **Code examples** in Python, Node.js/TypeScript, and curl

## Quick Start

After installing the skill, ask your AI coding assistant:

- "Help me integrate EUrouter into my Python app"
- "Set up EUrouter with strict GDPR compliance"
- "Migrate my OpenAI calls to use EUrouter"
- "Show me how to use EUrouter streaming with EU data residency"

## Links

- [EUrouter Website](https://eurouter.ai)
- [EUrouter Documentation](https://eurouter.ai/docs)

## License

MIT
