---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:155
    summary: D12 records a report found on disk before it attaches, so a strand still in its gate re-prompt loop is never attached, its failing gates are never seen, and the next batch spawns beside it
    fix: In D12, replace the order "first records a finished report that was not yet recorded, then attaches for an attempt in flight" with - for every attempt state records in flight, Weaver first attaches through AttachGated with the attempt's report path and gates; the attached Wait settles a run whose report is already on disk under its gates; Weaver records the report only from the result that Wait returns, and records a report directly only when the attach answers not found. Update the D12 acceptance criterion to add - a crash while a strand is re-prompted on a failing gate attaches to that strand, and the batch is not recorded done until its gates pass.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md:54
    summary: D3 and D11 do not pin the batch key or the batch list the shared checks get, and the ordinal numbering D3 implies breaks unbegunCards, isLastBatch and LastBegunCard, which key a batch by its first card number
    fix: In D3, replace "its batches numbered after the done ones" with - each batch's number is its first card's number, as the shared batch identity gives it; pending batches live only in the recorded partition and get no batch record until they begin. In D11, replace "The shared checks take that embedded state, so they read the partition D3 records as the batch list" with - the batch list Weaver hands the shared checks is its recorded partition mapped onto the current plan, the same mapping Webster's verbs use, so the later-reference check sees the pending cards and only the run's last batch reads as last.
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:92
    summary: D6 stops a strand only when its attempt fails or times out, so a machinery loss with a live pane, a Stuck return on a Weaver fault and a NEEDS_PARENT stop each leave a live strand beside the next attempt or the arbiter, and D13's held-back notice does not stop it
    fix: In D6, replace "An attempt that fails or times out has its strand stopped" with - every attempt that ends without being recorded done has its strand stopped through shuttle's stop verb before the next attempt starts or the row returns, whatever ended it - a failure, a timeout, a machinery loss, a NEEDS_PARENT report or a fault in Weaver's own config or state. In D13, drop the claim that holding the notice back keeps the leftover strand from acting beside the arbiter, and say instead that D6's stop removes it. Add to the D6 acceptance criterion - a machinery-loss retry and a Stuck return on a Weaver fault each find the attempt's strand stopped.
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md:129
    summary: D12's fixed strand identity, in-flight record and attach cover batch attempts only, so a crash during D10's verify fixer lets the verify re-entry spawn a second fixer beside the live one
    fix: In D12, change "Every attempt of a batch" to every attempt of a batch and every verify fixer strand, and say a fixer strand's identity is its own output file, fixed and recorded in flight before its spawn, so re-entry attaches to it as to a batch attempt. In D10, name the fixer's output file. Add to the D12 acceptance criterion - a crash during a fixer strand attaches to it and never spawns a second fixer.
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md:45
    summary: D2 claims the plan returns with the task branch to its state at the run's start, but the plan directory and _lym/weaver/ live in the weft worktree that D9's records commit writes to, so moving the task branch leaves the rename-rewritten pending cards and the per-batch records in place
    fix: In D2, state that lym weaver reset moves the code branch back to the first batch's recorded start commit and restores the plan directory in the weft to its content at the run's start, committed through fabric; that it archives _lym/weaver/ before clearing it; and that Webster's run directory is untouched. Drop "The plan returns with the branch to its state at the run's start". In D9, replace "puts it on the branch" with "puts it on the weft branch". Extend the D2 acceptance criterion so the test checks that the plan's pending cards read as at the run's start after the reset.
  - id: F6
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:47
    summary: The bound on lym weaver reset names only the run lock, which a crashed producer drops, so the reset can move the branch and clear .lym/weaver/ under a batch strand that outlived the producer
    fix: In D2, add to the reset's bound - it also refuses while state records an attempt in flight whose strand shuttle reports live, naming as the way forward re-stepping the row so that it attaches, or lym loom pause --after the implementer row; and it stops no strand itself. Add that case to the reset acceptance criterion.
  - id: F7
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:98
    summary: D7's no-spawn path records a stopped batch done on the post-batch checks and card gates alone, so a batch whose only failure was a row gates entry passes on re-entry, and D7 states no bound on that bypass
    fix: In D7, state the bound - the no-spawn path records the batch done only when its post-batch checks, its card gates and the row's gates entries all pass; since a gates entry needs a strand to run in, a batch whose last attempt failed on a row gates entry, as state records, never takes the no-spawn path and gets the last-rung attempt instead. Add that case to the D7 acceptance criterion.
  - id: F8
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md:100
    summary: D7 treats a NEEDS_PARENT stop like a ladder stop, so once the parent answers, the batch jumps to the last rung for one attempt, or is recorded done with no spawn if its cards check done, against D6's rule that a question does not climb
    fix: In D7, add - re-entry after a NEEDS_PARENT stop starts the next attempt on the rung that asked, with the ladder above it still open, and never takes the no-spawn path. Add that case to the D7 acceptance criterion.
  - id: F9
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:98
    summary: D7's re-entry runs the shared checks before any begin, against a recorded partition and plan fingerprint that an arbiter fix, a renumber or a Plan-Review reroute may have changed, and the shared refusals then name Webster's rebaseline and restore-plan verbs, which do not act on Weaver's run
    fix: In D7, state that re-entry first re-reads the plan, and that when the stopped batch's recorded cards are unchanged it restamps Weaver's plan baseline and maps the done and stopped batches onto the current plan before it runs the checks. A stopped batch whose own cards changed refuses, naming lym weaver reset as the way forward. In D11, state that every shared refusal names the active implementer's verbs in its way forward. Add a D7 acceptance case where the arbiter edits a pending card and the stopped batch is then recorded done.
  - id: F10
    severity: MEDIUM
    class: decision
    origin: handler
    location: _lym/discussion/decision-record.md:207
    summary: The board says the journal is pushed with the run so it outlives a move to another machine, while the implementer contract says the implementer never pushes and push-before-act-takeover owns pushes at shed transitions; only the parent can settle who pushes the journal mid-row
    fix: Raise to the parent - should Weaver push after each batch as an exception to the implementer never pushing, should push-before-act-takeover push at points inside a producer, or is a push at the row's end enough? Record the answer as a Decision and drop the Open risks line and Q16.
  - id: F11
    severity: LOW
    class: decision
    origin: handler
    location: _lym/discussion/decision-record.md:206
    summary: The operator's 2026-11-15 decision says each batch is a producer, and the record reads it as one producer that runs every batch; only the parent can confirm that reading of the operator's words
    fix: Raise to the parent - is one producer that runs every batch, with its stops going through shed's stop chain, an acceptable reading of the decision that each batch is a producer? Record the answer in D2 and drop the Open risks line and Q17.
  - id: F12
    severity: LOW
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md:176
    summary: D13 has merge-in's in-flight warning read Weaver's state, but Weaver's run directory has no outcome marker, so nothing defines when a Weaver run is finished and the warning would fire on every finished run
    fix: In D9, add an outcome marker to Weaver's run directory, written done when the row returns Done. In D13, state that the merge-in warning counts a Weaver run in flight while its state file exists and that marker is absent or not done, as for Webster.
  - id: F13
    severity: LOW
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md:92
    summary: D6 stops a failed strand and only then records the attempt failed, so a crash between the two leaves the attempt in flight with a stopped run, which D12 counts as a machinery loss and retries on the same rung
    fix: In D6, reverse the order - state records the attempt failed, then the strand is stopped through shuttle's stop verb. In D12, state that re-entry stops the live strand of an attempt recorded failed before it starts anything.
  - id: F14
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:32
    summary: D1's way forward of moving a corrupt state file aside makes it count as missing with no stated bound, so a run can switch implementer over built work, and the plan gate loses those done cards
    fix: In D1, state the bound on that way forward - a state file moved aside counts as missing only when the task branch's HEAD is the run's start commit; otherwise the seam keeps refusing and names lym webster reset --to start or lym weaver reset first.
  - id: F15
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md:157
    summary: D12 sends an attached strand every decision recorded since its attempt began, including those D13's live notice already delivered before the crash
    fix: In D12, replace "every decision entry recorded since its attempt began" with every decision entry recorded since the attempt began that state does not record as delivered to it, and have D13's live notice record each delivery in Weaver's state.
  - id: F16
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md:71
    summary: D5 lists weaver.yaml's keys as ladder and fixer_timeout_min, while D10 adds verify_gate
    fix: In D5, add verify_gate (D10) to the list of keys weaver.yaml holds.
  - id: F17
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/discussion/decision-record.md:138
    summary: D11's second paragraph lists parameter lists and carried types of postBatchInputs, checkRecoveredSuspects, failureDigestBlock and the fingerprint helpers, the exact-signature detail the writer is told not to gather and Plan computes fresh
    fix: Cut D11's line beginning "These pieces read Webster's state model" from the decision record and move it to the support log's Interview as evidence for Q12. Keep the requirement in D11 - the state types the shared pieces read move with them into a package both implementers import.
  - id: F18
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/discussion/decision-record.md:141
    summary: D11 names one run-directory field of Geometry, but Geometry carries several told directories, the run, reports, prompts and scratch directories, and the scratch one holds the state lock and state-mutation lease D3 and D13 rely on
    fix: In D11, replace "Geometry's run-directory field names the active implementer's run directory" with - every told directory of Geometry that names Webster's run, reports, prompts or scratch directory names the active implementer's under Weaver, so Weaver's state lock and state-mutation lease sit under .lym/weaver/.
