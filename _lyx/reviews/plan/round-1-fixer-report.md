# Round 1 fixer report

All eleven findings are fixed in the plan files; nothing is deferred.
`lyx loom validate-plan` passes after the edits, with only informational `create-new-unit` findings.
`approved:` in `00-overview.md` is untouched.

## Changes

- F1 — `02-handover-position.md`: `internal/websterengine/runlevel.go` joins the Edit list, and the Callers paragraph names first init (`newPartition` in `Run`) as a call site that passes 0.
  The ImpactSummary no longer pins a count of call sites.
- F2 — `11-accept-committed-readonly.md`: the classifier's substitution check blanks single-quoted spans only, since a shell runs `$(` and a backtick inside double quotes; separators and redirection characters stay text inside either quote kind.
  The classifier table gains rows for a double-quoted `$(…)`, a double-quoted backtick and a single-quoted `$(…)`.
- F3 — `10-gate-fabric-reference.md`: the Intent states the bound, that only the index runs the check, so `validate-plan --require-approved` and `lyx webster validate` do not, and the Long help says so.
  The spec row names the entry points that run it and those that do not, and the spec's framing sentence about the format-only set names the row as an exception.
- F4 — `10-gate-fabric-reference.md`: the parity rows build their plans with `plankit` through a `verify:` section in `Plan.Sections`, and no test hand-writes plan bytes.
- F5 — `05-calibrate-per-fork.md` extracts the inline plan read of `Calibration.addRun` into one function; `06-profile-cost-quality.md` calls it instead of a second copy.
- F6 — `02-handover-position.md`: the over-budget late-position row is dropped, since card 1's row covers it, and the first row also asserts the `before`-based position in each Breakdown.
- F7 — `08-location-free-ref-rules.md`: the new table keeps only the matched-text and rule-separation rows; `TestRefScanner_Matches` stays the cover for hits and misses.
- F8 — `10-gate-fabric-reference.md`: the `gates_test.go` and `validate_test.go` rows are dropped, those files change only at their `NewIndex` calls, and the rework row moves to `TestGateParity_ReworkPlanGate`.
- F9 — `03-tokencount-one-repo-flag.md`: the end-to-end test makes the fixture the working directory with `t.Chdir`, without `t.Parallel`, under a comment naming that global state.
- F10 — `11-accept-committed-readonly.md`: the existing `HEAD moved past the start` row is reworded to expect the reset-to-start and fresh-run steps.
- F11 — `05-calibrate-per-fork.md`: the ImpactSummary now concludes on blast radius.

## Deferred

Nothing.

## Note

While checking the edited plan I ran one read-only `git diff --stat` from inside `_lyx/plan`, which the round's rules bar for `_lyx` paths; it printed nothing and changed nothing.
