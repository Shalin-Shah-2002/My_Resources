---
name: api-design
description: API design guidance for creating, reviewing, or improving REST, GraphQL, gRPC, WebSocket, and async API contracts with strong naming, versioning, pagination, filtering, sorting, HTTP semantics, authentication, authorization, security, maintainable implementation practices, testing, lifecycle management, and deprecation strategy. Use when Codex needs to design API endpoints, write API code, audit API security, or document API best practices.
---

# API Design

Use this skill to design APIs that are easy for clients to consume, safe to expose, and maintainable in code.

## Workflow

1. Clarify consumers, use cases, data ownership, permission boundaries, latency expectations, and compatibility requirements.
2. Choose the API style deliberately: REST for resource contracts, GraphQL for flexible client field selection, gRPC for internal high-performance service calls, WebSockets for realtime push, and AMQP/queues for async work.
3. Define the contract before implementation whenever possible: endpoints or schema, request shape, response shape, errors, auth requirements, pagination, filtering, sorting, and versioning.
4. Design security into the contract: authentication, authorization, rate limits, CORS, CSRF, XSS prevention, injection prevention, and auditability.
5. Implement thin transport handlers that validate input, call domain/application services, and translate domain results into API responses.
6. Keep business logic out of controllers/resolvers where possible.
7. Add tests for contract behavior, validation errors, authorization failures, edge cases, and security-sensitive flows.
8. Document lifecycle expectations: stability, monitoring, maintenance, deprecation, and retirement.

## Default Answer Shape

For API design tasks, respond with:

1. API style choice and why
2. Resource or schema model
3. Endpoint/query/mutation/RPC definitions
4. Request and response examples
5. Error model and status codes
6. Authn/authz rules
7. Security controls
8. Implementation notes
9. Tests and maintenance plan

## Reference Loading
 
Read `references/api-design-handbook.md` for detailed REST, GraphQL, gRPC, HTTP anatomy, lifecycle, authentication, authorization, and security guidance.

Read `references/api-implementation-checklist.md` when writing or reviewing API code for maintainability, validation, testing, and security.

Use the broader `system-design` skill when the task is mainly about full-system architecture, scaling, storage, load balancing, or infrastructure trade-offs.