---

### [BLOCKING] D12 records a report on disk before attaching, so a live gated strand is never attached

**Location:** `_lym/discussion/decision-record.md:155`

**Issue:** D4 has the agent write its report as the strand's output file while must-pass gates re-prompt it, so the report can be on disk while the agent is still fixing gate failures.
Shuttle's attach already treats a running record whose output files all exist as attachable and waits on it under the gate (`internal/shuttleengine/attach.go:535`, `:223`).
D12 records "a finished report that was not yet recorded" first, which takes the attempt out of flight, so the attach never runs.
The batch is recorded done with failing gates, against D6, and the next batch spawns beside the still-live agent, the race D12's own rationale forbids.
The walkthrough and bound-breaker lenses each built this counterexample on their own.

### [MEDIUM] Batch key and batch list for the shared checks are unpinned

**Location:** `_lym/discussion/decision-record.md:54`

**Issue:** `unbegunCards` looks up `st.Batches[number]` with `batchIdentity`, which is the first card's number (`internal/websterengine/recordbatch.go:114`, `internal/websterengine/runlevel.go:982`).
`isLastBatch` compares by the same key (`internal/websterengine/cardgate.go:66`), and `LastBegunCard` reads every record in `Batches`.
D3's "numbered after the done ones" reads as an ordinal; under it, a begun batch reads as unbegun.
If pending batches got records, their cards would leave the later-reference check, and the renumber bound would pass cards that have not run.
`laterDeleteReferenceReasons` and `isLastBatch` take the batch list as a parameter, so D11's "they read the partition" needs saying how the list is built.
Every lens found this; the existing-paths, interactions and bound-breaker reports duplicate it.
Focus item 2 verdict: the record's intent, the full partition recorded at each begin, is right, and `isLastBatch` reads true only for the run's last batch.
The key and the list handed in are what is missing.

