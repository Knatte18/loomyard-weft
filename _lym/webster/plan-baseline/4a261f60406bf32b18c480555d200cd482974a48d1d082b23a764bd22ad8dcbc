# Card 12 — run-loop

**Create:**
- `plan:internal/weaverengine#Run` -> `func Run(deps RunDeps) (RunResult, error)`
- `plan:internal/weaverengine#RunResult` -> `type RunResult struct`
- `plan:internal/weaverengine#ErrRunBusy` -> `var ErrRunBusy = errors.New("weaver: run is already in progress")`
- `plan:internal/weaverengine#RunActive` -> `func RunActive(scratchDir string) (bool, error)`
- `plan:internal/weaverengine#runBatch` -> `func runBatch(deps RunDeps, st *State, plan *planparser.Plan, batch batcher.Batch, startRung int, reentry bool) (batchEnd, error)`
- `plan:internal/weaverengine#batchEnd` -> `type batchEnd struct`
- `plan:internal/weaverengine#outcomeRecord` -> `type outcomeRecord struct`
- `plan:internal/weaverengine#writeOutcome` -> `func writeOutcome(path string, rec outcomeRecord) error`
- `plan:internal/weaverengine#stopReason` -> `func stopReason(deps RunDeps, st *State, label, cause string) string`
- `plan:internal/weaverengine#TestRun` -> `func TestRun(t *testing.T)`

**Extends:**
- `internal/weaverengine/doc.go`

**Uses:**
- `internal/runplan#ResolveAtDispatch`
- `internal/runplan#WriteConflictRecord`
- `internal/runplan#RemoveConflictRecord`
- `internal/runplan#ConflictDir`
- `internal/runplan#PendingPartition`
- `internal/runplan#RecordBeginPartition`
- `internal/runplan#AcquireStateMutation`

**Intent:** The row's producer body: every batch in Go, then the plan-level verify (D2, D3, D4, D5, D6).

- `Run` holds Weaver's run lock, `run.lock` in the scratch directory taken with `lock.TryAcquireWriteLock`, for its whole call; a held lock returns `ErrRunBusy` wrapped with the way forward: wait for the running row to stop, then re-step it.
  `RunActive` probes the lock and releases it at once, as `websterengine.RunActive` does.
  On entry it removes a stale plan-conflict record from `runplan.ConflictDir(AnchorRoot)`,
  then removes `outcome.yaml` and `summary.md` from Weaver's run directory when they exist, before `resumeStrands` (card 13), as `websterengine.Run` moves its stale outcome and summary aside at entry.
  A re-stepped run therefore holds no `outcome.yaml` until a Done return writes both again.
- Each batch begins under the state-mutation lease: it re-reads the plan, partitions the remaining cards (`runplan.PendingPartition`), records the partition (`RecordBeginPartition`), and runs `runplan.ResolveAtDispatch` for the first fresh batch.
  Weaver's way forward for a conflict names the reroute: the parent fixes the named pending cards through plan review, then re-steps the row; it never names a `lym webster` verb.
  A `*PlanConflictError` writes the conflict record and returns Stuck with no `StopRecord`, so the producer takes the reroute (card 16) and the next call begins afresh from the current plan, a card Plan-Review inserted or removed included.
  It records the batch's card set, card hashes and start commit (HEAD) once, and, on the run's first begin, `PlanWeftCommit` from `WeftHead`.
  A begin also clears any `StopRecord` under the same lease, so a later re-step never re-enters a stop the run has moved past.
  The begin saves the partition, the batch record, `PlanWeftCommit` and the cleared `StopRecord` through `saveRunState` with `leaseHeld` true, then releases the lease.
  A batch's card set and start stay fixed for every attempt.
- Every state save of `Run`, `runBatch` and the resume (card 13) goes through `saveRunState` (card 10), so a delivery the decision verb recorded survives it; every save but the begin's passes `leaseHeld` false.
- `runBatch` runs attempts from `startRung`: it takes the next attempt number, fixes the report path, records the attempt in flight with its rung, marked `Reentry`, with `Stop.Reentered` set in the same save, when `reentry` is set and it is the call's first attempt, renders the prompt (card 8) with the previous attempt's failure digest and uncommitted paths on a retry, starts the strand (`StartGated` with `batchGateSpec`), records its guid and strand name, waits, judges it (`classifyAttempt`) and ends it through `endAttempt` (card 10), whatever the verdict.
  - `recordable` that `endAttempt` records done ends the batch done; one it fails counts as a failed attempt.
  - `failed` climbs to the next rung; a failure on the last rung stops the row.
  - `lost` retries once on the same rung; a second loss counts as a failed attempt.
    `runBatch` counts the lost attempts on the rung from the batch's latest `Reentry` attempt on, or from its first attempt when none is marked, so a re-entry gets its own retry.
    The count is read from recorded attempts, never from a counter of its own call, so a crash and a new call never reset it.
  - `needs_parent` stops the row at once with the report's `need`, without climbing.
  Every attempt that ends without being recorded done has its outcome and cause recorded first, then its strand stopped, before the next attempt starts or the row returns.
  A failed state write still stops the strand and returns Stuck naming the fault.
  A failed stop starts no next attempt and returns Stuck naming the guid and `lym reed remove <guid>`.
