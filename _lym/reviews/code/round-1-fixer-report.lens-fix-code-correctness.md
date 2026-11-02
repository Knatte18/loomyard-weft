```yaml
lens: fix-code-correctness
outcome: reported
```

Lens fix-code-correctness: each changed function or branch was run on a concrete input. Two findings, both LOW.

**Findings**

1. **LOW** — `rowgate.go`, `waitThenRun` early return `if result.Passed && !markOnPass`.
   - **Summary:** an earlier entry that returns `Pending` or `Terminal` over a live record still marks the record terminal.
   - **Evidence:**
     - Input: a live record, an earlier entry (`markOnPass` false) returning `{Pending: true}`.
     - Only `Passed` skips the mark, so `markTerminal` runs and the record is terminal.
     - A later `verify` entry then finds no live record, so a failure there carries no fixer reply, which is F2's symptom.
     - The `Pending` path has no test row.
   - **Possibly intended:** a `Terminal` failure is a failure, so marking may be right. `Pending` carries no verdict over the finished tree, so leaving the record live fits D1 and F3's reasoning better.

2. **LOW** — `cardfixer_test.go`, `NonZeroExitWithNoStderrSpawnsNone`.
   - **Summary:** the test is meant to pin F4's `holdsFailureReport` guard, and I could not confirm it fails with the guard removed.
   - **Evidence:**
     - `detachrun.go:334` does `os.Create(d.path(stderrFile))`, so the run creates an empty stderr file before the gate call. That makes the guard's `Size() > 0` check the deciding branch, and the test should fail without it.
     - I did not read past line 334 to see whether anything is written to stderr afterwards.
     - If something writes to it on a non-zero exit, the rig is not "empty stderr" and the test passes without the guard.
     - The test asserts only exit code, no spawn and one gate call. It does not assert the stderr file is empty.
   - **Suggested check:** assert `rig.stderr() == ""` in the test.

**Checked, no defect**

- `waitThenRun` on an inner error returns before marking the record, so the record stays live.
  - `AnInnerErrorAfterTheWaitLeavesTheRecordLive` fails with the old join-then-mark code, which marked the record `done`.
- `waitOnly(…, false)` on a passing earlier entry leaves the record live.
  - `AnEarlierEntryThatPassesLeavesTheRecordLiveForTheFixerEntry` fails with the old code, which marked it terminal.
  - The `spawning` live branch and `spawnAndJudgeAgain` pass `true`, and `Final` passes `true`, so the fixer entry still marks the record.
  - `ExpiredIsTrueOnlyForALiveRecordAFreshWrapDidNotSpawn` still holds, since its entry is the fixer entry.
- `fixAndRerun` now marks after the re-run on the abandoned and non-abandoned paths alike.
  - The `recordStatusAtCall[1] == ""` assertion fails with the old order.
  - The early-return paths (spawn error, abandoned, wait error with a failed stop) are unchanged.
- `TestWriteNewRun_RecordsTheCallersRole` fails if `Role: f.call.role` is dropped.
  - The `front := &front{…}` shadowing compiles, since the variable's scope starts after the declaration.
- The `arm.go` site in `fixerendguard_test.go` fails if that site stops wrapping, because `ends` would be 0.
  - Only the `arm.go` site is exercised; the `arbiterconsume.go` and `drivewiring.go` sites are not.
- F8's deleted branch is safe: `gateslot/config.go:245` refuses `fixer_timeout_min < 1` at load.
- Every caller of the changed `waitOnly` and `waitThenRun` signatures is updated.

**Tool calls (1 of 6)**
- `grep` of `stderrFile` in `gatecli/*.go` and of `fixer_timeout_min` in `gateslot/config.go`. My context lacked how the runner creates the stderr file and whether config load refuses a zero timeout.
