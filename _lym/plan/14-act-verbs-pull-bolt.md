# Card 14 — act-verbs-pull-bolt

**Create:**
- `plan:internal/pairteardown#ReadRunPulled` -> `func ReadRunPulled(prime *lymcwd.Location, slug string) (runfsm.Records, runfsm.Inputs, error)`
- `plan:internal/pairteardown_test#TestReadRunPulled` -> `func TestReadRunPulled(t *testing.T)`

**Edit:**
- `internal/loomcli#hubRunVerdict`
- `internal/tendercli#hubRunIntents`

**Extends:**
- `internal/pairteardown/doc.go`

**Uses:**
- `internal/boardengine#PullBolt`
- `internal/pairteardown#ReadRun`
- `internal/pairteardown_test#TestReadRun_ReadsTheSameStateWithoutTheCache`
- `internal/boardengine#WriteHold`

**Intent:** Make every verb that asks the act rule outside the dobby's pass fetch Bolt first when origin answers, so a hold another machine wrote is read (D5).

`ReadRunPulled(prime, slug)` pulls the hub's Bolt through `boardengine.PullBolt`, then returns `ReadRun(prime, slug)`.
An unreachable origin (`ErrOwnerOriginUnreachable`) reads the local Bolt and goes on.
Any other failed pull returns an error naming the pull's detail and the way forward: bring the board level as it names, then run the verb again; nothing is read.
`loomcli`'s `hubRunVerdict` (behind `lym loom start` and `lym loom resume`) and the `Verdict` seam of `tendercli`'s `hubRunIntents` (behind `lym tender resume <slug>`) read through it in place of `ReadRun`.
`lym batten cancel` already pulls first, and `lym batten run` reads origin through its catch-up (card 20).
Card 15 later sets the hold aside in tender resume's verdict and reads the hold through its own `BoltHold` seam; the pull then keeps that verdict's owner reading fresh.

`TestReadRunPulled`, tier1, runs over the hub fixture `TestReadRun_ReadsTheSameStateWithoutTheCache` builds, with a second clone of the hub's Bolt origin:
a hold the second clone writes through `WriteHold` after this hub's last pull reads `Held` with that clone's machine;
with the Bolt's origin URL pointing nowhere, the read goes on over the local Bolt;
a Bolt with an uncommitted change, which the pull skips as dirty, returns the error naming the way forward and reads nothing.
The two verdict readers are thin wiring over it, so their own tests stay as they are.

**ImpactSummary:** One new pairteardown reader with two production callers that swap ReadRun for it; their outputs are unchanged apart from reading a fresher board.
