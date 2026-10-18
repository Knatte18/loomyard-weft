# Card 9 — orch-creel-seam

**Create:**
- `internal/orchengine#Creel`
- `internal/orchengine#CreelMessage`
- `internal/orchengine#CreelStep`
- `internal/orchengine#CreelState`
- `internal/orchengine#CreelQueued`
- `internal/orchengine#CreelSending`
- `internal/orchengine#CreelInjected`
- `internal/orchengine#CreelUnconfirmed`
- `internal/orchengine#CreelExpired`
- `internal/orchengine#CreelWithdrawnState`
- `internal/orchengine#CreelOutcome`
- `internal/orchengine#CreelSent`
- `internal/orchengine#CreelWithdrawn`
- `internal/orchengine#CleanupRetiredNotices`
- `internal/orchengine#fakeCreel`
- `internal/orchengine#fakeCreel.Post`
- `internal/orchengine#fakeCreel.ReadMailbox`
- `internal/orchengine#fakeCreel.Inject`
- `internal/orchengine#fakeCreel.InjectOutcome`
- `internal/orchengine#fakeCreel.Sent`
- `internal/orchengine#fakeCreel.Hold`
- `internal/orchengine#fakeCreel.Release`
- `internal/orchengine#fakeCreel.Withdraw`
- `internal/orchengine#fakeSession.StrandName`
- `internal/orchcli#orchWatchProgram`
- `internal/orchcli#creelClient`
- `internal/orchcli#fakeCreelClient`
- `internal/orchcli#fakeCreelClient.Log`
- `internal/orchcli#creelSeam`
- `internal/orchcli#creelSeam.Post`
- `internal/orchcli#creelSeam.ReadMailbox`
- `internal/orchcli#creelSeam.Inject`
- `internal/orchcli#creelSeam.InjectOutcome`
- `internal/orchcli#creelSeam.Sent`
- `internal/orchcli#creelSeam.Hold`
- `internal/orchcli#creelSeam.Release`
- `internal/orchcli#creelSeam.Withdraw`
- `internal/orchcli#injectOutcomeOf`
- `internal/orchcli#sentOf`
- `internal/orchcli#runnerSession.StrandName`
- `internal/orchcli#TestCreelSeam_FoldsTheJournal`
- `internal/orchcli#fakeCreel`
- `internal/orchcli#fakeCreel.Post`
- `internal/orchcli#fakeCreel.ReadMailbox`
- `internal/orchcli#fakeCreel.Inject`
- `internal/orchcli#fakeCreel.InjectOutcome`
- `internal/orchcli#fakeCreel.Sent`
- `internal/orchcli#fakeCreel.Hold`
- `internal/orchcli#fakeCreel.Release`
- `internal/orchcli#fakeCreel.Withdraw`

