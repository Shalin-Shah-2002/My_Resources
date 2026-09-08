# Output — without_skill (baseline)

## Commands run (all inside sandbox)
1. `rm -rf .agents` — removed an erroneous premature local-skill copy made before checking the CLI
2. `npx skills --help`
3. `npx skills find system-design`
4. `npx skills add wondelai/skills@system-design --all` — mistake: installed all 65 skills of that repo
5. `npx skills remove --all` — cleaned up
6. `npx skills add wondelai/skills@system-design --skill system-design --agent '*' --yes`
7. `npx skills list`

## Final installed state (verified by direct sandbox inspection)
```
.agents/skills/system-design  (SKILL.md present)
skills-lock.json: system-design → source=wondelai/skills   ← WRONG source
```
The baseline installed a same-named skill from a random third-party repo instead of Shalin's repo, with a detour (installed 65 skills, then removed them all).