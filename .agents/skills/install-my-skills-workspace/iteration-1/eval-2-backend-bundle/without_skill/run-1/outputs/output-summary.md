# Output — without_skill (baseline)

## Commands run (all inside sandbox)
1. `npx -y skills find backend` — searched the ecosystem
2. `npx -y skills add wshobson/agents@nodejs-backend-patterns -y` — installed
3. `npx -y skills add affaan-m/ecc@backend-patterns -y` — installed
4. `npx -y skills list` — verified

## Final installed state (verified by direct sandbox inspection)
```
.agents/skills/backend-patterns        source=affaan-m/ecc
.agents/skills/nodejs-backend-patterns source=wshobson/agents
```
The baseline interpreted "my skills for backend" as generic ecosystem search and installed two unrelated third-party skills — it had no way of knowing Shalin's repo or the backend bundle concept.