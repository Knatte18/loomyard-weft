# loom becomes a task-run contract that tapestry and darn fulfil

`loom` stops naming one recipe and names a contract.
A loom recipe takes one board task in, holds a decision record as the task's decided scope, has a build segment that turns the record into the change, and ends with Finalize landing one change in the parent.
`tapestry` and darn are the loom recipes; pawl joins in `pawl-loom-recipe`.
What the two recipes wrote twice is now one copy driven by the recipe.

## What changes

- A recipe declares `contract: loom` at its top level.
  Shed's loader in `internal/shedbuild` checks it at build and refuses a recipe that lacks a record row, a build segment after the record row, a Finalize row ending its `on_done` path, or a `verify_source` key.
  The refusal names the broken clause and the way forward.
  The record, build and Finalize engine sets are declared once beside the registry in `internal/shedrecipe`.
- Every `lym loom` verb that addresses a run refuses a run whose recipe does not declare the contract, naming the `lym shed` verbs.
  The refusal sits in the `lym loom` command path, not in `loomcli.ArmAt`, which `lym shed` shares.
  Pawl runs are served as before.
- A verb that applies to one recipe refuses on another by the recipe's rows.
  `validate-plan`, `insert-card`, `remove-card` and `restore-renumber` refuse on a darn run, `decision add --amends-plan` refuses there too, and `lym loom peers` reads each peer against its own recipe.
- Darn's decision record is written mechanically by a Go `Record-Write` row from the board entry: the body verbatim in a fenced block under `## Goal`, the other sections empty, plus a support log, both committed before Drift-Baseline.
  A resume leaves an existing pair byte for byte and repairs a crash that left a partial pair.
- One discussion validator and gate serve every loom record.
  The history check runs on every record, the citation and relevant-list checks follow the record row's engine, and a top-level fence opens and closes by CommonMark's rule.
  `validate-discussion` and the gate report the same findings.
  Darn-Write has a post-session guard: a damaged record or support log halts the run Stuck with no re-prompt and names the restore steps.
- Darn's writer stop marker halts the run.
  An outdated-entry answer goes to the parent through `lym loom decision add` and the writer never edits the record.
- Darn's Code-Review perch measures against the decision record, and a darn run writes no `board-entry.md`.
- One Describe table and stencil serve both recipes, and the verify source follows `verify_source`.
  `loomcli`'s recipe-name branches now read the recipe's rows and keys.
- `tapestry` routes Parent-Align, Drift-Verify, the Code-Review perch, then Describe, so Finalize's reroute reaches Drift-Verify again.
- The Tapestry-Preflight row and its seed check go.
  Preflight loads `batcher.yaml` for any recipe with an Implementer row.
- The `beam` recipe alias goes; `lym shed` reports a seed recording it as an unknown recipe.
- Config splits: `loom.yaml` holds what every loom recipe shares and a new `tapestry.yaml` holds the `plan_*` and `implementer` keys.
  A hub `loom.yaml` that tuned `plan_max_bounces` carries the value over on reconcile.
- The arbiter, tackler and driver-kind keys live in `shed.yaml` alone.
  `loom.yaml` and `darn.yaml` refuse them, and a profile or task `run_settings` naming one under `loom` or `darn` is refused at its write and at Seed-Child.
- The pinned specs `loom-discussion-spec.md`, `shed-recipe-spec.md` and `loom-status-spec.md`, the package docs, stencils and the Code-Review rubric are updated to match.

`lym shed` and `lym loom` both stay with their verbs.
Pawl paths are untouched except the override-key move above.

## Design decisions

- `Config.WithTapestry` applies `tapestry.yaml`; `LoadConfig` never reads it.
  `reviewbudget` always checks the run-wide soft/hard pair.
- Two commits carry a `Pattern-Departure: PATTERN-fabric-vocabulary` trailer: the restore way forward says "the committed copy under _lym" because a bare `weft` token is forbidden outside its owner set.

## Check by hand

- Before landing, the orch must move any `arbiter*`, `tackler*` or `driver_kind` value in `profiles.yaml` blocks under `loom` or `darn` to `shed`, or those runs refuse.
  A seeded pair whose local `loom.yaml` or `darn.yaml` holds one refuses at load until it moves to the local `shed.yaml`.
  A hub base `loom.yaml` or `darn.yaml` that tuned one is dropped with no carry-over, so move each such value into a loom recipe's profile block under `shed`, or the hub silently takes `shed.yaml`'s value.
- Runs in flight may need `lym loom goto`: a darn run with no record parked on Drift-Baseline, Drift-Await or halted at Darn-Write goes to `Record-Write` then `lym loom resume`, and a `tapestry` run inside the Code-Review perch needs a goto because of the new edge order.
- `finalize-fix-in-place` removes Finalize's `on_reroute`; it must read `tapestry`'s new order after Parent-Align.
- Stale leftovers the change did not cover: `tools/tapestryrename/tables.go` still names `TapestryRetiredRows` and `NameTapestryPreflight`, `internal/hubreconcile/ensure_integration_test.go:556` mentions shed's `driver_kind`, and a comment at `internal/shedrecipe/entries_gate.go:124` says a darn run has no `PatternRecordPath`.
- Findings left open, to verify in the code:
  - `d13-retired-key-refusal-mechanism`: the mechanism that refuses a retired key under `loom` or `darn`.
  - `d4-decision-add-darn-refusals-way-forward`: the way-forward text of `decision add` refusals on a darn run.
  - `d4-differs-from-committed-restore-loses-answer`: restoring a darn record from its committed copy can lose a recorded answer.
  - `d4-inflight-darn-halt-unreachable`: the halt of an in-flight darn run with no record may be unreachable.
  - `d5-darn-guard-damaged-record-no-recovery`: recovery from a damaged darn record after the Darn-Write guard.
  - `card6-missed-moved-key-readers`: readers of the keys moved to `tapestry.yaml` that the move missed.
  - `tapestry-run-wide-key-routing`: routing of run-wide keys for `tapestry` between `loom.yaml` and `tapestry.yaml`.
  - `implementer-spec-stale-loom-implementer-sentence`: the implementer spec's `loom.implementer` sentence, fixed in the last commit and not re-reviewed.
- The verify gate first failed in `internal/loomrecipe`; the fix commit `cbec60f4` makes `TestDarnVerifyBudget` expect Darn-Write gates without the removed discussion gate.
