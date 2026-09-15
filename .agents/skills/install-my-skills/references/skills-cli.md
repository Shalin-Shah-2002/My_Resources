# skills.sh CLI Reference

The `skills` CLI (open source: https://github.com/vercel-labs/skills) is run via npx — no installation required.

## `skills add` — install skills

```bash
npx skills add <source> [options]
```

### Source formats

```bash
# GitHub shorthand (owner/repo)
npx skills add vercel-labs/agent-skills

# Full GitHub URL
npx skills add https://github.com/vercel-labs/agent-skills

# Direct path to a skill inside a repo
npx skills add https://github.com/vercel-labs/agent-skills/tree/main/skills/web-design-guidelines

# GitLab URL, any git URL
npx skills add git@github.com:owner/repo.git

# Local path
npx skills add ./my-local-skills

# A pack (bundle of skills)
npx skills add https://skills.sh/p/<pack-id>

# Direct SKILL.md or archive URL (.zip/.tar/.tgz)
npx skills add https://example.com/download/my-skill
```

Private repos work with the same commands — the CLI uses already-configured Git credentials (git credential helper → GitHub CLI → SSH fallback for GitHub). `GITHUB_TOKEN` / `GH_TOKEN` can be set explicitly for API access.

### Options

| Option | Description |
|--------|-------------|
| `-g, --global` | Install to user directory instead of project |
| `-a, --agent <agents...>` | Target specific agents (e.g. `claude-code`, `opencode`, `codex`); use `'*'` for all |
| `-s, --skill <skills...>` | Install specific skills by name; `'*'` = all skills; quote names containing spaces |
| `-l, --list` | List available skills without installing |
| `--copy` | Copy files instead of symlinking (use when symlinks unsupported) |
| `-y, --yes` | Skip all confirmation prompts |
| `--all` | Install all skills to all agents without prompts |

### Examples

```bash
# List skills in a repo without installing
npx skills add owner/repo --list

# Install specific skills
npx skills add owner/repo --skill frontend-design --skill skill-creator

# Non-interactive install (CI/CD friendly)
npx skills add owner/repo --skill frontend-design -g -a claude-code -y

# All skills to all agents
npx skills add owner/repo --all

# All skills to one agent
npx skills add owner/repo --skill '*' -a claude-code
```

### Installation scope

| Scope | Flag | Location | Use case |
|-------|------|----------|----------|
| Project | (default) | `./<agent>/skills/` | Committed with the project, shared with team |
| Global | `-g` | `~/<agent>/skills/` | Available across all projects |

Install method (when interactive): symlink (recommended, single source of truth) or copy.

The CLI auto-detects installed agents. If none are detected it prompts; with `-y` it needs a detectable agent or explicit `-a`.

## Other commands

| Command | Description |
|---------|-------------|
| `npx skills use <source>` | Use one skill without installing (prints generated prompt; `--agent <a>` starts the agent with it) |
| `npx skills list` (alias `ls`) | List installed skills, project + global |
| `npx skills find [query]` | Search ecosystem (interactive fzf-style without query; keyword search with one) |
| `npx skills remove [skills]` | Remove installed skills from agents |
| `npx skills update [skills]` | Update installed skills to latest versions |
| `npx skills init [name]` | Create a new SKILL.md template |

### `skills list`

```bash
npx skills list          # all installed (project and global)
npx skills ls -g         # global only
npx skills ls -a claude-code -a cursor   # filter by agents
```

### `skills find`

```bash
npx skills find typescript
npx skills find react --owner vercel   # search all repos of an owner
```

### `skills update`

```bash
npx skills update            # all skills (interactive scope prompt)
npx skills update my-skill   # one skill
npx skills update -g         # global only
npx skills update -p         # project only
npx skills update -y         # non-interactive (auto-detects scope)
```

Only changed skills are re-downloaded.

### `skills remove` (alias `rm`)

```bash
npx skills remove                      # interactive selection
npx skills remove web-design-guidelines
npx skills remove frontend-design web-design-guidelines
npx skills remove --global web-design-guidelines
npx skills remove --agent claude-code cursor my-skill
npx skills remove --all                # remove everything, no confirmation
npx skills remove --skill '*' -a cursor  # all skills from one agent
```

| Option | Description |
|--------|-------------|
| `-g, --global` | Remove from global scope instead of project |
| `-a, --agent` | Target specific agents (`'*'` = all) |
| `-s, --skill` | Skills to remove (`'*'` = all) |
| `-y, --yes` | Skip confirmation prompts |
| `--all` | Shorthand for `--skill '*' --agent '*' -y` |

### `skills init`

```bash
npx skills init          # SKILL.md in current directory
npx skills init my-skill # SKILL.md in a new subdirectory
```

## Agent target names and paths

Common agents (full list of 80+ in the CLI README). Project path is `./<project path>`; global path under `~/`.

| Agent | `--agent` value | Project path | Global path |
|-------|-----------------|--------------|-------------|
| OpenCode | `opencode` | `.agents/skills/` | `~/.config/opencode/skills/` |
| Claude Code | `claude-code` | `.claude/skills/` | `~/.claude/skills/` |
| Cursor | `cursor` | `.agents/skills/` | `~/.cursor/skills/` |
| Codex | `codex` | `.agents/skills/` | `~/.codex/skills/` |
| Codex (universal) | `universal` | `.agents/skills/` | `~/.config/agents/skills/` |
| Gemini CLI | `gemini-cli` | `.agents/skills/` | `~/.gemini/skills/` |
| GitHub Copilot | `github-copilot` | `.agents/skills/` | `~/.copilot/skills/` |
| Windsurf | `windsurf` | `.agents/skills/` | `~/.codeium/windsurf/skills/` |
| Amp | `amp` | `.agents/skills/` | `~/.config/agents/skills/` |

## Skill discovery in source repos

The CLI looks for `SKILL.md` files (each containing `name` + `description` frontmatter) in:

- Repo root
- `skills/` (and `skills/.curated/`, `skills/.experimental/`, `skills/.system/`)
- Known agent skill dirs (`.agents/skills/`, `.claude/skills/`, etc.)
- Catalog layouts up to 3 levels deep (e.g. `skills/<category>/<name>/SKILL.md`)

If nothing is found in standard locations, a **recursive search** runs — this is how non-standard folders (e.g. `Skills_By_Shalin/`) are still discovered. Use `--full-depth` to also discover SKILL.md outside container dirs.

Names come from each SKILL.md's frontmatter `name` field, which may differ from its folder name.

## Environment variables

| Variable | Description |
|----------|-------------|
| `INSTALL_INTERNAL_SKILLS=1` | Show/install skills marked `metadata.internal: true` |
| `DISABLE_TELEMETRY=1` / `DO_NOT_TRACK=1` | Disable anonymous usage telemetry |
| `GITHUB_TOKEN` / `GH_TOKEN` | Explicit token for GitHub API access (private repos, rate limits) |

## Troubleshooting

- **"No skills found"** — the repo has no valid SKILL.md with `name` + `description` frontmatter.
- **Skill not loading in agent** — verify install path (check the agent's row above), confirm frontmatter is valid YAML, restart/reload the agent.
- **Permission errors** — need write access to the target directory.
- **Symlink problems** — retry with `--copy`.