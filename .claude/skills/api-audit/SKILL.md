---
name: api-audit
description: Audit application and service APIs for design quality, end-to-end test readiness, and agentic workflow support.
allowed-tools: Read, Glob, Grep, Bash
---

Audit the API surface of this project. Discover endpoints by reading route registrations, handler files, OpenAPI/Swagger specs, proto definitions, GraphQL schemas, or any other API definition mechanism present in the codebase. Adapt your approach to whatever language and framework the project uses.

Produce a report with three sections. For each section, list specific findings with file paths and line numbers. Categorize each finding as one of:

- **PASS** — meets the standard
- **WARN** — deviation that may be intentional but should be reviewed
- **FAIL** — violates the standard and should be fixed

At the end, provide a summary count of PASS/WARN/FAIL per section.

---

## 1. API Design

Evaluate against the [Google API Design Guide](https://cloud.google.com/apis/design) and [Microsoft REST API Guidelines](https://github.com/microsoft/api-guidelines). Use Google as the primary authority for resource modeling and method semantics. Use Microsoft for operational patterns.

### Resource naming
- Resources are nouns, collections are plural (`/publishers`, `/books`)
- URLs use kebab-case or camelCase consistently — never mixed
- IDs are URL-safe (lowercase, alphanumeric, hyphens)
- No verbs in resource paths — actions use sub-resources or custom methods (`/books/123:archive`, not `/archiveBook`)
- Nesting reflects ownership (`/publishers/123/books`), not arbitrary grouping

### Standard methods
- GET for reads (no body, no side effects)
- POST for creates
- PUT for full replacement, PATCH for partial updates — not interchangeable
- DELETE returns 204 regardless of prior existence
- Collection GET returns an array, never a bare object

### Error handling
- Consistent error envelope across all endpoints: `{ error: { code, message } }` at minimum
- Use standard HTTP status codes correctly (400 for bad input, 401 for unauthenticated, 403 for unauthorized, 404 for not found, 409 for conflicts, 422 for validation, 429 for rate limiting, 500 for server errors)
- Error messages are actionable — they tell the caller what went wrong and how to fix it
- No stack traces or internal details leak in production error responses
- Permission checks happen before existence checks (return 403, not 404, unless revealing existence is itself a leak)

### Pagination
- Collection endpoints support pagination via `page_size`/`page_token` (Google) or `top`/`skip` (Microsoft) — pick one pattern and use it everywhere
- Response includes a `next_page_token` or equivalent when more results exist
- Pagination parameters don't change between pages (same filters, same ordering)

### Versioning
- API version is explicit (URL path prefix `/v1/` or query param `?api-version=`)
- No breaking changes within a version

### Idempotency
- PUT and DELETE are idempotent
- POST endpoints that create resources support an idempotency key or return 409 on duplicate

### Concurrency control
- Mutable resources return an `ETag` or `last-modified` header
- Update and delete operations support `If-Match` / `If-Unmodified-Since` to prevent lost updates
- Clients get a clear 412 Precondition Failed when a conflict occurs — not a silent overwrite

### Content negotiation
- Requests and responses use `Content-Type` and `Accept` headers correctly
- JSON responses use camelCase field names consistently
- Dates use ISO 8601 / RFC 3339

---

## 2. E2E Test Readiness

Evaluate whether this API can be driven reliably in an automated end-to-end test harness.

### Test data lifecycle
- API supports creating test data through the same endpoints tests will exercise (no back-door DB seeding required)
- Resources created during tests can be cleaned up via DELETE or a teardown mechanism
- No global shared state that causes test interference — each test run can operate in isolation

### Deterministic behavior
- Same input produces same output (no hidden randomness, no time-dependent logic without clock injection)
- Ordering of collection responses is deterministic (explicit default sort, not database-order)
- Async operations provide a way to poll or wait for completion

### Authentication in test environments
- Auth can be configured for test environments (service accounts, test tokens, API keys)
- No hard dependency on external identity providers that can't be stubbed in CI

### Observability
- Requests can be correlated via request IDs or trace headers
- Errors return enough context to diagnose failures without reading server logs

### Data isolation
- Multi-tenant APIs support test tenant/namespace isolation
- Tests can run concurrently without stepping on each other

### Health and readiness
- Service exposes a health or readiness endpoint that test harnesses can poll before sending traffic
- Health endpoint reflects actual dependency status (database, caches, downstream services), not just "process is running"

---

## 3. Agentic Readiness

Evaluate whether an AI agent can discover, use, compose, and recover from errors with this API.

### Discoverability
- API has machine-readable documentation (OpenAPI spec, proto files, GraphQL introspection)
- Endpoint naming is self-descriptive — an agent can infer purpose from the URL and method
- Response schemas are consistent and predictable across endpoints

### Error recovery
- Error responses include enough detail for an agent to self-correct (e.g., "field X is required" not just "bad request")
- Validation errors enumerate all failures, not just the first one
- Rate limit responses include `Retry-After` or equivalent

### Composability
- CRUD operations exist for all core resources — an agent can create, read, update, and delete without special workflows
- Related resources are linked (IDs or URLs) so an agent can navigate the object graph
- Bulk operations exist for resources that agents will commonly need to process in batches

### Idempotency and safety
- An agent can safely retry failed requests without causing duplicates or corruption
- GET requests have no side effects — an agent can explore freely
- State-changing operations are explicit (POST/PUT/PATCH/DELETE, not GET with query params)

### Predictable schemas
- Response fields are consistent — same field name means same type everywhere
- Nullable fields are explicit, not sometimes-present sometimes-absent
- Enums are documented and stable — an agent can rely on known values

---

## Summary

After completing the audit, output a table:

| Section | PASS | WARN | FAIL |
|---------|------|------|------|
| API Design | | | |
| E2E Test Readiness | | | |
| Agentic Readiness | | | |

Then list the top 3 highest-impact findings to address first.
