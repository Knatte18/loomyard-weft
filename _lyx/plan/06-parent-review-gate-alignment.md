# Card 6 — parent-review-gate-alignment

**Create:**
- `plan:internal/parentreview#Aligner` -> `type Aligner interface`
- `plan:internal/parentreview#Alignment` -> `type Alignment struct`
- `plan:internal/parentreview#AlignmentPoll` -> `type AlignmentPoll struct`
- `plan:internal/parentreview#AlignmentRunning` -> `const AlignmentRunning = "running"`
- `plan:internal/parentreview#AlignmentApplied` -> `const AlignmentApplied = "applied"`
- `plan:internal/parentreview#AlignmentFallback` -> `const AlignmentFallback = "fallback"`
- `plan:internal/parentreview#AlignmentStopped` -> `const AlignmentStopped = "stopped"`
- `plan:internal/parentreview#alignmentFile` -> `const alignmentFile = "alignment.json"`
- `plan:internal/parentreview#Store.RecordAlignment` -> `func (s Store) RecordAlignment(a Alignment) error`
- `plan:internal/parentreview#Store.SettleAlignment` -> `func (s Store) SettleAlignment(state, report, reason string) error`
- `plan:internal/parentreview#Store.WriteBrief` -> `func (s Store) WriteBrief(brief string) error`
- `plan:internal/parentreview#closures.spawnAlignment` -> `func (c *closures) spawnAlignment(r Round) (shuttleengine.GateResult, error)`
- `plan:internal/parentreview#closures.pollAlignment` -> `func (c *closures) pollAlignment(r Round) (shuttleengine.GateResult, bool, error)`
- `plan:internal/parentreview#closures.closeAlignment` -> `func (c *closures) closeAlignment(r Round, state, reason string) error`
- `plan:internal/parentreview#fakeAligner` -> `type fakeAligner struct`
- `plan:internal/parentreview#TestGate_Alignment` -> `func TestGate_Alignment(t *testing.T)`

**Edit:**
- `internal/parentreview#GateConfig`
- `internal/parentreview#Round`
- `internal/parentreview#Store.load`
- `internal/parentreview#closures.gate`
- `internal/parentreview#closures.open`
- `internal/parentreview#closures.final`
- `internal/parentreview#closures.pingDue` -> `func (c *closures) pingDue(r Round, now time.Time) bool`
- `internal/parentreview/doc.go`
- `internal/testkit/envkit#nilLegal`

**Intent:**
Make D4's gate spawn the alignment agent before any delivery to the run's parent, apply its verdict through the store, and fall back to today's delivery on everything else.
`internal/parentreview` keeps its import set (D1): the agent is reached through the new `Aligner` seam, which card 7 implements over `internal/alignment`.

The seam and record:
- `Aligner` has three methods: `Spawn(round int) (Alignment, error)`, which starts the agent for a round and is idempotent per round on the implementation's side; `Poll(a Alignment) (AlignmentPoll, error)`, which never blocks; and `Close(a Alignment, reason string) error`, which stops a live agent and closes its attempt.
- `Alignment` is the round's `alignment.json`: `Attempt` (the attempt directory), `Strand`, `SpawnedAt`, `State` (one of the four `Alignment*` constants), `Report` and `Reason`.
- `AlignmentPoll` carries `Done`, `Verdict` (`approve`, `reject`, `escalate`, or empty when no verdict applies), `Report` and `Reason`.
- `Round` gains `Alignment *Alignment`, and `Store.load` reads the file under the round lock beside the others.

The store methods, each under the latest round's lock:
- `RecordAlignment` writes the record.
- `SettleAlignment` sets `State`, `Report` and `Reason` on the latest round's record.
- `WriteBrief` replaces `brief.md`.

`GateConfig` gains two fields:
- `Aligner`: nil keeps today's gate exactly, which is how `skip_alignment` and a board-less run are wired, so both behave as before this task.
- `RenderAlignmentBrief func(report, reason string) (string, error)`: the reviewer brief after a fallback, naming the alignment report or why no verdict came.

