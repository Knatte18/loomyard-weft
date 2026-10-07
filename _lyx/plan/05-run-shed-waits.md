# Card 5 — run-shed-waits

**Edit:**
- `internal/battenshed/innerrun.go`
- `internal/battenshed/deps.go`
- `internal/battenshed/doc.go`
- `internal/battenshed/innerrun_test.go`
- `internal/battenshed/innerrun_halted_test.go`
- `internal/battenshed/innerrun_awaiting_test.go`
- `internal/battenshed/innerrun_ide_test.go`
- `contracts/recipes/batten-recipe.yaml`
- `internal/shedrecipe/entries_batten.go`
- `internal/shedrecipe/entries_batten_test.go`
- `internal/battenrecipe/recipe_test.go`
- `internal/battencli/wire.go`
- `internal/battencli/wire_seams_test.go`
- `internal/battencli/commitstatus.go`
- `internal/battencli/lifecycle_integration_test.go`
- `docs/overview.md`

**Intent:**
D3: the Run-Shed row waits on the child inside its `Call` instead of sleeping one poll interval and returning `Stuck` on every arm.

Each arm keeps its actions (spawn, adopt, revive, resume on a decision, done grace) and then waits in one shared wait loop in `innerrun.go`, which checks every `poll_interval_s` through `deps.Sleep` and returns only when:
- the child's `State` differs from the state the call began with: a budget-exempt `Stuck` whose `OutputPointer.Path` names the change (`child running → blocked at Plan-Review`, the producer taken from the new status's `CurrentProducer`) and whose `Reason` mirrors it, so each change is its own history entry and equal entries never fold across changes;
- the arm's own event fires: for an awaiting child a decision record not yet acted on (the resume runs as today, and the wait goes on until the child leaves `awaiting`); for a done child the driver strand ending or `driverExitGrace` elapsing since first sight (returns `Done`);
- batten's own status carries `pause_requested`, read through a new `InnerRunDeps.PauseRequested func() (bool, error)` seam: a budget-exempt `Stuck` whose `Path` is `pause requested at <time>`, the time being the check's (`deps.Now`), so a second pause is its own entry; the engine then pauses at the row boundary as today;
- the stat of the child's status file fails: a budget-exempt `Stuck` naming the file, leaving the next call's status read to decide (an absent file by today's absent-file handling, an unreadable one as the hard error);
- the context is cancelled, or decoding the child's status file fails (hard errors, as today).

Per check, the cheap file reads run every time: the status file's stat, the pause flag, the decision record, the done-seen marker.
The status file is decoded only when its modification time differs from the baseline (any difference, not only a later time), and also once per `notice_probe_s` whatever the time says.
The baseline modification time is stat'ed before the status read that defines the state the call began with (the spawn arm's second read included), so a write landing between that read and the first check is seen.
Every read that costs a process or a multiplexer round trip runs at most once per `notice_probe_s`: `DriverAlive` and `DriverStrand` wherever read, the review-wait note, the forced decode, and the retry of a resume the child's driver refused as `ErrChildNotParked` (which logs once per episode and goes on until the driver parks or the state changes).
The running arm writes its wait reason through `reportStuck` at entry and again when the review-wait note changes, so `lyx batten status` still shows what the child waits on.
Today's `noticeStep` runs at the call's start and once per `notice_probe_s` inside the wait (card 6 replaces it).
Every `Stuck` the row returns is budget-exempt; the row no longer returns a counted `Stuck`.

`InnerRunDeps` gains `PauseRequested` (a nil one reads as never paused) and `NoticeProbe time.Duration`; `innerRunEntry` reads a new `notice_probe_s` key (default 30, negative refused, the same shape as `poll_interval_s`) into it, and `defaultInnerRunPollIntervalS` becomes 2.
In `batten-recipe.yaml` `poll_interval_s` becomes 2, `notice_probe_s: 30` is added, `max_bounces: 1440` is removed, and the header comment is rewritten for the in-call wait (the self-route remains for the exempt returns; no running watch window).
`battenrecipe/recipe_test.go`'s twelve-hour pin is replaced by a pin on the new values and on the absence of a counted budget.
`battencli/wire.go` wires `PauseRequested` to batten's own status file (`StatusFile`/`StatusLock` of the prime location and slug) through `state.ReadJSONStrict`, and `NoticeProbe` arrives through the entry; `commitstatus.go`'s comment about per-poll status rewrites is updated, since an uncommitted per-bounce rewrite no longer happens every poll.

Bound, stated in the `Call` doc comment and `internal/battenshed/doc.go`: the row now waits on a running child without a time limit; `lyx batten pause` (honoured within one check), cancellation, and the notices (cards 6 and 7) bound it; a running child whose driver is dead and whose status file does not change returns nothing from the wait, the driver-dead notice being the only signal.
Rewrite `Call`'s disposition table and the producer type's comment for the wait; update the batten section of `docs/overview.md` where it describes the per-poll bounce, the 12-hour running budget and the 1440 history entries.

Tests, on the injected `Sleep`/`Now` and a status file the test rewrites between checks (extending the existing table and arm tests rather than adding parallel ones):
- a child that stays running for many checks makes the call return nothing until it changes to `blocked`, then one exempt `Stuck` whose `Path` names the change; a later change back to `running` returns a distinct `Path`;
- a write between the call's status read and its first check is seen on that check; a write that leaves the modification time unchanged is seen within `notice_probe_s`;
- a pause flag set while the child runs returns within one check with the `pause requested at` path, and a second pause later returns a distinct path;
- a failed stat returns an exempt `Stuck` naming the file;
- an awaiting child's decision record is acted on within one check, and the call returns once the child leaves `awaiting`; a not-parked refusal is retried no more than once per `notice_probe_s`;
- a done child returns `Done` once `DriverAlive` reports the driver ended, read no more than once per `notice_probe_s`, or once the grace elapses;
- `entries_batten_test.go`: `notice_probe_s` default, explicit value and negative refusal;
- `wire_seams_test.go`: `PauseRequested` reads batten's own `pause_requested`;
- `lifecycle_integration_test.go`: the short-poll recipe drops `max_bounces`, and the step-driven Run-Shed test asserts that a step returns once the child's state changes rather than after one poll interval.

**ImpactSummary:** rewrites the InnerRun producer's wait and its recipe row; `NewInnerRun`'s signature is unchanged, `InnerRunDeps` gains two optional fields, and `shedengine` is untouched.
