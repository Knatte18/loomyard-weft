# Round 1 fixer report

## Changes

- F1 (fixed as proposed): D4's Decision now says every process that builds a code role's spawn loads the list itself, including `lyx webster recover-batch`, so a recovery strand can load a newer list than its master; D4's Rationale now says one load gives every spawn of one process the same list.
  Premise checked: `internal/loomcli/wiring.go` states the recovery prompts are composed in a separate `lyx webster` process, and `internal/webstercli/recoverbatch.go` builds `RecoverDeps`.
- F2 (fixed as proposed): the third Open risk now says a misspelled skill is skipped at once with a logged warning and nothing else signals it; support-log Turn 3's scenarios line says the same.
  Premise checked: `ClassifySkillLoad` marks a skill unknown when the listing does not name it, and `settleLoadTurn` in `internal/shuttleengine/run.go` skips unknown skills without a retry.
- F3 (fixed as proposed): D7's Rationale gains a `Bound:` line; the second Open risk now says each refused command costs a turn and the template comment only steers the operator.
- F4 (fixed as proposed): the acceptance criterion now excludes the Discussion and Plan review rows and their fix rows.
- F5 (fixed as proposed): deleted the `PATTERN-sandbox-coverage` Constraints line.

## Disputed

None.

## Deferred

None.
