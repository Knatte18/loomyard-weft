# Plan-Burler round 2 — fixer report

Every finding was validated against the code and fixed; none is disputed or deferred.
`lyx loom validate-plan` passes on the fixed plan, with informational `create-new-unit` notes only.

## Fixed

- **F1** (card 13): `internal/shuttleengine/doc.go` joins the Edit list, and a Docs paragraph adds the wait loop's shadow logging to card 8's session-state section: state-change and disagreement logs, the fixed mapping, the three expected pairings, no branching on the state, and the cursor's start and its trailing-stamp stop.
- **F2** (card 12): `internal/websterengine/triage_test.go` joins the Edit list.
  The Intent states that only the `ran past <bound>` head of `expiredShellWarning` is replaced, and that the bound with its minutes, the `stopped waiting on the shell at a turn end, …` clause, `lyx did not stop the shell` and the strand-ending and final-outcome clauses stay byte-identical, which keeps `webstercli/verbs_test.go`'s `TestRunCmd_DiedMasterNotesExpiredShellOutcome` valid.
  The Tests paragraph names `TestBackgroundShellNoteAndWarning`, whose substring list keeps its entries and gains the new head's wording.
- **F3** (cards 10, 11): confirmed that `TestRun_Wait_HeldTurnEnd` (`wait_test.go:396`) drives held turn ends with a `fakeReed` recording `WaitMarkCalls`, and that `TestWait_HeldTurnEndNotice` (`hold_test.go:33`) drives `Wait` past the shell bound with `shellJump`.
  Card 10 folds its mark assertions into `TestRun_Wait_HeldTurnEnd`, swaps `waiting_test.go` for `wait_test.go` in its targets and drops its budget row raise and target.
  Card 11 adds the payload-shell deadline case as a row of `TestWait_HeldTurnEndNotice` (with `hold_test.go` in its targets), keeps one new function only for the log-once and next-turn-end behavior that needs both a clock jump and a script plus log capture, and replaces the false "no existing test" sentence.
  Card 11's `internal/shuttleengine` raise of 1 stands, for that one function.
- **F4** (card 11): `internal/shuttleengine/run.go` joins the Edit list, and the Docs paragraph rewords the `Result.ExpiredShells` field comment to cover both kinds of shell.
- **F5** (card 2): `internal/shuttleengine/template.go` and `internal/shuttleengine/spec.go` join the Edit list, with the `ConfigTemplate` and `Spec.Interactive` comments naming `claude_deny_python` and the python deny.
- **F6** (card 3): `consumed` now stops before the earliest line of the trailing run of unpaired stamp lines, and the existing `signals_test.go` table gains a row with two trailing stamps of different hooks that pair as a whole-data call does once their payloads arrive.
- **F7** (card 13): the signal cursor starts at the run record's `PromptOffset` on every path, fresh or attached, never at `run.offset`, which an attach resuming at an asking offset (`attach.go`'s `startOffset`) starts later.
- **F8** (card 5): the exit-status assertion names the ask record beside Stop.

## Disputed

None.

## Deferred

None.
