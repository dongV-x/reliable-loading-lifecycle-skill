# Verification And Enforcement

Scale verification with the target's risk. The goal is observable protection, not a large test suite.

## Minimum Positive Checks

- Each applicable runtime/protocol boundary classifies the failures it owns correctly; explicitly registered third-party or framework-native exceptions have focused tests.
- A late response cannot overwrite a newer scope.
- Identical reads deduplicate when the selected server-state layer promises deduplication.
- Identity switches advance a version guard and clear the protected namespace before the new identity renders.
- First load, cached refresh, empty, initial error, and refresh error render differently.
- Mutations do not retry blindly; duplicate activation is prevented.
- Consequential writes demonstrate server-side idempotency and unknown-outcome recovery.
- Background work cannot trigger page-wide feedback.
- Slow-load visuals keep stable dimensions and respect reduced motion.

Test long-job, file-stream, search, modal, or lazy-route behavior only when the target owns those profiles.

## Static Gate

Prefer an existing AST-aware linter or parser. Limit static gates to properties they can prove. Apply each gate to its declared runtime/protocol boundary and verify registered exceptions separately. Detect the target's actual bypass routes, which may include:

- direct `fetch`, `XMLHttpRequest`, `WebSocket`, `EventSource`, or installed HTTP clients
- aliases, `.call`/`.bind`, computed properties, wrapper functions, and re-exports
- unscoped query keys or a scoped helper call unrelated to the actual query
- routes or async owners absent from the target registry
- mutation calls outside the canonical transport

Fail closed on code the checker cannot safely interpret only inside a deliberately narrow protected boundary. A universal fail-closed parser can block valid metaprogramming and is not automatically better.

Count-based ratchets are useful during migration but are not semantic proof. Drive temporary allowances to zero or require a precise, reviewed reason for each one.

## CI Gate

Run the smallest relevant set on pull requests and the default branch:

1. Static loading contract.
2. Transport and cache-lifecycle tests.
3. Server idempotency tests when writes are in scope.
4. Build/artifact consistency checks.
5. Focused browser or visual checks for critical loading states.

If repository policy allows it, make the workflow a required status check. Creating a workflow file does not by itself make it required.

Design the required job so fork pull requests and missing secrets still run the static contract and non-secret tests. Separate secret-dependent integration work rather than skipping the required job. Exclude generated, vendor, fixture, and third-party code through reviewed path rules, and verify shipped first-party generated code elsewhere when it can contain network calls. Treat a skipped required contract job as not passed.

## Negative Probe

Start in an isolated local workspace:

1. Add one realistic bypass that is included in the shipped source.
2. Rebuild generated artifacts when the repository requires them.
3. Confirm the local gate fails for the intended reason.
4. When explicitly authorized, open or push the probe through the same CI path as ordinary work.
5. Confirm remote failure occurs in the loading-contract step, not in an earlier unrelated check, then close and delete the probe.

Do not run destructive probes against production or real data.

## Honest Completion Levels

| Claim | Evidence required |
|---|---|
| Shared helper exists | Representative callers use it |
| Pages migrated | Named route inventory is covered |
| Global mechanism | All routes and non-route async owners covered; critical runtime guards active |
| Future pages enforced | Pull-request CI rejects a realistic bypass |
| Default branch protected | Gate runs on the default branch and repository protection requires it when available |
| End-to-end write safety | Client and server idempotency plus reconciliation are tested |

List partial and pending scenarios explicitly. Examples: chunk-failure recovery not fault-injected, long-job resume absent, background offline behavior untested, p95 not measured, or required-check settings unavailable.

## Release Safety

For a high-risk shared migration, use the target project's existing rollout and rollback mechanism. Verify in isolated data first, then observe applicable signals such as blank-screen errors, request failures, duplicate requests, stale-response suppression, unknown mutation outcomes, and recovery success. Do not invent a new observability stack solely for this work.
