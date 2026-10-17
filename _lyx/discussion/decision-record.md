# Decision record — landing-trailer-text

## Goal

Let the operator decide the landing commit's trailer block: fixed lines from `landing.yaml`'s `trailer_text`, plus one line per engine that ran a session in the task, from `engine_trailers`, each copied verbatim and each switchable off.

## Scope

- `landing.yaml` loses `co_authored_by` and gains `trailer_text` (a string, possibly multi-line) and `engine_trailers` (a map from engine name to trailer text).
- Finalize composes the landing commit message from the change description and that trailer block, with no prefix of its own.
- Shuttle keeps a durable record of the engines its sessions in a worktree ran under, and exposes the set that ran.
- `lyx config reconcile` migrates an existing `co_authored_by` value into `engine_trailers.claude`.
- Docs, specs and stencils that name `co_authored_by` or the single `Co-Authored-By` trailer are updated.

Out of scope: the Describe validation, which keeps refusing a `Co-Authored-By` line in the change description; any new engine.

## Decisions

### Trailer block composition

The trailer block is `trailer_text` trimmed of surrounding whitespace, followed by the trimmed `engine_trailers` text of each engine that ran, in sorted engine-name order, joined by newlines.
An empty or whitespace-only value adds nothing.
The landing message is the change description's commit message, then a blank line and the block when the block is non-empty, and the bare commit message otherwise.

Rationale: the operator decision asks for verbatim copies, and sorted order keeps the message stable across runs.

### Which engines ran

Shuttle's Start records each started run's engine name in an engine ledger file at the root of the worktree's run directory, beside the per-run directories.
`Runner.EnginesRun` returns the sorted union of the ledger's names and the engine of every run record still under that root, a record with no engine reading as the engine set's default.
Finalize reads it through a new `Deps.EnginesRun` seam, which loomcli wires to the run's runner; a nil seam names no engine.

Rationale: per-run directories are swept once their strand is gone, so they alone cannot answer at landing; the orphan sweep removes directories only, so a file at the root survives the run.
The run-record scan covers sessions started by a binary that predates the ledger.
Every runner of a task worktree anchors at the same run directory root, so the set is the task's own sessions.
Bound: the ledger only grows within one worktree and is never tracked, so it dies with the worktree at teardown.

### Validation

`engine_trailers` keys must be engine names `modelspec` knows; an unknown key fails `LoadConfig`, naming the key and the known engines.
`trailer_text` and the map values take any text, empty included.

Rationale: a typo in an engine name would otherwise drop that engine's line silently.

### Config categories and open map

`trailer_text` and `engine_trailers` are `CategoryCrossPair`, like `co_authored_by` was, and `engine_trailers` is an open map so reconcile carries the operator's entries whole.

Rationale: both shape what lands on the parent branch.

### Migration

`landingshed.MigrateConfig`, wired as landing's `Migrate` hook, rewrites a present `co_authored_by: <value>` into `engine_trailers: {claude: "Co-Authored-By: <value>"}` and drops the old key.
A document already holding `engine_trailers` keeps it and drops `co_authored_by`, reporting the dropped value.

Rationale: keeps an existing hub's trailer unchanged across the upgrade, as the operator decision asks.

### Template default

`trailer_text: ""` and `engine_trailers: {claude: "Co-Authored-By: Claude <noreply@anthropic.com>"}`.

Rationale: the operator decision's default, which keeps today's landing commit for a Claude-only run.

## Constraints

- `PATTERN-tolerant-records`: the ledger is written and read through `internal/state`.
- `PATTERN-shuttle-provider-seam`: the ledger stores engine names only and names no provider.
- `PATTERN-told-geometry`: landingshed reads engines through a told seam and derives no shuttle path.
- `PATTERN-durable-vs-ephemeral-state`: the ledger lives under the never-tracked run directory root.

## Auto-mode assumptions

- A darn run holds no interview; every call here is the writer's own.
- The engines that ran in the task worktree are the engines of the run, since a task worktree hosts only that task's sessions.

## Open risks

- A run in flight when the binary changes may have had earlier run directories swept before the ledger existed; such a run lands without that engine's line unless a later session of the same engine started.

<!-- lyx:carry-over Darn-Review -->
- Darn-Review, round 1: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lyx/reviews/darn/round-1-review.md; fixer report: _lyx/reviews/darn/round-1-fixer-report.md.
  - start-ledger-write-failure-untested (class design, severity MEDIUM, open in rounds 1)
<!-- /lyx:carry-over Darn-Review -->

## Acceptance criteria

- A landing with an empty `trailer_text` and an empty `engine_trailers` has no trailer block.
- A landing with fixed lines and a Claude session lands with the fixed lines, then the Claude line.
- An engine that did not run, or one whose text is empty, adds no line.
- A run directory swept after its strand ended still counts its engine through the ledger.
- A run record with no engine field counts as the default engine.
- `lyx config reconcile` turns `co_authored_by: X` into `engine_trailers.claude: "Co-Authored-By: X"`, and a second pass changes nothing.
- An `engine_trailers` key naming an unknown engine fails `LoadConfig` with the key named.
