---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lym/plan/12-run-loop.md:42
    summary: The same-rung retry allowance read from a batch's recorded attempts on the rung gives a re-entry attempt no retry after two losses on that rung
    fix: >-
      Make the allowance count only the lost attempts of the rung that belong to the current ladder cycle,
      not every lost attempt ever recorded on the rung.
      In card 5, add `Reentry bool` to `AttemptState`, set on the first attempt a re-entry (card 13's `reenter`, after a batch stop or a needs-parent stop) starts.
      In card 12's `runBatch` bullet and card 13's `resumeStrands` step 4, replace "has no earlier lost attempt on that rung" and
      "reads that allowance from the batch's recorded attempts on the rung" with
      "counts the lost attempts on the rung from the batch's latest `Reentry` attempt on, or from its first attempt when none is marked";
      the count is still read from recorded attempts, so a crash never resets it.
      Say the same for a verify re-entry's last-rung fixer, whose `FixerState` takes the same marker.
      Add a `TestRun_Reentry` row: a batch stopped after two lost attempts on the last rung, then re-entered, whose re-entry attempt is lost, retries once on the last rung and stops only on a second loss.
  - id: F2
    severity: MEDIUM
    class: scope
    location: _lym/plan/12-run-loop.md:29
    summary: Run never clears the previous run's outcome.yaml or summary.md on entry, so merge-in reads a re-stepped run as not in flight
    fix: >-
      In card 12's `Run` entry bullet, after the run lock is held and before `resumeStrands`, remove `outcome.yaml` and `summary.md` from Weaver's run directory when they exist,
      as `websterengine.Run` archives its stale outcome and summary at entry.
      A Done return rewrites both.
      Add a `TestRun` row: a run directory holding `outcome.yaml` done and a plan with a card no batch records, begun anew, leaves `weaverengine.RunInFlight` true and no `outcome.yaml` until the row returns Done.
      Say in card 23's Intent that `RunInFlight` relies on this entry removal.
  - id: F3
    severity: MEDIUM
    class: design
    location: _lym/plan/13-run-reentry-and-resume.md:51
    summary: checkStoppedBatch runs the card and batch gate calls batchGateSpec renders, but no card target renders those calls apart from the strand closures
    fix: >-
      In card 9, add the Create target `plan:internal/weaverengine#batchGateCalls` -> `func batchGateCalls(geom Geometry, cards []planparser.Card, last bool) ([]string, error)`,
      which returns the `card` calls in card order and then the `batch` call, with the module's main packages when `last` is set.
      Say `batchGateSpec` wraps these calls in its closures and appends the row's `gates` entries.
      In card 13, add `plan:internal/weaverengine#batchGateCalls` to Uses and say `checkStoppedBatch` runs exactly those calls through `RunDeps.Verify`,
      with `last` taken from `runplan.IsLastBatch` over the recorded partition mapped onto the current plan.
      Checked with `lym loom validate-plan --overlay`, which reports no new blocking finding against the baseline.
  - id: F4
    severity: LOW
    class: consistency
    location: _lym/plan/23-merge-in-warning.md:25
    summary: outcome.yaml's writer and reader each restate the key set and no card names a shared type or writer function
    fix: >-
      In card 12, add the Create targets `plan:internal/weaverengine#outcomeRecord` -> `type outcomeRecord struct`
      (yaml keys `outcome`, `stuck_reason`, `batches_done`)
      and `plan:internal/weaverengine#writeOutcome` -> `func writeOutcome(path string, rec outcomeRecord) error`,
      and say `Run` writes `outcome.yaml` through `writeOutcome`.
      In card 23, add `plan:internal/weaverengine#outcomeRecord` to Uses and say `RunInFlight` decodes with that type, rejecting unknown keys.
      Drop the sentence "It decodes exactly the keys card 12 writes".
      Checked with `lym loom validate-plan --overlay` together with F3's fix, which reports no new blocking finding against the baseline.
  - id: F5
    severity: LOW
    class: scope
    location: _lym/plan/12-run-loop.md:53
    summary: No card says which code sets StopRecord and saves it before the row returns Stuck, so a stop's re-entry depends on an unowned write
    fix: >-
      In card 12, say that each batch stop, needs-parent stop and verify stop sets `st.Stop` (kind, batch, rung) and saves it through `saveRunState` before `Run` returns Stuck,
      a failed-stop Stuck after a last-rung failure included, and that a plan-conflict Stuck and a Weaver config or state fault set none.
      Add a `TestRun` row asserting the saved state of a last-rung failure holds `Stop` with the batch and rung, which card 13's `reenter` then reads.
---

### [BLOCKING] Re-entry attempt after two last-rung losses gets no same-rung retry

**Location:** `_lym/plan/12-run-loop.md:42`, `_lym/plan/13-run-reentry-and-resume.md:41`

**Issue:** The decision record's D6 and its acceptance criteria give a machinery-loss retry to every attempt, a re-entry attempt included.
Card 12 reads the allowance "from the batch's recorded attempts on the rung", and card 13 grants the retry "when the batch has no earlier lost attempt on that rung".
Take a batch on the last rung whose attempt is lost, whose retry is lost too, and which therefore counts as a failed attempt and stops the row (D5, D6).
D7 re-enters it with one last-rung attempt.
The batch already holds two lost attempts on that rung, so the re-entry attempt, if lost, counts as failed at once.
The same holds after a `NEEDS_PARENT` stop whose rung had an earlier lost attempt.
Card 13's `TestRun_Reentry` row, "a lost re-entry attempt retries once on the last rung with a new attempt number", passes only when the stop came from a failed attempt, so no row reaches this case.
No `AttemptState` field marks where a re-entry's cycle begins, so the allowance cannot tell a fresh cycle from the stopped one.
A verify re-entry's last-rung fixer has the same shape, since card 12 gives a lost fixer its retry from the fixer records.

### [MEDIUM] Stale outcome.yaml and summary.md survive a re-step

**Location:** `_lym/plan/12-run-loop.md:29`

**Issue:** D13 has `lym fabric merge-in` count a Weaver run in flight while its state file exists and `outcome.yaml` is absent or not done.
Card 12 writes `outcome.yaml` done on Done and removes nothing on entry.
Card 14 archives the file only in `lym weaver reset` and the PR rework archive.
After Done, a reroute can re-step the row, for example through a Plan-Burler-Review that adds cards, as D13's rationale names, or an arbiter goto.
The row then begins new batches, or reruns the verify with fixers, while the old `outcome.yaml` still reads done, so merge-in's warning is silent for a run in flight.
`websterengine.Run` archives both its stale `outcome.yaml` and its stale summary at entry (`internal/websterengine/runlevel.go`, the `archiveStaleOutcome` call), and Weaver's plan omits the same step.

### [MEDIUM] Stopped-batch gate calls have no card target

**Location:** `_lym/plan/13-run-reentry-and-resume.md:51`

**Issue:** The re-entry no-spawn path must run each card's `card` gate call and the batch's `batch` gate call at HEAD, "the calls `batchGateSpec` renders minus the row's `gates`".
Card 9's `batchGateSpec` returns a `shuttleengine.GateSpec` of closures, and no card target yields the calls on their own.
`checkStoppedBatch` would render them a second time, including the main-packages variant for the last batch, and the two renderings could drift.
Card 13's `Uses` also lists nothing it takes the gate-call rendering from.
The `last` flag `checkStoppedBatch` needs has no stated source either.

### [LOW] outcome.yaml has two independent descriptions of one key set

**Location:** `_lym/plan/23-merge-in-warning.md:25`

**Issue:** Card 12 writes `outcome`, `stuck_reason` and `batches_done`, and card 23's `RunInFlight` decodes "exactly the keys card 12 writes".
Both cards state the same keys in prose, so the focus question is met on paper.
No target names the writer or a record type, so the two sides are held in step by a sentence.
A key added on one side breaks `RunInFlight`'s unknown-key refusal at run time and no card's test catches it (PATTERN-code-conventions: a name that can go stale is one named constant).

### [LOW] StopRecord has no owning writer

**Location:** `_lym/plan/12-run-loop.md:53`

**Issue:** Cards 5, 10 and 13 say where `StopRecord` is cleared.
Card 12 says only that `stopReason` renders the reason and that `StopRecord` "records a batch, needs-parent or verify stop's kind, batch and rung".
No sentence says which code sets it, or that the state is saved before `Run` returns Stuck.
If the Stuck return skips the save, the next call finds no `StopRecord`, `reenter` never runs, and D7's no-spawn path and its last-rung attempt are lost.
`stopReason` is described as a renderer, so setting the record there would hide a state write in a string builder.
The same gap leaves open whether a failed-stop Stuck after a last-rung failure records the stop.

## Rejected

- Focus 1, cards 10 and 12, `saveRunState` re-acquiring the lease inside a lease-holding section: no such path exists.
  Only the batch begin holds the lease and it passes `leaseHeld` true.
  Every other caller (`endAttempt`, the resume, `recordDoneWithoutSpawn`) runs after the begin releases it.
  `runplan.ResolveAtDispatch` and `RestampBaseline` take no lease (`internal/runplan/conflict.go`, `fingerprint.go`).
  `AcquireStateMutation` is a blocking `lock.AcquireWriteLock` (`internal/runplan/state.go:37`), and card 12's begin row runs under a test deadline against a real scratch directory and asserts the lease is free afterwards, so a nested acquire fails the test.
- Focus 2, `StopRecord` left set by a re-entered attempt that ends done, or by the plan's last batch: card 10's `endAttempt` clears it on every recordable pass, card 13 clears it in `recordDoneWithoutSpawn` and on a passing verify re-run, and card 13's last row covers a last batch whose final record commit fails.
  The only gap found is the unowned write in the last finding.
- Focus 3, functions called by the re-entry and resume paths: `stopAttempt`, `classifyAttempt`, `StrandRunner.ProbeGated`, `endAttempt`, `checkCleanTree`, `recordDoneWithoutSpawn` and `Deliver` are each a target of cards 9, 10, 11, 13 or 21.
  The resumed or re-entered batch continues with the recorded card set and start commit (card 13, begin reuse) on the rung `resumeResult` or `reentry` names.
  The gaps are the gate-call rendering and the retry allowance above.
- Focus 4, `weaverengine.Run` wiring: card 18 fills `Env.WeaverRun` in `loomCLI.wire` with `weaverengine.Run` explicitly, beside `WebsterRun` (`internal/loomcli/wiring.go:853`), `weaverEntry` refuses a nil one, and `loomshed.NewWeaverProducer` (card 17) is the constructor card 18 returns.
  `RunActive`, `RunInFlight` and `Reset` have non-test callers in cards 14, 15 and 23, so PATTERN-test-only-code holds.
