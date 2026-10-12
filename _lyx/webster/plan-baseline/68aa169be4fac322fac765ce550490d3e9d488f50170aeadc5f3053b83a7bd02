# Card 9 — the orch watcher's hub-pause retry waits the settle across ticks and goes through the guard

**Create:**
- `plan:internal/orchengine#Watcher.pauseInterrupt` -> `func (w *Watcher) pauseInterrupt(st State) (sent bool)`
- `plan:internal/orchengine#fakeSession.InterruptSettle` -> `func (f *fakeSession) InterruptSettle() time.Duration`
- `plan:internal/orchcli#runnerSession.InterruptSettle` -> `func (s runnerSession) InterruptSettle() time.Duration`

**Delete:**
- `internal/orchengine#maxPauseInterrupts`

**Edit:**
- `internal/orchengine#Session`
- `internal/orchengine#Watcher`
- `internal/orchengine#Watcher.pauseTick`
- `internal/orchengine#Watcher.handleHubEntry`
- `internal/orchengine#fakeSession`
- `internal/orchengine#fakeSession.Interrupt`
- `internal/orchengine#TestWatcher_HubPauseInterruptsAtGraceEndHoldsAndQueuesTheResumeLine`
- `internal/orchengine/doc.go`
- `internal/orchcli#runnerSession.Interrupt`

**Uses:**
- `internal/shuttleengine#ErrNoRunningTurn`
- `internal/shuttleengine#ErrRewindPickerOpen`
- `internal/shuttleengine#Runner.InterruptSettle`

**Intent:**
D3's watcher log, D4's watcher half and D8's orch surface.
The watcher stays the only Go writer into the orch pane (`PATTERN-orch-pane-single-writer`).
Its settle rides the existing ticks and adds no wake (`PATTERN-daemon-wakeups`).

Seam:

- `Session` gains `InterruptSettle() time.Duration`, the provider's settle time.
- `runnerSession.InterruptSettle` returns `s.runner.InterruptSettle()`.
- `runnerSession.Interrupt`'s comment says `Runner.Interrupt` now sends the key only into a pane showing a running turn and closes an open rewind picker once.

`Watcher` gains `pauseFirstEsc map[int64]time.Time`: per hub entry key, in memory only, when the first key went.
Its comment states D4's bound: a watcher restarted during the settle has lost the time, finds the orch recorded in the entry, settles it through `settledWithoutInterrupt` and sends no second key.

`pauseInterrupt(st)` calls `w.session.Interrupt(st.Strand)` and returns whether the key went.

- On `ErrNoRunningTurn` it logs at Info "orch: the pause found no running turn on the orch pane, so the orch reads idle" with `strandGUID` and returns false.
  It never logs this at Warn, so a guard that withholds every orch key stays visible.
- On `ErrRewindPickerOpen` it logs at Warn "orch: the pause found the rewind picker open on the orch pane and closed it" with `strandGUID` and the error, and returns false.
- On any other error it logs today's Warn "orch: the pause could not interrupt the orch" and returns false.

Each refusal marks the entry handled (below), so each is logged at most once per hub entry.

`handleHubEntry(entry, record, st, now)`:

1. A handled entry returns, as today.
2. New, before `settledWithoutInterrupt`: when `pauseFirstEsc` holds the entry's key:
   - A tick with `now` less than `InterruptSettle()` past it returns nil and waits.
   - Otherwise it drops the key, re-probes with `pauseProbeIdle` and marks the entry handled.
     A probe error is returned, as today.
     An idle orch gets nothing more; a busy one gets `pauseInterrupt`, the second and last key.
   This step comes before `settledWithoutInterrupt` because the first key's record already makes the entry own the orch, which that check would read as settled.
   `pauseTick` reaches this step only while the entry is live, because its `record.Covering("")` check runs first.
   So a tick after a resume during the settle queues the resume line instead and sends no second key.
3. `settledWithoutInterrupt`, the grace-period branch and the idle check at the grace end stay as today.
   An idle orch at the grace end is marked handled.
4. A busy orch at the grace end is recorded in the entry as today; an `errPauseEntryGone` or `errPauseAlreadyOwned` marks it handled and returns nil.
   Then `pauseInterrupt` runs: when the key went, the entry's key and `now` go into `pauseFirstEsc` and the entry is not yet handled; when it did not, the entry is marked handled.

