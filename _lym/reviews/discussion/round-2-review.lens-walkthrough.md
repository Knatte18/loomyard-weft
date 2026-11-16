```yaml
lens: walkthrough
outcome: reported
```

Task: walkthrough-lens review of the Weaver decision record.

## Walkthrough list

- **D1:** Weaver run, then `tapestry.implementer` flipped to `webster` while Weaver has begun batches. The seam picks Weaver and the guard refuses. **Purpose met.**
- **D2:** a three-card plan split [1,2] and [3], run through to verify. The loop runs, and its stops go to shed's stop chain. **Purpose met.**
- **D3:** cards 1–4, partition [1,2] and [3,4], batch [1,2] done, then a re-partition into [3] and [4]. The shared checks get a batch list whose numbering and begun-set don't match how they key them. **Purpose not met** (finding 2).
- **D4:** first attempt of batch [1,2]. The prompt, gates and report output file all follow from D4. **Purpose met.**
- **D5:** sonnet fails, then opus, then opus[high] fails and the row stops. **Purpose met.**
- **D6:** an attempt times out. It is stopped through shuttle's stop verb, recorded failed, and the next rung starts. **Purpose met.**
- **D7:** re-entry after a `NEEDS_PARENT` stop on rung 1. The batch's post-batch checks fail because no work was done, so it gets one last-rung attempt. **Partly met** (finding 3).
- **D8:** a crash between the journal append and the state write. The keyed append skips the duplicate. **Purpose met.**
- **D9:** the commit after batch [1,2]. **Purpose met.**
- **D10:** verify fails, the fixer on rung 1 fixes it, verify passes. **Purpose met.**
- **D11:** the later-delete check under Weaver after batch [1,2]. It is the same mechanism as D3. **Purpose not met** (finding 2).
- **D12:** a crash while the agent has written its report and its strand is still re-prompted on a failing gate. Re-entry records the report first and never attaches, so the live strand keeps editing beside the next batch. **Purpose not met** (finding 1).
- **D13:** a reroute to Plan-Review after Weaver has built cards. The seam picks Weaver, so the gate freezes its done cards. **Purpose met.**

## Findings

1. **MEDIUM · design · decision-record.md D12 (lines 155–158).**
   On re-entry, D12 records a "finished report" before it attaches. That lets a gated strand that is still live keep running beside the next batch.
   - D4 has the agent write its report as the strand's output file while must-pass gates re-prompt it. So the report can be on disk while the agent is still fixing gate failures.
   - Shuttle settles that case itself. `AttachGated` reconstructs a running record whose output files all exist and re-waits it under the gate (`internal/shuttleengine/attach.go:535-537`, `:223`).
   - In D12's order, Weaver records the report first, so the attempt is no longer in flight. The attach is skipped and the live strand is never stopped. The batch is recorded done without its gates having passed, against D6's "gates still fail" rule, and the next batch spawns beside that agent.
   - The record also never says how Weaver tells a finished report from one written partway through a gate loop.
   - Fix: attach first, for every attempt that has a report path and no recorded outcome. Let the attached Wait settle the finished case under the gate. Record the report only from the result that Wait returns.

2. **MEDIUM · design · decision-record.md D3 (lines 54–56) and D11 (lines 139–142).**
   The record doesn't pin how the recorded partition maps onto the keys the shared checks read. In two readings it allows, the later-reference check and the last-batch flag come out wrong.
   - `unbegunCards` decides "begun" by looking up `st.Batches` under `batchIdentity(b)`, which is the batch's first card number (`internal/websterengine/recordbatch.go:114-124`, `runlevel.go:982-987`). `isLastBatch` compares batches by that same identity (`cardgate.go:66-73`).
   - D3 says the fresh batches are "numbered after the done ones" and that pending batches "stay recorded as pending".
   - If Weaver keys batch records by a running count (batch 2 for cards [3,4]), the lookup for key 3 misses. The in-flight batch's own cards then read as unbegun later cards.
   - If a pending batch gets a `BatchState` record, `unbegunCards` drops its cards and the later-reference check never sees them. `LastBegunCard` (`beginbatch.go:283-299`) also counts them as begun, which moves the renumber bound past cards that haven't run.
   - `laterDeleteReferenceReasons` and `isLastBatch` take the batch list as a parameter (`recordbatch.go:190`, `cardgate.go:66`); they don't read it from state. D11's "the shared checks take that embedded state, so they read the partition" doesn't hold as written.
   - Fix: D3 states three things. A batch's number is its first card's number, as `batchIdentity` gives it. Pending batches live only in `Partition` and never get a `Batches` record. The batch list handed to the shared checks is the recorded `Partition` mapped onto the plan, through `ExecutionBatches`/`mapPartition`.

3. **LOW · design · decision-record.md D7 (lines 98–100).**
   D7 treats every batch stop the same on re-entry, so a `NEEDS_PARENT` stop jumps to the last rung.
   - D6 stops the row "without climbing" on a `NEEDS_PARENT` report from, say, rung 1.
   - Once the parent answers and re-steps, D7 runs the post-batch checks. They fail, since the agent asked before it did the work, and the batch gets "one more attempt on the last rung".
   - So a question costs the batch every rung between its own and the last, and it gets one attempt instead of a ladder. That contradicts D6's choice not to climb on a question.
   - Fix: D7 says which rung a re-entry attempt runs on after a `NEEDS_PARENT` stop, for example the rung that asked.

Tool calls:
- grep across `internal/shedarbiter` and the recipes for the arbiter's recipe-description source, to check D8's claim; the manifest didn't hold it.
- grep for files that mention the arbiter, to find where the description is built; the first grep found nothing.
- grep for a description field in `loomengine`, `sheddriver`, `shedengine` and `shedrecipe`; nothing found, so I report no finding on D8.
