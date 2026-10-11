# Plan-Review round 1 — fixer report

Every finding's premise was checked against the code and held; all five are fixed.
`lyx loom validate-plan` passes after the fixes (`ok: true`, informational `create-new-unit` notes only).

## Changes

- **F1 (MEDIUM, consistency), fixed as proposed.**
  The premise holds: `internal/shuttlecli/smoke_*_test.go` are in package `shuttlecli`, which cannot name an unexported `claudeengine` constant, and `go vet -tags llm ./...` type-checks the file.
  Card 9 now creates the exported `PasteCollapseMinChars` and says it is exported so card 10 sizes its texts from it; card 10 lists `plan:internal/shuttleengine/claudeengine#PasteCollapseMinChars` under `**Uses:**` and sizes the texts from `claudeengine.PasteCollapseMinChars`, never a literal.
- **F2 (MEDIUM, consistency), fixed as proposed.**
  The premise holds: the pre-write hold quotes the draft (`internal/shuttleengine/readiness.go:84`), but card 6's foreign-box branch had a fixed text.
  Card 6's foreign-box error now quotes the box it read (`the input box holds text that is not this send's: %q`), naming cards 13 and 14 as the consumers, and its foreign-draft test row asserts the error text carries the draft.
- **F3 (LOW, scope), fixed as proposed.**
  The premise holds: `typeColor` returns nil on an empty `ColorSequence` (`internal/shuttleengine/run.go:1026`), and `Claude.ColorSequence` returns nil outside its palette.
  Card 8's `sendSequence` now returns nil before the guard on an empty choreography, and `TestRunner_TypeColor_PlaysTheSequenceAndWaitsItOut` gains a row asserting no `PasteSubmit` and no `OperatorActive` call for a color with no command.
- **F4 (LOW, design), fixed as proposed.**
  The premise holds: `Claude.SessionIdle` is `IdleSession`, which requires an on-screen box with a blank interior (`claudeengine/session.go:57`).
  Card 3's `Bound:` now states that on a query error the rename is still held by the namer's idle reading, so the fail-open can type only past an operator active in the pane who has not typed into the box (R2's window).
- **F5 (LOW, scope), fixed as proposed.**
  The premise holds: the only planned test took `childFound` and the child status as arguments, leaving `readChildStatus`'s path and run ID and the emitted key unpinned.
  Card 16 now adds a step to `TestBattenIntegration_Rows` in `internal/battencli/lifecycle_integration_test.go` (listed under `**Edit:**`): over a real pair, a done child status at `shedrun.StatusFile(task, shedrun.SelfRunID)` and a failed batten make the `status` envelope carry `child_done_batten_halted`, and removing the child status file omits it.
  The pure table shrinks to the state matrix; the step is not a top-level test, so the budget row is unchanged in scope.

## Disputed

None.

## Deferred

None.
