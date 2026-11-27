# Card 20 — dobby-push-summary

**Edit:**
- `internal/dobbyengine#Deps`
- `internal/dobbyengine#State`
- `internal/dobbyengine#daemon.tick`
- `internal/beamcli#hubDobbyDepsStarting`

**Create:**
- `plan:internal/dobbyengine#PushSummaryDeps` -> `type PushSummaryDeps struct`
- `plan:internal/dobbyengine#PushPeriod` -> `type PushPeriod struct`
- `plan:internal/dobbyengine#daemon.pushSummary` -> `func (d *daemon) pushSummary(ctx context.Context, now time.Time)`
- `plan:internal/dobbyengine#TestPushSummary` -> `func TestPushSummary(t *testing.T)`

**Uses:**
- `internal/dobbyengine/doc.go`
- `internal/beamcli/doc.go`
- `internal/dobbyengine#RunView`
- `plan:internal/runpusher#TargetRefused`
- `plan:internal/pushledger#List`
- `plan:internal/pushledger#Entry`
- `plan:internal/fabricengine#Config.PushTiming`
- `internal/fabricengine#FailedPushWayForward`

**Intent:** D3: no mail per failed push; the prime's dobby posts one summary once something stays owed past the threshold, and one all-clear once everything is pushed.

`Deps` gains `PushSummary PushSummaryDeps`: `Ledger`, the hub's entries through `pushledger.List`; `Refused`, a target's refusal through `runpusher.TargetRefused`; and `Threshold`, `fabric.push_notice_after_min` through `Config.PushTiming`.
A nil `Ledger`, as every beam's dobby and a hand-built `Deps` leave it, posts nothing; only the prime's dobby is wired, in `hubDobbyDepsStarting`.
`State` gains `PushPeriod`: the start of the current owed period and whether its summary was posted.
`daemon.tick` calls `pushSummary` each tick on the prime.

The owed period starts at the earliest of the ledger entries' `FirstOwed` and the listed runs' `WaitSince` (card 3), and lasts while any entry or waiting run stands.
Once it has lasted `Threshold`, `pushSummary` posts the hub orch, through `Deps.Notice`, one notice listing every owed branch with the minutes it has been owed, a refused one named refused with its way forward,
and every run the Go driver waits on with its producer; no further summary follows in that period.
Once the ledger is empty and no run waits after such a notice, it posts one notice that every branch is pushed and ends the period.
A refused target keeps its entry while its record stands, so it keeps the period open and blocks the all-clear; once the record's way forward clears it, the next tick drops the entry (card 6), and the all-clear posts.
A failed post is retried on the next tick, logged once per episode.
An owed push still writes no failed-push record (card 5), so issue #238's per-push mail is gone.

`TestPushSummary`, tier0, with an injected clock over a real ledger under `t.TempDir()` and a recording `Notice`, adding about a second to one tier0 run, pins:
one summary listing every owed branch and the waiting run once a target has stayed owed for the threshold, and none before it;
one summary for a run that kept a wait record for the threshold with an empty ledger;
no second summary in the same owed period; one all-clear once the ledger empties;
a refused target's entry named refused and blocking the all-clear, and the all-clear posted on the tick after its entry is dropped;
and a summary whose post fails posted on the next tick, once.

**ImpactSummary:** Adds one per-tick step to the prime's dobby and one field to its state record; beams' dobbies are unchanged.
