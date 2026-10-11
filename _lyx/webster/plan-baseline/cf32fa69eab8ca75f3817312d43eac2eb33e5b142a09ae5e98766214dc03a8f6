# Card 1 — shuttle-hold-deferral

**Edit:**
- `internal/shuttleengine#Spec`
- `internal/shuttleengine#Run`
- `internal/shuttleengine#Run.Wait`
- `internal/shuttleengine/hold_test.go`
- `internal/shuttleengine/doc.go`

**Create:**
- `plan:internal/shuttleengine#holdNoticeSettle` -> `const holdNoticeSettle = 45 * time.Second`
- `plan:internal/shuttleengine#holdNoticeMaxDeferral` -> `const holdNoticeMaxDeferral = 20 * time.Minute`
- `plan:internal/shuttleengine#Run.settleDeferredHold` -> `func (run *Run) settleDeferredHold()`

**Uses:**
- `internal/shuttleengine#Run.notifyHeld`
- `internal/shuttleengine#Run.pollEventsTick`

**Intent:**
Give a shuttle run a told deferral predicate for its held-turn notice (decision D10), the mechanism the chair's deferral in card 5 rides.

`Spec` gains `HoldDeferral func() bool`, documented beside `QuietHold`: when set, it reports whether outstanding work elsewhere defers this run's held-turn notice.
`validate` does not inspect it, and it reaches an attached run through the spec its attach is given, as every other field does.
A run without a predicate notifies exactly as today.

For a run that sets it, `Wait` no longer calls `notifyHeld` at the held turn end.
It records the held turn end and the run clock's time on the `Run` (two new fields, a pending `*heldTurnEnd` and its time), replacing any earlier pending one.
A held turn end read in the same batch as a turn start past it is not recorded, since the run has already moved past it.
On every later tick whose events read succeeded with a new event or a turn start past the pending turn end (the condition that already ends the held wait), the pending turn end is dropped unnotified.
Then, on every tick, `settleDeferredHold` decides a pending turn end that is still held:
once the run clock is at least `holdNoticeSettle` past its time and the predicate reports false, or at least `holdNoticeMaxDeferral` past it whatever the predicate says, it calls `notifyHeld` once with it and clears it.
The predicate is never called before the settle interval has passed.
`notifyHeld`'s own skips (interactive, quiet hold, no notifier, an offset at or below `NotifiedOffset`) still apply unchanged, so the deferred notice goes through the runner's notifier like every other (`PATTERN-orch-pane-single-writer`).
The re-check rides the wait loop's existing poll, adding no wait of its own (`PATTERN-daemon-wakeups`).

Bound: the deferral delays a notice and never drops one for a turn end the run is still held on, and never past `holdNoticeMaxDeferral`.
`holdNoticeSettle` is short enough to notify a real stall within a minute and long enough to cover a SendMessage's delivery to an idle peer;
`holdNoticeMaxDeferral` is long enough for a peer to answer an ordinary batch and short enough that a wedged peer hides a stall from the parent well before a review or discussion table's timeout.
Both are one constant each, in `hold.go`.

Tests, as rows of `TestWait_HeldTurnEndNotice` in `hold_test.go` (a predicate field and a per-Sleep clock jump on the row table, scripted by the row):
- a deferred run whose predicate always reports outstanding work sends exactly one notice, and only once the clock is past `holdNoticeMaxDeferral`;
- a deferred run whose predicate reports outstanding work and then none sends exactly one notice, after the settle interval and after the predicate turned false;
- a deferred run with no outstanding work sends its one notice no earlier than `holdNoticeSettle` after the turn end;
- a deferred run whose agent starts a new turn before the settle interval passes sends no notice for the old turn end.
The existing rows keep pinning that a run without a predicate notifies at once.

Add the deferral to the held-turn-end paragraphs of `internal/shuttleengine/doc.go`, beside `QuietHold`.

**ImpactSummary:** One new optional Spec field and a wait-loop branch taken only when it is set; every existing run notifies as before.
