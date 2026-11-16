```yaml
lens: bound-breaker
outcome: reported
```

Bound-breaker lens: I attacked each bound, guard, ordering promise and claim about existing code in the decision record. Each claim is listed below as held or broken, followed by the findings.

**Claims attacked**

- D12: the attach finds an attempt's run by its declared output file, with no guid. **Held** (`internal/shuttleengine/attach.go:111-115`, `:434-470`).
- D12: a finished report is recorded first, then the attach runs only for an attempt still in flight. **Broken.** A report already on disk is not a finished report while gates still re-prompt the strand (finding 1).
- D12: a failed attempt is never attached. **Held.** Report paths are unique per attempt, so a failed attempt's record never matches a later attempt's spec.
- D12 with D6: no second agent ever runs on the tree. **Broken** on the machinery-loss retry (finding 2) and on a Stuck return for a Weaver fault (finding 3).
- D6: a failed or timed-out attempt's strand is stopped before the next attempt. **Held as stated.** A crash between the stop and the state write turns a timeout into a machinery loss (finding 6).
- D13: holding back the notice keeps a halted run's leftover Weaver strand from acting beside the arbiter. **Broken** (finding 3).
- D7: a batch that passes the post-batch checks on re-entry is recorded done with no spawn. **Broken.** The no-spawn path skips the row's `gates` entries and a pending NEEDS_PARENT question (finding 4).
- D7 and D11: re-entry after an arbiter fix runs on Weaver's state. **Broken** when the fix edits the plan (finding 5).
- D3: the shared checks see the pending cards. **Held only** if Weaver keys batches by first card number (finding 7).
- D3: only the true last batch reads as last. **Held.** The partition recorded at each begin covers every remaining card, and `internal/websterengine/cardgate.go:66` compares against its tail.
- D2: `lym weaver reset` refuses while a live producer holds the lock, and the kernel drops the flock on exit. The flock claim **holds** (`internal/lock/lock.go:13`). The safety the bound implies is **broken** (finding 8).
- D1: `lym webster reset --to start` clears Webster's begun batches. **Held.** It archives the run record (`internal/websterengine/reset.go:67-81`).
- D1: moving a corrupt state file aside makes it count as missing. **Broken as a bound** (finding 9).
- D5: batch attempts are bounded. **Held.** Each attempt gets one retry, a second loss counts as a failed attempt, and the parent re-step is unbounded by design.
- D8: the journal append is idempotent across a crash. **Held,** with the batch key caveat in finding 7.
- D12: decisions recorded since the attempt began are sent once. **Broken.** Live notices already delivered are sent again (finding 10).

**Findings**

1. **MEDIUM, design. D12: a crash during a gate re-prompt skips the strand's gates and leaves a live agent working beside the next batch.**
   - Shuttle evaluates a gated run's gates at turn ends once the output files exist, and re-prompts the agent with the report already on disk (`internal/shuttleengine/gate.go:73`, `:142`).
   - Counterexample: the agent writes a `DONE` report and a card gate fails. The producer crashes while shuttle re-prompts the agent. On re-entry, D12 "first records a finished report that was not yet recorded".
   - The batch is then no longer in flight, so the attach never runs. The gate failure is never seen, and the live strand keeps editing while Weaver begins the next batch.
   - `dispositionCandidate` already attaches a running record whose files exist (`attach.go:535`). The record-first step should apply only to a run shuttle no longer holds as running.

2. **MEDIUM, design. D6 and D12: a machinery-loss retry can spawn beside a live strand.**
   - D6 stops a strand only for an attempt that fails or times out. A machinery loss with a live pane, such as shuttle's API-error verdict, is retried without a stop.
   - The retry takes a new attempt number and so a new report path (D12). `AttachGated` removes superseded strands only when the output-file set is exactly the same (`attach.go:53-57`, `:230-245`), so the old strand is never removed.
   - Result: two agents on one tree, which D12's rationale forbids.

