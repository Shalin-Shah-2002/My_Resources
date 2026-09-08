# Output — with_skill

## Commands run (all inside sandbox)
1. `curl -fsSL .../Skills_By_Shalin/catalog.md` → 404, fell back to bundled snapshot (no dataviz → ecosystem search)
2. `npx skills find dataviz` — 19 candidates; top by installs: `wentorai/research-plugins@dataviz-skills` (236 installs, 2x runner-up)
3. `webfetch https://skills.sh/wentorai/research-plugins/dataviz-skills` — vetted: passed Gen/Socket/Snyk audits
4. `npx skills add wentorai/research-plugins --skill dataviz-skills -y`
5. `npx skills list`

## Final installed state (verified by direct sandbox inspection)
```
.agents/skills/dataviz-skills
skills-lock.json: dataviz-skills → source=wentorai/research-plugins
```
Correct flow (fallback → not found → find → install), but picked an obscure 236-install repo. A far better candidate exists: `anthropics/knowledge-work-plugins@data-visualization` (11.8K installs, official Anthropic) — the run stopped at the first exact-ish name match instead of weighing source quality.