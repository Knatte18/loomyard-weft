# Card 5 — orch-conditional-notice

**Edit:**
- `internal/orchengine#Notice`
- `internal/orchengine#ListNotices`
- `internal/orchengine#QueueNotice`
- `internal/orchengine#noticeName` -> `func noticeName(now time.Time, when *NoticeCondition) (string, error)`
- `internal/orchengine#noticeNames`
- `internal/orchengine#Watcher.deliverNotice`
- `internal/orchengine#Session`
- `internal/orchengine#Watcher.logIdleStateDisagreement`
- `internal/orchengine/doc.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Create:**
- `internal/orchengine#NoticeCondition`
- `internal/orchengine#QueueConditionalNotice`
- `internal/orchengine#TestQueueConditionalNotice`
- `internal/orchengine#TestWatcher_ConditionalNoticeDeliveredOnlyWhileItsStateReads`

**Intent:**
The orch notice queue gains a conditioned notice, which `revive` queues for the orch instead of typing into its pane (D5, `PATTERN-orch-pane-single-writer`).
The watcher stays the one writer into the orch's pane, and drops a revive notice whose orch has recovered before delivery.

`NoticeCondition` names a session state, `State shuttleengine.SessionStateName` and `Cause string`, and is declared in `notice.go`.
`Notice` gains `Condition *NoticeCondition`, nil for an unconditioned notice.

`QueueConditionalNotice(p, line, when, now)` refuses and logs exactly as `QueueNotice` does: a multi-line or slash-leading line is an error, and with no strand recorded nothing is queued.
It queues nothing and returns `queued` false with a nil error when a notice with the same line and the same condition is already queued, so a second revive before delivery queues no second notice.
Two such calls racing may each queue one, which the orch reads as a repeated carry-on line; the record accepts that.
The condition is carried in the file name, as `.when-<state>+<cause>` after the random suffix, and the name ends in `.cnotice` in place of `.notice`: `<time>-<rand>.when-<state>+<cause>.cnotice`.
`noticeName` builds that name for a non-nil condition and today's `.notice` name for nil, which `QueueNotice` passes, and `noticeNames` lists both suffixes; since every name starts with its arrival time, one sort keeps the two kinds in arrival order.
The file's content stays the line.
A watcher built before this change lists only `.notice` names, so it leaves a conditioned notice queued until a watcher of this build delivers or drops it, and never types it unchecked.
`ListNotices` parses that tag into `Notice.Condition`.

`Watcher.deliverNotice`, for the oldest notice carrying a condition, reads `w.session.SessionState(st.Strand)` before the idle probe.
When the state's name or cause differs from the condition, it removes the notice without typing it, logs one Info naming it, and delivers nothing on that tick.
When they match it delivers as today; a state read error returns the error and keeps the notice.
The `Session` interface's `SessionState` comment, which says the watcher only logs the state, now says the watcher also reads it to drop a conditioned notice.
`Watcher.logIdleStateDisagreement`'s doc comment clause "nothing the watcher decides depends on the state" now names the conditioned notice's drop as the one decision that reads the state.

`doc.go`'s "Notices" section states the conditioned notice, its duplicate skip and the drop at delivery.
`doc.go`'s session-state paragraph likewise drops "The watcher only logs it" and "no probe result, cycle, delivery or State write depends on the state", and names the conditioned notice's drop as the one decision that reads the state.

`TestQueueConditionalNotice` in `notice_test.go`, a new top-level test for a new exported function:
- the queued notice lists with its line and its `Condition`, and its file name does not end in `.notice`;
- a second call with the same line and condition returns false and the queue still holds one notice;
- the same line with no condition, through `QueueNotice`, or with another condition is queued beside it.
The line refusals share `QueueNotice`'s validation, which `TestQueueNotice_RefusesMultiLineAndSlashLeading` pins, so they get no row here.

`TestWatcher_ConditionalNoticeDeliveredOnlyWhileItsStateReads` in `watcher_test.go`, over `newWatchEnv` and its `fakeSession`, whose `sessionState` field the rows set:
- `sessionState` reads idle-stalled with cause `api-error`: the tick types the notice and removes it;
- `sessionState` reads busy, or idle-done: the tick types nothing and the queue is empty after it;
- an unconditioned notice queued behind a dropped one is typed on the next tick;
- `sessionStateErr` is set: the tick returns the error, types nothing and keeps the notice queued.
Both tests queue through `QueueConditionalNotice` on the env's `paths`.

`test-budget.yaml`'s `internal/orchengine` row rises from 140 to 142.

**ImpactSummary:** The orch notice queue gains a condition carried in the file name; unconditioned notices, their writers and the watcher's delivery of them are unchanged.

**Commit:** 5: orch: a conditioned notice the watcher drops once its state no longer reads
