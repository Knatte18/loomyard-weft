# Card 8 — batten holds an arbiter-eligible stop's notice until the arbiter has handed on

**Edit:**
- `internal/battenshed#InnerRunDeps`
- `internal/battenshed#NewInnerRun`
- `internal/battenshed#noticeEpisode`
- `internal/battenshed#innerRunProducer.stateChangeReady`
- `internal/battenshed/doc.go`
- `internal/battenshed#noticeHarness`
- `internal/battenshed#noticeHarness.producer`
- `internal/battencli#battenCLI.wire`
- `internal/battencli#TestWire_MarkerAndReportSeamsRefuseAnAbsentTaskWorktree`
- `internal/battencli#stepStopReport_ReadsTheParkMarkersContentAndTime`
- `internal/testkit/envkit#nilLegal`
- `cmd/lyx/testdata/test-budget.yaml`

**Create:**
- `internal/battenshed/stophold.go`
- `plan:internal/battenshed#failArbiterStop` -> `const failArbiterStop = "battenshed: arbiter stop unreadable; holding nothing"`
- `plan:internal/battenshed#innerRunProducer.eligibleStopHold` -> `func (p *innerRunProducer) eligibleStopHold(ctx context.Context, w *childWait, now, parkAt time.Time, parkFound bool) (holding bool, end time.Time)`
- `plan:internal/battenshed#stopHold` -> `func stopHold(reading ChildArbiterStop, since, now, parkAt time.Time, parkFound bool, offSeen time.Time) (holding bool, end time.Time)`
- `plan:internal/battenshed#TestNotice_EligibleStopHold` -> `func TestNotice_EligibleStopHold(t *testing.T)`

**Uses:**
- `plan:internal/battenshed#ChildArbiterStop`
- `plan:internal/arbiterstop#HoldReading`
- `internal/battenshed#innerRunProducer.arbiterHolds`
- `internal/battenshed#TestNotice_LiveArbiterHoldsTheStateChange`
- `internal/battenshed#TestNotice_AwaitingLeavesTheRelayToTheDriver`
- `internal/battencli#readStopReport`
- `internal/battencli#TestWire_EverySeamFilled`

