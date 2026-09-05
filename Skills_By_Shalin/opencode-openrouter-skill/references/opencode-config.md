# OpenCode Config Reference

Everything about where opencode config lives, how it merges, and how credentials/substitution work.

## Locations and precedence

Configs are **merged, not replaced** — later sources override earlier ones only on conflicting keys. Load order (later wins):

1. **Remote config** (`.well-known/opencode`) — organizational defaults
2. **Global**: `~/.config/opencode/opencode.json` — user-wide preferences (providers, models, permissions)
3. **Custom**: `$OPENCODE_CONFIG` env var path
4. **Project**: `opencode.json` in project root (highest standard precedence; safe to commit to git)
5. `.opencode` directories (agents, commands, plugins, skills)
6. **Inline**: `$OPENCODE_CONFIG_CONTENT` env var
7. **Managed files**: `/Library/Application Support/opencode/` (macOS), `/etc/opencode/` (Linux), `%ProgramData%\opencode` (Windows) — admin-controlled
8. **macOS MDM** (`.mobileconfig`, `ai.opencode.managed` domain) — cannot be overridden

So: project config beats global, which beats organizational remote defaults. Use global for "I always want OpenRouter", project for "this repo pins this model".

Both `opencode.json` and `opencode.jsonc` (JSON with comments) work everywhere. OpenCode looks in the current directory then traverses up to the nearest git directory. TUI-specific settings live separately in `tui.json`.

Custom directory: `$OPENCODE_CONFIG_DIR` behaves like an extra `.opencode` and can override global + project `.opencode` settings.

Verify the fully resolved config: `opencode debug config`.

## Credentials

- Keys added via `/connect` (TUI) or desktop Settings are stored in `~/.local/share/opencode/auth.json` — same path on macOS/Linux/Windows, shared by TUI and desktop. Substitute `$XDG_DATA_HOME` if set.
- Manual write format for auth.json:
  ```json
  {
    "openrouter": { "type": "api", "key": "sk-or-v1-..." }
  }
  ```
- Check what's stored: `opencode auth list`.

## Variable substitution

Never hardcode keys in opencode.json (it's often committed). Two mechanisms:

```json
{
  "provider": {
    "openrouter": {
      "options": {
        "apiKey": "{env:OPENROUTER_API_KEY}"
      }
    }
  }
}
```

- `{env:VAR}` — environment variable; empty string if unset.
- `{file:path}` — file contents; path relative to the config file, or absolute/`~`.

## Provider visibility

- `disabled_providers: ["openai"]` — never load these, even if creds exist or env vars are set.
- `enabled_providers: ["openrouter"]` — allowlist; everything else ignored.
- `disabled_providers` wins if a provider appears in both.

## Per-provider options

Under `provider.<id>.options`:

- `baseURL` — custom endpoint (proxies/gateways). Not usually needed for OpenRouter; the built-in provider handles it.
- `apiKey` — inline key or substitution (see above).
- `timeout` (default 300000 ms), `headerTimeout`, `chunkTimeout` — request/stream timeouts; `false` disables.
- `setCacheKey` — force a cache key for the provider.

Per-model extra: each entry in `models` accepts `name` (display name), `limit: { context, output }`, and `options` (where OpenRouter routing config goes — see `references/provider-routing.md`).

## Hiding models in the picker

```json
"provider": {
  "openrouter": {
    "blacklist": ["some/model-id"],
    "whitelist": ["~anthropic/claude-sonnet-latest"]
  }
}
```

Model IDs match what `/models` shows. `whitelist` narrows first, then `blacklist` removes from the remainder.

## Other keys worth knowing (context)

- `model` / `small_model` — top-level model selection (see SKILL.md).
- `share`: `"manual"` (default) | `"auto"` | `"disabled"`.
- `permission` — e.g. `{ "edit": "ask", "bash": "ask" }`.
- `instructions` — extra instruction files/globs for the model.
