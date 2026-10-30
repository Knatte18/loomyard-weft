```yaml
lens: code-correctness
outcome: reported
```

Task restated: the code-correctness lens, running each changed function, branch and error path of the diff with concrete inputs.

**Per function, case run, verdict**

- `runNotices`
  - A nil `ReadChild` or a non-probe tick returns early: correct.
  - An unreadable state record logs once and sends nothing: correct.
  - A run in a non-running batten state or with a cancel marker is skipped: correct.
  - A slug that leaves the listing has its `deliveries` entries dropped: correct.
  - A superseded episode's failed delivery entry stays until the slug leaves the listing: negligible leak, no finding.
- `runNotice`
  - A read error logs per episode and returns with no state change: correct.
  - An absent child (`found == false`) returns before keys or memory are touched: correct.
  - First sight with a zero `Modified` dates `Since` from `now`: correct.
  - A state change resets `Since`, and `keepEpisode` drops the old keys: correct.
  - `kept` and `memory` are captured by the deferred closure, so the appended key and the `&memory` mutation from `apiErrorNoticeDue` are saved: correct.
  - A failed marker write repeats the notice each probe tick. Batten's `saveEpisode` did the same, so nothing regressed.
- `noticeCondition`
  - A non-running child, a pause-covered child, a pause read error and a driver read error each give no condition: correct.
  - A dead driver is checked before the agents, and the cap text is built when `RespawnCapped` is set: correct, but see F1 and F2.
- `judgeAgents`
  - With no runs and a quiet window of zero, nothing is sent: correct.
  - With no runs, quiet falls back to the later of `NewestHistory` and `Since`: correct.
  - Precedence is usage-limit, then api-error, then quiet: correct.
  - The `waitLive` hold on quiet and the sleep and pause subtraction through `AwakeSince` are unchanged from the removed code: correct.
- `apiErrorNoticeDue`
  - First sight revives and gives false: correct.
  - The next sight of the same key gives true: correct.
  - The third revived episode with no status write gives true: correct.
  - A status write resets the count: correct.
  - A revive error still counts as the revive: correct.
  - The streak survives a re-exec because it rides in `RunNoticeState`, and `saveNoticeMemory` writes only on change: correct.
- `deliverRunNotice`
  - A failure increments the attempt count, and the third failure marks the notice dropped with a Warn: correct.
  - A dropped notice is never retried, and its key is not recorded as sent: correct.
  - A nil `Notice` seam returns false: correct.
  - A success clears the delivery entry: correct.
- `keepEpisode`
  - A key that is malformed or whose state differs is dropped: correct.
  - Quiet and api-error keys survive when the activity is unknown: correct.
  - A usage-limit key is dropped once no run reads usage-limited: correct.
  - A driver-dead key is kept only while its stamp equals `Since`: correct.
- `saveNoticeMemory`
  - Writes go through `UpdateState` as a read-modify-write. The other writers of `RunState` in `owned.go` and `passes.go` also use it, so none overwrites `Notice` from a stale copy: correct.
- Batten's `noticeCondition`, `keepEpisode` and `conditionText`
  - A running or done child gives no finding: correct.
  - A halted child gives state-changed, then driver-not-revived: unchanged.
  - The `conditionText` default `driver strand is dead` is removed. The only remaining driver-dead finding carries `driverNotRevivedText`, so nothing reads the default: correct.
- Removed `InnerRunMemory` api-revive fields
  - The record is tolerant, so an old record with those keys still reads: correct.
- `DobbyRunNotices`, `readNoticeChild` and `noticeDriverAlive`
  - An absent task worktree or status file reads as not found: correct.
  - An unreadable lock reads as alive: correct.
  - `RespawnCapReached` reads `shedrun.ScratchDir(prime, slug)`, the same directory as batten's `BattenDir`: correct.
- `runShedDuration`
  - A missing key falls back, and the `int`, `int64` and `float64` cases are scaled by the unit: correct.
- `battenRunCommitPaths` and the `NoticeEpisodeFile` and `NoticeEpisodeRel` pair
  - The marker is added only when it exists, and both paths use `RunDir` and `runSegment`: correct.

**Findings**

- **F1 — LOW, consistency** — `internal/dobbyengine/runnotice.go:43` redeclares `respawnCap = 3`. It builds the notice text, while `RespawnCapped` is computed from `battenshed`'s own `respawnCap` (`internal/battenshed/innerrun_memory.go:202`, constant at `innerrun.go:181`). If one changes, the text names a number the capped reading did not use. Evidence: both lines above.
- **F2 — LOW, design** — `RespawnCapReached` (`innerrun_memory.go:199-202`) reads only the saved `Respawns`. The reset on a later status write, `baseline.After(p.respawnStatusTime)`, runs only in the pass at `innerrun.go:428`, and only when the pass is about to respawn. After the child writes its status the stale record still reads capped, so a dead driver found in that window gets the "respawned 3 times without progress" text. The removed code evaluated `p.respawns` after that same-pass reset. The window lasts until the next batten pass, so the effect is a wrong notice text. A memory-read error in `readNoticeChild` (`internal/battencli/dobbynotice.go:171-174`) also returns not found, which hides every condition of the run, including a dead driver, until the record reads again. Fix: treat a memory-read error as "not capped" and keep the child reading, and compare the child's status mtime with `RespawnStatusTime` in `RespawnCapReached`.

No blocking or medium defects found in this lens.

**Tool calls**

1. `grep` of `Runs[` writers in `internal/dobbyengine`, plus `ScratchDir`, `RunDir` and `respawnCap` in `internal/shedrun` and `internal/battenshed`: the manifest lacked other writers of the state record and the scratch path.
2. `grep` of `BattenDir`, `innerrun.go:425-440` and `innerRunMemoryFile`: to confirm the marker scratch directory and where the respawn reset happens.
