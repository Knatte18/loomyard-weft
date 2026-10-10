# Plan round 2: fixer report

The one finding was validated against the plan, the decision record and `internal/orchengine/watcher.go`, and fixed.
`lyx loom validate-plan` reports `ok: true` after the edits, with informational `create-new-unit` notes only.

## Fixed

### F1: the two added watcher decisions are settled by opposite precedence rules

Confirmed.
Shared Decision `watcher-index-binary-reading` let D4, Scope and the acceptance criteria outrank "Watcher index binary", while card 14 let "Missing-binary bound at auto-reload" outrank D4's Bound, Scope Out's carve-out and the acceptance criterion, with no stated reason.
At HEAD, `startAutoReload` (`internal/orchengine/watcher.go:834-836`) returns a `RenderRoleFile` error as a failed tick, while `startClearing` (`:760-761`) and `reloadAfterCompaction` (`:1077-1078`) go idle with the reason.

I took the review's first option and kept the stated rule for both, because each added decision rests on a premise the record's final text contradicts.
"Missing-binary bound at auto-reload" says Scope keeps the reload steps out, but Scope Out names the `startAutoReload` change as its one exception.
"Watcher index binary" declines launch-path plumbing that Scope "In" lists and points at an Open risks entry that does not exist.

- `00-overview.md`: Shared Decision `watcher-index-binary-reading` is replaced by `added-watcher-decisions-yield-to-d4`, which states one rule (where an added decision contradicts D4, Scope and the acceptance criteria, the plan follows those sections) and names both conflicts and how each is settled.
- Card 14's `Intent`: every render site, `startAutoReload` included, goes idle with the reason when the index source or `RenderRoleFile` fails, never a failed tick, and this is named as the only change to the reload steps.
  The auto-compaction reload test's failing-source case now asserts idle with the reason and no tick error.
  `ImpactSummary` names the `startAutoReload` behavior change.
- Card 13 is unchanged: the rule keeps it in scope.

## Disputed

None.

## Deferred

None.

## Constraint conflicts

The decision record still contradicts itself in its two "Added after Discussion" watcher entries; the plan now resolves both by one stated rule rather than silently.
