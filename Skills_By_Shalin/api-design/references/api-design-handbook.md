# API Design Handbook

## Table of Contents

1. API design process
2. Mental model and foundational concepts
3. HTTP request and response anatomy
4. REST design
5. JSON data format
6. GraphQL design
7. gRPC design
8. Realtime and async protocols
9. Authentication and identity
10. Authorization models
11. API security
12. Lifecycle, maintenance, and deprecation
13. Anti-patterns and pitfalls
14. Quick-reference cheat sheet
15. Self-check questions
16. Glossary

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

## 2. Mental Model and Foundational Concepts

### The API as an Interface

Think of an API as the specific surface area of potential interactions exposed by an application. It acts as a contract that hides internal complexity (such as database schemas) while exposing defined capabilities to consumers. The internal implementation can change freely as long as the contract remains stable.

### The API as a Decoupler

APIs enable two primary architectural structures:

- **Enhancement**: Using a third-party API (for example, Google Maps) to add functionality to your app without building it from scratch.
- **Product Architecture**: Splitting a single product into a front-end and a backend that communicate via API, allowing separate codebases, independent deployments, and managed data access.

### Architectural Styles at a Glance

| Style | Transport | Format | Best For |
|---|---|---|---|
| REST | HTTP | JSON | Public resource contracts, most web APIs |
| GraphQL | HTTP | JSON | Flexible client field selection, BFF layer |
| gRPC | HTTP/2 | Protobuf | Internal high-performance microservice calls |
| WebSockets | WS | Any | Realtime bidirectional push (chat, notifications) |
| SOAP | HTTP | XML | Legacy/enterprise integrations |
| AMQP/Queues | TCP | Any | Async background jobs and event-driven work |

### Defining a Resource Interface (Step-by-Step)

Use this procedure when designing a new resource endpoint set:

1. **Define the base and version**: `[base URL]/api/[version]` — for example `mysite.com/api/v1`.
2. **Determine parentage/nesting**: Decide if the resource is bucketed under a parent (for example `/posts/{id}/comments`) or exists as a top-level collection.
3. **Map methods to actions**:
   - Retrieve all: `GET /posts/{id}/comments`
   - Create new: `POST /posts/{id}/comments` (data in request body)
   - Retrieve specific: `GET /comments/{id}`
   - Full replace/update: `PUT /comments/{id}` (requires full document in body)
   - Partial update: `PATCH /comments/{id}` (specific attributes only)
   - Remove: `DELETE /comments/{id}`
4. **Define specialized interactions**: For sub-actions like replies use `POST /comments/{id}/reply`; for alternate parent access use `GET /users/{id}/comments`.

## 3. HTTP Request and Response Anatomy

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

## 4. REST Design

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

### Nesting vs. Filtering Decision Rule

Choose between nested paths and query parameters based on access complexity:

- **Use nested paths** for simple data with few access patterns and a clear parent-child relationship, for example `/posts/1/comments`.
- **Use query parameters** for complex access patterns, multiple filter dimensions, or when the same resource is accessed from many different parents, for example `/comments?post_id=123&sort=desc`.
- **Over-nesting anti-pattern**: Deep hierarchies such as `/posts/{id}/comments/{id}/replies/{id}` become hard to read and maintain. Flatten beyond two levels by switching to query parameters.

### Identification vs. Filtering

- Use the **path** to identify a specific resource, typically via ID: `/users/42`.
- Use **query parameters** for optional modifiers such as filtering, sorting, and pagination: `/users?role=admin&page=2`.

### Security Partitioning in URLs

Never put sensitive data — passwords, API keys, tokens, or secrets — in the URL. URLs are stored in browser history, proxy logs, and server access logs. Place sensitive attributes in the **request body** or in **request headers** instead.

## 5. JSON Data Format

JSON (JavaScript Object Notation) is the standard data exchange format for REST APIs.

### Supported Types

| Type | Example |
|---|---|
| String | `"hello"` |
| Number | `42`, `3.14` |
| Boolean | `true`, `false` |
| Object | `{ "key": "value" }` |
| Array | `[1, 2, 3]` |
| Null | `null` — use this when an attribute has no value |

### Types NOT Supported by JSON

- **Date/DateTime**: There is no native date type. Represent dates as ISO 8601 strings: `"2024-01-15T10:30:00Z"`.
- **Functions**: Functions cannot be serialized; JSON is data only.
- **`undefined`**: JavaScript's `undefined` is not a JSON value. Use `null` for missing values.
- **Binary**: Binary data must be Base64-encoded before embedding in JSON.

Never assume a JSON object is identical to a JavaScript object — these type gaps cause runtime errors when serializing and deserializing.

## 6. GraphQL Design

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

## 7. gRPC Design

gRPC is a high-performance RPC framework. It uses Protocol Buffers and HTTP/2, including bidirectional streaming.

Use gRPC for:

- Internal microservice communication.
- Strongly typed contracts.
- Low-latency service calls.
- Streaming between services.

Avoid gRPC as the default public browser API unless the client ecosystem supports it.

## 8. Realtime and Async Protocols

### WebSockets

WebSockets keep a bidirectional connection open after one handshake. Use for live chat, collaborative editing, realtime notifications, games, and live dashboards.

### AMQP and Message Brokers

AMQP supports async work through queues. A producer adds tasks; a consumer processes them when capacity is available.

Exchange types include:

- Direct: one-to-one routing.
- Fan out: broadcast-style routing.
- Topic: pattern-based routing.

Use queues for background jobs, order processing, email sending, media processing, and traffic smoothing.

## 9. Authentication and Identity

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

## 10. Authorization Models

Authorization determines what a known user may do.

