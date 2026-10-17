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
- A Baseline producer early in the run, a side action with a config flag, runs the generator on the parent commit before any card lands, inside a gate slot.
- The baseline is cached hub-wide keyed by the parent commit, so runs from the same parent share one baseline run.
- When the parent moves and the run merges it in, the baseline is taken again at the new parent (a cache hit if another run made it), so other tasks' drift is not blamed on this one.
- The compare runs at the final gate only, never per card, to keep the cost down.

## Open

- How long the generator takes on the work machine, and whether it can be scoped like the other tiers.
- Whether `preserve` should also run the compare per batch, to point at the card that caused drift.
- Where the profiles live: run state under `.lyx/` or the records repository.

## Related

Notes `gate-language-neutral` and `dotnet-scoped-verify`.
