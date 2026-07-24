# API Design Handbook

## Table of Contents

1. API design process
2. HTTP request and response anatomy
3. REST design
4. GraphQL design
5. gRPC design
6. Realtime and async protocols
7. Authentication and identity
8. Authorization models
9. API security
10. Lifecycle, maintenance, and deprecation

## 1. API Design Process

APIs are contracts. A good API is designed from user workflows and maintained as a product surface, not treated as accidental controller output.

### Design Approaches

- Top-Down: Start from user needs and workflows, then define endpoints or operations. This is the most common system design interview approach.
- Bottom-Up: Start from existing data models and capabilities. This fits legacy systems or company databases that already exist.
- Contract-First: Define exact request and response contracts before implementation. This keeps frontend, backend, mobile, and partner teams aligned.

### Lifecycle

1. Design: Gather requirements, consumers, data, permissions, and expected outcomes.
2. Development and Testing: Implement locally with validation, tests, and contract examples.
3. Deployment and Monitoring: Release to staging or production and monitor latency, errors, traffic, and abuse.
4. Maintenance: Keep the API simple, intuitive, documented, and easy for developers to maintain.
5. Deprecation and Retirement: Phase out older versions after communication and migration, such as replacing `/api/v1/` with `/api/v2/`.

## 2. HTTP Request and Response Anatomy

### Request

An HTTP request includes:

- Method: `GET`, `POST`, `PUT`, `PATCH`, `DELETE`, and others.
- Resource URL: for example, `/api/products`.
- HTTP version.
- Host domain.
- Headers such as `Authorization`, `Accept`, `Content-Type`, and `User-Agent`.
- Optional body for create and update operations.

### Response

An HTTP response includes:

- HTTP version.
- Status code such as `200 OK`, `201 Created`, `400 Bad Request`, or `500 Server Error`.
- Headers such as `Content-Type: application/json`.
- Cache instructions such as `Cache-Control`.
- Optional response body.

## 3. REST Design

REST is stateless and resource-based. URLs name resources, and HTTP methods express actions.

### Resource Naming

Use plural nouns:

- `GET /products`
- `GET /products/{id}`
- `POST /products`
- `PATCH /products/{id}`
- `DELETE /products/{id}`

Avoid verb endpoints such as `/getProducts`.

Use explicit versioning for public APIs:

- `/api/v1/products`
- `/api/v2/products`

### HTTP Methods

- `GET`: Retrieve data. Safe and idempotent.
- `POST`: Create a new resource or trigger a non-idempotent action.
- `PUT`: Replace the entire resource. Idempotent when used correctly.
- `PATCH`: Partially update a resource.
- `DELETE`: Remove a resource. Idempotent from the client's perspective when repeated deletes still result in absence.

### Status Codes

Use status codes consistently:

- `200 OK`: Successful read or update with a response body.
- `201 Created`: New resource created.
- `204 No Content`: Successful operation with no body.
- `400 Bad Request`: Malformed request or missing required auth material such as a required API key header.
- `401 Unauthorized`: Authentication failed, credentials invalid, token invalid, or API key invalid.
- `403 Forbidden`: Authenticated user lacks permission.
- `404 Not Found`: Resource does not exist or should not be revealed.
- `409 Conflict`: State conflict or uniqueness conflict.
- `422 Unprocessable Entity`: Valid syntax but semantic validation failed, if the API chooses this convention.
- `429 Too Many Requests`: Rate limit exceeded.
- `500 Internal Server Error`: Unexpected server failure.

### Pagination, Filtering, and Sorting

Avoid unbounded list endpoints.

Common patterns:

- Page and limit: `/products?page=2&limit=50`
- Offset and limit: `/products?offset=100&limit=50`
- Cursor: `/products?cursor=abc&limit=50`
- Filtering: `/products?category=books&in_stock=true`
- Sorting: `/products?sort=price_asc`

Cursor pagination is preferred for large, changing datasets.

### REST Contract Quality

Design each endpoint with:

- Stable request and response examples.
- Validation rules.
- Auth requirements.
- Error shape.
- Idempotency expectations.
- Rate limits.
- Cache behavior where relevant.

## 4. GraphQL Design

GraphQL was created to solve overfetching and underfetching. Clients call one endpoint, usually `/graphql`, and request exactly the fields they need.

### Core Concepts

- Queries read data.
- Mutations create or update data.
- Subscriptions provide realtime updates.
- Schema defines the contract.

### Best Practices

- Mirror the business domain model in the schema.
- Use designated input types for mutations.
- Avoid leaking database internals into public schema names.
- Add query depth limits and complexity limits.
- Add pagination for list fields.
- Keep authorization checks at field/resource boundaries where sensitive data can appear.

