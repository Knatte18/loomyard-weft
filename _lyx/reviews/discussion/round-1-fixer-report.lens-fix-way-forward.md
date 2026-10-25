```yaml
lens: fix-way-forward
outcome: reported
```

Task: check, through the fix-way-forward lens, that each refusal, next step and criterion this round's fix added can be reached and leads to progress.

Ways forward the fix introduced:
1. The pass refuses `--continue-pass` and names a plain `lyx batten run <slug>` (D4).
2. The spawn hold and batten's `PauseHold` end the pass as a yield, and a later pass picks the run up (D4, D12).
3. A wake leaves every run owed a full pass, carried in the state record (D4).
4. D12 runs the lift pass "as soon as that pass exits".
5. Teardown leaves the task `draft`, and `lyx board set-status <slug>` releases it (D5).
6. D7's notice keys are cleared on a pass that exits 0 or on a claim.
7. The respawn-cap count resets (D4).
8. `lyx reed down` spares the beam's dobby (D1).
9. The darn copy carries over only hub-layer values (D8).

Findings:

- MEDIUM. D4: "batten's wiring of the Go driver's `PauseHold` ends the pass as a yield does, exit 0 with the run `running` at its row", together with "A pass dobby schedules into a running batten run at Run-Shed is a continue pass".
  Summary: a pause can now leave a run `running` at a row other than Run-Shed with no pass in flight, so every probe-tick pass under the pause is a full pass.
  Evidence: `holdPause` runs before any step (`internal/sheddriver/drive.go:251,315,418,509`), not only at Run-Shed. The fix's own bound refuses `--continue-pass` unless the status "reads `running` at Run-Shed". D4's triggers still include "on a probe tick, once per batten's `notice_probe_s`" for a `running` run, and nothing exempts a covered slug. A run held before Land therefore gets a full pass every `notice_probe_s` for the whole pause, each one an origin reconcile plus an Open-Window re-run. Before the fix there was one held process. Either skip passes of a slug a pause covers until the lift, or allow the continue pass at any row.

- MEDIUM. D4: "records each sleep it sees in its state record, and owes every run it drives a full pass" and "the state record of the parent's dobby names the run as one it drives with no full pass owed".
  Summary: no text says when an owed full pass is cleared, so the refusal path is ambiguous whichever way the clear is placed.
  Evidence: D4 says "A pass that exits non-zero is retried on the next probe tick". If the owed mark clears when dobby launches the full pass, a failed full pass is retried as a continue pass and the post-wake reconcile is skipped, the F1 defect again. If nothing clears it, every pass stays full. The first pass after a dobby start isn't described as "owed" at all, so the same gap applies there. State that the owed mark clears only when a full pass of that run exits 0, and that a start owes one too.

- LOW. D7: "one per task and refusal text for a refused start, cleared when a pass claims the task".
  Summary: if the refused task is never claimed, the key is never cleared, so a later refusal with the same text after a re-release posts nothing.
  Evidence: D5 retries a refused start "at once on any change of the task's board entry". After the operator sets the task back to `draft` and later releases it, an unchanged refusal text matches the standing key, and the orch is not told again. Also clear the key when the task's entry changes, or when it leaves an empty status.

- LOW. D4: "the pass refuses `--continue-pass`, naming a plain `lyx batten run <slug>`".
  Summary: the named way forward is a hand command, but in practice the caller is dobby, and D4 doesn't say what dobby does on that refusal.
  Evidence: D4 says "A pass that exits non-zero is retried on the next probe tick; three failures in a row with the same error text post one notice per error text". A stale record, or a race where the row moves off Run-Shed, makes dobby's continue pass fail repeatedly and the orch receives the refusal text. Nothing has dobby fall back to a full pass. Have dobby schedule a full pass on this refusal, or state that the refusal never reaches a dobby-scheduled pass.

- LOW. D4 and the acceptance criterion: "A batten pass under a covering pause or a standing owner exits at once, leaving the run `running` at its row".
  Summary: a hand `lyx batten run <slug>` under a pause or an owner hold now exits 0 with no visible reason, so the operator gets no next step.
  Evidence: before the fix, `holdSpawnWhileCovered` wrote `pauseHoldReason` or `ownerHoldReason` and kept waiting. The new yield text doesn't say whether the pass reports that reason, such as "take it over with lyx adopt", on its exit. Have the yield exit print the hold reason.

No further tool calls were made; everything above comes from the context and the code reads the fixer had already done (`internal/sheddriver/drive.go` `holdPause` callers, `innerrun_pause.go`).
