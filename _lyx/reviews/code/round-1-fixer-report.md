## Changes

- F2 (LOW), fixed as proposed: `internal/gatecli/green_integration_test.go` adds `TestGreenRunRecordPass`, a tier1 test with a positive control; a pass whose tree changed before `recordPass` leaves `green.json` without an entry.
- F3 (LOW), fixed as proposed: `internal/gatecli/emptytier_test.go` adds rows for `-skip`, `-list`, `-bench`, `-test.run=X` and `--run`, each expecting no failure line.
- F4 (LOW), fixed as proposed: the `internal/gatecli/doc.go` gate.yaml example compiles `tier1,integration` and `tier2,tmux`, and vets `tier3,llm`.
  The self-check also had the tier0 example compile `-tags tier0`.
- F5 (LOW), fixed as proposed: `newestWholeAncestor` takes the newest by history and compares write time only between commits unrelated by history.
  Its `isAncestor` seam takes `(commit, of)`, and `scope.go` passes it through.
  `TestGreenRecord_NewestWholeAncestor` gains rows with an older commit written later.
- F6 (NIT), fixed as proposed: `greenRecord.with` counts only whole entries with empty args against `greenWholeCommitsKept`, and keeps args-carrying whole entries only for the current tree.
  A new `TestGreenRecord` subtest pins it.
  The constant's comment, the `recordGreen` comment and `doc.go` state the rule.
- Commits made after tests were green: `99355ea88` (F3), `1b60b5791` (F4), `65e148384` (F2), `4d5eb791a` (F5 and F6 together, as one change touches both in `green.go`), `17b45269b` (self-check follow-up).

## Disputed

- F1, ground `premise false`: `TestRelaunch` in `internal/shedengine/goto_test.go` already has the rows the finding asks for.
  "backward target keeps the pause conditions and clears the entered row" seeds `enteredSeed(StateBlocked, "B", "B")` with target A and asserts `got.EnteredRow == ""`.
  "no target writes the status unchanged, the entered row included" and "target in effect writes the status unchanged, the entered row included" assert the seeded entered row is kept, through the table's unchanged-status comparison.
  Reverting `cur.EnteredRow = ""` in `Relaunch` fails the first row.
  The rows were added by commit `3b620f678`, which precedes this review.

## Deferred

Nothing deferred.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency, LOW, `doc.go` pruning paragraph stale against F6: fixed as proposed.
- fix-consistency, LOW, `loomshed` fixtures still use the old tags: disputed, `premise false`.
  The fixture module's test files carry no build tag, so the empty-tier check has nothing to trip on.
  The fork itself marked this unverified.
- fix-consistency, third item: the fork reported no finding.
- fix-proposal, LOW, tier0 doc example lacks `-tags tier0`: fixed as proposed.
- fix-affected-set, LOW, stale `greenWholeCommitsKept` comment and `doc.go` budget text: fixed as proposed, merged with the fix-consistency finding above.

Gate: `lyx gate test --upto 1 ./internal/gatecli` was green before the follow-up commit, and `--tier 0 ./internal/gatecli` was green after it.
