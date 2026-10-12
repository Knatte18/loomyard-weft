# Fixer report: discussion round 3

## Changes

- F1 (BLOCKING), fixed as proposed.
  D10 now tests `hand` only for runs mapped by the exact join, from the tag tree's loom history, and states a time-joined run is never `hand` because Finalize writes the checkpoint it joined through.
  An untagged time-joined run is identified by its slug, contributes to the landing table only, and has no `land-incomplete` flag; the flag sentence now names tagged runs.
  Scope Out's in-flight bullet gains that a landed pair not yet torn down maps its landing but enters no per-run figure.
  The criterion on `hand` and not-yet-torn-down landings asserts label kind kept, never `hand`, no flag and no per-run row.
  Support-log Q7 updated to match.
- F2 (MEDIUM), fixed as proposed.
  D10's time join now opens a tagged run's window at the newest checkpoint dated in D4's previous-tag-to-this-tag interval and an untagged run's at the newest checkpoint after the slug's newest tag, closes it at the first board done commit after that checkpoint, and reads committer dates for every comparison.
  New criteria: a reused slug maps each landing to its own run, a run with two checkpoints opens at the newer, and a run with no checkpoint in its interval or no later done commit gets no time join.
- F3 (MEDIUM), fixed as proposed.
  D10 states a fast-forward landing's task-branch commits belong to that landing and are neither landings nor `unmapped`; D11 states they are never blamed.
  New criterion for a non-squash fast-forward of several commits.
- F4 (LOW), fixed as proposed with the recommended option.
  The window criterion is narrowed to a commit another tagged child's exact join already claimed; the residual hand-landing and direct-push case is in Open risks, and the rejected exclusion rule is in the support log's rejected alternatives.
- F5 (MEDIUM), fixed as proposed.
  D6's fourth bullet defines a failure episode.
  Walking D4 to D11, criteria were added or extended for: the generic wake kind, keep-alive pings, a round with no delivery, Worktree-Teardown episodes and a second Land episode, per-point alignment outcomes, a Merge-In first verify failure as an automatic step and a second as a wake, `paused` reported as its kind, another-recipe tags as `partial`, `shedrun.ReadSeed` staying strict, a `blocked` stop's anomaly class, unapplied arbiter verdicts, `hand` landings as LLM steps, parent decisions per `by`, a numbered round without a review file, and `feature` and `other` landings never blamed.

## Disputed

None.

## Deferred

None.
