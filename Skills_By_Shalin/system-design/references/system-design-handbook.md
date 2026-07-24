# System Design Handbook

## Table of Contents

1. Architecture and scaling foundations
2. Load balancing strategies
3. Database selection
4. API architectural styles
5. Network layers and protocols
6. Authentication and identity
7. Authorization models
8. API security techniques
9. API lifecycle and HTTP anatomy
10. Specific tools, examples, and clarifications

## 1. Architecture and Scaling Foundations

Senior system design work is about creating high-level architectures from scratch, optimizing storage and performance, and making trade-offs explicit.

### Single Server Setup

A single-server setup hosts the web application, database, and cache on one machine. When a user enters a domain name such as `app.demo.com`, DNS maps the domain to the server IP. The client then sends HTTP requests to that IP.

Typical responses:

- Browsers receive HTML, CSS, and JavaScript.
- Mobile apps usually call APIs and receive JSON, such as product ID, name, description, and price.

The simple setup is useful as a baseline, but it struggles under traffic and creates a single point of failure.

### Vertical Scaling

Vertical scaling, or scaling up, means adding CPU, RAM, storage, or network capacity to one machine.

Strengths:

- Simple to understand and operate.
- Good enough for moderate traffic.

Limits:

- Hardware has hard upper bounds.
- The system still has a single point of failure.
- Larger machines are often disproportionately expensive.

### Horizontal Scaling

Horizontal scaling, or scaling out, means adding more servers and distributing traffic across them.

Strengths:

- Better fault tolerance.
- Better scalability by adding more machines.
- One failed app server does not have to take the whole system down.

Costs:

- Requires a load balancer.
- Requires shared state decisions: external sessions, shared databases, distributed caches, object storage, queues, and idempotent operations.
- Introduces more deployment, monitoring, and consistency complexity.

## 2. Load Balancing Strategies

A load balancer sits between clients and servers and routes traffic so no single server is overwhelmed. It improves reliability by avoiding unhealthy servers.

### Algorithms

- Round Robin: Send requests sequentially across servers. Best when servers have similar capacity.
- Least Connections: Send traffic to the server with the fewest active connections. Useful when session lengths vary.
- Least Response Time: Prefer the fastest responding server while considering active connections.
- IP Hash: Hash the client IP so a user consistently reaches the same server. Useful when local server session state still exists.
- Weighted: Assign more traffic to stronger machines. Example: a server with 64 GB RAM receives more traffic than one with 16 GB.
- Geographical: Route traffic to the closest region, such as US East, US West, or Europe, to reduce latency.
- Consistent Hashing: Map users and servers onto a hash ring. When a server is added or removed, only a portion of keys move.

### Health Checks and Resilience

Load balancers continuously check server health. If a server goes offline, the load balancer stops routing traffic to it until it recovers.

Prevent load balancer SPOF by using:

- Redundant load balancers.
- Health monitoring for the load balancers themselves.
- Self-healing infrastructure that replaces failed load balancer instances.

### Load Balancer Types

- Software: NGINX, which can also act as a web server, and HAProxy, an open-source load balancer.
- Hardware: F5 and Citrix appliances.
- Cloud: AWS Elastic Load Balancing, Azure Load Balancer, and Google Cloud Load Balancing with auto-scaling and health monitoring.

## 3. Database Selection

Choose the database from access patterns, data shape, consistency requirements, relationship complexity, and operational needs.

### Relational SQL Databases

SQL databases store data in tables, columns, and rows, similar to spreadsheets. Examples include PostgreSQL, MySQL, Oracle, and SQLite.

Strengths:

- Well-structured data with clear relationships.
- Complex joins.
- Strong transaction guarantees.
- Mature query languages and tooling.

ACID compliance:

- Atomic: The whole transaction succeeds or fails as one unit. A bank transfer is the classic example.
- Consistent: Transactions leave the database in a valid state.
- Isolated: Concurrent transactions do not interfere incorrectly.
- Durable: Committed data survives crashes.

Use SQL for e-commerce, banking, finance, orders, inventory, and other structured domains where correctness matters.

### Non-Relational NoSQL Databases

NoSQL databases are useful for massive scale, flexible schemas, low latency, and specialized data models.

Types:

- Document Stores: MongoDB stores nested JSON-like documents. A user document can include embedded orders or preferences, reducing joins.
- Wide-Column Stores: Cassandra and Azure Cosmos DB are built for massive scale and high write throughput.
- Graph Databases: Neo4j and Amazon Neptune model entities and relationships. Amazon Neptune is useful for recommendation-style relationship queries.
- Key-Value Stores: Redis and Memcached keep values in RAM for extremely fast reads and writes.