**Edit:**
- `internal/orchengine#Session`
- `internal/orchengine#Watcher`
- `internal/orchengine#NewWatcher` -> `func NewWatcher(session Session, creel Creel, cfg Config, paths Paths, stencilsDir string, skills []string, index func() (string, error), clock Clock) *Watcher`
- `internal/orchengine#Watcher.tick`
- `internal/orchengine#Watcher.queueResumeLine`
- `internal/orchengine#Watcher.Run`
- `internal/orchengine#Paths`
- `internal/orchengine#fakeSession`
- `internal/orchengine#watchEnv`
- `internal/orchengine#newWatchEnv`
- `internal/orchengine#watchEnv.newWatcher`
- `internal/orchengine#TestWatcher_HubPauseInterruptsAtGraceEndHoldsAndQueuesTheResumeLine`
- `internal/orchengine#TestRun_ReturnsWhenStrandGone`
- `internal/orchengine#TestRun_StopsAtErrorCap`
- `internal/orchengine#TestRun_SuccessfulTickResetsErrorCount`
- `internal/orchengine#TestRun_ChecksTheBinaryOnlyAtARestartSafeTick`
- `internal/orchcli#orchCLI`
- `internal/orchcli#Command`
- `internal/orchcli#orchPaths`
- `internal/orchcli#orchCLI.watchCmd`
- `internal/orchcli#orchCLI.cleanupCmd`
- `internal/orchcli#TestCleanup_Envelope`
- `internal/orchcli#newTestCLI`
- `internal/orchengine/doc.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Delete:**
- `internal/orchengine/notice.go`
- `internal/orchengine/notice_test.go`
- `internal/orchengine#Watcher.deliverNotice`
- `internal/orchengine#Watcher.dropQueueWithoutStrand`
- `internal/orchengine#watchEnv.queue`
- `internal/orchengine#watchEnv.noticeLines`
- `internal/orchengine#TestWatcher_NoticesDeliveredOldestFirstOnePerTick`
- `internal/orchengine#TestWatcher_NoticeWaitsOnANonIdleProbe`
- `internal/orchengine#TestWatcher_NoticeWaitsOutANonIdlePhase`
- `internal/orchengine#TestWatcher_TickThatStartsACycleDeliversNoNotice`
- `internal/orchengine#TestWatcher_HeldNoticeWaitsForTheOperatorAndRecordsWhy`
- `internal/orchengine#TestWatcher_ConditionalNoticeDeliveredOnlyWhileItsStateReads`
- `internal/orchengine#TestRun_NoticeQueueAtExit`
- `internal/orchengine#TestWatcher_DraftHoldLogsOnceWithoutTheDraftText`
- `internal/orchengine#TestWatcher_LogsDisagreementBetweenIdleProbeAndSessionState`

**Intent:**
The watcher reaches creel through a seam beside its `Session` seam (decision record D10), its notice queue goes (Scope, D6), and its pause-resume line becomes creel mail (D9).

`orchengine.Creel` is the seam, with the methods `Post(receiver, body string) (int64, error)`, `ReadMailbox(handle func([]CreelMessage) error) error`, `Inject(receiver string, step CreelStep, ttl time.Duration) (int64, error)`, `InjectOutcome(id int64) (CreelOutcome, error)`, `Sent(receiver string, since time.Time) ([]CreelSent, error)`, `Hold(receiver string, duration time.Duration) error`, `Release(receiver string) error` and `Withdraw(receiver string) ([]CreelWithdrawn, error)`.
`CreelMessage` holds a mailbox message's id, sender, `reply_to` and body; `CreelStep` an inject's command name and argument, or its text; `CreelOutcome` an inject's `CreelState` (queued, sending, injected, unconfirmed, expired, or withdrawn for an inject expired with reason `withdrawn`), the time of its newest `sending` record and the stall reason its `expired` record names; `CreelSent` one post or inject the caller sent, with its id, time, whether it is mail, its step and its subject; `CreelWithdrawn` a withdrawn message's id and subject.
`internal/orchengine` imports no creel package, and its import allowlist in `seam_enforcement_test.go` gains none (`PATTERN-told-geometry`).

`Session` gains the interface method `StrandName(guid string) (string, error)`, the strand's full name read from reed at each call, which the watcher uses to address creel calls.
`NewWatcher` takes the `Creel`, stored on `Watcher`.

The notice queue goes: `notice.go` with `QueueNotice`, `QueueConditionalNotice`, `ListNotices`, `RemoveNotice`, `DropNotices` and the cap, and `notice_test.go`; `Watcher.deliverNotice` and its call from `tick`; `Watcher.dropQueueWithoutStrand`.
`Paths.NoticesDir` stays, documented as the retired queue directory, and `Paths` gains `RetiredNoticeLogPath`, the retired `.lyx/notices.log` of the prime, which `orchPaths` joins through `lyxdirs.DotLyxDirName`.
`CleanupRetiredNotices` lists both when present and, with `apply`, removes them; `lyx orch cleanup` (`cleanupCmd`) calls it beside `CleanupHandoffs` and reports what it listed or removed under `retired` (D8).

`Watcher.Run` reads its mailbox once before its first tick, so creel knows the program as a receiver before a reply can arrive; this card's handler logs each message and drops it, since no request is pending yet, and card 10 gives that read the reply gate's handler.
A failed start read is logged and changes nothing else, and is not counted as a tick error.
`queueResumeLine` posts `pauserecord.ResumeLine` as creel mail from the watcher to the orch strand's name, which creel reminds once the pause is lifted; mail has no TTL, so the line cannot expire while the orch stays busy.
It marks the entry noticed only after the post succeeds, so a failed post is posted again on the next tick.

