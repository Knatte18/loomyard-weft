# Parent review as a step in Discussion-Write's gate

All 14 batches finished `done`, and the plan-level verify (`go build`, `go test`, integration tests, smoke vet) reported `OK`.

- **01 gate-pending-state:** `shuttleengine` gate results gained a pending state, a `Final` closure and a `waiting` entry state, and the wait loop holds them at turn boundaries.
  Deviations: `completionsignal_enforcement_test.go` (tripwire list moved with the extracted helper) and `gate_test.go` (new `evaluateGate` argument).
- **02 loom-parent-review-config:** `parent_review_wait_min` and the parent-review directory accessors in `loomengine`.
  Deviations: `loomcli` and `webstercli` `wiring_test.go` fixtures needed the new required key.
- **03 parent-review-stencils:** delivery and brief stencils plus their compose functions.
- **04 parent-review-store:** `internal/parentreview` round store and transitions.
- **05 parent-review-gate:** the gate and finalize closures over the store.
- **06 recipe-parent-review-gate:** `parent-review` joined `shedrecipe`'s gate vocabulary, pass-on-cap only.
- **07 loom-review-verbs:** `lyx loom review notify|delivered|approve|reject` with refusal rows.
- **08 shedverbs-waiting-hook:** a `Waiting` hook on the generic status verb.
  Deviations: `shedverbs/spec.go` and `status.go`.
- **09 loom-status-review-wait:** `lyx loom status` reports the reviewer wait.
- **10 batten-review-wait:** batten's InnerRun wait message names the child's reviewer wait.
- **11 discussion-stencil-parent-review:** Discussion-Write reads the board and handles parent-review prompts.
- **12 wire-parent-review:** the entry is enabled on Discussion-Write and the round directories are committed with the discussion.
- **13 orch-review-fork:** the orch stencils handle a review notice through a one-shot fork.
- **14 parent-review-integration:** integration test and smoke test for the whole exchange.
  Deviations: `wiring_commitstatus_integration_test.go` (helper renamed to avoid a clash with batch 07's `writeReviewFile`).

The fork for batch 14 reported `gofmt -l` flags `internal/loomcli/wiring.go` (from batch 12); it was left untouched.
