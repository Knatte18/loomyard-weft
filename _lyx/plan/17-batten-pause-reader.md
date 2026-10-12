# Card 17 — batten-pause-reader

**Create:**
- `plan:internal/battenshed#innerRunProducer.coverage` -> `func (p *innerRunProducer) coverage(w *childWait) pauserecord.Coverage`

**Edit:**
- `internal/battenshed#InnerRunDeps`
- `internal/battenshed#NewInnerRun`
- `internal/battenshed#childWait`
- `internal/battenshed#innerRunProducer.Call`
- `internal/battenshed#innerRunProducer.check`
- `internal/battenshed#innerRunProducer.noticeCondition`
- `internal/battenshed#innerRunProducer.judgeAgents`
- `internal/battenshed#innerRunProducer.lockStep`
- `internal/battencli#battenCLI.wire`
- `internal/testkit/envkit#nilLegal`
- `internal/battenshed#noticeHarness`
- `internal/battenshed#noticeHarness.producer`
- `internal/battenshed#lockHarness`
- `internal/battenshed#TestNotice_QuietOnlyForRunningChildWithLiveDriver`
- `internal/battenshed#TestNotice_APIErrorStall`
- `internal/battenshed#TestNotice_DeadDriverWhileRunning`
- `internal/battenshed#TestNotice_StateChangeNotifiesOncePerEpisode`
- `internal/battenshed#TestInnerRun_ParentLockReleasesAStalledLoopsRecord`
- `internal/battenshed/doc.go`

**Intent:**
Make batten's InnerRun read the pause record (D5): no quiet, api-error or driver-dead notice for a covered run, idle clocks that subtract every paused interval, no idle release of a covered holder's parent lock, and no spawn, resume or revive of a covered child, while a state-changed notice still goes for a run that halts.

- `InnerRunDeps` gains `Pause func() (pauserecord.Coverage, error)`; `NewInnerRun` resolves nil to never covered.
  `battenCLI.wire` fills it with `pauserecord.ReadCoverage` at the prime's `hubgeom.PauseGeometry` record path for the slug, and `envkit.nilLegal` lists `InnerRun.Pause`, since the recipe kit's full env leaves it nil.
- `coverage(w)` reads the seam once per check into `w.pause`; an error reads uncovered, warned once per distinct message, as the pause flag's read is.
- `check`, under a covered reading, still runs `checkChild` (a state change still ends the wait) and the notice step, reports `held by the pause of <scope> since <paused_at>` as the wait reason, and skips the arm's step, so no resume spawn, stale-base resume, Publish retry or driver revive runs while covered; batten's own `pause_requested` is still honoured.
- `Call`'s first spawn, for a fresh or unconfirmed child, waits while the slug is covered, reporting the same reason and re-reading coverage each poll interval, so a covered child is never started and a `lyx loom start` refusal (card 15) never reaches the row as a hard error.
- `noticeCondition` returns no finding for a running child while covered; a halted child's state-changed notice is decided as today.
- `judgeAgents` passes `w.pause.Intervals` to `awakeSince` (card 16), so after the resume the quiet and api-error clocks exclude the paused interval, and a run idle for its window after the resume is noticed as before.
- `lockStep`, while covered, resets `w.lock` and returns: a covered holder keeps its record, and the idle watch restarts after the resume so paused time never counts as waiting.

Bound: a pause only suppresses notices and spawns and delays the lock's idle release; it changes no state and ends no wait except through a state change or batten's own pause.

Tests (harness gains a `pause` coverage field fed through `InnerRunDeps.Pause`):
- `TestNotice_QuietOnlyForRunningChildWithLiveDriver` gains `ACoveredChildSendsNoQuietNoticeHoweverLong` (two hours of idle agents under a live entry, no notice) and `AfterTheResumeThePausedIntervalIsNotIdleTime` (an hour covered, then the entry resumed: no notice until 45 unpaused minutes pass, then one reporting 45m).
- `TestNotice_APIErrorStall` and `TestNotice_DeadDriverWhileRunning` each gain a covered row that sends nothing.
- `TestNotice_StateChangeNotifiesOncePerEpisode` gains a covered row whose child halts: the state-changed notice still goes once.
- `TestInnerRun_ParentLockReleasesAStalledLoopsRecord` gains a row whose agents wait past the bound under a live entry: the record is kept, and after the resume it is released only once the bound passes again.
`doc.go` names the pause reader beside the notices and the lock step.

**ImpactSummary:** battenshed's InnerRun gains a told pause seam read once per check; batten's wire fills it and the envkit nil list names it, with no change while no entry covers the slug.
