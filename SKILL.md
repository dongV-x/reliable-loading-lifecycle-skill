---
name: reliable-loading-lifecycle
description: Audit and implement a shared, resilient loading lifecycle for web applications. Use when pages load slowly, fail into blank screens, show inconsistent loading states, leak stale or cross-account data, duplicate requests, retry writes unsafely, or when old and new pages need enforceable loading rules in CI. 适用于页面加载慢、白屏、失败恢复不一致及全局加载机制治理。
---

# Reliable Loading Lifecycle

Build a project-specific loading system from shared invariants. Do not copy a reference implementation before inspecting the target stack and existing behavior.

## Start With The Real System

1. Read the target project's local instructions, package manifests, route definitions, request clients, server-state/cache layers, auth lifecycle, CI, and the highest-traffic pages.
2. Trace two or three representative flows end to end: initial read, scope switch, background refresh, mutation, long job, modal/search, upload/download, or lazy route loading when they exist.
3. Record current evidence before changing code: request count, blank/loading/error behavior, stale-data behavior, cancellation, and same-condition latency measurements when the request includes speed improvement.
4. Decide whether the contract is fully applicable, partly applicable, or not the primary fix. Only after that decision, inventory every in-scope route and non-route async owner and classify them with [references/core-contract.md](references/core-contract.md).

Do not infer that a smooth page has a sound lifecycle. It may only have smaller data, a warm cache, fewer joins, or faster endpoints. Separate backend latency, payload/render cost, asset loading, and frontend lifecycle defects. If the bottleneck is primarily backend, asset, or rendering work, use this skill only for feedback and recovery while fixing the measured bottleneck elsewhere. Do not claim speed improved without comparable before/after measurements.

## Choose The Smallest Adapter That Fits

Read [references/adapters.md](references/adapters.md), then reuse the project's existing request and server-state libraries. Add a dependency only when the application has enough shared server state that local helpers cannot enforce the contract cleanly.

The reusable unit is the behavior contract, not one framework implementation. A React Query project, an SWR project, a GraphQL client, a server-rendered framework, and a small fetch-only app should not receive identical code.

## Implement Shared Invariants

Implement only the layers the target needs, but do not simplify away these applicable invariants:

- Map an auditable boundary for each runtime and protocol. Route controllable first-party requests through the appropriate shared boundary; explicitly register and test framework-native loaders, server runtimes, third-party SDKs, streams, and other justified exceptions. Centralize applicable deadline handling, cancellation, authentication refresh, error classification, response parsing, and request IDs at each boundary.
- Give protected data a complete identity key: user/account/tenant scope plus every business filter that changes the result. Audit query caches, normalized caches, persistence, Service Workers, SSR/server caches, CDN policy, and browser storage when the target uses them. On identity change, clear only client-side protected namespaces and preserve unrelated public cache and drafts; isolate shared server/CDN caches with complete keys and correct cache headers.
- Deduplicate only semantically identical, safe reads. Define identity from method, normalized URL/parameters/body, authenticated scope, representation-changing headers, and cache mode. Exclude streams, range requests, signed URLs, and semantically non-idempotent reads unless the target proves equivalence. Pass cancellation signals when supported and also enforce an identity/version guard before committing a response, because client cancellation may not stop server work.
- Distinguish first load, background refresh, empty, error, and stale-data refresh. Do not replace valid data with a blocking spinner during refresh.
- Keep feedback local to the owner. Background polling and telemetry must not activate a page-wide loader.
- Preserve layout during slow loads. Use a measured or project-appropriate delay, with 200-300 ms only as a starting point. Keep stable navigation when the architecture supports it; use a structural placeholder, progressive rendering, or framework-native fallback. Support reduced motion and expose `aria-busy` or an equivalent accessible status.
- Disable blind retries for mutations. Coalesce concurrent auth refreshes, and never replay a consequential write after auth refresh unless server-enforced idempotency makes it safe. Treat timeout/network/abort after dispatch as an unknown outcome that requires authoritative reconciliation.
- Represent resumable long jobs with an operation ID and poll that operation. Never repeat job creation merely because polling failed.
- Handle lazy-route or chunk failure without silently reloading over unsaved input.

Never use a global in-flight request counter as the source of page loading truth. It cannot distinguish foreground work from telemetry or polling and commonly produces stuck overlays.

## Migrate Without A Cosmetic Rewrite

Fix the shared root first, then migrate high-value and high-risk surfaces, then close the inventory. Preserve existing page layout and controls unless the user asked for UI changes. Every unmigrated path or permanent exception needs an owner, reason, bounded scope, and corresponding test; unexplained bypasses mean the migration is incomplete.

For complex applications, keep a small machine-readable registry of routes and async owners. For small applications, an existing lint rule plus focused tests may be enough. Do not introduce a registry solely for ceremony.

## Enforce The Result

Read [references/verification.md](references/verification.md) before claiming completion.

Use the project's existing parser, linter, and test runner to create the narrowest effective gate. Static checks should reject provable architectural violations such as unapproved network entry points, unscoped protected queries, unregistered routes or async owners where a registry is warranted, and unsafe mutation defaults. Verify visual state semantics with component or browser tests. Prefer syntax-aware checks to regex when code can be aliased, wrapped, re-exported, or computed.

Run a local negative probe in an isolated temporary workspace: intentionally add one bypass and prove the gate rejects it for the intended reason. Only push a probe through real pull-request CI when explicitly authorized, then remove it afterward. Without that remote evidence, report that the gate exists but pull-request blocking is unverified. Do not mutate production, merge, change branch protection, or publish externally without the authorization required by the target project.

## Completion Standard

Report four things:

1. The target-specific adapter and why it fits.
2. What routes and async profiles are covered, including remaining exceptions.
3. Positive test evidence, local negative-gate evidence, and remote CI-probe evidence when authorized.
4. Before/after performance evidence for speed claims and known limits such as untested chunk failure, missing long-job recovery, absent latency baselines, or a CI check that is not required on the default branch.

Do not say "global" or "all pages" merely because a shared helper exists. Those claims require complete inventory coverage, runtime protection for critical invariants, CI enforcement on the default branch, and no unexplained bypasses.
