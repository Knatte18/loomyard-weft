```yaml
round: 1
fixed: [F1, F2, F3]
disputed: []
deferred: []
```

## Changes

- **F2** (commit `849c7d77`, fixed with my own change):
  - The reviewer proposed a test that calls `checklistPoll` with a `Waiting` hook returning distinct Text and Steady.
  - `checklistPoll` built its hook internally through `reviewWaitingFor(location)`, so no test could inject one.
  - `checklistPoll` now takes the hook as a second parameter, and its single caller, the `checklist` verb at `internal/loomcli/checklist.go:127`, passes `reviewWaitingFor(location)`, so production behaviour is unchanged.
  - `TestChecklistPoll_WaitLineKeepsTheNoteText` in `internal/loomcli/checklist_test.go` drives it over a temp-dir location with a seeded status file and asserts the frame's last line is `waiting Build: verify running 6m` for a running run and `waiting on nothing` for a done one.
- **F1** (commit `ddae134b`, fixed as proposed, partly):
  - `TestStatusCmd_WaitingNote_RunningOnly` now returns a `WaitNote` whose `Steady` is `"steady form of " + note`, so putting `Steady` into the envelope's `waiting` key fails the first subtest.
  - The human-view half of the proposed assertion is not made: `execEnvelope` gives a non-terminal writer, so `statusCmd` never prints the human view there.
  - The envelope and the human view both take `waiting.Text` in `status.go`.
- **F3** (commit `ddae134b`, fixed as proposed):
  - `TestStatusWatchLine_AppendsWaitingNote` now polls a spec over a missing status file and asserts line and key both equal `UnavailableLine("loom")`.
  - F1 and F3 edit the same file, so they share one commit.

Gate: `lym gate test ./internal/shedverbs ./internal/loomcli` answered tier0 1617 passed, 0 failed, 48 skipped, exit 0.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-affected-set: no findings.
- fix-proposal: one note below LOW — the Done row of the F2 test checks the state gate rather than Text versus Steady, so it may repeat a `renderChecklist` case.
  Disposition: fixed as proposed, no change.
  The `renderChecklist` cases pass `waitNote` in directly, so none exercises the poll's running-state gate at `checklist.go:167`, and this row is the one that does.

## Disputed

None.

## Deferred

None.
