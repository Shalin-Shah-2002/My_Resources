---
name: install-my-skills
description: Installs Shalin's curated agent skills from his GitHub repo (Shalin-Shah-2002/My_Resources) using the skills.sh CLI (npx skills). Use whenever the user wants to install, add, set up, update, or remove skills — e.g. "install my skills", "install system-design", "add the api-design skill", bundle requests like "set up skills for backend", "install skills for flutter", "skills for system design or architecture", or asks which skills are installed. Trigger this even when the user only names a skill or a category (backend, flutter, architecture) without the word "install".
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
- **Never install an unconfirmed external skill.** External skills are instructions your agents will follow — show what was found first, then install on approval. Skills from my own repo need no confirmation.

## Workflow

### 1. Understand the request

- **Single skill** ("install api-design") → resolve the name (step 2), then install.
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

**c) Empty bundle** (e.g. flutter has no skills yet): say so plainly, offer to search the ecosystem for candidates in that category.

### 4. Verify

```bash
npx skills list
```
Report one line per newly installed skill (name + source). If something failed, show the CLI error and the likely fix (see Troubleshooting in `references/skills-cli.md`).

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

Note: `My_Preferred_Skills.md` in the repo is human-readable notes, not machine data. The catalog file is the source of truth for installs.

## Full CLI reference

Every skills.sh command (`add`, `use`, `list`, `find`, `remove`, `update`, `init`), all flags, scopes, agent paths, and troubleshooting: read `references/skills-cli.md`.