---
name: install-my-skills
description: Installs Shalin's curated agent skills from his GitHub repo (Shalin-Shah-2002/My_Resources) using the skills.sh CLI (npx skills). Use whenever the user wants to install, add, set up, update, remove, list, try out, or create skills — e.g. "install my skills", "install skill system-design", "add the api-design skill", bundle requests like "set up skills for backend", "install skills for flutter", "skills for system design or architecture", pastes a GitHub link to the My_Resources repo or its Skills_By_Shalin folder, asks which skills are installed, wants to try a skill without installing, or asks to scaffold a new skill. Trigger this even when the user only names a skill or a category (backend, flutter, architecture) without the word "install".
---

# Install My Skills

Install agent skills with the skills.sh CLI (`npx skills`). Skills come from two places:

1. **My repo** — `Shalin-Shah-2002/My_Resources` (skills live under `Skills_By_Shalin/`)
2. **The public ecosystem** — anything on skills.sh (only when a skill isn't in my repo)

## Ground rules

- **Project scope by default.** Install with no `-g` flag — skills land in `./<agent>/skills/` of the current project. Only use `-g` when the user explicitly says "global" or "for all projects".
- **Auto-detect agents.** Omit `-a`; the CLI detects installed agents. Add `-y` to skip prompts.
- **Skill names come from SKILL.md frontmatter, not folder names.** Example: folder `opencode-openrouter-skill` contains skill `opencode-openrouter`. Always resolve through the catalog or `--list` — never guess a name from a folder name.
- **One install pattern covers everything:**
  ```bash
  npx skills add <owner>/<repo> --skill <skill-name> -y
  ```
  Multiple skills from the same repo: repeat `--skill`. Skills from different repos: one command per repo.
- **Catalog entries are pre-approved.** Shalin curates the catalog himself, so anything listed there (my skills, external skills like `mattpocock/skills`, plugins like `ponytail`) installs without asking. Only skills found by ecosystem search need confirmation before install.

## Workflow

### 1. Understand the request

- **Single skill by name** ("install skill system-design", "add api-design", "get me the openrouter skill") → resolve the name (step 2), then install.
- **By link** (pastes a GitHub URL) → if the link points at `My_Resources` or its `Skills_By_Shalin/` folder, it's my own skills: run `npx skills add Shalin-Shah-2002/My_Resources --list -y` to see what's available, then install what the user wants. A link to a specific skill folder installs just that skill. A link to any other repo is the install source — use it as-is (still confirm before installing non-my-repo skills).
- **Bundle** ("set up skills for backend", "install skills for flutter", "system design and architecture") → load the catalog, find the matching bundle(s), install every skill in them.
- **Everything** ("install all my skills") → `npx skills add Shalin-Shah-2002/My_Resources --all -y`.
- **Vague** ("get me some good skills") → list what's available and ask.

### 2. Resolve the skill name

1. Fetch the live catalog — it's kept in my repo so I can update it without touching this skill:
   ```bash
   curl -fsSL https://raw.githubusercontent.com/Shalin-Shah-2002/My_Resources/main/Skills_By_Shalin/catalog.md
   ```
2. If that fails (offline, repo moved), use the bundled snapshot `references/catalog-fallback.md`.
3. If the skill still isn't there, enumerate the repo directly:
   ```bash
   npx skills add Shalin-Shah-2002/My_Resources --list
   ```

Catalog entries map `skill name → source repo (owner/repo)`. My own skills use `Shalin-Shah-2002/My_Resources`. If the requested skill is in neither the catalog nor the repo, treat it as an ecosystem skill (step 3b).

### 3. Install

**a) Known source** (my repo, or a catalog entry pointing elsewhere):
```bash
npx skills add Shalin-Shah-2002/My_Resources --skill system-design --skill api-design -y
```

**b) Unknown skill (ecosystem search):**
```bash
npx skills find <name>
```
One obvious exact-name match → install it (`npx skills add <owner>/<repo> --skill <name> -y`) and say where it came from. Several plausible candidates → show them and let me pick.

**c) Bundles that span sources** (e.g. backend mixes my repo, external repos, and a plugin): group by repo — one `npx skills add` per repo — then install any plugins listed in the bundle (see Plugins below). The catalog holds the exact recipe for each bundle.

