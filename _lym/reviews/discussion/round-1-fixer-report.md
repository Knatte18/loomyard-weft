# Fixer report — discussion round 1

F5 is `decision` class and stays with the parent; this report leaves it out.

## Changes

All in `_lym/discussion/decision-record.md` unless named otherwise.

- F1: D9 registers a pass under its worktree's owner, `runproc.HubOwner` in the prime and the task's slug in a task, so the pair's teardown ends it per `PATTERN-child-processes`.
  Its Bound limits the one-pass refusal and `TestMapRunning` to passes of the same worktree.
  Added the D9 criterion and a `PATTERN-child-processes` Constraints line.
- F2: D11 imports each old map whose package has no map in the prime, and removes an old map only once the prime holds a committed map for its package.
  The Bound is restated that way, and there is a new criterion for a call after a killed import.
- F3: D6 states the two conditions for sharing, the machine env layer and the commit being present in the reader's store, and says both degrade to whole-package runs.
  The Constraints env-digest fact names the machine layer.
- F4: D7 moves every reader that takes `commit == HEAD` to mean current over to the planned HEAD, and names the audit's candidate filter.
  There is a new D7 criterion for the audit.
- F6: D7's Bound cites the `internal/fabricengine` package doc and the `MissCommitUnreadable` → `UpkeepMiss` → whole rebuild outcome.
- F7: D7's skip condition now reads "whose entries, queue and failure record are unchanged by the plan".
- F8: D8 names `internal/state`'s compact form, which keeps the unknown-key carry-over, and says "field order".
- F9: D4 keeps the exit record and build locks under `.lym/testcover/` through a scratch accessor beside `testcover.Dir`, and puts each map's lock where `lock.TrackedFileLockPath` puts it.
  The `PATTERN-lym-files` Constraints line follows.
- F10: D11 names the operator's hand `lym gate testmap` in the prime as the first call, and Open risks states the window with no maps until that call.
- F11: D11 runs the removal before D6's refusal on any machine and the import after it on the owner only, and says how a non-owner's old bolt maps go.
  Added a criterion for a non-owner call.
- F12: D10 says "in the prime" and "a layer that holds the key", and its criterion follows.
- F13: D7's restatements of Constraints are cut as proposed.
- Outside the findings: Scope read "D1 to D10" although the record holds D11, so it now reads "D1 to D11".
- `_lym/discussion/support-log.md`: Review rounds records round 1.

## Self-check

- fix-consistency, LOW, D6's lead-in promised conditions but listed failure cases: fixed as proposed, the lead-in now states both conditions.
- fix-claims, MEDIUM, and fix-proposal, MEDIUM (merged): the audit, which runs inside the pass, cannot read the exit record, and the guard in `testmappackage.go:82` would skip an unwritten map. Fixed with my own change.
  D7 now says that inside a pass the planned HEAD is the pass's own HEAD, and it names both readers. It adds that a test run into an unwritten map writes the planned map at HEAD.
- fix-claims, LOW, D7 Bound said "rewrite" where the doc says "rewind" and left out that the object survives: fixed as proposed.
- fix-proposal, LOW, D11 might re-import an old map that has no committed counterpart: fixed with an own sentence. D4's commit takes every imported map whether or not the pass plans its package, and no code deletes a map, so each old map gets a counterpart.
- fix-proposal, LOW, teardown might treat a task's `RoleTestMap` record as busy: disputed, premise false. Teardown ends the slug's processes and refuses only on survivors of that end (`internal/pairteardown/teardown.go:342`). `ownedProcesses` feeds only the refusal's text (`teardown.go:611-614`).
- fix-way-forward: no findings.

## Disputed

None of the review's findings.

## Deferred

None.

## Rulings

None reached this round.