**Intent:**
D9's batten half: at an arbiter-eligible stop the parent hears about the stop only once the arbiter has handed on, or when no arbiter will settle it.
This closes the incident windows the record names (before the driver writes the arbiter marker, while the marker's strand reads not alive, and after the verb removed the marker but before the driver acts on the verdict), since the hold reads neither the marker nor its strand.

`internal/battenshed`:

- `InnerRunDeps` gains `ArbiterStop func(ctx context.Context, status shedengine.Status) (ChildArbiterStop, error)`, resolved on Call like every task-worktree seam; `NewInnerRun` resolves a nil one to a reading that is not eligible, and its doc comment says so.
- `noticeEpisode` gains `arbiterOffAt time.Time`, the first time this process saw the arbiter off while the stop was eligible in this episode; a new episode starts it at zero.
- `stophold.go` holds the hold, with `failArbiterStop` as its seam-failure key beside the other `fail*` constants' meaning.
  `stopHold` is pure and decides, in this order:
  a stop that is not `Eligible` is not held, ending at `since`;
  a non-zero `AppliedAt` ends it at the later of `AppliedAt` and `since`;
  `On` false ends it at the later of `offSeen` and `since`;
  no spawns left and no attempt at the key never start it, ending at `since`;
  a park marker found with `parkAt` at or after `since` ends it at `parkAt`, unless `HandOverAt` is non-zero and at or after `since`, which marks the Go driver's hand-over marker, whose own hand-back writes no settle record;
  an `Attempt` whose `HandOnAt` is non-zero and at or after `since` ends it at `HandOnAt`;
  the backstop, the later of `since` plus `Timeout` plus `noticeReportWait` and, with an `Attempt`, `AttemptDeadline` plus `noticeReportWait`, ends it at the backstop once `now` reaches it;
  otherwise the stop is held.
  So the start is the episode's `since`, read from the status file, and every end time comes from a record or the backstop, and is never before `since`.
  The end conditions are checked on every pass, so switching the arbiter off mid-hold ends it at the next pass.
- `eligibleStopHold` calls `ArbiterStop` only for a `blocked` or `awaiting` status, so error stops keep the marker hold alone; it records `arbiterOffAt` at the first eligible reading with `On` false, and calls `stopHold` with the episode's `since` and that time.
  A seam error warns once per episode through `seamFailed(failArbiterStop, err)`, holds nothing and ends at `since`; a later good read recovers it.
- `stateChangeReady` asks `arbiterHolds` first, as today, then `eligibleStopHold` with the park marker's time from `stopReport`, and waits while either holds.
  Once neither holds, it measures `noticeReportWait` from the later of `since` and the hold's end, in place of `since`, so a hold that outlasts the report wait does not find it already spent and send `report none yet` before the driver's park marker.
  A report still qualifies at or after `since`, and the awaiting relay rule, `noticeRelayWait` from the report with the decision-record check, is unchanged.
  So a watched awaiting stop with a parent notice waits `noticeReportWait` from the hand-on for the park marker, then `noticeRelayWait` from that marker, and the driver's own relay after a hand-on is not doubled.
  A stop the hold never covered keeps today's rules: its end is `since`.
  The doc comment states the order.
- `doc.go`'s notice paragraph adds the eligible-stop hold beside the marker hold, with its end conditions, its backstop and the restart rule: the start comes from the status file and the end from a record, so a batten restart neither resets the start nor loses the end, and after the arbiter was switched off a restarted batten takes its own first observation.

`internal/battencli`: `battenCLI.wire` fills `InnerRun.ArbiterStop` with a closure that resolves the task worktree through `locateTask`, returning its error, which names the slug, for an absent pair, and returns `arbiterstop.HoldReading(taskLocation, status)`.
`TestWire_EverySeamFilled` then finds the new seam filled with no change.

Tests:
`noticeHarness` gains `arbiterStop ChildArbiterStop` and `arbiterStopErr error`, and `noticeHarness.producer` wires `ArbiterStop` to them.
`TestNotice_EligibleStopHold` is a table run with fake clocks, each row leaving running into a `blocked` stop, or an `awaiting` one with a parent notice and the watched marker held, with an eligible, on reading, a 30 minute timeout and spawns left unless the row says otherwise. Its rows:
- the incident shape: with the driver alive, no report and the clock past `noticeReportWait`, no notice goes while there is no arbiter marker yet, while `ArbiterInFlight` reports a marker whose strand is not alive, and while the attempt exists with no marker and no hand-on;
- a tackler-led attempt whose deadline lies past `since` plus the timeout plus `noticeReportWait`: no notice at that point, and a notice once its deadline plus `noticeReportWait` passes;
- a park marker written after `since` sends the blocked stop's notice at once with its report;
- a hand-on attempt more than `noticeReportWait` after the stop, with no park marker, at a watched awaiting stop: no notice until `noticeReportWait` after `HandOnAt`; and with a park marker written after the hand-on, no notice until `noticeRelayWait` after the marker with no decision record;
- an applied resume at the key ends the hold, and the notice goes `noticeReportWait` after `AppliedAt`;
- the arbiter switched off mid-hold ends it, and the notice goes `noticeReportWait` after the pass that first saw it off;
- with no attempt, the backstop at `since` plus timeout plus `noticeReportWait` sends the notice, even when the driver strand has ended earlier;
- a park marker at or after `since` with `HandOverAt` at or after `since` keeps the hold, so a blocked stop whose driver handed over gets no notice before the successor's attempt hands on; the same marker with `HandOverAt` zero, the successor's tackler having handed back, ends it at once with the marker's report;
- a restart mid-hold, a second producer over the same scratch directory, still holds until the same backstop; a restart after a hand-on and before the notice measures the wait from `HandOnAt`, after an applied resume from `AppliedAt`, after the backstop from the backstop, and after the arbiter was switched off from its own first pass;
- the hold does not start, so the notice goes `noticeReportWait` after `since` as today, for a reading that is not eligible, one that is off from the start, one with no spawns left and no attempt at the key, and one with an `AppliedAt` before `since`;
- a seam error holds nothing and the notice goes as today.
`TestWire_MarkerAndReportSeamsRefuseAnAbsentTaskWorktree` asserts `InnerRun.ArbiterStop` returns the absent-worktree refusal naming the slug.
`stepStopReport_ReadsTheParkMarkersContentAndTime`, before it writes the park marker, writes a driver stop report file under `shedrun.DriveReportsDir(childLocation, shedrun.SelfRunID)`, as the Go driver's start report does before its tackler or arbiter runs, and asserts `StopReport` still reports none: such a file is no park marker, so it cannot end the hold.
`internal/testkit/envkit`'s `nilLegal` gains `"InnerRun.ArbiterStop"`, since a nil seam is a documented default, reading not eligible, so `FullEnv` keeps passing `NilSeams`.
`TestNotice_LiveArbiterHoldsTheStateChange` and `TestNotice_AwaitingLeavesTheRelayToTheDriver` keep passing unchanged, since the harness's zero reading is not eligible.
`cmd/lyx/testdata/test-budget.yaml` raises `internal/battenshed` from 113 to 114.

**ImpactSummary:** batten's state-changed notice gains a hold for arbiter-eligible blocked and awaiting stops, read through a new InnerRunDeps seam that battencli wires to arbiterstop; the report wait is measured from the hold's end; stops the hold never covers keep today's timing.

**Commit:** 8: batten: hold an arbiter-eligible stop's notice until the arbiter hands on