Use NoSQL when the domain has flexible structure, extremely high volume, low-latency access, large write scale, or relationship traversal patterns that do not fit relational joins.

## 4. API Architectural Styles

APIs are contracts between components. Select the style according to client needs, performance constraints, team maturity, and communication pattern.

### REST

REST is resource-based and stateless. URLs use nouns and HTTP methods define actions.

Good resource names:

- `/products`
- `/products/123`
- `/api/v1/products`

Avoid verb-style URLs such as `/getProducts`.

Methods:

- `GET`: Retrieve data. Safe and idempotent.
- `POST`: Create data.
- `PUT`: Replace the entire resource.
- `PATCH`: Partially update a resource.
- `DELETE`: Remove a resource.

Status code families:

- 200s: Success, such as `200 OK`, `201 Created`, `204 No Content`.
- 300s: Redirection.
- 400s: Client errors, such as `400 Bad Request`, `401 Unauthorized`, `404 Not Found`.
- 500s: Server errors.

REST best practices:

- Use plural resource names.
- Use explicit versioning such as `/api/v1/`.
- Support pagination, filtering, and sorting to avoid excessive bandwidth.
- Keep requests stateless.

### GraphQL

GraphQL solves overfetching and underfetching. The client sends a query to one endpoint, usually `/graphql`, specifying exactly the fields needed.

Operations:

- Queries read data.
- Mutations create or update data.
- Subscriptions provide realtime updates.

Schema guidance:

- Model the schema after the business domain.
- Use designated input types for mutations.
- Add query depth limits to prevent malicious or accidentally expensive nested queries.

Error handling:

- GraphQL usually returns `200 OK` even on logical failure.
- Failures are embedded in an `errors` array.
- A useful error object includes `message`, custom status code, and `path`, such as the failed schema field `user`.

GraphQL evolves without explicit URL versioning, but schema governance becomes critical.

### gRPC

gRPC is a high-performance Remote Procedure Call framework from Google. It uses Protocol Buffers and HTTP/2. It supports bidirectional streaming and is especially strong for internal microservice-to-microservice communication.

Use gRPC when services are internal, strongly typed contracts matter, latency matters, and clients can support Protocol Buffers.

## 5. Network Layers and Protocols

### Application Layer

HTTP and HTTPS:

- HTTP follows request/response.
- HTTPS adds TLS/SSL encryption, improving confidentiality, integrity, and SEO for web platforms.
- Polling over HTTP can waste bandwidth for realtime workloads.

WebSockets:

- Start with a handshake, then keep a continuous bidirectional connection open.
- Let the server push data to the client.
- Fit live chat, collaboration, realtime dashboards, and games.

AMQP:

- Supports asynchronous work through message brokers.
- A producer places work on a queue.
- A consumer pulls work when it has capacity.
- Exchange types route messages to queues, including direct, fan out, and topic-based patterns.

### Transport Layer

TCP:

- Reliable and connection-oriented.
- Uses a three-way handshake: client sends sync, server syncs and acknowledges, client acknowledges.
- Guarantees ordered delivery and resends lost packets.
- Slower but correct for payments, emails, banking, and user data.

UDP:

- Fast and lightweight.
- No connection setup.
- No guarantee of delivery or ordering.
- Useful for video calls, live streaming, online gaming, and cases where dropping a frame is better than pausing.

## 6. Authentication and Identity

Authentication proves who the user is.

### Basic and Digest

Basic sends Base64 credentials on every request. Digest uses MD5-style challenge response. Treat both as outdated for modern API design unless dealing with legacy constraints.

### API Keys

API keys are unique strings, often sent through an `X-API-KEY` header.

Best practices:

- Store only a key hash, not the raw key.
- Store allowed scopes with the key.
- Return `401 Unauthorized` for an invalid key.
- Return `400 Bad Request` if a required key is completely missing from the request.
- Support rotation and revocation.

### Session-Based Authentication

Session auth is stateful. The user logs in, the server stores a session ID, and the client receives a cookie.

Pitfalls:

- In-memory session variables log everyone out if the server crashes.
- File-system sessions are possible but not scalable.
- Redis is preferred because it supports shared lookup and automatic key expiration.

### JWT

JSON Web Tokens are signed JSON bearer tokens carrying claims such as user ID, roles, and expiration. The server validates the signature without a database lookup, which scales well.

Clarifications:

- Bearer token is the access pattern: whoever holds the token gets access.
- JWT is a token format, commonly used as a bearer token.
- A token carries identity and claims; it is not an authorization model.

### Access and Refresh Tokens

Use short-lived access tokens for API calls, often 15 minutes to 1 hour. Use longer-lived refresh tokens to obtain new access tokens. Store refresh tokens in HTTP-only secure cookies to reduce XSS exposure.

