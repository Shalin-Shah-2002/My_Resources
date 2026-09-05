# Model Slugs, Aliases, and Variants Reference

Every OpenRouter model is addressed by a slug: `author/model-name`. In OpenCode those slugs become keys under `provider.openrouter.models`, and the full OpenCode model ID gains the `openrouter/` prefix (e.g. `openrouter/anthropic/claude-sonnet-4.5`). Browse all slugs at [openrouter.ai/models](https://openrouter.ai/models).

## `~author/family-latest` aliases

Slugs starting with `~` are **not typos**. `~anthropic/claude-sonnet-latest` always resolves to the newest model in the Claude Sonnet family. When a lab ships a new version, the alias retargets automatically — old configs keep working and pick up the new model.

```json
"models": {
  "~anthropic/claude-sonnet-latest": {},
  "~google/gemini-flash-latest": {},
  "~z-ai/glm-latest": {}
}
```

How resolution works:
1. The `~` prefix identifies the model family.
2. The newest visible model in that family is selected (aliases and hidden models are excluded).
3. The response's `model` field reports the *concrete* model that actually served the request — useful for logging version rollovers.
4. If a family has no eligible model, the request errors rather than falling back to something unrelated.

**Pricing and capabilities track the target, not a snapshot** — the models page and `/api/v1/models` report the pricing, context length, modalities, and parameters of whatever the alias currently resolves to.

**Compatibility contract**: when an alias retargets to a model that requires reasoning, OpenRouter remaps unsupported reasoning parameters to the nearest supported value (e.g. `effort: "none"` upgrades to the lowest supported effort) instead of returning 400. Concrete slugs keep strict validation — sending `effort: "none"` to a reasoning-mandatory concrete model is a 400.

### When to use aliases vs pinned slugs

| Use `~latest` alias | Use pinned concrete slug |
|---|---|
| User-facing agents that should always get the best current model | Regression tests / reproducibility requirements |
| Prototypes ("latest Claude Opus" is fine) | Exact parameter semantics (no remapping) |
| Rolling migrations before a release stabilizes | Downgrading to a known-good version |

Limitations: aliases only ever point to the newest version — there's no "second newest" or rollback through the alias; switch to a concrete slug for that. A newer model can roll in at any time.

## Variant suffixes

Append to any slug (they compose with aliases):

| Suffix | Effect |
|---|---|
| `:nitro` | Sort by throughput + makes priority service-tier endpoints eligible. Superset of `provider.sort: "throughput"`. |
| `:floor` | Sort by price + makes flex service-tier endpoints eligible. Superset of `provider.sort: "price"`. |
| `:free` | Free variant of a model (separate slug on the models page, e.g. `deepseek/deepseek-r1:free`). |

Example in OpenCode:

```json
"models": {
  "meta-llama/llama-3.3-70b-instruct:nitro": {}
}
```

With `:nitro`/`:floor`, remember the full OpenCode ID: `openrouter/meta-llama/llama-3.3-70b-instruct:nitro`.

## Finding and verifying slugs

1. [openrouter.ai/models](https://openrouter.ai/models) — search, copy exact slug including `~` and suffixes.
2. A model missing from the `/models` picker just needs to be added under `provider.openrouter.models` (see SKILL.md).
3. "Model not found" errors: re-check the slug character-by-character — most failures are a missing `~`, a typo, or a forgotten `openrouter/` prefix in the top-level `model` key.
4. To see which concrete model served past requests, check the OpenRouter [activity log](https://openrouter.ai/activity) or the response's `model` field.
