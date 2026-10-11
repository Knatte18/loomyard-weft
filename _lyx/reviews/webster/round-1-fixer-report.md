# Webster round 1 — fixer report

Both findings held against the code and the fasit, and both are fixed and committed.

## Changes

- **F1** (LOW, design) — fixed, commit `0a136194d`.
  Premise holds: card 7 and decision-record R8 say a slash command's retry always types afresh, but `retryLanded` read any turn start past the recorded offset as landed.
  The Fix line's alternatives were open, so the change is mine. It takes the first one's direction and still records the pending send, as the decision record says both failures do:
  `sendContext` gains `startsNoTurn`, `sendSequence` sets it, and `retryLanded` never reads such a send as landed, though it still clears the record.
  Regression row: `checkPastedSessionCommand` gains "a pending record of the command with a turn started since pastes it afresh". It covers `/clear`, `/compact`, `/reload-plugins` and `/color`, and all four rows failed before the fix.
  `doc.go` already stated the intended behavior, so it is unchanged.
- **F2** (LOW, design) — fixed as proposed, commit `42256006a`.
  Premise holds: tmux stops a command list at its first failing command, so `paste-buffer -d` never runs after a failure.
  `pasteSubmitLocked` now runs a best-effort `delete-buffer -b <name>` on a failed invocation and drops its error.
  Regression row: `TestPasteSubmit` gains "a failed paste deletes its buffer". It also asserts that a successful paste issues no separate `delete-buffer`.
  `internal/reedengine/doc.go` notes the cleanup.

Verification: `lyx gate test ./internal/shuttleengine` and `lyx gate test ./internal/reedengine` are green, and `lyx loom lint-comments` passes.

## Disputed

None.

## Deferred

None.