`internal/orchcli`: `orchCLI` gains `newCreel func() (orchengine.Creel, error)`, set in `Command`'s pre-run to a `creelSeam` over `creelcli.NewProgramClient(location, orchWatchProgram)`, so the watcher always signs as `ORCHWATCH` whatever `$LYX_STRAND_NAME` it inherited; `watchCmd` passes it to `NewWatcher`, and `runnerSession.StrandName` answers the tracked strand's name or an error naming the guid.
`creelSeam` holds a `creelClient`, the interface of the `GoClient` methods it calls, which `*creelcli.GoClient` satisfies, and maps each method onto the call of the same name; `InjectOutcome` and `Sent` fold `GoClient.Log` answers through `injectOutcomeOf` (`log --id`: the newest of `sending`, `injected`, `unconfirmed`, `expired`, with `expired` plus reason `withdrawn` read as withdrawn, and the time of the newest `sending` record for the id, since `markSending` journals one `sending` line per typing attempt and `requeue` after `ErrBusy` resets only in-memory state) and `sentOf` (`log --from` the watcher `--to` the receiver `--since`: each `posted` event, mail or inject, with its command, argument or subject), so the engine never reads a log entry.

Tests:
`fakeCreel` records posts, injects, holds, releases and withdraws, answers scripted outcomes, mailbox messages and errors, and `watchEnv` builds the watcher with it and gives `fakeSession` a strand name.
`TestWatcher_HubPauseInterruptsAtGraceEndHoldsAndQueuesTheResumeLine` asserts the resume line is one creel post to the orch strand's name and the entry is marked noticed after it, and gains a row where the first post fails and the next tick posts it once and marks it; its rows that asserted queued notice lines assert the fake creel's posts instead, and its row where a notice and a cycle request wait under a live entry keeps only the cycle request, since no notice passes through the watcher.
`TestRun_ReturnsWhenStrandGone` asserts `Run` made one mailbox read before its first tick, and gains a row where that read fails and `Run` still ticks and returns when the strand is gone, with no tick error recorded; `TestRun_StopsAtErrorCap`, `TestRun_SuccessfulTickResetsErrorCount` and `TestRun_ChecksTheBinaryOnlyAtARestartSafeTick` build their watcher with the fake creel.
The six notice delivery tests and `TestRun_NoticeQueueAtExit` go with the queue, and so do `TestWatcher_DraftHoldLogsOnceWithoutTheDraftText` and `TestWatcher_LogsDisagreementBetweenIdleProbeAndSessionState`, which drive the watcher's idle probe through a queued notice; that probe and its hold logging go in card 10.
In `internal/orchcli`, `fakeCreel` is the same kind of recording fake, and `newTestCLI` sets `newCreel` to answer one, so `TestWatch` and card 11's start tests never reach a daemon.
`TestCleanup_Envelope` gains rows: a dry run lists a retired `notices/` directory and `notices.log` without removing them, and `--apply` removes both.
`TestCreelSeam_FoldsTheJournal`, a new untagged test in `internal/orchcli`, drives `creelSeam.InjectOutcome` and `creelSeam.Sent` over a `fakeCreelClient`, which embeds `creelClient` and answers `Log` with scripted entries, and asserts each outcome state, the `sending` time, the later time for an id with two `sending` records (a busy attempt, then the typed one), the withdrawn reading, the expired stall reason and the sent list with command, argument and subject; `cmd/lyx/testdata/test-budget.yaml` raises `internal/orchcli` by one.
`internal/orchengine/doc.go` replaces "Notices" with the creel seam, updates the `.lyx/orch/` layout and the hub pause's resume line.

**ImpactSummary:** Adds the watcher's creel seam and one Session method, re-signs NewWatcher (one production caller in orchcli, three test callers listed), and deletes the notice queue after cards 6 and 7 removed its last outside callers.
