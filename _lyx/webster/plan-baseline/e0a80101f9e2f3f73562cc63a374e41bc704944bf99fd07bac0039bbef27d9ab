# Card 7 — bouncer-usage-entries

**Create:**
- `plan:internal/shedadapters#newPassUsage` -> `func newPassUsage(model burlerengine.ModelChoice, result shuttleengine.Result) halfUsage`
- `plan:internal/shedadapters#Bouncer.recordPass` -> `func (b *Bouncer) recordPass(round int, seed bool, result shuttleengine.Result)`
- `internal/shedadapters/bouncer_usage_test.go`
- `plan:internal/shedadapters#TestBouncer_UsageRecord` -> `func TestBouncer_UsageRecord(t *testing.T)`

**Edit:**
- `internal/shedadapters#Bouncer.runSeedSpawn`
- `internal/shedadapters#Bouncer.judgeCall`
- `internal/shedadapters#Bouncer.Call`
- `internal/shedadapters#Bouncer.awaitLiveJudge` -> `func (b *Bouncer) awaitLiveJudge(round int) (shuttleengine.Result, bool, error)`
- `internal/shedadapters#Bouncer.awaitLiveSeed` -> `func (b *Bouncer) awaitLiveSeed() (shuttleengine.Result, bool, error)`
- `internal/shedadapters#TestBouncer_Clear_ApprovedRunDirClearsAndReseeds`
- `cmd/lyx/testdata/test-budget.yaml`

**Prosa:**
- `internal/shedadapters/doc.go`

**Intent:** Record every seed and judge pass in the round's usage record, as D10 decides.
`newPassUsage` builds one entry from the pass's model and effort and the shuttle result's start, end and `Usage`, in the half shape with wall seconds set only when both times are set and token fields set only when the reading is known; the fork fields stay unset.
`recordPass` appends the entry to round 1's `seed` list when `seed` is true, with `b.cfg.Models.Seed`, and otherwise to the given round's `judge` list with `b.cfg.Models.Judge`, through `mergeRoundUsage` with `b.cfg.Now`; a failed write only logs a warning and never changes the Call's outcome.
`runSeedSpawn` calls it once a result is in hand, from an attach or from a `Run` that returned no error, before the outcome check; `judgeCall` calls it the same way, before the harvest, so a judgment the guard refuses is still recorded and a re-judged round shows every pass it took.
The two entry-time probes of `Call` attach to a pass too, and its result reaches this Call: `awaitLiveJudge` and `awaitLiveSeed` return the attached `shuttleengine.Result` beside the attach flag, their doc comments no longer saying they carry no result out, and `Call` calls `recordPass` on an attach, the judge's for the round it probed before its harvest or fall-through, and the re-bounce seed's for round 1.
A pass whose `Run` returned an error records nothing, and a judgment a later Call settles from disk records nothing, since no result reached that Call.
The seed entry of a cleared-and-re-seeded generation lands in the recreated run directory's round-1 record by construction, because the clear archives the whole directory before the seed spawns; `TestBouncer_Clear_ApprovedRunDirClearsAndReseeds` gains an assertion that the fresh record holds exactly one seed entry and the archived directory's record is untouched.
`TestBouncer_UsageRecord` is one table in the new file `bouncer_usage_test.go`, which pins the Bouncer's usage-record contract that no existing test file covers: a seed pass writes round 1's `seed` entry with the seed model and effort, times, wall seconds and known tokens; a judge pass of round N appends to round N's `judge` list and leaves a Burler-written review half in place; a refused CONVERGED followed by a re-judge leaves two judge entries; a judge harvested by the entry-time attach records one judge entry for its round, and a seed attached on the re-bounce records one seed entry; a result the provider could not read records `tokens_known: false` and no token fields; a `Run` error records nothing; a record the Bouncer cannot write, such as a directory at the record's path, leaves the Call's outcome and pointer unchanged.
Raise the `internal/shedadapters` row of `test-budget.yaml` so the package's count, which this card and card 5 each grow by one, stays within it.
In `doc.go`, extend the sentence describing `round-<N>-usage.yaml` with the `seed` and `judge` lists, the merge-on-write rule and that a record without a review half reads as missing.

**ImpactSummary:** Two new helpers, call sites in the Bouncer's two spawn paths and its two entry-time probes, and two probes re-signed whose one caller is `Bouncer.Call`; the judge and seed passes gain a usage write whose failure is logged and never alters an outcome.
