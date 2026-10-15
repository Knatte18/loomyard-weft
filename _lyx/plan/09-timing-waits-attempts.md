# Card 9 — timing-waits-attempts

**Create:**
- `internal/reportengine#WaitParentReview`
- `internal/reportengine#WaitPRGate`
- `internal/reportengine#WaitEscalation`
- `internal/reportengine#WaitHalt`
- `internal/reportengine#waitInterval`
- `internal/reportengine#battenRowHalts`
- `internal/reportengine#runWaits`
- `internal/reportengine#netWaits`
- `internal/reportengine#AttemptFigure`
- `internal/reportengine#TimingAttempts`
- `internal/reportengine#runAttempts`

**Edit:**
- `internal/reportengine#RunRow`
- `internal/reportengine#RunRow.readAlignmentWakes`
- `internal/reportengine#TimingRun`
- `internal/reportengine#TimingBucket`
- `internal/reportengine#timeRun`
- `internal/reportengine#TestComputeTiming`
- `internal/reportengine/doc.go`

**Intent:**
Add the time a run spent waiting on someone else, and the arbiter and alignment attempts that served those waits, to each timing run and bucket.

Waits (`runWaits`), each an interval of one kind, in this order:
`WaitParentReview`, a parent-review round from its request's `opened_at` to its verdict's `recorded_at`;
`WaitPRGate`, a batten change into awaiting at `PR-Gate`;
`WaitEscalation`, a change into awaiting at any other row;
`WaitHalt`, a change into blocked or failed.
A change's wait runs from batten's entry for it to the next row of either history stamped after it.
A batten-row halt is a batten history entry at one of batten's own rows, never `Run-Shed`, that left batten awaiting (an `awaiting` outcome) or blocked (a `stuck` outcome no `OnStuck` bounced, its `routed_to` empty); `battenRowHalts` returns their indexes in the batten record, card 10 counts them as stops, and each is a wait of the kind its state gives, `WaitEscalation` for an awaiting one and `WaitHalt` otherwise, from its own entry to the next row.
A wait with no later row and no verdict runs to the span's close, which is `SnapshotAt` for a run in flight, so an open wait there runs to the snapshot with no finding.
A parent-review round with no request opened no wait and adds none.
A batten `Stuck` entry written before `routed_to` existed has it empty even when `OnStuck` bounced it, so an older batten record reads such a bounce as a batten-row halt; `doc.go` states this bound, as the decision record's open risks state the alignment stamp's.
`netWaits` clips every wait to the span, removes sleep first, then unions each kind and removes from it every earlier kind's union, so overlapping waits count once, under the kind listed first, and a sleep inside a wait counts as sleep alone.
Waits stay inside their phase's time; `TimingRun` gains `waits` (kind to seconds, every kind present), and `timeRun` sets `active_seconds` to the span less sleep less the union of every net wait, so active plus waits plus sleep equals the span.

Attempts (`runAttempts`): every arbiter attempt (`RunRow`'s arbiter records), every PR-Gate alignment attempt and every parent-review round's alignment record is one spawn.
`readAlignmentWakes` keeps each decoded `alignment.Attempt` on a new unexported `RunRow` field, as `readArbiter` keeps its records.
An attempt's time runs, net of sleep, from its `spawned_at` to the first row of either history stamped after it, or to `SnapshotAt` in a run in flight.
In a landed run, an attempt whose `spawned_at` falls outside the span, or that has no row after it, has its time absent and adds a finding naming its file; its spawn still counts.
`AttemptFigure` is `count int` and `seconds *int64`, the summed time of the attempts whose time is known, nil when none is; `TimingAttempts` holds one figure each for `arbiter`, `alignment` and `parent_review_alignment`; `TimingRun` gains `attempts`.
Attempt time lies inside the wait it serves and is reported beside the waits, never added to them.

`TimingBucket` gains `waits` (kind to seconds, clipped to the bucket like the phases) and `attempts`, each spawn counted in the bucket its `spawned_at` falls in and its time clipped to each bucket it overlaps.

`TestComputeTiming` gains rows: a parent-review round overlapping a halt counted once as `parent_review`; an awaiting change at `PR-Gate` as `pr_gate`, one at `Plan-Bouncer` as `escalation`, a blocked and a failed change as `halt`; a batten `Land` entry `stuck` with no `routed_to` as a `halt` wait and an `awaiting` one as `escalation`; a sleep inside a halt counted as sleep and not as wait; a landed run whose active, waits and sleep sum to its span; a run in flight whose open halt wait and open arbiter attempt run to `SnapshotAt` with no finding; a landed run's arbiter attempt spawned before the span with absent seconds, a finding and a count of 1; a landed run's arbiter attempt spawned after its last row of either history with absent seconds, a finding and a count of 1; a PR-Gate alignment attempt and a parent-review alignment record each counted with its time to the next row; batten `Run-Shed` entries, a budget-exempt `stuck` self-bounce among them, and a batten `Land` entry `stuck` with `routed_to` set adding no wait; a landed run's parent-review round with no verdict whose wait runs to the span's close, and a round with no request adding no wait; and, at the bucket level, a halt wait and an arbiter attempt spanning UTC midnight clipped across both day buckets, the attempt's spawn counted only in the bucket of its `spawned_at`.
`internal/reportengine/doc.go`'s Timing section gains the waits and attempts as above.

**ImpactSummary:** readAlignmentWakes keeps what it already decodes, so convergence's figures are unchanged; the rest extends this plan's own timing types.

**Commit:** 9: Add waits and arbiter and alignment attempt time to the timing report
