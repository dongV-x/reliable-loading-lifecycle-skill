# Core Contract

Use this reference while classifying async work and defining target-specific acceptance criteria.

## Profiles

| Profile | Typical owner | Required user-visible behavior | Important safety behavior |
|---|---|---|---|
| `page-query` | Page or dashboard | Stable page skeleton on first load; local error and retry | Complete scope key; cancel obsolete work |
| `table-query` | Ranking or data table | Preserve columns and table height; no white replacement | Old scope rows never appear under new filters |
| `local-query` | Panel or widget | Feedback stays inside the panel | Does not block unrelated page work |
| `modal-query` | Dialog or drawer | Modal-sized loading/error/retry state | Closing cancels or safely ignores late results |
| `search-query` | Typeahead or explicit search | Inline progress; newest input wins | Debounce when useful; cancel/ignore obsolete results |
| `mutation` | Save, confirm, send | Disable duplicate action; preserve input on failure | No blind retry; idempotency and unknown-outcome recovery when consequential |
| `long-job` | Import, scan, generation | Durable progress and explicit recovery | Create once, resume with operation ID |
| `background` | Polling, health, telemetry | No blocking loader; quiet failure policy | Bounded work; must not steal foreground state |
| `file-stream` | Upload or download | Progress or download-scoped feedback | Cancellation, size/type errors, explicit write identity when applicable |
| `lazy-route` | Code-split route | Keep application shell visible; recoverable asset error | Preserve unsaved state before reload |
| `local-only` | Client computation | Only local busy feedback when needed | No network contract required |

Profiles describe behavior, not components. One page can own several profiles.

## State Semantics

| Situation | Correct state |
|---|---|
| No cached data, request pending briefly | Keep stable shell; avoid a flash |
| No cached data, request remains pending | Structural skeleton or scoped loading state |
| Cached data, same key refreshes | Keep data; show subtle updating status |
| New key has no cache | Show loading for the new scope; never label old data as new |
| Initial load fails | Error state with useful reason and retry |
| Refresh fails while cached data exists | Keep data; show non-blocking refresh failure |
| Request is intentionally cancelled | Usually silent; do not label as timeout |
| Request times out or loses network | Classified error; retry reads only when policy allows |
| Mutation may have reached the server | Outcome unknown; reconcile with authoritative state |
| No records after a successful response | Empty state, not error and not loading |

## Identity And Cache Keys

A key must contain every dimension that can change the response, including the authenticated scope. Common dimensions include tenant, account, shop, brand, region, period, report mode, ranking mode, filters, search term, pagination, and permission context.

Do not put secrets in keys. Use a stable non-secret cache scope. On logout, account switch, or permission downgrade: advance an identity epoch/version, cancel protected requests where possible, clear protected client cache and persistence namespaces, then render the next identity. A response must verify the current identity epoch before committing even when cancellation is unavailable.

SSR request-local caches should die with the request. Shared server and CDN caches require complete cache keys plus suitable `private`, `no-store`, or `Vary` policy; purge only when the platform supports it and invalidation is actually required. Service Worker and browser persistence need explicit protected-namespace handling.

## Transport Error Taxonomy

Keep at least these classes distinguishable when the platform exposes them:

- intentional abort
- deadline timeout
- offline/network failure
- authentication/authorization failure
- HTTP application failure
- response parse/schema failure
- unknown mutation outcome

Carry a server request or trace ID when available. User messages should explain recovery; technical details can remain in expandable diagnostics or logs.

## Retry And Idempotency

- Reads may retry only when the operation is semantically safe, the error is plausibly transient, and the user experience benefits. Bound attempts and use backoff.
- Writes default to no automatic retry.
- Authentication refreshes should coalesce. Do not automatically replay a dispatched consequential write unless server-enforced idempotency makes replay safe.
- Consequential writes should carry a stable idempotency key that the server validates and deduplicates.
- A client key without server enforcement is not idempotency.
- Reuse the same key only for the same logical operation, especially after an unknown outcome.
- File/binary writes need an explicit operation identity when request bodies cannot be fingerprinted reliably.

## Performance Diagnosis

Measure these separately:

1. Route asset/chunk load.
2. Authentication/session bootstrap.
3. Server latency and database work.
4. Payload transfer and parsing.
5. Client render and chart/table work.
6. Duplicate or obsolete requests.

A loading component improves feedback, not underlying latency. Fix excessive queries, payloads, joins, serialization, or rendering when the measurements point there. Before implementation, agree on the measured stage, statistic such as median or p95, sample count, cache/network/data conditions, and a success threshold larger than expected measurement noise. Use the same fixture/data scope, cache condition, network condition, and viewport for before/after comparisons.
