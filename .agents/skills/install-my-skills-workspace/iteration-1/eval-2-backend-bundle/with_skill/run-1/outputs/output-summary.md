# Output — with_skill

## Commands run (all inside sandbox)
1. `curl -fsSL .../Skills_By_Shalin/catalog.md` → 404, fell back to bundled `references/catalog-fallback.md`
2. `npx skills add Shalin-Shah-2002/My_Resources --skill api-design --skill system-design -y` → installed both, non-interactive, agent auto-detect
3. `npx skills list`

## Final installed state (verified by direct sandbox inspection)
```
.agents/skills/api-design      (SKILL.md present)
.agents/skills/system-design   (SKILL.md present)
skills-lock.json:
  api-design    → source=Shalin-Shah-2002/My_Resources
  system-design → source=Shalin-Shah-2002/My_Resources
```
Exactly the two backend-bundle skills, both from Shalin's repo, nothing else.