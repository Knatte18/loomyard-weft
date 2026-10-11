# parent-review-wait: keep-alive pings, no pass-at-cap, bounded wait

All three batches reported `done`, with no deviations from the declared file-ops.
Each record-batch returned only informational scope warnings, listed below.

## Batch 01 — engine-prompt-cache-lifetime (head `95c8e6506`)

- Cards 1–4 landed.
- The engine answers a session's warm prompt-cache lifetime (`Engine.PromptCacheLifetime`).
- The one-line keep-alive stencil and its renderer exist, and the discussion stencil gained its third message.
- The gate sends up to three counted keep-alive pings per delivered round.
- Loom wiring sets the keep-alive interval from the writer's cache lifetime (`keepAliveMargin`).
- Scope warnings: `loomcli#keepAliveMargin`, the parentreview gate-test helpers, `loomengine` stencil-name constants and `Engine.PromptCacheLifetime`.

## Batch 05 — parent-review-no-pass-at-cap (head `2c69b67de`)

- Cards 5–9 landed.
- The gate's pass-at-cap mode is gone, `pass_on_cap` is refused on parent-review, and the recipe has two reviewed rounds.
- A gate list with an entry after parent-review is refused.
- A fresh spawn reuses a latest round that holds no request.
- `approve --review` refuses a missing, unreadable or empty file.
- `delivered` on a round with a verdict exits 0 and writes nothing.
- Scope warning: `parentreview#wantFindingsFailure`.

## Batch 10 — review-refusals-name-status (head `072a991ce`)

- Cards 10–14 landed.
- Review refusals name `lyx batten status <slug>` for a slug and `lyx loom status` for none.
- The brief and the orch role say no message goes to the writer.
- A pending gate with no new arrival is re-evaluated at most every 5 seconds (`gateRecheckDue`).
- `GateEntry` gained an expiry probe the wait loop calls off the boundary, and parent-review wires a read-only probe through its resolver.
- Card 13 raised the shuttleengine row in `cmd/lyx/testdata/test-budget.yaml`.
- Scope warnings: `loomcli` review-test helpers, `parentreview#snapshotTree` and `shuttleengine` `Run.gateRecheckDue`.

## Verify gate fixes

- Findings (attempt 1 of 3): `battencli.TestWire_EverySeamFilled` and three `envkit` tests (`TestFullEnv_FillsEverySeam`, `TestNilSeams_ReportsExactPaths`, `TestNilSeams_SkipsNilLegalPaths`) failed because `ParentReview.RenderKeepAlive`, the seam card 3 added, was nil after wiring and not on the nil-legal list.
- The fixer added `ParentReview.RenderKeepAlive` to the nil-legal seam list in `internal/testkit/envkit/envkit.go`, beside `RenderDelivery` and `RenderBrief`, which `loomcli` wiring fills at runtime.
- Commit `30a88e0cc`: `fix: list ParentReview.RenderKeepAlive among the nil-legal env seams`.
- The fixer reports `lyx gate test --tags integration ./internal/battencli ./internal/testkit/envkit` passing and the worktree clean.