### [MEDIUM] Strands that end any way but failure or timeout stay live

**Location:** `_lym/discussion/decision-record.md:92`

**Issue:** A machinery loss with a live pane, such as shuttle's API-error verdict, is retried under a new attempt number and so a new report path.
`AttachGated` removes only superseded strands whose output-file set is exactly the same (`internal/shuttleengine/attach.go:230`), so it never removes the old strand.
A Stuck return on a fault in Weaver's own state, D2, and a NEEDS_PARENT stop leave the strand live while the tackler or arbiter acts.
D13 says the held-back notice keeps such a strand from acting beside the arbiter.
`heldBackReason` only withholds a notice (`internal/loomcli/decision.go:440`) and stops no agent.
Focus item 5 verdict: a failed or timed-out attempt is stopped as stated.
Re-entry attaches only by the in-flight attempt's own report path, so it cannot attach to an earlier attempt's stale strand.
It does not remove that strand either.

### [MEDIUM] The verify fixer has no crash identity

**Location:** `_lym/discussion/decision-record.md:129`

**Issue:** D12's identity, in-flight record and attach are scoped to "every attempt of a batch".
D10's fixer strand names no output file, and the record gives it no attach.
D6 applies the machinery-loss retry to fixer strands, but nothing detects their loss after a crash.
D7's verify re-entry can start a second fixer beside a live one.

