# Adapter Selection

Combine the rows that match each runtime and data owner in the target. Reuse installed libraries and existing ownership boundaries.

| Target shape | Preferred adaptation | Do not assume |
|---|---|---|
| TanStack Query | Shared transport plus one configured `QueryClient`; scoped query-key helper; signal propagation; mutation defaults | That adding Query automatically fixes incomplete keys or unsafe writes |
| SWR | Shared fetcher/provider, complete serialized keys, scoped cache reset, local mutation policy | That request cancellation and mutation idempotency are provided automatically |
| RTK Query | Shared `baseQuery`, complete endpoint arguments/cache serialization, tag invalidation, auth reset | That every network call already uses RTK Query |
| Apollo/GraphQL client | Auth/error/retry links, variables containing full scope, normalized-cache identity rules, mutation IDs | That GraphQL operation names alone isolate cached results |
| Next.js/Remix/server-first app | Framework loading/error boundaries plus client cache only where interactive server state needs it | That every server component needs a client query library |
| Angular/RxJS | `HttpClient` interceptor, typed errors, cancellation via subscription lifecycle, route/component-scoped feedback | That one global interceptor should own visual loading state |
| Vue Query or similar | Framework-native query client plus the same identity, cancellation, refresh, and mutation invariants | That React-specific code should be ported |
| Small fetch-only app | One thin request helper, `AbortController`, explicit local state, and focused tests | That a new cache dependency is justified |
| Legacy mixed app/iframes | Stabilize one shared transport per runtime; define cross-frame ownership and migrate incrementally | That a single JavaScript singleton can span separate documents or processes |

## Decision Rules

1. Map runtimes and protocols first; a browser SPA, server loader, worker, iframe, GraphQL client, and third-party SDK may need separate boundaries.
2. Keep server-state ownership in the established server-state library.
3. Keep transport concerns out of visual components.
4. Keep visual loading ownership at page/table/panel/modal/action scope.
5. Use framework-native route loading and error boundaries for code chunks.
6. Add a registry only when route and async-owner coverage cannot otherwise be proved.
7. Add runtime guards for high-impact invariants such as authenticated cache isolation; use static gates for architectural boundaries such as banned raw transports.

## Existing Backend Constraints

Inspect the server before promising client behavior:

- Is cancellation propagated or merely ignored by the client?
- Does the API return request IDs and structured errors?
- Does it validate idempotency keys and persist/replay results?
- Can long jobs be resumed by operation ID?
- Are cache headers, ETags, pagination, and payload shape usable?
- Which layers cache protected data: query client, normalized cache, SSR/CDN, persistence, Service Worker, IndexedDB, or browser navigation cache?

When the backend cannot support an invariant, document the boundary and implement the safest client fallback. Do not describe a client-only approximation as end-to-end correctness.
