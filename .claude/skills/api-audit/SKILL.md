---
name: api-audit
description: Audit application and service APIs for conformance to Google AIPs (API Improvement Proposals). Findings cite specific AIP rules with quoted text.
allowed-tools: Read, Glob, Grep, Bash, WebFetch
---

Audit the API surface of this project for conformance to the Google AIPs at google.aip.dev. Every finding must be grounded in text fetched from the AIP at audit time — never in the model's recollection of API design principles.

## Step 1: Fetch the AIPs

Fetch all core AIPs in parallel (one assistant turn, multiple WebFetch calls) before reading any project code.

Core AIPs — always in scope:

- https://google.aip.dev/121 — Resource-oriented design
- https://google.aip.dev/122 — Resource names
- https://google.aip.dev/130 — Standard methods
- https://google.aip.dev/131 — Standard methods: Get
- https://google.aip.dev/132 — Standard methods: List
- https://google.aip.dev/133 — Standard methods: Create
- https://google.aip.dev/134 — Standard methods: Update
- https://google.aip.dev/135 — Standard methods: Delete
- https://google.aip.dev/136 — Custom methods
- https://google.aip.dev/140 — Field names
- https://google.aip.dev/158 — Pagination
- https://google.aip.dev/160 — Filtering
- https://google.aip.dev/185 — API versioning
- https://google.aip.dev/190 — Naming conventions
- https://google.aip.dev/193 — Errors
- https://google.aip.dev/211 — Authorization checks

When fetching each AIP, ask WebFetch for the full MUST / SHOULD / MAY rules and any examples, not a summary. You will quote from this text in findings.

Conditional AIPs — fetch only if the API uses the corresponding pattern (decide after Step 2):

- https://google.aip.dev/151 — Long-running operations (if any endpoint returns an Operation or handles async work)
- https://google.aip.dev/157 — Partial responses (if any endpoint supports field selection)
- https://google.aip.dev/161 — Field masks (if PATCH endpoints use field masks)
- https://google.aip.dev/164 — Soft delete (if resources can be soft-deleted or restored)
- https://google.aip.dev/194 — Automatic retry configuration (if the API publishes client retry guidance)
- https://google.aip.dev/231, /233, /234, /235 — Batch methods (if any batchGet / batchCreate / batchUpdate / batchDelete)

## Step 2: Discover the API surface

Find every endpoint the project exposes. Adapt to whatever the project uses:

- OpenAPI / Swagger specs — common paths: `openapi.yaml`, `swagger.json`, `docs/api.yaml`, generated output under `build/` or `dist/`
- Route registrations in code — Go: `chi`, `gin`, `echo`, `gorilla/mux`, `net/http`; Python: FastAPI routers, Flask blueprints, Django `urls.py`, `aiohttp` routes
- Proto files (`*.proto`) if the API has a gRPC surface with HTTP transcoding

For each endpoint, record: HTTP method, path, handler function location (file:line), and request/response shape if available from the spec or code.

If you cannot enumerate the API surface with reasonable confidence, stop and report what you found. Do not audit a partial surface without saying so.

## Step 3: Evaluate each endpoint against the AIPs

For every endpoint, check it against every core AIP rule that applies to its method and shape. Use the fetched AIP text as the authority — quote from it, don't paraphrase.

Severity mapping (RFC 2119 terms used in AIPs):

- **MUST** violation → **FAIL**
- **SHOULD** violation → **WARN**
- **MAY** deviation → **PASS** unless clearly harmful

Produce one finding per (endpoint, violated rule) pair. Do not list PASS findings individually — roll them into the summary count.

Every WARN or FAIL finding must include:

- AIP reference: `AIP-NNN §<section-heading>`
- Quoted rule text — one sentence of what the AIP requires, verbatim
- Endpoint: `METHOD /path` with handler location `file:line`
- What the endpoint does instead
- Recommended fix, grounded in the AIP

If you cannot quote the specific rule from the fetched AIP text, do not cite the AIP — omit the finding rather than invent authority.

## Step 4: Output

### Summary

One row per AIP actually checked:

| AIP | Title | Endpoints checked | PASS | WARN | FAIL |
|-----|-------|-------------------|------|------|------|
| 122 | Resource names | N | N | N | N |
| 131 | Standard methods: Get | N | N | N | N |
| ... | | | | | |

### Findings

Grouped by AIP, FAIL before WARN within each group. Format:

**AIP-122 §Resource ID segments — FAIL**

> "Resource IDs must not contain characters that require URL encoding."

Endpoint: `GET /users/{user_email}` — handler at `api/users.go:47`
Uses email addresses (containing `@` and `.`) as the resource ID segment. Recommendation: use an opaque ID and expose email as a field on the User resource.

### Top 3 priorities

Three findings with the highest blast radius — rules that multiple endpoints violate, or violations that block programmatic or agentic consumers most severely.

## Scope and caveats

- This skill checks conformance to Google AIPs only. It does not audit observability, health/readiness endpoints, request-ID / tracing conventions, E2E test hooks, or agentic-workflow requirements. Those are out of scope.
- AIPs are opinionated toward resource-oriented REST with a proto / gRPC flavor. Some idiomatic patterns in FastAPI, Flask, or stdlib `net/http` handlers will show up as FAIL against strict AIP rules. That is working as intended — report them, and let the service team decide whether to conform or document an exception.
- Every citation must be backed by quoted AIP text fetched this run. If WebFetch fails for an AIP, note that AIP as "not checked" in the summary rather than falling back to recollection.