### Errors

GraphQL often returns `200 OK` even when a specific operation fails. Put failure details in the `errors` array.

A useful error object includes:

- `message`
- custom status code such as `404`
- `path`, pointing to the failed field such as `user`

## 5. gRPC Design

gRPC is a high-performance RPC framework. It uses Protocol Buffers and HTTP/2, including bidirectional streaming.

Use gRPC for:

- Internal microservice communication.
- Strongly typed contracts.
- Low-latency service calls.
- Streaming between services.

Avoid gRPC as the default public browser API unless the client ecosystem supports it.

## 6. Realtime and Async Protocols

### WebSockets

WebSockets keep a bidirectional connection open after one handshake. Use for live chat, collaborative editing, realtime notifications, games, and live dashboards.

### AMQP and Message Brokers

AMQP supports async work through queues. A producer adds tasks; a consumer processes them when capacity is available.

Exchange types include:

- Direct: one-to-one routing.
- Fan out: broadcast-style routing.
- Topic: pattern-based routing.

Use queues for background jobs, order processing, email sending, media processing, and traffic smoothing.

## 7. Authentication and Identity

Authentication proves who the user is.

### API Keys

API keys are often sent as `X-API-KEY`.

Implementation rules:

- Store only key hashes, never raw keys.
- Store scopes and metadata.
- Return `400 Bad Request` if a required key header is missing.
- Return `401 Unauthorized` if a key is present but invalid.
- Support rotation and revocation.

### Sessions

Session auth is stateful. Store sessions in Redis or a shared store with expiration. Avoid in-memory process sessions for production because a crash logs users out and prevents easy horizontal scaling. Avoid file-system sessions for scalable systems.

### JWT and Bearer Tokens

JWT is a signed JSON token format. Bearer token is the access pattern: whoever holds the token can use it.

JWT claims commonly include:

- subject/user ID
- roles or scopes
- expiration
- issuer
- audience

Use short-lived access tokens for API calls. Use longer-lived refresh tokens, stored in HTTP-only secure cookies, to issue new access tokens.

### OAuth2 and OpenID Connect

OAuth2 is authorization: it grants a third-party app limited access to resources and returns an access token.

OpenID Connect adds authentication on top of OAuth2 and returns an ID token with identity claims.

Typical code flow:

1. Identity provider returns an authorization code.
2. Backend exchanges the code for access and ID tokens.

### SSO and SAML

SSO is a user experience pattern that lets one identity provider login unlock multiple services. Modern SSO commonly uses OpenID Connect. Legacy enterprise SSO may use SAML, which returns an XML SAML assertion.

## 8. Authorization Models

Authorization determines what a known user may do.

- RBAC: Broad roles such as Admin, Editor, Viewer.
- ABAC: Dynamic rules based on user attributes, resource attributes, and environmental context such as time, location, VPN, or device type.
- ACL: Resource-specific permission lists, such as per-document sharing permissions.

Tokens carry identity and claims. They are not authorization models. Authorization logic must still check roles, attributes, or resource permissions.

## 9. API Security

### Rate Limiting

Apply limits by endpoint, user, IP, and system-wide threshold. IP-only limits are weak against botnets because each bot can get its own allowance. Add an overall traffic limit or circuit breaker for severe spikes.

Return `429 Too Many Requests` when limits are exceeded.

### CORS

CORS is a browser mechanism that controls which origins may call the API. Restrict it to approved frontend domains.

### Injection Prevention

Use parameterized queries or ORM query builders. Never concatenate untrusted input into SQL or NoSQL query structures.

### WAF

Use a Web Application Firewall such as AWS WAF to inspect payloads and block suspicious patterns, including SQL keywords, strange HTTP methods, or known attack signatures.

### VPN and Private APIs

Keep internal dashboards, admin APIs, and private operations behind VPN or private network controls when possible.

### CSRF

Cookie-authenticated state-changing requests need CSRF protection so a malicious website cannot silently use a user's active browser session.

### XSS

Escape output and sanitize user-generated content so submitted JavaScript cannot execute in another user's browser.

## 10. Lifecycle, Maintenance, and Deprecation

Good APIs age deliberately.

Maintenance practices:

- Keep controllers/resolvers thin.
- Validate input at the boundary.
- Separate transport, application, domain, and persistence concerns.
- Use stable error shapes.
- Add request IDs or trace IDs.
- Log security-relevant failures without leaking secrets.
- Monitor latency, error rates, status code distribution, and rate-limit events.
- Document deprecation windows and migration paths.

Deprecation practices:

- Announce old version retirement.
- Add response headers or docs warnings.
- Provide migration examples.
- Keep old versions long enough for real clients to migrate.
- Remove only after traffic and dependency review.
