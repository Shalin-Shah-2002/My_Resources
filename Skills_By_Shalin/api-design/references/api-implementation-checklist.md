# API Implementation Checklist

## Contract Checklist

- Define the API style: REST, GraphQL, gRPC, WebSocket, or async queue.
- Define consumers and use cases.
- Define resource names or schema types in business language.
- Define request examples.
- Define response examples.
- Define validation errors.
- Define auth requirements.
- Define authorization checks.
- Define pagination, filtering, and sorting for lists.
- Define idempotency for retries.
- Define versioning or schema evolution rules.
- Define caching behavior when relevant.

## REST Endpoint Checklist

- Use plural nouns.
- Keep methods semantically correct.
- Do not use verbs in resource URLs unless modeling a real command/action is unavoidable.
- Use `/api/v1/` for public versioned APIs.
- Return `201 Created` after creation.
- Return `204 No Content` for successful deletes without a body.
- Return `400` for malformed requests.
- Return `401` for invalid credentials.
- Return `403` for authenticated but unauthorized users.
- Return `404` when the resource does not exist or must not be revealed.
- Return `409` for conflicts.
- Return `429` for rate limits.
- Keep error bodies consistent.
- Never put sensitive data (passwords, API keys, tokens) in the URL path or query parameters.
- Use the request body or a secure header for sensitive attributes.

## GraphQL Checklist

- Schema mirrors the business domain.
- Mutations use input types.
- List fields are paginated.
- Query depth and complexity limits exist.
- Resolver-level or field-level authorization exists.
- Errors include message, path, and useful custom code.
- Sensitive fields do not leak through nested relationships.

## gRPC Checklist

- Proto contracts are stable and reviewed.
- Message names use domain language.
- Streaming is used only where it solves a real flow.
- Deadlines/timeouts are defined.
- Retries are safe and idempotent where enabled.
- Error codes map cleanly to client behavior.

## Code Maintainability Checklist

- Keep transport handlers thin.
- Move business logic into services/use cases.
- Use DTOs or schema validators for input boundaries.
- Never trust client-supplied IDs, roles, scopes, or prices.
- Centralize authentication middleware.
- Centralize authorization helpers/policies where possible.
- Keep error handling consistent.
- Avoid leaking stack traces or internal IDs to clients.
- Use request IDs for traceability.
- Keep database access behind repositories or query modules if the codebase already uses that pattern.
- Prefer existing project conventions over introducing a new architecture.

## Security Checklist

- Require HTTPS for credentials and tokens.
- Store API key hashes, not raw API keys.
- Store refresh tokens in HTTP-only secure cookies.
- Keep access tokens short-lived.
- Validate JWT issuer, audience, signature, and expiration.
- Check authorization for every resource access, not only at route level.
- Apply endpoint, user, IP, and global rate limits.
- Restrict CORS to approved origins.
- Use CSRF tokens for cookie-authenticated state changes.
- Escape or sanitize user-generated content.
- Use parameterized queries or ORM APIs.
- Add WAF or gateway rules for public APIs.
- Hide admin/internal APIs behind private networks or VPN where possible.
- Log suspicious auth and rate-limit events.

## Testing Checklist

- Happy path tests for every operation.
- Validation failure tests.
- Missing auth tests.
- Invalid auth tests.
- Authenticated but forbidden tests.
- Resource ownership tests.
- Pagination boundary tests.
- Filtering and sorting tests.
- Idempotency and retry tests for create/update flows.
- Rate-limit tests where practical.
- Injection attempts for risky query inputs.
- CSRF tests for cookie-authenticated writes.
- XSS tests for stored user content.

## Documentation Checklist

- State the API style and version.
- Show auth requirements.
- Show request and response examples.
- Show error examples.
- Document rate limits.
- Document pagination.
- Document deprecation policy.
- Document breaking changes.

## Anti-Pattern Avoidance Checklist

- No verb endpoints such as `/getUser` or `/createOrder`; use HTTP methods instead.
- No sensitive data in URL query parameters; use the request body or secure header.
- No deep nesting beyond two levels; flatten with query parameters.
- No `GET` requests that mutate state.
- No `POST` requests for deletes or replacements where `PUT`/`DELETE` is correct.
- Guard `POST` endpoints against accidental duplicate submission with idempotency keys or unique constraints.
- Apply POST/Redirect/GET pattern in browser flows to prevent form resubmission errors.
- Do not pass `Date` objects, functions, or `undefined` through JSON; serialize dates as ISO 8601 strings and use `null` for no-value attributes.
