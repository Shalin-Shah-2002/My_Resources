# Provider Routing Reference

OpenRouter routes each request to an upstream provider (Anthropic, OpenAI, Baseten, DeepInfra, Google Vertex, etc.). By default it load-balances across the cheapest healthy providers — prioritizing ones without recent outages, weighted by inverse square of price, keeping the rest as fallbacks. Setting `sort` or `order` disables load balancing.

In OpenCode, routing options attach to each model under `options.provider`. Everything below goes in that object.

```json
{
  "$schema": "https://opencode.ai/config.json",
  "provider": {
    "openrouter": {
      "models": {
        "~anthropic/claude-sonnet-latest": {
          "options": {
            "provider": {
              "<field>": "<value>"
            }
          }
        }
      }
    }
  }
}
```

## All fields

| Field | Type | Default | What it does |
|---|---|---|---|
| `order` | string[] | - | Provider slugs to try, in order. Others still available as fallbacks. |
| `allow_fallbacks` | boolean | `true` | When `false`, only the listed/selected providers are used — request fails if they fail. |
| `only` | string[] | - | Allow *only* these provider slugs. Sharpens `allow_fallbacks: false`; reduces recovery options. |
| `ignore` | string[] | - | Skip these provider slugs (e.g. one known to degrade quality for your workload). |
| `sort` | string \| object | - | `"price"`, `"throughput"`, or `"latency"`. As an object: `{ "by": "...", "partition": "model"\|"none" }`. Disables load balancing. |
| `require_parameters` | boolean | `false` | Only route to providers supporting all request parameters (tools, response_format, etc.). |
| `data_collection` | `"allow"` \| `"deny"` | `"allow"` | `"deny"` excludes providers that may store/train on data. |
| `zdr` | boolean | - | `true` = only Zero-Data-Retention endpoints. OR'd with account-wide privacy settings. |
| `enforce_distillable_text` | boolean | - | Only route to models that permit text distillation. |
| `quantizations` | string[] | - | Filter by quant level: `int4`, `int8`, `fp4`, `mxfp4`, `nvfp4`, `fp6`, `fp8`, `mxfp8`, `fp16`, `bf16`, `fp32`, `unknown`. |
| `preferred_min_throughput` | number \| object | - | Prefer ≥ this many tokens/sec. Number = p50; or object with `p50`/`p75`/`p90`/`p99` cutoffs. Deprioritizes, never excludes. |
| `preferred_max_latency` | number \| object | - | Prefer ≤ this many seconds latency. Same percentile form. Deprioritizes, never excludes. |
| `max_price` | object | - | Hard price cap, e.g. `{ "prompt": 1, "completion": 2 }` ($/M tokens). Also `request` and `image` keys. Request fails if nothing is cheap enough. |

Key distinction: `preferred_*` thresholds are soft (slow/expensive providers move to the back of the line, request still runs); `max_price` is hard (request fails if no provider meets it).

## Recipes

**Always one specific provider, no fallback:**
```json
"provider": { "order": ["baseten"], "allow_fallbacks": false }
```

**Cheapest stable option, never load-balance:**
```json
"provider": { "sort": "price" }
```

**Fastest (also enables priority-tier endpoints — superset of sort):**
```json
"provider": { "sort": "throughput" }
```
Shortcut: append `:nitro` to the model slug instead (e.g. `meta-llama/llama-3.3-70b-instruct:nitro`). The `:floor` shortcut does the same for price and enables flex-tier endpoints.

**Cheapest across multiple models/fallbacks that still meets a speed floor** — `partition: "none"` sorts endpoints globally across models instead of grouping by model:
```json
"provider": {
  "sort": { "by": "price", "partition": "none" },
  "preferred_min_throughput": { "p90": 50 }
}
```

**Privacy-hardened (no training, zero retention):**
```json
"provider": { "data_collection": "deny", "zdr": true }
```

**Exclude a flaky provider:**
```json
"provider": { "ignore": ["deepinfra"] }
```

**Max price ceiling:**
```json
"provider": { "max_price": { "prompt": 1, "completion": 2 } }
```

## Slug matching for variants

A base slug matches all endpoints of that provider; a suffixed slug targets exactly one:

| Slug in `order`/`only`/`ignore` | Matches |
|---|---|
| `"google-vertex"` | All Vertex endpoints, every region |
| `"google-vertex/us-east5"` | Only that region |
| `"deepinfra"` | Default + turbo endpoints |
| `"deepinfra/turbo"` | Only the turbo endpoint |

Service-tier endpoints (e.g. `openai/fast`, `google-vertex/flex`) are *not* matched by base slugs — they need the `service_tier` parameter or a tier-suffixed slug.

Provider slugs are visible on model pages at openrouter.ai (copy button next to each provider name). Also: when sending `tools`/`tool_choice` or `max_tokens`, OpenRouter best-effort routes to providers supporting them even without `require_parameters`.

## Default load-balancing example

Providers A ($1/M), B ($2/M, recent outage), C ($3/M): request goes to A (inverse-square weighting makes A 9× more likely than C), B is deprioritized for its outage and kept as last fallback. If you need different behavior than this, that's what `sort`/`order` are for.
