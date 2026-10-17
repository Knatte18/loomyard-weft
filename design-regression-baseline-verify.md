# Regression tier: baseline profiles before the run, compare at the final gate

`regression-baseline-verify` · Note · feature, shed, webster, batten, config

## Seen (operator, 2026-10-17)

The operator's C# work repository has a regression tier: a suite run before the implementation writes profiles, and the same suite run after it compares against them to catch drift.
Many tasks there are pure refactorings that must not change behaviour, and this tier is what proves it.
Operator view, not a decision: regression tests are not always wanted, since some tasks expect drift, for example when a model is changed; this must be handled, perhaps by a flag on the task in the board.

## Orch recommendation, not a decision

- A per-task setting in the board entry's `run_settings`, with three values:
  - `off`: no baseline, no compare.
  - `preserve`: any drift fails the final gate, as a verify failure.
  - `expect-drift`: baseline and compare still run, but the drift report goes to the reviewers and the alignment agent, who judge whether it matches the task's goal; drift alone fails nothing.
  The hub's default lives in config.
- The hub declares the tier in `language.yaml` as three commands: the profile generator, the compare (the repository's own tool, with its tolerances) and the profile directory. lyx only orchestrates and knows no format.
- Operator view, not a decision: the baseline belongs in shed as a producer; it often takes long, so it should start at the beginning as an async command while Discussion and the rest run.
- Orch recommendation, not a decision: shed runs one producer at a time today, so async becomes a generic shed pair of ordinary producers, usable by any recipe:
  - `Job-Start` (a side action with a config flag) starts a detached job, records its handle in run state and returns at once.
  - `Job-Join`, placed before the row that needs the result (here the final verify that runs the compare), waits for the job and fails with its log when it failed.
- The baseline runs on the parent commit in its own detached checkout, never in the task worktree, which changes during the run.
- The job takes a gate slot like every test run, and obeys a hub pause like the run's other work (note `pause-background-work`).
- Batten tears the job down with the run.
- The baseline is cached hub-wide keyed by the parent commit, so runs from the same parent share one baseline run.
- When the parent moves and the run merges it in, the baseline is taken again at the new parent (a cache hit if another run made it), so other tasks' drift is not blamed on this one.
- The compare runs at the final gate only, never per card, to keep the cost down.

## Open

- How long the generator takes on the work machine, and whether it can be scoped like the other tiers.
- Whether `preserve` should also run the compare per batch, to point at the card that caused drift.
- Where the profiles live: run state under `.lyx/` or the records repository.

## Related

Notes `gate-language-neutral` and `dotnet-scoped-verify`.
