# Card 13 — run-reentry-and-resume

**Create:**
- `plan:internal/weaverengine#resumeStrands` -> `func resumeStrands(deps RunDeps, st *State) error`
- `plan:internal/weaverengine#reenter` -> `func reenter(deps RunDeps, st *State, plan *planparser.Plan) (reentry, error)`
- `plan:internal/weaverengine#reentry` -> `type reentry struct`
- `plan:internal/weaverengine#reentryWayForward` -> `const reentryWayForward = "put the plan's card set back as the recorded partition holds it, or run lym weaver reset to start the run over from the plan it started with"`
- `plan:internal/weaverengine#TestRun_Reentry` -> `func TestRun_Reentry(t *testing.T)`
- `plan:internal/weaverengine#TestRun_Resume` -> `func TestRun_Resume(t *testing.T)`

**Extends:**
- `internal/weaverengine/doc.go`

**Uses:**
- `internal/runplan#MapPartition`
- `internal/runplan#RestampBaseline`
- `internal/runplan#PostBatchChecks`
- `internal/shuttleengine#Runner.ProbeGated`

**Intent:** What a new call of the row does after a stop or a crash (D7, D12), run by `Run` before it starts anything.

- `resumeStrands` runs first:
  1. It stops the strand of every attempt and fixer whose outcome state records, through `StopStrand`; a failed stop returns Stuck naming the guid and `lym reed remove <guid>`, and starts nothing.
  2. For every attempt recorded in flight it calls `ProbeGated` with the attempt's report path and gates, which finds the run by its output file even when state never recorded its guid.
     A found run is a handle not yet waited on: Weaver records its guid and strand name (from shuttle's run record), sends the strand, through `Deliver`, the decision entries its `Delivered` lacks and records the headings sent, then waits on the handle and judges the result with `classifyAttempt`, recording a report only from that result.
     A not-found answer, with the report on disk or not, carries no gate verdict and counts as a machinery loss (D6), never done; `ProbeGated` has already removed the strand of a respawn-eligible run.
  3. A fixer whose `gatefixer` record is live is attached by `gatefixer.SpawnOrAttach`, which returns the record before any wait; Weaver delivers its strand the undelivered entries the same way, then awaits it.
     A fixer number state records with no record is probed through `ProbeGated` with its reply path; a run it finds is stopped through the handle's `Stop`; the fixer counts as a machinery loss and the verify runs again before any new fixer.
  A `ProbeGated` error returns Stuck naming the error and shuttle's way forward, and spawns nothing.
- `reenter` runs when `StopRecord` is set; a call after a plan-conflict Stuck finds none and begins as card 12 does.
  It re-reads the plan; when every recorded card is a plan card and every plan card is in a recorded batch, it restamps the plan baseline and maps the recorded partition onto the plan (`runplan.MapPartition`), so an edit to a card's text, the stopped batch's included, carries into the next attempt; otherwise it refuses with `reentryWayForward`.
  - After a `needs_parent` stop the next attempt starts on the rung that asked, with the ladder above it open, and never takes the no-spawn path.
  - After any other batch stop it runs the post-batch checks (card 10) on the stopped batch against its recorded start commit; a pass with a clean tree records the batch done with no spawn, unless the row has `gates` entries.
    Otherwise the batch gets one attempt on the last rung, which retries once on the same rung after a machinery loss; its failure stops the row again.
  - After a verify stop it runs the verify again, and a failure gets one fixer on the last rung before the row stops again.

`TestRun_Reentry` is tier0: a passing stopped batch records done with no spawn, also after the arbiter edits a pending card; a passing batch on a dirty tree is not recorded done and gets the last-rung attempt; a failing one, and one whose row has `gates`, gets one last-rung attempt carrying an edit to its own card's text; a lost re-entry attempt retries once on the last rung with a new attempt number;
a plan that lost a recorded card or gained an unrecorded one refuses naming both ways forward; a re-step after a plan-conflict Stuck, with a card Plan-Review inserted, begins normally;
a `NEEDS_PARENT` re-entry starts on the asking rung and never records done with no spawn;
a verify re-entry reruns the verify, and a failure spawns one last-rung fixer and stops again.
`TestRun_Resume` is tier0: a recorded-outcome strand is stopped first, and a failed stop returns Stuck naming its guid and spawns nothing; an in-flight attempt is found by its report path with no guid recorded and its result recorded; a not-found probe with the report on disk, and one without it, are each a machinery loss; the probed strand gets the undelivered decisions once, before the wait; no second agent spawns for one attempt; a strand re-prompted on a failing gate is awaited and not recorded done until its gates pass; a live fixer record is attached and delivered to; a fixer with no record has its probed strand stopped and the verify rerun; a probe error returns Stuck and spawns nothing; a retry and a re-entry attempt each take a new attempt number.

**ImpactSummary:** Only adds to the new weaverengine package's doc; no existing symbol changes.
