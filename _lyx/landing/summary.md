# Implementer seam: batching inside Webster, engine by loom.implementer

Loom's implementer becomes a seam ahead of Weaver: batching moves into the implementer, the implementer row's contract is written down, and a setting chooses the row's engine.
Behaviour is otherwise unchanged; Webster still partitions the plan once per run.

## What changes

- The `Batchifier` row leaves `contracts/recipes/loom-recipe.yaml`, and its engine (`internal/loomshed/batchifier.go`), registry entry, row name constant and interrupt-policy entry go with it.
  `Plan-Bouncer`'s `on_done` now names `Webster`.
- The Webster producer keeps resolving `batcher.Active` at the start of every `Call`.
  The reason prefix and way forward the row shared with Webster and Loom-Preflight move into `internal/loomshed/webster.go`.
- A new `Implementer` registry engine in `internal/shedrecipe` builds the implementer row.
  The row's Config names its setting as `implementer_setting: loom.implementer`, read through `Env.Setting`; the value selects a constructor from a closed table that holds `webster` alone.
- `loom.yaml` gains `implementer`, default `webster`, settable in the local layer and per task through `run_settings`.
- `contracts/specs/implementer-spec.md` is new and writes down the implementer row's contract: in, the approved plan and the task branch; out, commits on the branch with a clean tree and the implementer's run directory with its state file; outcomes Done, a Stuck reroute to plan review, or a halting Stuck.
  `webster-spec.md` points at it.
- `lyx report timing` maps a past run's `Batchifier` row to the webster phase, so old histories count batching with the implementer instead of under other.
- Docs follow: `PATTERN.md`, `README.md`, `docs/overview.md`, the `shedrecipe` and `loomshed` package docs, and the loom-plan and shed-recipe specs.

## Design decisions

- **Dropping the row loses no early failure.**
  The `Batchifier` row was a fail-fast gate that Loom-Preflight's `batcher.yaml` load already covers before any LLM row; a batcher config that breaks later still stops at Webster.
- **The setting only chooses among constructors of the same row.**
  Every implementer builds the same row with the same edges, so no recipe row is skipped or entered conditionally.
  An absent, empty, non-string or unknown value refuses at recipe build, naming the setting, the accepted values and the `lyx config loom --set implementer=<implementer>` command.
- **The row keeps the name `Webster`**, and the `Webster-Review` segment is unchanged.
  The row name is a durable on-disk identity, and the name Weaver's row carries is Weaver's decision.
- **The implementer table is a second `map[string]Constructor` in `internal/shedrecipe`.**
  `PATTERN-shed-recipe-registry` is reworded to bar it from the no-`init()`/`Register` rule's scope.
- **No status migration.**
  A loom run parked at `Batchifier` is restarted, as the recipe header records for earlier retired rows.
- **Out of scope:** Weaver itself and other Weaver groundwork, such as a cold-agent spawn per batch in its own worktree.

## To check by hand

- A hub whose `loom.yaml` predates the `implementer` key reads the template default through the layer stack; reconcile adds the key on the next binary change.
  Not run against a real hub.
