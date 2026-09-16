# Skills Catalog — Shalin's skills

Live source (wins when fetchable):
https://raw.githubusercontent.com/Shalin-Shah-2002/My_Resources/main/Skills_By_Shalin/catalog.md

This file is a bundled fallback snapshot.

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

**backend bundle recipe** (mainly for new projects):

```bash
npx skills add Shalin-Shah-2002/My_Resources --skill api-design -y
npx skills add mattpocock/skills --skill grill-me --skill grill-with-docs --skill improve-codebase-architecture -y
# then install the ponytail plugin for the current harness — see SKILL.md "Plugins"
```

## Plugins

| plugin | source | notes |
|--------|--------|-------|
| ponytail | DietrichGebert/ponytail | Minimal-code ruleset; per-harness install, in the backend bundle |

## Adding entries

- New skill in my repo: add a row under Skills with source `Shalin-Shah-2002/My_Resources`, then add it to any bundles.
- External skill: source is the skill's GitHub `owner/repo`.
- New bundle: add a row under Bundles.