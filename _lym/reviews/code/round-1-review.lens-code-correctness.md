```yaml
lens: code-correctness
outcome: reported
```

Task: run the code-correctness lens over the diff, with a case per function or branch.

## Cases run

| Function or branch | Case run | Result |
|---|---|---|
| `RunCapture`, exit 0 with no file | `true` command, empty log | Correct. The reason is `exit 0 but wrote no file…; it ran in <dir> and printed nothing`, `WroteNoFile` is set, and `OnEnd` gets the record under the lock. |
| `RunCapture`, non-zero exit, timeout and signal | `exit 3`, a timeout, ctx cancel (`exit -1`) | Correct. Each carries `commandReport`. |
| `RunCapture`, `runStep` or `holdsFile` error | Missing `dir` before spawn | Correct for the record: `failCapture` writes the reason, so the Drift-Await halt states the cause. `OnEnd` is not called, so the child writes no error envelope and only sets exit 1. This matches the `OnEnd` doc ("once the command ran"). |
| `RunCapture`, `WriteCapture` fails | | Correct. It returns `rec, err` and skips `OnEnd`. |
| `OnEnd` in `runCaptureChild` | Failed, timed-out and passed records | Correct. A non-passed record writes an error envelope and sets the exit code. Passed does nothing. A `Check` refusal calls `fail` and then `SetExit(1)`, as before. |
| `FailedCaptureError` | Plain, `WroteNoFile` with an empty skip list, `WroteNoFile` under skips | Correct. `awaitWaysForward(true, …)` is sound because `Check` guarantees HEAD equals the branch point. |
| `startMainCapture` | Record is failed with `Pid == pid` | Correct. It returns `FailedCaptureError`. |
| `startMainCapture` | Record still `running` | Returns success with exit 0. See finding 1. |
| `startMainCapture` | `ReadCapture` errors after the child started | The error is returned although a capture is running. See finding 3. |
| `commandReport`, `boundOneLine` | Empty log, blank lines, over-long last line, unreadable log | Correct. The `max(0, …)` slice bound, the cut ellipsis and the unreadable-log clause all hold. `readChildReport` behaves as before with `boundOneLine`. |
| `commandReport` | A log of hundreds of MB | See finding 4. |
| `RunAcceptedCapture`, `RunCompare` | Failure reasons | Correct. The order is `…; it ran in … ; log <path>`. |
| `RebaseDirsOntoRoot`, same dir | `worktreeRoot == anchorDir` | Correct. The bytes come back unchanged. |
| `RebaseDirsOntoRoot`, empty or comment-only document | | Correct. It returns `existing`, because the document kind is not a mapping. |
| `RebaseDirsOntoRoot`, `.` with anchor `dotnet` | | Correct. It rewrites to `dotnet`. |
| `RebaseDirsOntoRoot`, dir under the root alone | | Correct. Kept, and a second run changes nothing. |
| `RebaseDirsOntoRoot`, dir naming neither | | Correct. Flagged and the file is not rewritten. |
| `RebaseDirsOntoRoot`, absolute or `""` | | Correct. Kept. |
| `RebaseDirsOntoRoot`, unparseable YAML | | Correct. Returns an `ErrInvalid` error. |
| `RebaseDirsOntoRoot`, `dir: null` or `~` | | Wrong. See finding 5. |
| `RebaseDirsOntoRoot`, dir existing under both anchor and root | | The anchor wins by design. See finding 6. |
| `applyReinterpretations`, marker present | | Correct. Skipped. |
| `applyReinterpretations`, base file absent | | Correct. Only the marker is written. |
| `applyReinterpretations`, rewrite returns an error | | Correct. A `FileError` aborts the board write and the snapshot restores it. |
| `applyReinterpretations`, flagged but not rewritten | | The marker is still written. See finding 2. |
| `reconcileHubWide` | Skipped pull, commit failure, success | Correct. Notices are sent only after the commit lands and are dropped on error or skip. The `slices.Contains` dedupe holds. `snapshotConfig` covers the markers, so a failed write removes a marker it wrote. |
| `notifyReinterpretations` | No lines, no notifier, mail error, mail not posted | Correct. Silent on no lines, and a failure is logged only. |
| `InstallReconcileNotifier`, `hubLocation` | Outside a hub, another hub's board | Correct. Nothing is installed outside a hub, and another hub's notice is reported as not posted. The extraction from `InstallPushNotifier` keeps its behaviour. |

## Findings

1. **MEDIUM, design.** `lym drift capture` can exit 0 for a capture that will fail with no baseline file.
   - Location: `internal/driftcli/cli.go:349-356`.
   - The verb reads the record right after `StartCapture` sees the child's pid. The child writes that record as `running` before the command starts, so the check only catches a capture that has already finished.
   - A command that takes a slot wait plus a spawn, such as `dotnet run` exiting 0 with no file, normally still reads `running` at that point.
   - The Goal's first Part 1 item says the verb "fails with a non-zero exit and a stated cause when it produced no baseline file".
   - The shipped behaviour only meets that when the failure happens before the poll. The child does exit non-zero, and the record and halt carry the cause.
   - Evidence: `cli.go:353`, `spawn.go:115-143` (returns on the first record carrying the pid), `capture.go:84-87` (the running record is written before the command).

2. **LOW, design.** A flagged value is told once, and a notice that was not delivered is lost.
   - Location: `internal/hubreconcile/reinterpret.go:632-639` with `walk.go:262`.
   - The marker is written even when the file was only flagged and not fixed. If no notifier is installed, no orch is recorded, or the mail fails, the line stays only in the Warn log.
   - No later walk re-flags it. The Goal says "or flags with a way forward" and tells the orch, so an undelivered flag has no durable trace.

3. **LOW, design.** A `ReadCapture` error after a successful start reports a failure for a capture that is running.
   - Location: `internal/driftcli/cli.go:349-352`.
   - The detached child keeps running while the verb exits with an error and no pid, so an operator who retries gets the held-lock refusal.

4. **LOW, design.** `commandReport` reads the whole log into memory and cuts the end of the quoted tail.
   - Location: `internal/driftengine/steprun.go:164-178`.
   - A long test log is read whole just to take three lines.
   - `boundOneLine` cuts from the end, so the final line is the one clipped when the joined tail exceeds 400 runes. That final line is usually the error. The test `an over-long last line is cut…` pins this.

5. **LOW, design.** A null `dir` is treated as the literal directory "null".
   - Location: `internal/driftengine/dirbase.go:356-357` (`dir.Value == ""`, `scalarAt`).
   - `dir: null` and `dir: ~` have `Value` "null" and "~" with tag `!!null`. They are neither rewritten nor kept; they are flagged as "names no directory". The result is a false notice to the orch.
   - The `!!null` tag is not checked.

6. **LOW, design.** The anchor-wins rule can undo a hand fix.
   - Location: `internal/driftengine/dirbase.go:363-370`.
   - A hub that already fixed its dir by hand for the root base, to a value that also exists under the anchor (for example `src`), is rewritten to `<anchor>/src`. That changes the meaning the operator had set.
   - The change description names this trade-off for `.` but not for this case. It is reachable only for hubs that ran `c896481c` before this binary.

## Tool calls

None. My context already held every file the procedure needed.
