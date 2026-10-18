# Decision record — implementer-seam

## Goal

Make loom's implementer a seam before Weaver: batching sits inside the implementer, the implementer row's contract is written down, and a setting chooses the row's engine.

## Scope

- The `Batchifier` row leaves `contracts/recipes/loom-recipe.yaml`, and its engine, registry entry, row name constant and interrupt-policy entry go with it; `Plan-Bouncer`'s `on_done` names `Webster`.
- A new `Implementer` registry engine builds the implementer row from the engine a `loom.implementer` setting names; the `Webster` row uses it.
- `loom.yaml` gains `implementer`, default `webster`, settable per task through `run_settings`.
- `contracts/specs/implementer-spec.md` writes the implementer row's contract down.
- `lyx report timing` counts a retired `Batchifier` row under the webster phase.
- Out of scope: Weaver itself and any other Weaver groundwork, such as a cold-agent spawn per batch in its own worktree; renaming the `Webster` row or the `Webster-Review` segment.

## Decisions

### Webster alone resolves the batcher

Decision: the Webster producer keeps resolving `batcher.Active` at the start of every `Call` and partitions the plan once per run, as it does today; the `Batchifier` row and its engine are deleted, and the reason prefix and way forward they shared with Webster and Loom-Preflight move into the Webster producer's file.
Rationale: the row was only a fail-fast gate that Loom-Preflight's `batcher.yaml` load already covers before any LLM row, so dropping it loses no early failure, and batching becomes the implementer's own concern.

### An Implementer engine chooses the row's engine from a setting

Decision: the registry gains an `Implementer` engine whose row Config names its setting as `implementer_setting: loom.implementer`, read through `Env.Setting`; the value selects one constructor from a closed implementer table in `internal/shedrecipe`, which holds `webster` alone, and the remaining Config keys pass to that constructor.
An absent, non-string or unknown value refuses at recipe build, naming the setting, the accepted values and the command that sets it.
Bound: the setting chooses only among the table's constructors, every one building the same row with the same edges, so no row is skipped or entered conditionally.
Rationale: PATTERN-recipe-setting-lookup routes a plain row value through `Env.Setting`, and an engine table beside the registry keeps every engine name declared in `internal/shedrecipe`.

### The row keeps its name

Decision: the implementer row stays named `Webster`, in the `Webster-Review` segment's ring unchanged.
Rationale: the row name is a durable on-disk identity, and the name Weaver's row should carry is Weaver's decision.

### loom.implementer is a plain loom.yaml key

Decision: `implementer: webster` joins `loom.yaml`'s template with category none, so the local layer and a task's `run_settings` may set it; the `Implementer` entry is its only validator.
Bound: a local value can only pick another implementer from the closed table, and every implementer passes the same Webster-Review perch afterwards.
Rationale: one validator keeps the accepted values in one table; the refusal reaches a run at recipe build, before any LLM row.

### The contract is a reference spec

Decision: `contracts/specs/implementer-spec.md` pins the implementer row's contract: in, the approved plan under `_lyx/plan/` and the task branch checked out in the worktree; out, commits on the task branch with a clean tree, the implementer's own durable run directory with its state file, committed on Done, and an outcome of Done, a Stuck reroute to plan review, or a halting Stuck; batching and its config are the implementer's own.
It is an unregistered reference doc beside `webster-spec.md`, which points at it.
Rationale: no agent reads it at run time, so it needs no deploy entry.

### Timing folds batching into the implementer phase

Decision: the loom phase table maps the retired row name `Batchifier` to the webster phase, so a past run's batching time counts with the implementer.
Rationale: run histories written before the change still carry the row, and a missing mapping would count it under other with a finding.

### A run parked at Batchifier restarts

Decision: no status migration; a loom run whose status names the retired row is restarted, recorded in the recipe header as for earlier retired rows.
Rationale: that is the recipe's standing policy for retired rows.

## Constraints

- PATTERN-shed-recipe-registry: the new engine and the implementer table live in `internal/shedrecipe`, with no `init()` and no `Register`.
- PATTERN-recipe-setting-lookup: the implementer row reads `loom.implementer` through `Env.Setting`.
- PATTERN-refusal-way-forward: the `Implementer` entry's refusals name the way forward, and a test reaches each.
- `internal/loomrecipe`'s coverage guard keys loom's rows off `internal/loomshed`'s `Name*` constants, so the constant and the recipe row leave together.

## Auto-mode assumptions

- The board's "a setting such as `loom.implementer`" is taken as exactly `loom.implementer`, with value `webster`.
- The Open question about other Weaver groundwork is left to Weaver.

## Open risks

- A hub whose `loom.yaml` predates the key reads the template default through the layer stack; reconcile adds the key on the next binary change.

<!-- lyx:carry-over Darn-Review -->
- Darn-Review, round 2: converged; each finding below was fixed or disputed by the round's fixer, and no fresh reviewer has seen the result; the fixer report says which.
  Review: _lyx/reviews/darn/round-2-review.md; fixer report: _lyx/reviews/darn/round-2-fixer-report.md.
  - summary-claims-reverted-creel-fix (class consistency, severity MEDIUM, open in rounds 2)
<!-- /lyx:carry-over Darn-Review -->

## Acceptance criteria

- The loom recipe has no `Batchifier` row, and Plan-Bouncer's Done routes to Webster.
- With `loom.implementer` at `webster`, the `Webster` row builds the Webster producer, gates included.
- An unknown, empty or non-string `loom.implementer`, and a row without `implementer_setting`, refuse at build with a way forward.
- A broken `batcher.yaml` still stops a run at Loom-Preflight, and at Webster when it breaks later.
- `lyx report timing` counts a `Batchifier` row in a past run's history under the webster phase.
- `contracts/specs/implementer-spec.md` exists and states the in, the out and the outcomes.
