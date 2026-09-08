# Output — with_skill

## Commands run (all inside sandbox)
1. `curl -fsSL https://raw.githubusercontent.com/Shalin-Shah-2002/My_Resources/main/Skills_By_Shalin/catalog.md` → 404, fell back to bundled `references/catalog-fallback.md` (confirmed `system-design → Shalin-Shah-2002/My_Resources`)
2. `npx skills add Shalin-Shah-2002/My_Resources --list` — confirmed skill exists (3 skills found)
3. `npx skills add Shalin-Shah-2002/My_Resources --skill system-design -y` — project scope, agent auto-detect
4. `npx skills list` — verification

## Final installed state (verified by direct sandbox inspection)
```
.agents/skills/system-design  (SKILL.md present)
skills-lock.json: system-design → source=Shalin-Shah-2002/My_Resources
```
`npx skills list` output:
```
Project Skills
system-design  ./.agents/skills/system-design
  Source: Shalin-Shah-2002/My_Resources
```