With `Aligner` set, the gate runs in this order:
1. **Open.** `open` writes the request and brief as today, then calls `spawnAlignment` instead of carrying a prompt. The same happens on a later evaluation when the round has an open request, no alignment record, no prompt carried and no delivery. `spawnAlignment` calls `Aligner.Spawn` and records the result as `running` under the round lock, then returns pending with no prompt. This runs before the liveness hold and before `noReviewer`, so the agent spawns with the orch not live or absent (D4). The round's record means a driver restart re-reads it and never spawns again.
2. **Parent verdict or expiry.** On a round whose alignment is `running`, a verdict recorded by the parent's verbs, or the round reaching `WaitBound`, first runs `closeAlignment` with `stopped` and the reason. That calls `Aligner.Close` and then `SettleAlignment`, so the parent's verdict stands and the agent's verdict is dropped. The existing verdict and expiry logic then runs unchanged.
3. **Poll.** While the record is `running`, `pollAlignment` polls on every evaluation (the gate's existing re-evaluation cadence):
   - Pending: carry a due keep-alive ping, or nothing.
   - `approve`: `RecordVerdict(approve, report)`, the plain record and never the cap-reject supersede path. Then settle `applied`, close, and pass.
   - `reject`: `RecordVerdict(reject, report)`, settle `applied`, close and evaluate again. The existing consume, re-prompt and cap logic then applies unchanged, so a second reject at `attempts: 2` halts the run `blocked`.
   - A `RecordVerdict` refused with `ErrVerdictRecorded` means the parent's verdict landed first: close the alignment and evaluate again, so the parent's verdict is read.
   - `escalate`, or `Done` with no verdict (timeout, died, missing, invalid, or the card-7 snapshot check): close with `fallback`, the report and the reason. Then write the brief through `RenderAlignmentBrief(report, reason)` and `WriteBrief`, and continue into today's delivery path in the same evaluation.
4. **Today's delivery.** A record in `fallback`, `applied` or `stopped` is never polled again, so a verdict the agent writes after the fallback is never applied. The round runs today's delivery logic, where `noReviewer` (now checked here when `Aligner` is set), the liveness hold, the notify switch, the prompt throttle and the cap govern only this fallback.

A `lyx loom review notify` while the agent runs adds a waiting notify as today, and the gate carries nothing while the record is `running`. The notify is therefore held until the agent ends, then counts in the fallback's notify switch.

`pingDue` takes the round: while the record is `running`, its base is the request's `OpenedAt` instead of `DeliveredAt`, so the keep-alive clock runs from the round's opening while the agent works (D4), within the same `maxKeepAlives` cap.
`final` first closes a `running` alignment with `stopped`, so the writer's session never ends beside a live agent.
With `Aligner` set, `final` then checks `noReviewer` only for a round with no verdict, so an agent reject on the latest round fails terminally as a parent reject does, even with an empty `Reviewer`, and D4's bound holds: the agent cannot pass a discussion past a reject.
With `Aligner` nil, `final` keeps its existing order.
`envkit`'s `nilLegal` adds `ParentReview.Aligner` and `ParentReview.RenderAlignmentBrief`, whose nil is the documented "alignment off". `envkit.NilSeams` therefore passes for `FullEnv` and for the loom and batten wiring seam tests, which leave both fields nil whenever alignment is off.
`NewExpiryProbe` and `WaitBound` are unchanged and cover the agent and a fallback review together.

`TestGate_Alignment` is one table test over `fakeAligner`, which counts spawns, polls and closes, returns scripted poll results, and can record a parent verdict into the store from inside `Poll` for the race row. Rows:
- The first evaluation spawns once and carries no prompt or delivery, with `ReviewerLive` false and with an empty `Reviewer` alike.
- An approve passes, `verdict.json` is `approve`, `review.md` is the report's copy, the record is `applied`, `Close` ran once and no prompt was ever carried.
- A reject comes back as the findings result and is consumed. With `Cap` 2, a second round's agent reject fails terminally with the cap's way out.
- An escalate rewrites `brief.md` through `RenderAlignmentBrief` with the report, and the next evaluation carries the delivery prompt.
- A `Done` poll with only a reason (timeout) writes a brief naming the reason, and delivery follows.
- A parent `RecordVerdict(approve)` while the record is `running` closes the agent with `stopped` and passes. The race row, where the parent's verdict lands inside `Poll` before the agent's approve, keeps the parent's verdict.
- An `AddNotify` while `running` is not carried, and after the fallback it is carried as a notify prompt.
- Fresh closures over the same store, standing in for a restart, never call `Spawn` again and apply the next polled approve.
- After a fallback, a fake that would now poll `approve` is never polled: the poll count stays put and delivery continues.
- With `KeepAlive` set and the clock past `OpenedAt` plus `KeepAlive`, a ping is carried while `running`.
- Reaching `WaitBound` while `running`, and calling `final` while `running`, each close the agent and pass.
- With an empty `Reviewer`, `final` on a round whose agent reject is recorded fails terminally.

`doc.go` gains `alignment.json` in the round file list, plus a paragraph on the alignment order and its fallback.
`internal/parentreview`'s budget row has room for the one new test.

**ImpactSummary:** The gate's closures, `pingDue` and `Store.load` change inside `internal/parentreview`, plus two `envkit` nil-legal entries; with `Aligner` nil every existing caller and test sees today's behavior, and only card 7 sets it.
