```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens on the `ReadAgentActivity` / `noteWaiterlessRun` diff.

**Per-branch list (case run, verdict)**

- Unreadable record (`err != nil`, lines 79-83): the waiterless episode is left open and the `continue` skips every waiterless call. A torn read mid-episode keeps the episode open, and the next readable read resumes it. No duplicate Warn, no stale state. Correct.
- `!found` (absent record, line 85): `waiterlessRuns.Recover` fires only if an episode is open. `EpisodeSet.Recover` returns early on a missing key, so nothing is logged otherwise. It logs one Info, then deletes the key. Correct.
- `!runIsLive`, running with pid 4343 dead (`noteWaiterlessRun`): `Fail` creates the episode on the first skip and logs Warn. Later reads log Debug (policy in `Episode.Fail`, not re-read). Correct.
- `!runIsLive`, running with pid 0 (empty or boundary value): `rs.PID > 0` is false, so it takes `Recover`. That is a no-op with no open episode, so nothing is logged. A pid-0 record that follows a waiterless episode logs one Info. That is acceptable. Correct.
- `!runIsLive`, outcome terminal with a dead or live pid: the guard is false, so `Recover` ends an open episode with one Info and does nothing otherwise. Correct.
- `!runIsLive`, running with a pid that is alive: unreachable, because `runIsLive` is then true.
- Live run (line 93): `Recover` is called on the run's own key before the engine lookup. This is the "live again" Info. If `engineOfRun` fails afterwards, the episode is already ended, and the unknown-engine path has its own episode set. Correct.
- Key isolation: `waiterlessRuns` is separate from `unreadableFiles`, so the per-read `unreadableFiles.Recover` does not end the new episode. Correct.
- Concurrency: `EpisodeSet` holds a mutex, and the package-level var is zero-value usable. Correct.
- Test sequence trace: reads 1-3 give Warn once and Debug twice. The live read gives Info (the helper expects 2 for the first part, which matches the Warn and Info pair as I read it). Turning terminal gives an Info, and the following reads are no-ops. The vanish case gives an Info. The totals of 2 and 3 are consistent. run-b has no pid and run-c is finished, so neither is named in the log.

Wrong results, swallowed errors and inconsistent state on failure paths: none found.

**Findings**

- LOW, design, `internal/shuttleengine/activity.go:109-112`: `waiterlessRuns` is process-global and never pruned for a run directory that is deleted. A directory removed while its episode is open is only recovered by an absent-record read. A removed directory is never listed again, so its key stays in the map and its recovery Info is never logged. The leak is small and bounded by the number of run directories, and the same property holds for `unreadableFiles`. This is an observation, and no change is required.

No other findings.

**Tool calls (1)**
1. Read `activity.go` lines 60-110. My context held only the diff hunks, not the full loop with the unreadable-record branch.
