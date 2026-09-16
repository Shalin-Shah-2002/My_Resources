# Skills Catalog — Shalin's skills

This file is the source of truth for installs (see `Skills_By_Shalin/install-my-skills/SKILL.md`).
A snapshot of this file is bundled at `install-my-skills/references/catalog-fallback.md`.

## Skills

| name | source |
|------|--------|
| api-design | Shalin-Shah-2002/My_Resources |
| system-design | Shalin-Shah-2002/My_Resources |
| opencode-openrouter | Shalin-Shah-2002/My_Resources |
| install-my-skills | Shalin-Shah-2002/My_Resources |
| grill-me | mattpocock/skills |
| grill-with-docs | mattpocock/skills |
| improve-codebase-architecture | mattpocock/skills |

## Bundles

| bundle | skills |
|--------|--------|
| backend | api-design, grill-me, grill-with-docs, improve-codebase-architecture + ponytail plugin |
| system-design | system-design |
| architecture | system-design, api-design |
| flutter | (empty — add flutter skills here) |

**backend bundle install recipe** (mainly for new projects):

```bash
# 1. My skill
npx skills add Shalin-Shah-2002/My_Resources --skill api-design -y
# 2. Matt Pocock's skills (one command, one repo)
npx skills add mattpocock/skills --skill grill-me --skill grill-with-docs --skill improve-codebase-architecture -y
# 3. ponytail plugin — install for the harness in use (see SKILL.md "Plugins")
```

## Plugins

Not skills.sh skills — each has its own per-harness install. Details in `install-my-skills/SKILL.md`.

| plugin | source | notes |
|--------|--------|-------|
| ponytail | DietrichGebert/ponytail | Minimal-code ruleset; included in the backend bundle, install for the current harness |

## Adding entries

- New skill in my repo: add a row under Skills with source `Shalin-Shah-2002/My_Resources`, then add it to any bundles.
- External skill: source is the skill's GitHub `owner/repo`.
- Plugin: add a row under Plugins and mention it in the bundle line.
- New bundle: add a row under Bundles.
