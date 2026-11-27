# Card 20 — dobby-push-summary

**Edit:**
- `internal/dobbyengine#Deps`
- `internal/dobbyengine#State`
- `internal/dobbyengine#daemon.tick`
- `internal/beamcli#hubDobbyDepsStarting`
- `internal/dobbyengine/doc.go`

**Create:**
- `internal/dobbyengine#PushSummaryDeps`
- `internal/dobbyengine#PushPeriod`
- `internal/dobbyengine#daemon.pushSummary`
- `internal/dobbyengine#TestPushSummary`
- `internal/beamcli#TestHubDobbyDeps_PushSummaryWaitsReadBattenAndPairRecords`

**Uses:**
- `internal/beamcli/doc.go`
- `internal/dobbyengine#RunView`
- `internal/runpusher#TargetRefused`
- `internal/pushledger#List`
- `internal/pushledger#Entry`
- `internal/pushledger#Entry.Owed`
- `internal/fabricengine#Config.PushTiming`
- `internal/fabricengine#FailedPushWayForward`
- `internal/battencli#HubRunSlugs`
- `internal/sheddriver#ReadDriveWait`
- `internal/shedrun#DriveWaitRecord`

**Intent:** D3: no mail per failed push; the prime's dobby posts one summary once something stays owed past the threshold, and one all-clear once everything is pushed.

`Deps` gains `PushSummary PushSummaryDeps`: `Ledger`, the hub's entries through `pushledger.List`; `Refused`, a target's refusal through `runpusher.TargetRefused`;
`Waits`, every run of the hub the Go driver waits on, the prime's and each beam's alike, with its producer and `Since`;
and `Threshold`, `fabric.push_notice_after_min` through `Config.PushTiming`.
`hubDobbyDepsStarting` fills `Waits` from `battencli.HubRunSlugs` and `sheddriver.ReadDriveWait`, a wait standing under a held lock,
over each pair's wait record and driver lock and over each batten run's own record and driver lock in the prime (card 2), so a batten run waiting at Land is listed.
A nil `Ledger`, as every beam's dobby and a hand-built `Deps` leave it, posts nothing; only the prime's dobby is wired, in `hubDobbyDepsStarting`.
`State` gains `PushPeriod`: the start of the current owed period and whether its summary was posted.
`daemon.tick` calls `pushSummary` each tick on the prime.

The owed period starts at the earliest of the owed entries' `FirstOwed` and the `Waits` read's `Since`, and lasts while any `Entry.Owed` entry or waiting run stands.
An entry holding only `HeldRecords` or `Dropped` is listed in a summary as held, until its pair goes or its commit leaves the parent, not as owed: it neither opens nor holds an owed period and does not block the all-clear.
An entry holding landings whose `Worktree` no longer stands, which `PruneLedger` keeps (card 6), is listed owed with its worktree named gone, so a pair removed outside lym shows in the summary instead of holding the period open unseen.
Once it has lasted `Threshold`, `pushSummary` posts the hub orch, through `Deps.Notice`, one notice listing every owed branch with the minutes it has been owed, a refused one named refused with its way forward,
and every run the Go driver waits on with its producer; no further summary follows in that period.
Once no entry is owed and no run waits after such a notice, it posts one notice that every branch is pushed and ends the period.
`internal/dobbyengine/doc.go` gains a paragraph on the prime's push summary: the threshold, the one summary per owed period, the held entries and the all-clear.
A refused target keeps its entry while its record stands, so it keeps the period open and blocks the all-clear; once the record's way forward clears it, the next tick drops the entry (card 6), and the all-clear posts.
A failed post is retried on the next tick, logged once per episode.
An owed push still writes no failed-push record (card 5), so issue #238's per-push mail is gone.

`TestPushSummary`, tier0, with an injected clock over a real ledger under `t.TempDir()` and a recording `Notice`, adding about a second to one tier0 run, pins:
one summary listing every owed branch and the waiting run once a target has stayed owed for the threshold, and none before it;
one summary for a run that kept a wait record for the threshold with an empty ledger;
one summary naming a waiting run that the prime's dobby does not list, a beam member's, from the `Waits` read alone;
no second summary in the same owed period; one all-clear once the ledger empties;
one all-clear while an entry holding only `HeldRecords`, or only `Dropped`, stands, which the summary listed as held;
an entry holding a landing whose worktree is gone listed with its worktree named gone;
a refused target's entry named refused and blocking the all-clear, and the all-clear posted on the tick after its entry is dropped;
and a summary whose post fails posted on the next tick, once.
`TestHubDobbyDeps_PushSummaryWaitsReadBattenAndPairRecords`, tier1, over a hub fixture with a batten run and its pair, adding about two seconds to one tier1 run, pins that the wired `Waits` lists the batten run whose record stands in the prime under its held driver lock, with producer `Land`,
lists a pair's run whose record stands under its held lock, and lists neither once its lock is free.

**ImpactSummary:** Adds one per-tick step to the prime's dobby and one field to its state record; beams' dobbies are unchanged.