`pauseTick`'s godoc, which says the orch is interrupted once per entry at its grace end, says instead that the first guarded key goes at the grace end and a second only on a tick past the settle.
The `maxPauseInterrupts` loop and constant go; the two-key bound is now the structure above: one key at the grace end, one on a settled tick.

Tests, `watcher_test.go` and `watcher_pause_test.go`:

- `fakeSession` gains `settle time.Duration`, answered by `InterruptSettle`, and `interruptErrs []error`, consumed in order by `Interrupt` after it records the call, nil once drained.
  `flakySession` embeds `*fakeSession` and inherits both.
- `TestWatcher_HubPauseInterruptsAtGraceEndHoldsAndQueuesTheResumeLine`, existing rows:
  - "a busy orch is interrupted at the grace end and not before, twice when still busy, and recorded" sets `e.s.settle = 2 * time.Second`.
    After the grace-end tick it asserts one interrupt, then a second tick at the same time still one, then `advance(2s)` and a tick two.
    It is renamed "...and a second time only on a tick past the settle while still busy, and recorded".
  - "an idle orch with an open orch marker is interrupted when it reads busy at the grace end" expects 1 interrupt from its one grace-end tick.
  - "a watcher running before the entry interrupts a busy orch on the first tick after a zero grace" expects 1 interrupt and the orch recorded.
  - "a notice and a cycle request wait while the entry is live, then the resume line follows once": the orch reads idle at the next tick, so no second key goes.
    Its `want` becomes `[interrupt, send:n1, send:<resume line>]`.
- New rows:
  - "an orch idle at the settled re-probe gets no second Esc": settle 2 s, busy at the grace end, then `e.s.idle = true`, `advance(2s)` and two ticks.
    One interrupt.
  - "a resume during the settle sends no second Esc": settle 2 s, busy, the grace-end tick, `resumeHub()`, `advance(2s)`, and a tick with the orch still busy.
    One interrupt, and the orch's resume line queued as a notice.
  - "a guarded refusal reads as an idle orch and is logged once at Info": `logcapture.CaptureVerbose`.
    `interruptErrs [fmt.Errorf("%w (strand %q)", shuttleengine.ErrNoRunningTurn, "s1")]` with settle 2 s; the grace-end tick, `advance(2s)`, two more ticks.
    One `Interrupt` call, the Info message counted once and naming `s1`, and zero counts of the "could not interrupt" and "rewind picker" Warn messages.
  - "a guarded refusal of the settled second Esc is logged once at Info and ends the entry": the same capture, `interruptErrs [nil, wrapped ErrNoRunningTurn]`, settle 2 s, the orch busy throughout; the grace-end tick, `advance(2s)`, two more ticks.
    Two `Interrupt` calls, the Info message counted once, and no Warn.
  - "an open rewind picker is logged at Warn and gets no further Esc": `logcapture.CaptureVerbose`, `interruptErrs [wrapped ErrRewindPickerOpen]`, settle 2 s; the grace-end tick, `advance(2s)`, a tick with the orch still busy.
    One `Interrupt` call, and the Warn message counted once and naming `s1`.
  - "a watcher restarted during the settle sends no second Esc": settle 2 s, the grace-end tick, `e.w = e.newWatcher()`, `advance(2s)`, a busy tick.
    One interrupt.
- No new top-level test.

`orchengine/doc.go`'s Hub pause section, the "At the grace end" bullet, now says:

- A busy orch is recorded and interrupted once through shuttle's guarded interrupt.
- A later tick at least the provider's settle time after it re-probes and, when the orch still reads busy, sends the guarded second and last key.
- A guarded refusal reads as an idle orch with one Info line; an open rewind picker gets one Warn line and no further key.
- A resume during the settle, or a restarted watcher, sends no second key.

**ImpactSummary:** `Session` gains one method with three implementers (`runnerSession`, `fakeSession` and `flakySession` through embedding); the deleted constant has no reference outside `pause.go`; the hub-pause interrupt's second key moves from the same tick to a later tick.

**Commit:** 9: Orch watcher waits the settle across ticks before a guarded second Esc
