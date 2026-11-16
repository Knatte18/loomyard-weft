# Card 12 — run-loop

**Create:**
- `plan:internal/weaverengine#Run` -> `func Run(deps RunDeps) (RunResult, error)`
- `plan:internal/weaverengine#RunResult` -> `type RunResult struct`
- `plan:internal/weaverengine#ErrRunBusy` -> `var ErrRunBusy = errors.New("weaver: run is already in progress")`
- `plan:internal/weaverengine#RunActive` -> `func RunActive(scratchDir string) (bool, error)`
- `plan:internal/weaverengine#runBatch` -> `func runBatch(deps RunDeps, st *State, plan *planparser.Plan, batch batcher.Batch, startRung int) (batchEnd, error)`
- `plan:internal/weaverengine#batchEnd` -> `type batchEnd struct`
- `plan:internal/weaverengine#stopReason` -> `func stopReason(deps RunDeps, st *State, label, cause string) string`
- `plan:internal/weaverengine#TestRun` -> `func TestRun(t *testing.T)`

**Extends:**
- `internal/weaverengine/doc.go`

**Uses:**
- `internal/runplan#ResolveAtDispatch`
- `internal/runplan#WriteConflictRecord`
- `internal/runplan#RemoveConflictRecord`
- `plan:internal/runplan#ConflictDir`
- `plan:internal/runplan#PendingPartition`
- `plan:internal/runplan#RecordBeginPartition`
- `internal/runplan#AcquireStateMutation`

**Intent:** The row's producer body: every batch in Go, then the plan-level verify (D2, D3, D4, D5, D6).

- `Run` holds Weaver's run lock, `run.lock` in the scratch directory taken with `lock.TryAcquireWriteLock`, for its whole call; a held lock returns `ErrRunBusy` wrapped with the way forward: wait for the running row to stop, then re-step it.
  `RunActive` probes the lock and releases it at once, as `websterengine.RunActive` does.
  On entry it removes a stale plan-conflict record from `runplan.ConflictDir(AnchorRoot)`.
- Each batch begins under the state-mutation lease: it re-reads the plan, partitions the remaining cards (`runplan.PendingPartition`), records the partition (`RecordBeginPartition`), and runs `runplan.ResolveAtDispatch` for the first fresh batch.
  Weaver's way forward for a conflict names the reroute: the parent fixes the named pending cards through plan review, then re-steps the row; it never names a `lym webster` verb.
  A `*PlanConflictError` writes the conflict record and returns Stuck with no `StopRecord`, so the producer takes the reroute (card 16) and the next call begins afresh from the current plan, a card Plan-Review inserted or removed included.
  It records the batch's card set, card hashes and start commit (HEAD) once, and, on the run's first begin, `PlanWeftCommit` from `WeftHead`.
  A batch's card set and start stay fixed for every attempt.
- `runBatch` runs attempts from `startRung`: it takes the next attempt number, fixes the report path, records the attempt in flight with its rung, renders the prompt (card 8) with the previous attempt's failure digest and uncommitted paths on a retry, starts the strand (`StartGated` with `batchGateSpec`), records its guid and strand name, waits, judges it (`classifyAttempt`) and ends it through `endAttempt` (card 10), whatever the verdict.
  - `recordable` that `endAttempt` records done ends the batch done; one it fails counts as a failed attempt.
  - `failed` climbs to the next rung; a failure on the last rung stops the row.
  - `lost` retries once on the same rung; a second loss counts as a failed attempt.
  - `needs_parent` stops the row at once with the report's `need`, without climbing.
  Every attempt that ends without being recorded done has its outcome and cause recorded first, then its strand stopped, before the next attempt starts or the row returns.
  A failed state write still stops the strand and returns Stuck naming the fault.
  A failed stop starts no next attempt and returns Stuck naming the guid and `lym reed remove <guid>`.
- After the last batch, `Run` runs the verify (card 11): a failure spawns one fixer on the next rung through `gatefixer.SpawnOrAttach` and `gatefixer.Await` over `fixerStore`, records the fixer number first, records its outcome when `Await` returns terminal, and runs the verify again before any next fixer; a lost fixer gets one same-rung retry after the re-run.
  An `Await` result warning that the fixer's stop failed starts no next fixer and returns Stuck naming the strand's guid and `lym reed remove <guid>`.
  A runner crash stops the row; a failure after the last rung stops the row; a pass writes `summary.md`, writes `outcome.yaml` done, commits the run record and returns Done.
  A failure of that last commit returns Stuck naming it and the re-step as the way forward.
- `stopReason` renders every stop's reason: the batch and its cards or the verify's findings, the plan directory, the state file, the last report, the journal and the decision record, and `ReentryStep`.
  `StopRecord` records a batch, needs-parent or verify stop's kind, batch and rung for the re-entry (card 13).

`TestRun` is tier0, over a fake `StrandRunner` that writes scripted reports, a fake fixer, a fake verify and an index fake:
a three-card plan the batcher splits in two runs two strands in order on the first rung and returns Done after the verify, spawning nothing else, with `outcome.yaml` done, `summary.md` written and the run record committed;
the first begin records `PlanWeftCommit`; a batch's card set and start commit stay the same across its attempts;
a batch reporting a rename partitions the next batch from the rewritten pending card;
a failed attempt starts the next on the next rung's model; a failure on the last rung returns Stuck naming the batch, state file, report and journal;
a `FAILED` report, a failing gate and an unclean tree each climb, and each failed attempt's journal entry is appended and committed;
a machinery loss retries once on the same rung with a new attempt number, and a second loss climbs;
every ended attempt's strand is stopped after its outcome is recorded, also when the state write fails;
a failed stop returns Stuck naming the guid and `lym reed remove` and starts nothing;
`NEEDS_PARENT` returns Stuck with its need at once;
a plan conflict at begin writes the conflict record, records no stop, names no `lym webster` verb and returns Stuck;
a verify failure spawns one fixer on the ladder under `fixer_timeout_min` and reruns the verify after it, a lost fixer included; a dirty tree hands its paths to the next fixer; a fixer whose stop failed returns Stuck naming its guid and spawns no next fixer; a runner crash spawns no fixer; a failure after the last rung returns Stuck;
a held run lock returns `ErrRunBusy` naming the re-step.

**ImpactSummary:** Only adds to the new weaverengine package's doc; no existing symbol changes.
