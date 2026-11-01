# Bench Parts-Write and Parts-Plan-Write against Plan-Write

Adds the research report of the plan-split study under `docs/research/plan-split-bench-2026-11-01/`, with its script and data.
No code, config or PATTERN entry changes.

## What the study did

The two plan paths, `Plan-Write` alone and `Parts-Write` then `Parts-Plan-Write` (`tapestry.plan_split: true`), wrote the plan of the same input, replayed at the start of Plan on a scratch worktree at the input's base commit.
The inputs are the landed runs `warp-visibility` and `run-verbs-shared`, with the models and efforts the hub's config gives each producer.
Each plan got one Plan-Review round, stopped before its fixer.

## Findings

- On `warp-visibility` the split cost $16.69 to write against $14.93 and took 15m39s against 25m07s; its largest session peaked at 245k context against 408k for the plan writer.
- The split plan had 34 cards against 15, with 80 card dependencies against 29, and its review found one more blocking finding (4 against 3).
- On `run-verbs-shared` the plan arm cost $28.03 and its writer peaked at 674k context; the split arm failed at the chair wait when reed lost the chair's strand, so that input has no comparison.
- The split's joined plan passed the `plan` gate first time only after two hand edits to `parts.md`, each made because a gate would otherwise have refused the run.
  The edits fixed an export that broke `PATTERN-cli-cobra` and names that broke `PATTERN-fabric-vocabulary`, both defects of `Parts-Write`.
- The report recommends keeping `plan_split` off, and making the parts check refuse those two export defects before a repeat.

## Design decisions

- The scratch worktrees came from the orch (`lym fabric add --local`), because the verb is refused in a task session; `plan_split` was set only in the two split worktrees' own config layer.
- The `run-verbs-shared` split arm stopped at the reed failure, with no rerun beyond the cap of four plan writings, as the parent decided.
- `Drift-Baseline` did not run in the replays, to stay inside the cap of agent runs.
- The hand-edit rule is the parent's: a replay is edited only when a gate would otherwise refuse it.

## Verification

No code changed, so no test applies.
The report's figures come from `go run ./tools/tokencount`, `go run ./tools/plandag` and `session-context.sh` over the four scratch runs; their outputs are in the report directory.