3. **MEDIUM, design. D2, D6 and D13: a Stuck return for a Weaver config or state fault leaves the attempt's strand live while the tackler and arbiter act.**
   - D2 returns Stuck "on a fault in Weaver's own config or state". D6 stops strands only for failed or timed-out attempts.
   - Example: the state write after the spawn fails, so the row returns Stuck with the strand working.
   - D13 claims that holding back the decision notice means the strand "never acts beside the arbiter". Withholding a notice does not stop an agent that is working. `heldBackReason` only suppresses notices (`internal/loomcli/decision.go:440-449`).

4. **MEDIUM, design. D7: re-entry records a batch done without the row's `gates` entries, and over an unanswered NEEDS_PARENT stop.**
   - D7 runs the post-batch checks and the card gates at HEAD, but not the row's `gates` entries that D4 attaches to every strand.
   - Counterexample: a last-rung attempt fails only a row `gates` entry and the row stops. On re-entry the done checks and card gates pass, so the batch is recorded done and the guard is bypassed.
   - Likewise, a NEEDS_PARENT stop on a batch whose cards check as done is recorded done on re-step, whether or not the parent's answer was applied.

5. **MEDIUM, design. D7 and D11: an arbiter or human plan edit during a stop has no rebaseline path under Weaver.**
   - D11 shares the plan fingerprint, the plan-conflict refusal and the partition mapping. `mapPartition` refuses an id the plan no longer holds with a way forward of `lym webster rebaseline` or `lym webster restore-plan` (`internal/websterengine/partition.go:131`).
   - Counterexample: the arbiter, now allowed to edit, fixes a pending card or runs a renumber past the last begun card. D7's checks on re-entry then hit a fingerprint or partition mismatch.
   - The way forward names Webster verbs that act on Webster's directory, and the record gives Weaver no restamp. D7's "the arbiter or a human fixed it" path cannot complete.

6. **LOW, design. D6 and D12: a crash between the stop and the failed-state write reclassifies a timeout as a machinery loss.**
   - D6's order is: stop the strand, then record the attempt failed. A crash between the two leaves the attempt in flight with a `stopped` run record.
   - `stopRecorded` writes `stopped` when the run has no report (`internal/shuttleengine/stop.go:92-97`). On re-entry the attach finds no live run, and D12 counts it a machinery loss.
   - The batch gets a same-rung retry instead of climbing.

7. **MEDIUM, consistency. D3 against D11: "numbered after the done ones" conflicts with the shared code's batch key.**
   - `unbegunCards` looks up `st.Batches[number]` with `batchIdentity`, which is the batch's first card number (`internal/websterengine/recordbatch.go:117`, `internal/websterengine/runlevel.go:982-986`).
   - If Weaver numbers batches by position, as D3's wording reads, the lookup misidentifies which batches are begun. A pending batch then reads as begun, or the reverse.
   - That breaks D3's claim that the later-reference check sees the pending cards, and keys D8's journal headings inconsistently.

8. **MEDIUM, design. D2: after a producer crash, `lym weaver reset` can move the branch under a live batch strand.**
   - The lock bound holds: the kernel drops the flock on exit (`internal/lock/lock.go:13`). But the batch strand runs in its own reed pane and outlives the producer.
   - After a crash the reset proceeds, moves the task branch back and clears `.lym/weaver/` while the agent is still committing.
   - The verb's bound names only the lock. It does not stop or refuse on an in-flight attempt that state records.

9. **LOW, design. D1: the "move a corrupt state file aside" way forward is an unbounded bypass of the switch guard.**
   - Once the file is moved aside it counts as missing. Example: a corrupt Webster state whose batches are done is moved aside, and the run switches to Weaver.
   - Weaver then rebuilds the work already in the tree. D13's seam also loses those done cards for the plan gate's freeze.
   - D1 states no bound on what this way forward lets through.

10. **LOW, design. D12 and D13: a notice can be delivered twice.**
    - While the run runs, D13 sends live Weaver strands the inline decision notice. D12 sends "every decision entry recorded since its attempt began" on re-attach.
    - An entry delivered live before the crash is delivered again, and state records no delivered mark.

Tool calls:
1. Grepped `internal/websterengine/reset.go` and `internal/lock`: to check whether reset to start archives Webster's state and whether the lock is a flock. The context had neither.
2. Grepped the shuttle gate and wait re-prompt code: to confirm that a gated run re-prompts with its output file already on disk. The context had `attach.go` but not the gate file.
