# Card 13 — run-reentry-and-resume

**Create:**
- `internal/weaverengine#resumeStrands`
- `internal/weaverengine#resumeResult`
- `internal/weaverengine#reenter`
- `internal/weaverengine#reentry`
- `internal/weaverengine#checkStoppedBatch`
- `internal/weaverengine#recordDoneWithoutSpawn`
- `internal/weaverengine#reentryWayForward`
- `internal/weaverengine_test#TestRun_Reentry`
- `internal/weaverengine_test#TestRun_Resume`

**Edit:**
- `internal/weaverengine#Run`

**Extends:**
- `internal/weaverengine/doc.go`

**Uses:**
- `internal/runplan#MapPartition`
- `internal/runplan#RestampBaseline`
- `internal/runplan#PostBatchChecks`
- `internal/runplan#BatchDepartures`
- `internal/runplan#IsLastBatch`
- `internal/shuttleengine#Runner.ProbeGated`
- `internal/weaverengine#batchGateCalls`

**Intent:** What a new call of the row does after a stop or a crash (D7, D12), run by `Run` before it starts anything.

- `Run`, after taking its run lock and removing the stale plan-conflict record, `outcome.yaml` and `summary.md` (card 12), calls `resumeStrands` and then, when `StopRecord` is set, `reenter`, before it begins any batch.
- `resumeStrands` runs first:
  1. It stops the strand of every attempt and fixer whose outcome state records, through `StopStrand`; a failed stop returns Stuck naming the guid and `lym reed remove <guid>`, and starts nothing.
  2. For every attempt recorded in flight it calls `ProbeGated` with the attempt's report path and gates, which finds the run by its output file even when state never recorded its guid.
     A found run is a handle not yet waited on: Weaver records its guid and strand name (from shuttle's run record), sends the strand, through `Deliver`, the decision entries its `Delivered` lacks and records the headings sent, then waits on the handle and judges the result with `classifyAttempt`, recording a report only from that result.
     A not-found answer, with the report on disk or not, carries no gate verdict and counts as a machinery loss (D6), never done; `ProbeGated` has already removed the strand of a respawn-eligible run.
  3. A fixer whose `gatefixer` record is live is attached by `gatefixer.SpawnOrAttach`, which returns the record before any wait; Weaver delivers its strand the undelivered entries the same way, then awaits it.
     A fixer number state records with no record is probed through `ProbeGated` with its reply path; a run it finds is stopped through the handle's `Stop`; the fixer counts as a machinery loss and the verify runs again before any new fixer.
  A `ProbeGated` error returns Stuck naming the error and shuttle's way forward, and spawns nothing.
  4. It returns a `resumeResult`, which `Run` hands to `runBatch`: whether a begun batch with no terminal record is to continue, and the rung its next attempt starts on, judged from the batch's last attempt, whether a crash interrupted it in flight or after its outcome was recorded.
     A recordable attempt ends its batch done through `endAttempt`, and nothing continues.
     A failed attempt starts the next attempt on the next rung; a failure on the last rung, or a `needs_parent` attempt, stops the row as `runBatch` does.
     A lost attempt, a not-found probe included, retries once on its own rung unless the batch has another lost attempt on that rung, counted from its latest `Reentry` attempt on, or from its first attempt when none is marked; else it counts as failed.
     The count is read from recorded attempts, so a crash never resets it.
- A begin that finds a non-terminal batch record for its first fresh batch reuses that record's card set and start commit, and starts `runBatch` at the rung `resumeResult` or `reentry` names, with `reentry` set when a `reentry` names the batch and `Stop.Reentered` is unset, so only the stop's first re-entry attempt is marked `Reentry`.
  A fresh partition whose first batch differs from the recorded one refuses naming `reentryWayForward`.
- `reenter` runs when `StopRecord` is set; a call after a plan-conflict Stuck finds none and begins as card 12 does.
  It re-reads the plan; when every recorded card is a plan card and every plan card is in a recorded batch, it restamps the plan baseline and maps the recorded partition onto the plan (`runplan.MapPartition`), so an edit to a card's text, the stopped batch's included, carries into the next attempt; otherwise it refuses with `reentryWayForward`.
  It returns a `reentry` that `Run` branches on: the batch to run, the rung its attempt starts on, and whether the verify stage is next, set when no batch is left to run.
  - After a `needs_parent` stop the next attempt starts on the rung that asked, with the ladder above it open, and never takes the no-spawn path.
  - After any other batch stop it calls `checkStoppedBatch`, which runs `checkCleanTree` (card 10) and `runplan.PostBatchChecks` with no renames from the batch's recorded start commit to HEAD, and returns the deviations and the failing reasons.
    It checks no report's `head_sha`, since a repair the parent or arbiter committed after the stop moves HEAD past the last report.
    When it returns no reason, it runs at HEAD exactly the calls `batchGateCalls` returns for the batch's cards, each through `RunDeps.Verify` (`verifytree.Verify` in held gate slots with the geometry's pause hold),
    with `last` taken from `runplan.IsLastBatch` over the recorded partition mapped onto the current plan.
    It records the batch done with no spawn, through `recordDoneWithoutSpawn`, only when every gate passes, and never when the row has `gates` entries.
    `recordDoneWithoutSpawn` marks the batch record terminal done with no digest, since `runplan.Distill` needs a report, but with the deviations `checkStoppedBatch` returned, its departures (`runplan.BatchDepartures`) and card trail as `endAttempt` records them; it sets `BatchEnds` and `LastEnd` from HEAD, clears `StopRecord`, saves through `saveRunState` and commits through `commitRecords`.
    Otherwise, a failing gate included, the batch gets one attempt on the last rung, which retries once on the same rung after a machinery loss; its failure stops the row again.
  - After a verify stop it runs the verify again, and a failure gets one fixer on the last rung, marked `Reentry` with `Stop.Reentered` set in the same save while `Stop.Reentered` is unset, before the row stops again; a lost one retries once on that rung as card 12 counts it.
  - `StopRecord` is cleared and saved through `saveRunState` the moment any batch is recorded done, by `endAttempt` (card 10) or by `recordDoneWithoutSpawn`, and the moment the verify re-run passes, so a later re-step runs no second re-entry, also when a re-entered attempt ends done after `reenter` has returned.