- After the last batch, `Run` runs the verify (card 11): a failure spawns one fixer on the next rung through `gatefixer.SpawnOrAttach` and `gatefixer.Await` over `fixerStore`, records the fixer number first, and runs the verify again before any next fixer; a lost fixer gets one same-rung retry after the re-run.
  That allowance counts the lost fixers on the rung from the latest `Reentry` fixer on, or from the first fixer when none is marked, read from the fixer records.
  When `Await` returns a terminal result, `Run` sets the fixer's `Record.Status`, and `DeadReason` when it has one, from that result and saves the record through `fixerStore.Save`, then sets `FixerState.Outcome`, since `Await` never marks the record terminal and `SpawnOrAttach` would otherwise return the ended record as live.
  An `Await` result warning that the fixer's stop failed starts no next fixer and returns Stuck naming the strand's guid and `lym reed remove <guid>`.
  A runner crash stops the row; a failure after the last rung stops the row; a pass writes `summary.md`, writes `outcome.yaml` done through `writeOutcome`, commits the run record and returns Done.
  `outcomeRecord` is `outcome.yaml`'s one shape, with the yaml keys `websterengine`'s `outcome.yaml` holds, so the two engines' files read alike: `outcome` set to `done`, `stuck_reason` empty and `batches_done` the count of batches this call recorded done.
  A failure of that last commit returns Stuck naming it and the re-step as the way forward.
- Each batch stop, needs-parent stop and verify stop sets `st.Stop` to its kind, batch and rung and saves it through `saveRunState` before `Run` returns Stuck, a failed-stop Stuck after a last-rung failure included.
  `runBatch` sets it for a batch or needs-parent stop, `resumeStrands` (card 13) for one it judges from a recorded attempt, and `Run` for a verify stop: a failure after the last rung, a runner crash or a fixer whose stop failed.
  Card 13's `reenter` reads it.
  A plan-conflict Stuck, a fault in Weaver's config or state, and a failed per-batch or final record commit set none, since the batch or verify they follow has already passed.
- `stopReason` renders every stop's reason and writes no state: the batch and its cards or the verify's findings, the plan directory, the state file, the last report, the journal and the decision record, and `ReentryStep`.

`TestRun` is tier0, over a fake `StrandRunner` that writes scripted reports and records each spawn's spec, a fake `Git`, a fake fixer, a fake verify, a recording commit seam and an index fake.
It is the one test of `Run`'s unexported helpers (cards 9 to 11), so it holds their rows:
a three-card plan the batcher splits in two runs two strands in order on the first rung and returns Done after the verify, spawning nothing else, with `outcome.yaml` done, `summary.md` written with its title and no triage or fix section, and the run record committed;
a run directory holding `outcome.yaml` done and `summary.md`, with a plan card no batch records, begun anew, holds neither file while the new batch's attempt runs, and its Done return writes both again;
the first batch, of two cards, carries two `card` gates, then `batch`, then the row's `drift` entry, in that order; the last batch's `batch` call carries the main packages and the first's does not; a gate config lacking `card`, and one lacking `batch`, each refuse naming the config's way forward;
the first begin records `PlanWeftCommit`; a batch begin over a real scratch directory and state file, run under a test deadline, saves its records and starts the batch's attempt, and once it returns `lock.AcquireWriteLockWithin` on the state-mutation lease succeeds, so a nested acquire fails the test naming the lease instead of hanging it; a batch's card set and start commit stay the same across its attempts;
a batch reporting a rename partitions the next batch from the rewritten pending card;
a failed attempt starts the next on the next rung's model; a failure on the last rung returns Stuck naming the batch, state file, report and journal, and the saved state holds `Stop` of kind `batch` with that batch and rung;
a `FAILED` report, a failing gate, an unclean tree (its paths recorded as `Uncommitted` and told to the next attempt), an `OutcomeTimeout` with no report file and an unparseable report each climb, and each failed attempt's journal entry is appended, its end commit recorded and the batch committed;
a `head_sha` that is not HEAD climbs, and an abbreviated `head_sha` and a clean parent merge on top of it are recorded done;
a done-check finding and a removed later target each climb with their reason;
a report recorded twice after a crash appends one journal entry; a failed per-batch commit returns Stuck naming the re-step;
a `Delivered` heading written to the state file during an attempt survives that attempt's end;
a machinery loss, an `OutcomeDied`, an `OutcomeAPIError` and a wait error from reed alike, retries once on the same rung with a new attempt number, and a second loss climbs;
every ended attempt's strand is stopped after its outcome is recorded, also when the state write fails;
a failed stop returns Stuck naming the guid and `lym reed remove` and starts nothing;
`NEEDS_PARENT` returns Stuck with its need at once;
a plan conflict at begin writes the conflict record, records no stop, names no `lym webster` verb and returns Stuck;
a verify failure, failing on its rerun too, writes the verify-failure record and spawns one fixer on the ladder under `fixer_timeout_min` with its findings, and reruns the verify after it, a lost fixer included; a first fixer that ends dead, with the re-run verify still failing, gets its record saved terminal and a second fixer spawns;
a verify that fails then passes on its rerun returns Done, spawns no fixer and lists the flaky identities under `summary.md`'s triage section; a verify timeout is not rerun; a fixer's findings and `fix:` commits are listed under `summary.md`'s fixes section;
a dirty tree hands its paths to the next fixer; a fixer whose stop failed returns Stuck naming its guid and spawns no next fixer; a runner crash spawns no fixer and writes no verify-failure record; a failure after the last rung returns Stuck;
a failed commit of the run record after the verify passes returns Stuck naming the failure and the re-step;
a held run lock returns `ErrRunBusy` naming the re-step.

**ImpactSummary:** Only adds to the new weaverengine package's doc; no existing symbol changes.