### OAuth2 and OpenID Connect

OAuth2 is an authorization framework. It delegates limited access to resources and returns an access token.

OpenID Connect builds on OAuth2 to add authentication. It returns an ID token containing identity claims.

OAuth2 and OIDC usually use an authorization code flow:

1. The provider returns an authorization code to the app.
2. The backend exchanges the code for access and ID tokens.

### SSO and SAML

Single Sign-On is a user experience pattern: one identity provider login grants access to multiple services.

Modern SSO commonly uses OpenID Connect and JWTs. Legacy enterprise systems such as Salesforce may use SAML, an XML-based protocol that returns a SAML assertion.

## 7. Authorization Models

Authorization determines what an authenticated user can do.

### RBAC

Role-Based Access Control assigns broad permissions through roles.

Example:

- Admin: read, write, delete.
- Editor: read, write.
- Viewer: read only.

### ABAC

Attribute-Based Access Control evaluates dynamic conditions:

- User attributes, such as department.
- Resource attributes, such as internal-only classification.
- Environment attributes, such as time of day, location, VPN state, or device type.

ABAC is flexible but harder to reason about and test.

### ACL

Access Control Lists are resource-specific permission lists. A Google Drive document with explicit read/comment/write users is a typical ACL-style example.

## 8. API Security Techniques

APIs expose databases and privileged workflows. Protect them as first-class attack surfaces.

- Rate Limiting: Stop brute force and DDoS pressure by limiting requests per endpoint, IP, user, or globally.
- Overall Rate Limit: IP-only limits can be bypassed by botnets with many IPs, so add a global circuit breaker for massive traffic spikes.
- CORS: Restrict which browser origins may call the API.
- Parameterized Queries and ORMs: Prevent SQL and NoSQL injection.
- WAF: Web Application Firewalls, such as AWS WAF, inspect payloads and block recognizable attack patterns like suspicious SQL keywords or strange HTTP methods.
- VPN: Keep internal/admin APIs private behind a company network.
- CSRF Tokens: Prevent a malicious site from using a logged-in browser's cookies to perform unwanted actions.
- XSS Prevention: Prevent malicious JavaScript from being stored in public text fields and executed in another user's browser.

## 9. API Lifecycle and HTTP Anatomy

### API Design Approaches

- Top-Down: Start with requirements and workflows, then define endpoints and operations. Common in interviews.
- Bottom-Up: Start with existing data models and system capabilities. Common when building around an established database.
- Contract-First: Define exact requests and responses before implementation.

### API Lifecycle

1. Design: Discuss requirements and outcomes.
2. Development and Testing: Implement locally and test behavior.
3. Deployment and Monitoring: Release to staging or production and watch performance and errors.
4. Maintenance: Keep the API intuitive and easy to evolve.
5. Deprecation and Retirement: Phase out older versions such as `/api/v1/` after migration to `/api/v2/`.

### HTTP Request Anatomy

Requests include:

- Method, such as `GET` or `POST`.
- Resource URL, such as `/api/products`.
- HTTP version.
- Host domain.
- Headers, such as `Authorization`, `Accept`, and `User-Agent`.
- Optional body for create/update operations.

### HTTP Response Anatomy

Responses include:

- HTTP version.
- Status code, such as `200 OK` or `500 Server Error`.
- Headers, such as `Content-Type: application/json`.
- Cache instructions, such as `Cache-Control`.
- Optional body.

## 10. Specific Tools, Examples, and Clarifications

### Tools Mentioned

Databases:

- PostgreSQL, MySQL, Oracle, SQLite.
- MongoDB.
- Cassandra.
- Azure Cosmos DB.
- Neo4j.
- Amazon Neptune.
- Redis.
- Memcached.

Load balancers:

- NGINX.
- HAProxy.
- F5.
- Citrix.
- AWS Elastic Load Balancing.
- Azure Load Balancer.
- Google Cloud Load Balancing.

### Developer Confusions to Correct

- Postman groups many mechanisms under an authorization dropdown, but authentication methods and authorization frameworks are different things.
- Bearer token and JWT are not identical. Bearer is a possession pattern; JWT is a signed token format.
- Tokens are not authorization models. RBAC, ABAC, and ACL are authorization models.

### Analogies

- SQL tables, rows, and columns are like spreadsheets.
- ACID atomicity is like a bank transfer: either the whole transfer succeeds or none of it does.

### Topics Mentioned as Next Steps

The course treats these as further study areas:

- Caching and CDNs.
- Big data processing.
- Real-world system design case studies such as WhatsApp, Spotify, and TinyURL.
- System design interview strategy.
