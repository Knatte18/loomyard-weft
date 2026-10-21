---
verdict: APPROVED
---

## Scope

The diff of pull request 788 at `90108f75` against `main` builds what the board entry asks for: `go_parallel` and `slots` take a machine-local override that Bolt never carries.
It follows the decision record:

- D1: `configengine` gains the machine layer at `<BoardDir>/.lyx/config/<module>.machine.yaml`, stacked between the hub file and the local file only when `Dirs` carry both `Board` and `Anchor`.
- D2: `CategoryMachine` and `MachineKeys`, with a `configreg` test that refuses a per-worktree module classifying a machine key.
- D3: gate's `slots` and `go_parallel` are classified `machine`, `busy_quit_sec` stays cross-pair, and the gate load failures in `gateslot`, `hubgeom.GateSlots` and `lyx gate test` name both files, both ways forward and the clause for a session that is not the prime.
- D4: `lyx config --layer machine` for `--print`, `--set` and the editor, behind the hub-write guard, with `--resolved` and menu marks.
- D5: an `Ensure` integration test pins the machine file as untouched by reconcile.

Docs land in the same change: `configengine`, `configcli`, `gatecli`, `gateslot` and `hubgeom` doc files, `docs/overview.md`, and the two PATTERN entries the record declares (`PATTERN-config-layers`, `PATTERN-hub-wide-config`), plus the background line in `pattern/PATTERN-durable-vs-ephemeral-state.md` the record's notes call for.
The `contracts/specs/loom-plan-spec.md` and `internal/planglyph/doc.go` lines and the `websterengine` test string follow from D3's changed failure text and are in scope.

The headroom trim in `454d1e41` shortens three other PATTERN index lines (`PATTERN-hub-containment`, `PATTERN-hub-store-gc`, `PATTERN-recipe-setting-lookup`) without changing what they require; the dropped detail stays in their background files.
That is bookkeeping for the headroom lint, not a scope deviation.

## Carried-over risks

- Webster-Review F1 and F2 (dropped index clauses) are fixed: the final diff leaves the `PATTERN-durable-vs-ephemeral-state` and `PATTERN-push-both-sides` index lines as on `main`.
- Discussion-Review's `gate-load-failure-no-pair-way-forward` is met by D3's role clause, which the diff carries in every gate load failure.
- The Plan-Review findings are design detail for the review segments, and the diff breaks no recorded decision over them.
- The record's open risk on `gate-language-neutral` stays open for that entry: it must classify `parallel` as `machine`.

## Overlaps and landing order

`own_row` is 21.

- `run-reentry` (row 11): both change `internal/configreg/configreg_test.go` at different places; run-reentry adds a run-wide loom and darn key and has no decision on categories or the machine layer.
  Text-only overlap. This run lands first, and run-reentry takes the change at Merge-In.
- `origin-reconcile` (row 11): both change `internal/hubreconcile/ensure_integration_test.go` at different places; its origin reconcile is a git-level branch check and never reads config files, so it does not contradict D5.
  Text-only overlap. This run lands first, and origin-reconcile takes the change at Merge-In.
- `scratch-defeats-test-cache` (row 7): no changed paths and no shared decision; no overlap.
- Parent commits: none listed.

## Superseded

None: `bench-osl1046-xdr` and `loom-gates-xdr-stacking` are notes this run does not resolve, and `gate-language-neutral` builds on it rather than being replaced.
