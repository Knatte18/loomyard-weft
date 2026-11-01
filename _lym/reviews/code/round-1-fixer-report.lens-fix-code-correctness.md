```yaml
lens: fix-code-correctness
outcome: reported
```

Findings: 2 LOW. The other changes ran clean on a concrete input each.

**LOW — F5 test passes with the defect in place.**
- Changed text: `TestDetachRun_ClaimPublishesCompleteRecord` in `detachrun_test.go`.
- The test claims once, checks the record decodes, checks a second claim returns `os.ErrExist`, and checks only one directory entry remains.
- The old exclusive-create-then-write `claim` passes all of those too, since the gap it left is only visible to a concurrent reader.
- The test therefore does not pin the temp-file-and-link behaviour. This matters only if a regression test for F5 is wanted. The review's F5 `fix` key asked for none.

**LOW — F5 changes the process file's mode from 0644 to 0600.**
- Changed text: `os.CreateTemp(d.dir, processFile+".*.tmp")` followed by `os.Link` in `claim`.
- `CreateTemp` creates the file as 0600, and the link shares its inode, so `process.json` is now 0600. It was 0644 before.
- Every reader (fronts, runner, tests) runs as the same user, so nothing breaks today. A different-user reader would fail to decode the file.
- A `Chmod(0o644)` before the link would keep the old mode.

Traced with no defect found:
- **F2:**
  - A failing stdout writer returns an error from `io.Copy`, `errors.Join` is non-nil, and `delivered` is not written.
  - A missing file is no error.
  - `TestFrontReplay_DeliveredMark/StdoutWriterFails` fails with the old unconditional write.
- **F3:**
  - An unreadable lease gives `leaseLapsed` false, an absent lease gives true, and a passed lease gives true.
  - `LeaseUnreadable` fails with the old negated `leaseHolds`.
  - `runDetached` still uses the strict `leaseHolds` for the startup refusal.
- **F4:**
  - A lease that passes after the call returned leaves `cancelledDuringCall` false, so the exit is the real verdict.
  - A lease that passes during the call stores `passed` before it cancels, so `cancelledDuringCall` is true and the run is abandoned.
  - The signal case is checked first, as before.
  - No test pins this race, which is acceptable because the review did not ask for one.
- **F7:**
  - The compiler and the tier-0 and tier-1 gate runs show every `answerRunGoing`, `replayedBusyLine` and `abandonedLine` caller updated.
  - `cardverify.go:151` sets `call.words` to the card id, so a real card-verify request always has `Args`.
  - The empty-`Args` check runs in `readRequest` before `claim`, and `CardVerifyWithoutCard` fails without it.
  - `endStrayLine` still returns early on a `readRequest` error.
- **F6:**
  - `pruneOldRuns` runs under the key lock, after a successful launch.
  - It skips the new run, skips non-directories (`lock`, `current.json`), and skips a live runner.
  - `TestFrontPruneOldRuns` fails without the age check or without the live-runner check.
  - A live run's directory mtime stays fresh because `writePhase` renames files into it.
- **F8:**
  - The dispute holds.
  - The cap-0 row at `test_integration_test.go:959` sets `strandEnvEntry` and `helperCapEnv=0` and asserts exit 7 plus the gate's own pid as `Enclosing`.
  - Without the `answer.Cap <= 0` branch the call would be capped and answer 75.
  - The inherited-slot row at `:1201` runs under a strand and asserts the held slot and the watchdog log from an in-process run.
- **F9:** doc text only.

Further calls made:
1. `grep strandEnvEntry` and `verbCardVerify`: my context lacked the strand constant's value and where card-verify requests get their `Args`.
