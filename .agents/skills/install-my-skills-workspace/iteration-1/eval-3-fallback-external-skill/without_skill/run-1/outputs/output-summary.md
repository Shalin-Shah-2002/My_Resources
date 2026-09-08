# Output — without_skill (baseline)

## Commands run (all inside sandbox)
1. `npx skills find dataviz` — only surfaced obscure sub-100-install skills
2. `npx skills find data visualization` + `curl -s https://api.github.com/repos/anthropics/skills/contents/` — cross-check
3. `npx skills add anthropics/knowledge-work-plugins@data-visualization -y`
4. `npx skills list` + `ls .agents/skills/data-visualization`

## Final installed state (verified by direct sandbox inspection)
```
.agents/skills/data-visualization
skills-lock.json: data-visualization → source=anthropics/knowledge-work-plugins
```
The baseline dug deeper when the first search looked low-quality (tried a second query, cross-checked GitHub) and landed on the official Anthropic dataviz skill with 11.8K installs.