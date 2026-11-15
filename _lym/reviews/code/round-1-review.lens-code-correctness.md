```yaml
lens: code-correctness
outcome: reported
```

Task: lens code-correctness, covering `clearDoneRuns`, `clearDoneRun`, `ClearDoneRun`, `restoreDoneRunRecords`, and the `d.landed` plumbing.

**Cases run**
- `clearDoneRuns`, nil `Deps.ClearDoneRun`: returns at once. Correct.
- `clearDoneRuns`, `ReadState` fails: warns and returns, so nothing is cleared. Correct.
- `clearDoneRuns`, a run in `d.landed` with `CountedLandings` unset (a tidy agent is in flight, or the landing is new this tick): skipped and retried at the next tick. Correct.
- `clearDoneRuns`, nil `Deps.Landings`: `d.landed` stays nil, so the run is cleared with no counting gate. Correct, since nothing counts landings there.
- `clearDoneRuns`, `Landings()` errors: `countLandings` returns the error and the tick returns before `clearDoneRuns` runs. `d.landed` is never read stale. Correct.
- `clearDoneRuns`, a run with an undecided landing notice: `rememberRuns` keeps the memory entry while a notice is owed, so the clear skips it. Correct.
- `clearDoneRuns`, a clear that succeeds: logs the line and deletes `CountedLandings[slug]`. The run no longer appears in `DobbyLandings`, so it is not counted twice. Correct.
- `clearDoneRuns`, `UpdateState` fails after a successful clear: only a warning is logged, and the stale `CountedLandings[slug]` stays. Defect, see F2.
- `clearDoneRuns`, the seam returns `(false, nil)`: no log line and the count is kept, so it is retried. Correct.
- `clearDoneRuns`, the seam returns an error: `Fail` is called with the slug as key, and `Recover` runs on the next success. Correct.
- `clearDoneRun` with `admitsAnyRun` and `recipe == ""`: the commit message reads "batten: remove the done run of <slug>". Correct.
- `clearDoneRun` with `admitsAnyRun`, run lock held or status file missing: returns `false, nil`. Correct.
- `clearDoneRun` with `admitsAnyRun`, status not done: returns `false, nil`. Correct.
- `clearDoneRun`, `removeRunRecords` fails with a commit error: `restoreDoneRunRecords` writes the status back, writes the child recipe only when it is non-empty, and writes the filing mark when one stood. The status lock is still held, and the errors are joined. Correct.
- `clearDoneRun`, the scratch-empty error: only warns, and `cleared` is true. Correct.
- `ClearDoneRun` from a non-prime dobby: `ResolveBattenPrime(prime)` returns the prime again, and the seam is bound to `prime` in `hubDobbyDeps`. Correct. Only one `Deps{}` construction site exists outside tests, so there is no nil seam to fill elsewhere.
- Run listing, owner: `dobbyParentRuns` does not filter by owner machine. See F1.

**Findings**

F1 (MEDIUM, design) `internal/dobbyengine/donerun.go:34-44`, with `internal/battencli/dobbyreads.go:75-111` and `:275-289`.
- The clear is gated on `d.landed`, which `DobbyLandings` limits to this machine's names.
- The run list in `dobbyParentRuns` carries no owner-machine filter.
- So a done run whose landing was recorded by another machine, with its records synced here, is not in `d.landed` and passes the gate.
- This machine's dobby then deletes and pushes the removal of that run's records before the other machine's dobby has counted its landing or decided its notice.
- The other machine's tidy count and landing notice can be lost.
- This conflicts with PATTERN-owner-machine, which says to act only on the entry's owner machine.
- Fix: skip a done run that `DobbyLandings` would not credit to this machine. Alternatively, gate on the run's owner being this machine, for example through `current.Runs[slug].OwnedElsewhere`, which `handover.go:107` already reads. I did not verify whether `RunView` already carries an owned-elsewhere flag.

F2 (LOW, design) `internal/dobbyengine/donerun.go:58-68`.
- If the `UpdateState` that deletes the cleared slugs from `CountedLandings` fails, only a warning is logged.
- The stale `true` entry stays, and nothing removes it later.
- A later run under the same slug whose landing is read would then be treated as already counted, so its board-tidy count is lost silently.
- Fix: retry the deletion on a later tick. For example, keep the pending slugs on the daemon and drop `CountedLandings` entries for slugs absent from the current `d.landed`. The second approach also covers a clear done by another path, such as `lym batten cleanup`.

**Tool calls**
1. `grep` of `DobbyLandings` and `removeRunRecords`: their bodies were not in my context.
2. `grep` of `runStateDone` and `EpisodeSet.Fail/Recover`: to check the episode keying and the state constant's other readers.
3. `grep` of `dobbyParentRuns`: to check whether the run listing filters by owner machine.
