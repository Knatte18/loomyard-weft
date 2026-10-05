# Round 2 fixer report

All four findings fixed in `_lyx/discussion/decision-record.md`; `support-log.md` is unchanged.

## Fixed

- F1: D9 now defines a role's opening stencils as the first stencil the spawning module reads per D2 row, lists every stencil for multi-stencil rows, excludes step and body files, and has forks inherit the directive through the master.
- F2: D6 states that the marker records its request time, that the timeout runs from it across restarts, and that a watcher removes a marker older than the timeout and acts on nothing.
  A matching acceptance line is added.
- F3: D1 now states that a skipped `ly:board` also loses the orch's board mechanics, and names D7's single handoff timeout as the bound.
  D7 persists a skip in the phase so a restart does not re-type it.
  The two acceptance lines now say "loaded or, on a timeout, logged and skipped" and exempt a persisted skip from re-typing.
- F4: D10's `PATTERN-parent-directive` entry and test description now state that the scan reads stencil text only and that interactive questions come from the `{{.mode_rules}}` marker, with `modeRules` in `internal/loomengine/prompt.go` as Go text the scan never sees.

## Verdicts on the focus questions

- The record names the interactive discussion mode as an exception with a stated bound (Goal, D9); not a defect.

## Deferred

None.
