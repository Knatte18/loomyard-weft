# Fixer report — discussion round 1

## Changes

- F1, fixed as proposed, with F2's narrower rule folded in: D2's Bound now says the step skips the guard tests of a dropped guard package directory for the cards from the emptying card on, and says what still catches them: a renamed guard test runs in the batch gate when its new directory is a target directory of the renaming card, and in the plan-level verify; a deleted one runs nowhere.
  Added an Open risks entry for this skip.
- F2, fixed as proposed: D2's drop clause now drops a directory only for the emptying card and the cards after it, and only once every Go file of the directory is renamed out or deleted.
  The acceptance row is restated, and a row was added for the card before the renaming card and for a one-file move.
- F3, fixed as proposed: D1's Bound and the matching Open risks entry now place the over-split on earlier glyph or handle targets whose unit is the renamed file's package directory.
- F4, fixed as proposed: D1's file-level bullet now covers a glyph or `plan:` handle, member or not, whose unit is a package directory equal to the file's directory, citing `planparser.RefScope`.
- F5, fixed as proposed, with a walk read error moved out of the degrade rule: D3 now degrades only on a `GuardScanError` or an over-long step, giving the over-long step its own rationale.
  begin-batch and recover-batch refuse on a walk read error, naming the path, and rerunning the verb once the path reads is the way forward.
  The prompt line asks the fork to fix the named file under its card's commit when the file lies in a package its card targets.
  The Bound says the error recurs at every later begin-batch.
  Added an Open risks entry for a persistent scan error and an acceptance row for the read-error refusal.
  The support log's rejected alternative "Refuse begin-batch on a guard-scan error" now says `GuardScanError`, to match.
- F6, fixed as proposed: D4's Bound says the fork may change a guard's pinned set or allowlist, that the prompt steers it to the code fix first, and that such an edit stays visible as a scope-guard deviation.
  Added an Open risks entry.
- F7, fixed as proposed: D2's walk skips `testdata` directories and `_`- or `.`-prefixed directories, as `go list` does.
  Added an acceptance row that a `_test.go` file under `testdata` is never scanned.
- F8, fixed as proposed: cut D2's plan-time-check Rationale sentence, and D3's Rationale now reads "the plan-level verify already reaches the fault".
- F9, fixed as proposed: Q6 now precedes Q7 in the support log's Question ledger.

## Disputed

None.

## Deferred

None.