### [MEDIUM] lym weaver reset moves the code branch, but the plan and run records live in the weft

**Location:** `_lym/discussion/decision-record.md:45`

**Issue:** `_lym` in this worktree links into a separate weft worktree on its own branch.
`fabricengine.CommitRecordsPaths` commits there (`internal/fabricengine/commitweftpaths.go:21`).
D9 commits the plan directory and `_lym/weaver/` through that path, so moving the task branch back leaves the plan's rename-rewritten pending cards in place.
D2's claim that "the plan returns with the branch to its state at the run's start" is false.
A Weaver run started again after the reset would read cards rewritten for glyphs the reset removed.
The interactions lens's reading, that the reset deletes `_lym/weaver/` before archiving it, is rejected below for the same reason.

### [MEDIUM] The reset's bound ignores a strand that outlived the producer

**Location:** `_lym/discussion/decision-record.md:47`

**Issue:** The flock claim holds, but a batch strand runs in its own reed pane and outlives a crashed producer.
After a crash, the reset proceeds and moves the branch under an agent that is still committing.
The refusal names only the lock.

### [MEDIUM] D7's no-spawn path bypasses the row's gates entries

**Location:** `_lym/discussion/decision-record.md:98`

**Issue:** D7 runs the post-batch checks and the card gates at HEAD, but not the row's `gates` entries that D4 puts on every strand.
The support log rejects running those entries from Go, since they need a strand to re-prompt.
Suppose a last-rung attempt fails only on a row gates entry.
On re-entry it passes D7's checks and is recorded done, and D7 states no bound on that.

### [MEDIUM] A NEEDS_PARENT stop re-enters as a ladder stop

**Location:** `_lym/discussion/decision-record.md:100`

**Issue:** D6 does not climb on a question, and support-log Q5 says a question is not a model failure.
D7 gives every re-entry after a batch stop one last-rung attempt, so an answered question at rung 1 skips the middle rungs.
When the cards already check done, D7 records the batch done whether or not the parent's answer was applied.

### [MEDIUM] Re-entry after a plan edit has no Weaver rebaseline

**Location:** `_lym/discussion/decision-record.md:98`

**Issue:** The arbiter may now edit and commit, and a reroute to Plan-Review may rewrite pending cards.
D7's checks run before any begin, against the partition and fingerprint D3 recorded.
`mapPartition` refuses a plan card that no recorded batch holds.
Its way forward names `lym webster rebaseline` and `lym webster restore-plan` (`internal/websterengine/partition.go:131`), which act on Webster's run directory.
D7's "the arbiter or a human fixed it" path cannot complete under Weaver.

### [MEDIUM] Open risk for the parent: who pushes the journal mid-row

**Location:** `_lym/discussion/decision-record.md:207`

**Issue:** The board's 2026-11-15 operator decision says the journal "is pushed with the run, so it outlives a crash or a move to another machine".
The implementer contract has the implementer never push, and `push-before-act-takeover`, a draft entry, owns pushes at shed transitions.
The whole Weaver row is one such transition.
This is a conflict between the entry and another entry's planned work, which only the parent settles.
Round 1 declined to file the two Open risks, reasoning that the record already raises them.
The parent is reached only through a `decision` finding, though, and a round later neither question has an answer: there is no `Decision for weaver:` note and no added entry.
So both are filed here, F10 and F11.

### [LOW] Open risk for the parent: one producer for every batch

**Location:** `_lym/discussion/decision-record.md:206`

