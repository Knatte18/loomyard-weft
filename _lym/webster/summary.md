# Fabric core foundation: layout and core packages, snapshot builder, executor loop, snapshot reader

All batches of the plan are recorded `done`, with head `f4fd753550f4d5dea8ff62d02a1f3f7d2a55175c`.
Each batch's fork reported its batch gate passing on tier0 and tier1; tier2 and tier3 did not run.
No digest listed a departure or an off-plan rename, and no batch needed recovery.

## What was built

- A `scankit` check that fails a package importing a banned path, stdlib included.
- `internal/fabriclayout`, holding fabric's path classification, hub paths, ref names, mutation record data and the plain origin, push, verb and read types, with callers retargeted to it.
- `internal/fabriccore`, holding the snapshot, phase, action list and outcome types.
- `internal/testkit/fabrickit`, a shared snapshot builder.
- In `internal/fabricengine`, the executor loop with its error-to-class mapping and the phased snapshot reader with its git fallback.

The batch 15 fork reported that the fabric `_test.go` line total went from 58594 at the part's base to 58200.

## Forks' notes on their work

- Batches 10, 11 and 15 reported `Loosened-Check` trailers: `internal/fabriclayout` was admitted in the import allowlists of `mergeresolve` (batch 10) and `hubreconcile` (batch 11), and batch 15 carries three.
- Batch 04 reported admitting `internal/fabriclayout` in the `landingshed` and `reportengine` import allowlists.
- Batch 03 reported one tier0 failure on its first card gate, `internal/shedverbs` `TestUntilStop_EnvelopeCurrency`, in code the card did not change; the batch gate then ran with none failing.
- Batch 12 chose field names and the `Action` fields each kind reads where card 12 did not say.

## Scope guard

The deviations are informational: files changed outside a batch's own targets.

- Batch 01: caller retargets and tests in `internal/fabricengine`, `internal/fabriclayout`, `internal/testkit/scankit`, `internal/weavercli` and `internal/webstercli`.
- Batch 03: a wide retarget of callers and their tests across `cmd/` and `internal/`, plus `internal/fabriclayout/hubpaths.go` and its test.
- Batch 04: its list was cut from Master's view of the digest, so it is not named here.
- Batch 05: `docs/logging-inventory.md`, `internal/beamcli/cli.go`, `internal/cleanupcli/cleanup_integration_test.go`, `internal/fabriclayout/mutation.go`, `internal/orchcli/start.go`, `internal/webstercli/reset.go`, `internal/webstercli/reset_test.go`.
- Batch 07: two docs under `docs/research/no-fake-other-modules/` and caller retargets across `internal/`, plus `internal/fabriclayout/origintypes.go`.
- Batch 08: one doc under `docs/research/loom/` and caller retargets across `internal/`, plus `internal/fabriclayout/pushtypes.go`.
- Batch 10: two docs under `docs/research/no-fake-other-modules/`, `cmd/lym/stencilseed_integration_test.go` and caller retargets across `internal/`, plus `internal/fabriclayout/verbresults.go` and its test.
- Batch 11: two docs under `docs/research/` and caller retargets across `internal/`, plus `internal/fabriclayout/readtypes.go` and `origin_test.go`.
- Batch 12: `internal/fabriccore/answer_test.go`.
- Batch 13: `internal/fabricengine/executor_test.go`, `internal/testkit/fabrickit/builder_test.go`, `internal/testkit/fabrickit/testmain_test.go`.
- Batch 15: none listed.