- RBAC: Broad roles such as Admin, Editor, Viewer.
- ABAC: Dynamic rules based on user attributes, resource attributes, and environmental context such as time, location, VPN, or device type.
- ACL: Resource-specific permission lists, such as per-document sharing permissions.

Tokens carry identity and claims. They are not authorization models. Authorization logic must still check roles, attributes, or resource permissions.

## 11. API Security

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

## 12. Lifecycle, Maintenance, and Deprecation

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

## 13. Anti-Patterns and Pitfalls

### Non-Idempotent POST Duplicates

`POST` is not idempotent — submitting the same request twice creates two records. Guard against accidental duplicates by using idempotency keys, unique constraints at the database level, or redirect-after-post patterns in browser flows.

### Confirm Form Resubmission

When a browser page loaded via a `POST` request is refreshed, the browser warns the user that they are about to repeat a non-idempotent action. Prevent this by redirecting to a `GET` after a successful `POST` (the POST/Redirect/GET pattern).

### Over-Nesting

Deep URL hierarchies such as `/posts/{id}/comments/{id}/replies/{id}/likes` become hard to read, document, and maintain. Flatten beyond two levels of nesting by switching to query parameters or making the nested resource top-level.

### Verb Endpoints

Endpoints like `/getUser`, `/deleteComment`, or `/createOrder` mix HTTP semantics with URL verbs. Use nouns in the path and rely on HTTP methods to convey the action.

### Sensitive Data in URLs

Putting passwords, API keys, or session tokens in query parameters exposes them in browser history, proxy logs, web server logs, and referrer headers. Always use the request body or a secure header.

### Wrong Method Semantics

Using `GET` requests that mutate data, or `POST` requests that delete records, violates consumer expectations and breaks caching, idempotency, and tooling assumptions. Follow standard HTTP method semantics strictly.

### JSON Type Misconceptions

Assuming JSON supports JavaScript's `Date` objects, functions, or `undefined` leads to silent data loss or runtime errors. Serialize dates to ISO 8601 strings and never attempt to pass functions through JSON.

## 14. Quick-Reference Cheat Sheet

### HTTP Methods

| Method | Action | Idempotent? | Data Location |
|---|---|---|---|
| `GET` | Retrieve | Yes | URL (path / query params) |
| `POST` | Create | No | Request body |
| `PUT` | Replace (full) | Yes | Request body |
| `PATCH` | Partial update | Typically yes | Request body |
| `DELETE` | Remove | Yes | URL (path) |

### Response Code Categories

| Range | Meaning | Common Examples |
|---|---|---|
| 2xx | Success | 200 OK, 201 Created, 204 No Content |
| 3xx | Redirect | 301 Moved Permanently, 302 Found |
| 4xx | Client error | 400 Bad Request, 401 Unauthorized, 403 Forbidden, 404 Not Found, 409 Conflict, 429 Too Many Requests |
| 5xx | Server error | 500 Internal Server Error |

### Data Placement Rules

| Data | Where to Put It |
|---|---|
| Resource identity (ID) | Path segment: `/users/42` |
| Filtering, sorting, pagination | Query parameters: `?sort=price_asc&page=2` |
| Create/update payload | Request body |
| Sensitive credentials | Request body or secure header — never URL |
| Content format metadata | Headers: `Content-Type: application/json` |

## 15. Self-Check Questions

Use these to verify understanding before finalizing an API design:

1. Explain the difference between `PUT` and `PATCH` in terms of payload requirements.
2. Why is it dangerous to put a password in a query parameter?
3. What is the main advantage of gRPC/Protobufs over REST/JSON for internal services?
4. If a `DELETE` request is executed 10 times on the same resource ID, why is it considered idempotent?
5. How does a GraphQL endpoint differ from a REST API in terms of URL structure and data fetching?
6. Write a RESTful URL for getting the 5th page of blue cars sorted by price ascending.
7. What JSON value should you use when an attribute specifically has no value?
8. When should you use nested paths versus query parameters for filtering?
9. What is the POST/Redirect/GET pattern and why does it exist?
10. Name three pieces of data that must never appear in a URL.

## 16. Glossary

| Term | Definition |
|---|---|
| API (Application Programming Interface) | The surface area of potential interactions exposed by an application. |
| Interface | The capabilities an application exposes to the outside world. |
| Resource / Entity | A thing you interact with via an API, for example a User, Comment, or Product. |
| Endpoint | The combination of an HTTP method and a path URL. |
| Idempotency | The property where executing an operation multiple times yields the same result as executing it once. |
| Protobufs (Protocol Buffers) | A method of serializing structured data into a compact binary format for fast transfer, used by gRPC. |
| Query Parameter | Key-value pairs following a `?` in a URL, used for filtering, sorting, and pagination. |
| JSON (JavaScript Object Notation) | The standard text-based data exchange format for REST APIs. |
| Base URL | The root address shared by all endpoints of an API, for example `mysite.com/api/v1`. |
| CORS | Cross-Origin Resource Sharing, a browser security mechanism controlling which origins may call an API. |
| CSRF | Cross-Site Request Forgery, an attack where a malicious site triggers state-changing requests using a victim's active session. |
| JWT (JSON Web Token) | A signed, self-contained token carrying identity claims, used as a bearer token for API authentication. |
| OAuth2 | An authorization framework that grants third-party apps limited, scoped access to resources. |
| Rate Limiting | Enforcing a maximum number of API requests per time window to prevent abuse and protect backend resources. |
| Pagination | Splitting a large list result into pages so the API returns a manageable subset at a time. |
| WAF (Web Application Firewall) | A layer that inspects HTTP traffic and blocks malicious requests before they reach the application. |
