---
name: opencode-openrouter
description: Master guide for configuring OpenRouter as a provider in OpenCode (terminal, desktop, and IDE). Use this skill whenever the user mentions OpenRouter, wants to add/switch/select models in opencode, set a default model, fix "model not found" or auth errors, configure provider routing/failover/quantization/price limits, connect an API key, edit opencode.json, or asks which model opencode is using — even if they never say the word "config".
---

# OpenRouter + OpenCode

OpenRouter gives OpenCode unified access to hundreds of models (Anthropic, OpenAI, Google, Meta, DeepSeek, and more) through one API key, with automatic provider failover. OpenCode treats OpenRouter as a built-in provider — no custom npm package needed.

Both the terminal (TUI) and desktop apps read the same `opencode.json` and the same stored credentials, so connecting once makes OpenRouter available everywhere.

## Workflow decision tree

Ask (or infer) what the user actually needs, then jump to the right section:

| User wants | Go to |
|---|---|
| Set up OpenRouter from scratch | [Setup](#setup) |
| Add a model that's not in the picker | [Add models](#add-models) |
| Set/override the default model | [Default model](#default-model) |
| Fix auth or "model not found" errors | [Troubleshooting](#troubleshooting) |
| Control which upstream providers serve requests (routing, price, privacy, quantization) | Read `references/provider-routing.md` |
| Understand `~latest` aliases, `:nitro`/`:floor` variants, pinning versions | Read `references/model-slugs.md` |
| Config file locations, precedence, env/file substitution, credentials, hiding models | Read `references/opencode-config.md` |

## Setup

1. Get an API key: [openrouter.ai/settings/keys](https://openrouter.ai/settings/keys) → **Create API Key** (starts with `sk-or-v1-...`).
2. In the OpenCode TUI, run `/connect`, search for **OpenRouter**, paste the key.
3. Run `/models` — many OpenRouter models are preloaded by default; pick one.

Desktop app: Settings (or command palette → **Connect provider**) → **OpenRouter** under Popular providers → **API key** method → paste. If models don't appear, fully quit and reopen the app (the catalog loads at startup).

Headless / CI (no TUI): write the key directly to `~/.local/share/opencode/auth.json` (respect `$XDG_DATA_HOME` if set):

```json
{
  "openrouter": {
    "type": "api",
    "key": "sk-or-v1-your-key-here"
  }
}
```

Never put the raw key in `opencode.json` — use `{env:OPENROUTER_API_KEY}` or `{file:~/.secrets/openrouter-key}` substitution instead (see `references/opencode-config.md`).

Verify: `opencode auth list` should show openrouter.

## Add models

Many models are preloaded; add any others under `provider.openrouter.models`, keyed by the model's exact OpenRouter slug (find them at [openrouter.ai/models](https://openrouter.ai/models)):

```json
{
  "$schema": "https://opencode.ai/config.json",
  "provider": {
    "openrouter": {
      "models": {
        "~anthropic/claude-sonnet-latest": {},
        "~google/gemini-flash-latest": {},
        "z-ai/glm-4.7": {}
      }
    }
  }
}
```

The leading `~` is not a typo — `~author/family-latest` slugs are OpenRouter aliases that always resolve to that lab's newest flagship model. Pinned slugs like `anthropic/claude-sonnet-4.5` work the same way. See `references/model-slugs.md` before "correcting" a slug that starts with `~` — it's intentional.

## Default model

OpenCode composes model IDs as `provider_id/model_id`, so OpenRouter models get an extra `openrouter/` prefix — the full ID is `openrouter/~anthropic/claude-sonnet-latest`, not `~anthropic/claude-sonnet-latest`. Getting this wrong is the most common "model not found" cause.

```json
{
  "$schema": "https://opencode.ai/config.json",
  "model": "openrouter/~anthropic/claude-sonnet-latest",
  "small_model": "openrouter/~google/gemini-flash-latest"
}
```

- `model` — main agent model, applies to TUI, desktop, and `opencode run`.
- `small_model` — cheap model for lightweight tasks (session titles, etc.).
- Override per invocation: `opencode run -m "openrouter/z-ai/glm-4.7"`.

## Per-model routing config

Each model entry accepts an `options.provider` object that OpenRouter uses for routing (order, fallbacks, sorting, price caps, privacy):

```json
{
  "$schema": "https://opencode.ai/config.json",
  "provider": {
    "openrouter": {
      "models": {
        "~anthropic/claude-sonnet-latest": {
          "options": {
            "provider": {
              "order": ["anthropic"],
              "allow_fallbacks": true
            }
          }
        }
      }
    }
  }
}
```

The complete field reference (sort, only/ignore, quantizations, max_price, data_collection, zdr, throughput/latency thresholds, `:nitro`/`:floor` variants) lives in `references/provider-routing.md`. Consult it whenever the user cares about *which* upstream provider serves the request, cost control, or data privacy — not just *which model*.

## Troubleshooting

Work through these before guessing:

- **Auth errors**: confirm the key exists with `opencode auth list`; re-run `/connect`; verify the key is active at openrouter.ai/settings/keys.
- **Model not found**: the slug must be exact, including any leading `~` (e.g. `~anthropic/claude-sonnet-latest`). In the top-level `model` key, remember the `openrouter/` prefix. Check the slug at openrouter.ai/models.
- **No models after connecting**: fully quit and restart OpenCode — the provider catalog loads at startup.
- **Model missing from picker**: add it under `provider.openrouter.models` in config, then restart.
- **Requests hitting the wrong provider or wrong price**: that's routing config — see `references/provider-routing.md`.
- **Privacy**: OpenRouter doesn't log source-code prompts unless prompt logging is opted in. For stricter guarantees set `data_collection: "deny"` or `zdr: true` in routing options.

## Reference files

- `references/provider-routing.md` — every `provider` routing field with opencode.json examples. Read when tuning routing, cost, performance, or privacy.
- `references/model-slugs.md` — `~latest` aliases, `:nitro`/`:floor` variants, version pinning, resolution behavior. Read when the user asks about model versions, speed/cost variants, or reproducibility.
- `references/opencode-config.md` — config locations & precedence, merging, credentials, variable substitution, blacklist/whitelist, enabled/disabled providers, timeout options. Read when config placement, key storage, or provider visibility is in question.
