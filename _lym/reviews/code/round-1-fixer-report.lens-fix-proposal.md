```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's carried-out proposals (F1, F2, F3, F6–F13) by building counter-cases.

Proposals checked: F1, F2, F3, F7, F8–F13. F6 and F12 are text-only and I found nothing against them.

**1. MEDIUM — F7: a transient End failure now permanently skips the end.**
- Changed text: `internal/orchengine/owner.go` `ownerTick`, which now calls `Stop` even when `End` failed, and returns `errors.Join(endErr, stopErr)`.
- Counter-case: `endSubjectProcesses` (now `pairteardown.EndSubjectProcesses`) fails on a transient `boardengine.ReadOwners` or `runproc.Owners` read error, or on a `SurvivorsError`.
- Result: `Stop` removes the orch strand in the same tick.
- Next tick: `tick()` sees `StrandAlive == false` and returns `WatcherExit = "strand gone"` before `ownerTick` runs again, as `watcher.go:272-279` shows.
- The end is never retried. The orch provider record, any survivors and the runs stay up on a machine that no longer owns the beam. Before the diff, a failed end kept the strand up and was retried.
- The review's fix says "so the next tick retries the end". That is false once `Stop` has succeeded, and the fixer's own doc text admits it.
- A narrower alternative: stop the strand despite a failed end only for a persistent failure such as a `SurvivorsError`, and retry a read failure.

**2. LOW — F3: leaving a live dobby up reopens the race that "dobby last" was meant to close.**
- Changed text: `pairteardown.EndSubjectProcesses(prime, owner, false)` in `cli.go`, and the `endDobby` skip in `runproc/end.go`.
- Counter-case: the watcher ends the runs and the orch, but the dobby keeps running until its own `probeOwner` call (`dobbyengine/loop.go:550`).
- The dobby's passes between the watcher's end and its next probe tick can restart a run that was just ended. `handOver` then calls `pauseRunsForOwner`, which pauses only runs it still sees as drivable.
- I did not verify whether `loop.go` reads the owner before it starts each pass, so this may not occur. The test only pins "dobby not signalled", not that nothing restarts.

**3. LOW — F1: no end path for the orch once the beam is torn down.**
- Changed text: `endOwnedProcesses` now spares `RoleOrch`.
- Counter-case: a `lym reed down` or `fabric remove` of the beam (`teardown.go:335`) now leaves the orch provider process and its record under the removed slug.
- The orch is not ended there, and the only remaining end is the watcher's owner-elsewhere path, which a removed beam never reaches. Whether `BeamDown` removing the strand kills the provider process is unproven in the diff.
- No test pins that the spare happens, so a regression would pass silently.

**No findings** on F2's spare list, F8, F9, F10, F11 or F13. The `end_test.go` rows I reasoned through produce the stated signals, and `subjectRuns` matches the old selection logic.

**Calls made**
1. `grep` and `sed` on `internal/dobbyengine/loop.go` and `handover.go`: my context lacked the dobby's owner-probe ordering and `handOver` body, which I needed for finding 2.
