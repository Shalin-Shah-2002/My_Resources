# System Design Checklists

## Interview Answer Checklist

Use this checklist to keep architecture answers complete without wandering.

1. Restate the problem and clarify the product goal.
2. Define functional requirements.
3. Define non-functional requirements: scale, latency, availability, consistency, durability, security, and cost.
4. Estimate traffic and storage if the prompt implies scale.
5. Start with a simple architecture.
6. Add load balancing and horizontal scaling only where the simple design breaks.
7. Pick databases from access patterns, not from preference.
8. Define APIs and protocols.
9. Add auth, authorization, and abuse protection.
10. Explain failure handling.
11. Discuss observability.
12. Call out trade-offs and future improvements.

## Architecture Review Checklist

Look for:

- Single points of failure in app servers, load balancers, databases, caches, queues, or regions.
- Unbounded reads or writes.
- Missing pagination, filtering, or backpressure.
- Stateful app servers that block horizontal scaling.
- Session state that should move to Redis or another shared store.
- Database choice mismatched to query patterns.
- Missing health checks.
- Missing rate limits.
- Missing CORS, CSRF, XSS, and injection defenses.
- Missing authorization checks at the resource level.
- No monitoring, alerting, or rollback story.
- No deprecation path for APIs.

## Storage Decision Guide

Prefer SQL when:

- Relationships are clear and important.
- Transactions must be correct.
- Joins are central to the workflow.
- Data shape is stable.
- Examples include banking, orders, invoices, inventory, and accounting.

Prefer document stores when:

- Data is nested or semi-structured.
- Reads often need one aggregate document.
- Schema needs to evolve frequently.

Prefer wide-column stores when:

- Writes are massive.
- Access patterns are known.
- Horizontal scale is more important than ad hoc querying.

Prefer graph databases when:

- Relationship traversal is the core product value.
- Recommendations, social graphs, fraud links, or knowledge graphs dominate.

Prefer key-value stores when:

- Low latency is the main need.
- Access is by key.
- Values can expire.
- Data can be recomputed or is safe to cache.

## Load Balancing Decision Guide

- Use round robin for similar servers and simple traffic.
- Use least connections when sessions vary in duration.
- Use least response time when performance varies dynamically.
- Use IP hash when sticky routing is needed.
- Use weighted routing when hardware capacity differs.
- Use geographical routing to reduce user latency across regions.
- Use consistent hashing when stable key ownership matters and membership changes are expected.

## Protocol Decision Guide

- Use REST for public CRUD-style APIs, broad compatibility, and simple resource contracts.
- Use GraphQL when clients need flexible field selection and multiple UI surfaces over the same graph.
- Use gRPC for internal service calls that need strong contracts and high performance.
- Use WebSockets for realtime bidirectional communication.
- Use AMQP or queues for async background work and load smoothing.
- Use TCP when correctness and ordering matter.
- Use UDP when low latency matters more than perfect delivery.

## Security Review Checklist

- Are credentials or tokens sent only over HTTPS?
- Are access tokens short-lived?
- Are refresh tokens stored in HTTP-only secure cookies?
- Are API keys stored as hashes with scopes?
- Are sessions stored in Redis or another shared store with expiration?
- Is authorization enforced with RBAC, ABAC, ACL, or a clear combination?
- Are there endpoint, user, IP, and global rate limits?
- Are browser origins restricted with CORS?
- Are all database queries parameterized or ORM-backed?
- Is there CSRF protection for cookie-authenticated state changes?
- Is user-generated content escaped or sanitized to prevent XSS?
- Are internal/admin APIs behind VPN or private network controls?
- Is a WAF inspecting suspicious payloads?