**Issue:** The operator's decision says "each batch is a producer, so the Shed driver's tackler and arbiter handle a stuck batch".
D2 keeps the purpose, stops reaching the tackler and arbiter, with one producer looping over batches.
Confirming that this reading fits the operator's words is the parent's call, not the writer's.

### [LOW] Merge-in's in-flight warning has no Weaver done marker

**Location:** `_lym/discussion/decision-record.md:176`

**Issue:** `RunInFlight` counts a run in flight while `state.json` exists and `outcome.yaml` is absent or not done (`internal/websterengine/inflight.go:21`).
Weaver's run directory, D9, holds no outcome file, and its state stays committed after Done.
The record never says what marks a Weaver run finished.

### [LOW] Stop-then-record turns a crash into a machinery loss

**Location:** `_lym/discussion/decision-record.md:92`

**Issue:** `stopRecorded` writes `stopped` for a run with no report (`internal/shuttleengine/stop.go:92`).
A crash after the stop and before the failed-state write leaves the attempt in flight with a dead run.
D12 then counts that as a machinery loss, and the batch gets a same-rung retry instead of climbing.

### [LOW] Moving a corrupt state file aside has no bound

**Location:** `_lym/discussion/decision-record.md:32`

**Issue:** Moving a corrupt state file aside makes it count as missing, which lifts D1's switch guard and D13's seam over work already built.
D1 introduces this way forward without saying what bounds it.

### [LOW] Decisions delivered twice to an attached strand

**Location:** `_lym/discussion/decision-record.md:157`

**Issue:** D13 sends live Weaver strands each decision inline.
D12's attach resends every entry recorded since the attempt began, including those already delivered before the crash.

### [LOW] D5's key list omits verify_gate

**Location:** `_lym/discussion/decision-record.md:71`

**Issue:** D5 says `weaver.yaml` holds `ladder` and `fixer_timeout_min`; D10 adds `verify_gate`.

### [LOW] D11 carries signature detail the writer is told not to gather

**Location:** `_lym/discussion/decision-record.md:138`

**Issue:** The writer's stencil bars exact signatures and interface shapes, which Plan computes fresh.
D11's line listing what `postBatchInputs` carries and what `checkRecoveredSuspects`, `failureDigestBlock` and the fingerprint helpers take is that class of fact.
The requirement it supports, moving the state types into a shared package, already stands on the next line, so moving the line to the support log loses nothing Plan needs.

### [LOW] D11 names one Geometry directory where several move

**Location:** `_lym/discussion/decision-record.md:141`

**Issue:** `Geometry` tells webster its run, reports, prompts and scratch directories (`internal/websterengine/geometry.go:36`).
The scratch directory holds the state lock and the state-mutation lease (`internal/websterengine/state.go:90`).
D11 says only that "the run-directory field" names the active implementer's.
Focus item 4 verdict: D11 does settle the question.
The shared checks take the embedded run-plan state, which holds the batch records and the partition, so the choice is not left to Plan.
Only which told directories switch with it is left unclear.

## Focus verdicts

- Focus 1, D12 strand identity: settled for identity.
  The report path is fixed and recorded before the spawn, and `AttachGated` finds the run by its output files with no guid.
  The re-entry order breaks the guarantee (F1), and the verify fixer is not covered (F4).
- Focus 2, batch list: F2.
- Focus 3, active-implementer seam: held.
  Begun batches in the live run directory decide which implementer is active, with `tapestry.implementer` as the fallback.
  A missing state file counts as no begun batches; a corrupt one refuses in the D1 guard, the plan gate, the renumber verbs and Describe, and only the two advisory readers fall back.
  The one gap is the move-aside bypass, F14.
- Focus 4, state shape: F18.
- Focus 5, strand lifecycle: F3, F13.

## Rejected

- interactions F1, that the reset archives `_lym/weaver/` after the hard reset deleted it: `_lym` is a link into the separate weft worktree, so moving the task branch never touches `_lym/weaver/`. The real defect is F5.
- interactions F3, that the clean-tree check fails every attempt on lym's own uncommitted run-directory files: those files live in the weft worktree behind the `_lym` link, so they never appear in the code tree's git status.
