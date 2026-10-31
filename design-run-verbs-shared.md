# loom becomes the task-run contract that tapestry and darn fulfil

`run-verbs-shared` · Task · enhancement, shed, loom, darn, landing, drift · draft

## Seen (operator, 2026-10-14)

Since `0850f76e` a darn run has a decision record, and `lym loom decision add` works on it.
The operator found it absurd that a `loom` command acts on something that is not a loom run.
The same holds for the other run verbs the orch uses on darn runs: `lym loom status`, `goto`, `resume`, `start` and `circling`.
A further recipe is planned, `Hardener` (the note `hardener`).
`tapestry-rename` renamed today's loom recipe to `tapestry`; this task gives `loom` its new meaning.

## Three layers (operator, 2026-10-31)

- Shed is the engine: the state machine, the recipe loader and the producer registry (`internal/shedrecipe/registry.go`), seeds, the perch machinery, arbiter, tackler and driver, and the `lym shed` verbs.
- Batten is the frame around a run, the same for every recipe: it makes the worktree, runs the recipe, deploys after landing and tears the worktree down.
- Loom is the task run, and `loom` names a contract rather than a body of producers:
  a loom recipe takes one board task in, holds a decision record as the task's decided scope, has a writer that writes the code, and ends with Finalize landing one change in the parent.
  Shed checks a recipe that declares itself a loom against that contract when it loads it.
  The `lym loom` verbs act on every run whose recipe is a loom, and nowhere else.

`tapestry` and darn are the loom recipes; this task does not touch pawl.
A recipe is a recipe file choosing producers from the registry, and a producer stays in the module that owns it: Parent-Align, Finalize, PR-Review, Describe and the merge fixer in `landing`, the Drift rows in `drift`, the perch in `shed`.
A variant owns only the producers no other recipe uses: Plan, its perch and Webster for `tapestry`, Darn-Write, tree-scoped verify and the prior-work respawn for darn.
Any recipe may take any other producer, so darn can add Parent-Align later with no code change.

## The decision record

The record keeps today's format and parser, and a recipe makes it one of two ways:
- with Discussion: a Discussion writer makes the record thorough from the board entry and a perch reviews it, as `tapestry` does today and always will;
- without: the run writes the record mechanically, with no LLM, the board entry's body verbatim under `## Goal` and every other section empty, and the writer plans and writes the code from it in one session, as darn does by default.
  This fits `loom-beam-darns`, whose darns are pieces a Discussion already cut.

Either way `lym loom decision add` appends later decisions to the record as `Added after Discussion` entries, as today.
The record is the decided scope: Code-Review checks the change against the record, never against the board entry's text, so darn's snapshot of the entry is no longer what its review compares with.

## Shared code written once

Both recipes already share one CLI (`lym loom`), one wiring path (`internal/loomcli/wiring.go`, with `wireDarn` overriding fields) and one recipe builder (`internal/loomrecipe`).
What each writes twice today becomes one copy driven by the recipe: Describe (`DescribeTable` and `DarnDescribeTable` in `internal/landingshed/describe.go`, with two stencils), the description commit, the writer wrappers (`loomshed/darnwrite.go` and `loomshed/discussionwrite.go`), the Discussion gate and validator (`NewDiscussionGate` and `NewDarnDiscussionGate` in `internal/loomshed/gates.go`), the verify source, and the routing, shed and retired-row lists (`Routing` and `DarnRouting`, `New` and `NewDarn`).

The row order is one for every loom recipe: Drift-Verify runs before Code-Review, as darn does today, since Drift-Verify can change the code a good deal, in `tapestry` especially, and the review must see those changes.

## Verbs that apply to one recipe

A `lym loom` verb that applies to one recipe refuses on another, saying it does not apply; the task checks each gap below and fixes what is real:
- `lym loom peers` reports a darn run's row position against tapestry's rows (`internal/loomcli/peers.go`, `hubPeersDeps`).
- `lym loom decision add` seems to accept `--pattern-*` and `--amends-plan` on a darn run, which has no PATTERN record or plan (not verified).
- `validate-plan`, `insert-card`, `remove-card` and `restore-renumber` seem to carry no recipe check, so on a run without a plan they fail on a missing plan (not verified).

## Config split

`loom.yaml` holds what every loom recipe shares: `discussion_*`, `code_*`, `review_*` and `friction`.
A new `tapestry.yaml` takes what only `tapestry` has: `plan_*`, `implementer` and the other Plan and Webster keys.
`darn.yaml` keeps darn's own keys: `writer`, `writer_timeout_min`, `tree_scope` and `verify_attempts`.
The hub's `profiles.yaml` blocks then name the module each key now sits in; the orch rewrites the hub's own copy after landing, and the run writes no migration for it.

## Tapestry-Preflight goes

The Tapestry-Preflight row is removed.
Its seed check repeats what Shed's own status read before every row already refuses (`internal/loomengine/seed.go`, `CheckSeed`), so it goes with the row.
Its check that `batcher.yaml` loads moves to where Webster is used: into Preflight for a recipe that runs Webster, or into the Webster row itself.

## Old recipe alias goes

`legacyRecipeBeam` in `internal/shedrun/seed.go`, which reads a seed recording the pawl recipe's former name `beam` as pawl, is removed with its handling in `decodeSeed`: no run seed records `beam`.

## Open

- `arbiter*`, `tackler*` and `driver_kind` sit today in `shed.yaml`, `loom.yaml` and `darn.yaml`; whether they live in `shed.yaml` alone, a recipe setting its own through its profile block.
- Whether `lym loom validate-discussion` accepts a record whose sections bar `## Goal` are empty, or the skeleton needs a fixed line in each.
- Where the check that the entry is not outdated (landed work or another entry contradicting it) lives once the record is written mechanically: in darn's writer, as today, or a step of its own.
- How `lym loom` relates to `lym shed`, which already carries `run`, `step`, `status`, `pause` and `goto`.
