# Implementer seam: batchifier inside Webster, engine chosen by setting

`implementer-seam` · Task · enhancement, webster, loom, shed, report · running · Merge-In

## Operator decisions (2026-10-17)

- The plan writer writes a flat plan; how that plan is implemented is the implementer's call, and batching is part of implementing.
- So the batchifier belongs inside the implementer, and the type of batchifier is the implementer's own config.
- No recipe row is skipped, or entered conditionally, on which implementer runs.
- The groundwork Weaver needs is its own task, built and landed before Weaver itself.

## Seen

- Loom's recipe runs `Batchifier` as a row of its own before `Webster` (`contracts/recipes/loom-recipe.yaml`, engine in `internal/loomshed/batchifier.go`), and it partitions the whole plan once.
- A recipe row names its engine directly (`internal/shedrecipe/registry.go`); no setting chooses a row's engine.
- `internal/reportengine/timingspans.go` reports `Batchifier` as a phase.

## Idea

- Webster runs the batcher itself when it starts and partitions the plan once, as the row does today; the `Batchifier` row leaves the loom recipe; behaviour is otherwise unchanged.
- The implementer row's contract is written down in `contracts/specs/`: in, the approved plan and the task branch; out, commits on the branch, the run's state file and an outcome (done, reroute to plan review, stuck), so the review, Merge-In and later rows do not depend on which implementer ran.
- A setting such as `loom.implementer` chooses the implementer row's engine, settable per task through `run_settings`; `webster` is its only value until Weaver lands.
- `lyx report timing` folds batching into the implementer phase.

### Open

Other groundwork Weaver needs (see `weaver`), such as a cold-agent spawn per batch in its own worktree, may belong here or in Weaver itself.