`TestRun_Reentry` is tier0: a passing stopped batch whose card and batch gates pass at HEAD records done with no spawn, also after the arbiter edits a pending card; a passing batch on a dirty tree is not recorded done and gets the last-rung attempt; a batch whose post-batch checks pass but whose card gate fails at HEAD is not recorded done and gets the last-rung attempt;
a re-step after a successful re-entry, followed by a plan-conflict Stuck, runs no second re-entry; a failing one, and one whose row has `gates`, gets one last-rung attempt carrying an edit to its own card's text; a batch stopped after two lost attempts on the last rung, then re-entered, whose re-entry attempt is lost, retries once on the last rung with a new attempt number and stops only on a second loss; a crash during that retry marks no second `Reentry` attempt, so its loss stops the row;
a plan that lost a recorded card or gained an unrecorded one refuses naming both ways forward; a re-step after a plan-conflict Stuck, with a card Plan-Review inserted, begins normally;
a `NEEDS_PARENT` re-entry starts on the asking rung and never records done with no spawn;
a verify re-entry reruns the verify, and a failure spawns one last-rung fixer and stops again; a verify stopped after two lost last-rung fixers, then re-entered, retries its lost re-entry fixer once on the last rung;
a stopped batch that is the plan's last, whose last-rung attempt ends done, with the verify passing and the final record commit failing, returns Stuck, and the next call begins no second re-entry.
`TestRun_Resume` is tier0: a recorded-outcome strand is stopped first, and a failed stop returns Stuck naming its guid and spawns nothing; an in-flight attempt is found by its report path with no guid recorded and its result recorded; a not-found probe with the report on disk, and one without it, are each a machinery loss; the probed strand gets the undelivered decisions once, before the wait; no second agent spawns for one attempt; a strand re-prompted on a failing gate is awaited and not recorded done until its gates pass; a live fixer record is attached and delivered to; a fixer with no record has its probed strand stopped and the verify rerun; a probe error returns Stuck and spawns nothing;
a crash after a failed attempt on rung 0 runs the next attempt on rung 1 with the recorded card set and start commit; a crash after a failed last-rung attempt returns Stuck with `Stop` of kind `batch`, that batch and rung saved;
a crash after a lost attempt retries once on the same rung, and a second loss on that rung climbs.

**ImpactSummary:** Only adds to the new weaverengine package's doc; no existing symbol changes.
