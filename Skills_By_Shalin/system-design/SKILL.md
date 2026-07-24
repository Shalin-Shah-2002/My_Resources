---
name: system-design
description: Comprehensive system design guidance for designing, reviewing, or explaining backend architectures, scaling strategies, load balancing, database choices, networking protocols, authentication, authorization, API security, and system design interview answers. Use when Codex needs to create an architecture from scratch, compare trade-offs, reason about reliability, performance, storage, distributed systems, or prepare a system design response.
---

# System Design

Use this skill to turn an ambiguous product or interview prompt into a clear backend architecture with explicit trade-offs.

## Workflow

1. Clarify the goal, actors, core user journeys, read/write patterns, latency expectations, data retention, compliance constraints, and availability target.
2. Start with a simple baseline architecture, then scale only the parts that pressure the requirements.
3. Identify bottlenecks and single points of failure before adding components.
4. Choose storage based on data shape, relationship complexity, consistency needs, query patterns, and write/read volume.
5. Choose communication protocols based on interaction style: request/response, realtime push, async background work, or internal service calls.
6. Add authentication, authorization, and API security early enough that they shape the design, not as a final checklist.
7. Explain trade-offs in plain language: what improves, what gets more complex, and what failure mode remains.
8. End with operational concerns: monitoring, health checks, backups, rate limits, deployment, maintenance, and deprecation.

## Default Answer Shape

For system design interviews or architecture proposals, structure the answer as:

1. Requirements and assumptions
2. High-level architecture
3. Data model and storage choices
4. API and protocol choices
5. Scaling and reliability plan
6. Security and access control
7. Observability and operations
8. Trade-offs, bottlenecks, and next steps

## Reference Loading

Read `references/system-design-handbook.md` when the task needs concrete definitions, examples, tool names, or nuanced comparisons from the course material.

Read `references/system-design-checklists.md` when the task asks for an interview answer, review checklist, or architecture critique.

Use the API-focused skill `api-design` for deep API contract design, endpoint naming, implementation hygiene, and API security details.