**d) Empty bundle** (e.g. flutter has no skills yet): say so plainly, offer to search the ecosystem for candidates in that category.

### 4. Verify

```bash
npx skills list
```
Report one line per newly installed skill (name + source). If something failed, show the CLI error and the likely fix (see Troubleshooting in `references/skills-cli.md`).

## Other operations

These are first-class requests too: "update my skills", "remove the system-design skill", "which skills do I have?", "let me try that skill before installing", "make me a new skill".

### Update my skills

```bash
npx skills update -y
```

Project scope by default; add `-g` for global. Only changed skills re-download, so this is cheap to run. Finish with `npx skills list` and report what changed.

### Remove skills

```bash
npx skills remove <skill-name> -y
```

Default scope is project; add `-g` for global. If the user doesn't say which scope, ask — removing is destructive. Resolve the exact name against `npx skills list` before running (frontmatter names differ from what people call skills). `--all` wipes everything — never run it unless the user explicitly asked to remove all skills.

### List installed skills

```bash
npx skills list
```

Report project and global installs separately, one line per skill (name + agent + scope). If the user instead asks "what skills can I get?", fetch the catalog (step 2) and call out which ones aren't installed yet.

### Try a skill without installing

```bash
npx skills use Shalin-Shah-2002/My_Resources --skill <skill-name>
```

Prints the skill's generated prompt without touching the project — good for "show me what this does before installing it".

### Create a new skill

```bash
npx skills init <skill-name>
```

Scaffolds a SKILL.md. Fill in real frontmatter (`name` plus a description that says both what the skill does and when to trigger it — vague descriptions under-trigger), write the body, then add an entry to the catalog (below) so it's installable.

## Plugins

Some catalog entries are plugins, not skills.sh skills — they install through their own harness-specific mechanism. Which harness? The one you're running in right now. If that's somehow ambiguous, ask.

**ponytail** (`DietrichGebert/ponytail` — makes the agent write minimal code; part of the backend bundle):

| Harness | Install |
|---------|---------|
| Claude Code | `/plugin marketplace add DietrichGebert/ponytail` then `/plugin install ponytail@ponytail` (two separate prompts) |
| OpenCode | Add `{ "plugin": ["@dietrichgebert/ponytail"] }` to `opencode.json` |
| Codex | `codex plugin marketplace add DietrichGebert/ponytail` then `codex plugin add ponytail@ponytail` |
| Gemini CLI / Antigravity | `gemini extensions install https://github.com/DietrichGebert/ponytail` |
| GitHub Copilot CLI | `copilot plugin marketplace add DietrichGebert/ponytail` then `copilot plugin install ponytail@ponytail` |
| Devin CLI | `devin plugins install DietrichGebert/ponytail` |
| Anything else | Fetch the ponytail README install section and use the matching harness |

When a bundle line includes a plugin (e.g. backend), installing it is part of the bundle — do it right after the skills, and report it alongside them in the verify step.

## Catalog maintenance

The catalog lives in my repo at `Skills_By_Shalin/catalog.md`. When asked to add a skill or bundle, edit that file in the repo — the bundled fallback is only a snapshot, refreshed after the repo changes. Format:

```markdown
## Skills
| name | source |
|------|--------|
| api-design | Shalin-Shah-2002/My_Resources |

## Bundles
| bundle | skills |
|--------|--------|
| backend | api-design, system-design |
```

### Refresh the catalog

After adding, renaming, or removing skills in the repo, sync the catalog with reality:

1. Enumerate what's actually in the repo: `npx skills add Shalin-Shah-2002/My_Resources --list -y`
2. Diff that against the catalog's Skills table — add new rows (source `Shalin-Shah-2002/My_Resources`), drop deleted ones, fix renamed ones, and update any bundles that reference them.
3. Update the bundled snapshot `references/catalog-fallback.md` to match.

Note: `My_Preferred_Skills.md` in the repo is human-readable notes, not machine data. The catalog file is the source of truth for installs.

## Full CLI reference

Every skills.sh command (`add`, `use`, `list`, `find`, `remove`, `update`, `init`), all flags, scopes, agent paths, and troubleshooting: read `references/skills-cli.md`.