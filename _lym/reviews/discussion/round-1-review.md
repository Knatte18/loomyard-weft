---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D12
    summary: D12's crash guard rests on a false premise; shuttleengine.FindRun resolves a known strand guid and cannot find a strand state never recorded, so a crash between spawn and state write spawns a second agent on the tree
    fix: In D12, state that Weaver fixes the attempt's strand identity before spawning and records it in state with the attempt and rung (for example a strand name derived from batch and attempt, or a guid chosen before the spawn), and that the attach on the next call looks the run up by that recorded identity; drop the claim that FindRun finds a run state never recorded. Keep the acceptance criterion's never-spawns-twice clause.
  - id: F2
    severity: BLOCKING
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md D3, D11
    summary: The post-batch checks D11 shares read the whole recorded partition (State.Partition, unbegunCards, laterDeleteReferenceReasons, isLastBatch), but D3 keeps only the first batch of each partition, so every batch reads as the last and later cards are in no batch
    fix: In D3, state what Weaver hands the shared checks as the batch list and how it numbers batches - for example it records the full fresh partition of the remaining cards at each batch begin (done batches kept, the first new batch numbered after them), so the later-card and last-batch checks see the pending cards - and state in D11 that the shared checks take that list. Add an acceptance criterion that a batch removing a symbol a pending card names is refused by the later-reference check under Weaver.
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md D1, D13
    summary: D13 reads the active implementer's state without saying what decides active, and D1's guard runs only at the implementer row, so flipping tapestry.implementer after a Weaver reroute makes the plan gate read Webster's empty state and unfreeze built cards; an unreadable state file's effect is unstated
    fix: In D13, state that the seam picks the implementer whose run directory holds begun batches in the current generation, falling back to tapestry.implementer only when neither has begun one, and that both holding begun batches or either state file failing to load refuses with a way forward. In D1, state that the guard reads through the same seam, so a missing state file counts as no begun batches and a corrupt one refuses. Add an acceptance criterion that the plan gate freezes Weaver's done cards after the setting is flipped to webster.
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md D13 renumber bullet
    summary: The renumber verbs check their bound under Webster's state-mutation lease, the lease Webster's begin-batch takes; D13 moves only the read, so a renumber can pass its bound while Weaver begins a batch over the cards it moves
    fix: In D13's renumber bullet, state that the renumber verbs take the active implementer's state-mutation lease around the bound check and the renumber, and in D3 or D12 that Weaver records a batch's begin under that same lease. Add the race to the D13 acceptance criterion.
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md D13 decision-notice bullet, D12
    summary: D13 holds a decision notice back from a halted run's Weaver strand on Webster's ground that re-entry starts a fresh session, but D12's re-entry attaches to that live strand, so a decision recorded during the halt never reaches it
    fix: In D13's decision-notice bullet, drop the hold-back for Weaver's strands and state that a halted run's live Weaver strand gets the notice, since D12 re-attaches to it and its report is recorded; keep the inline treatment. Change the D13 acceptance criterion's held-back clause to match.
  - id: F6
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md D9, D10, D13
    summary: Describe reads only Webster's summary.md (RunRecordPath from websterGeom.WebsterDir) and prior generations only from prior-generation/webster, and the rework seam archives only Webster's record, so Weaver's triage and verify-fix sections never reach the change description
    fix: Add to D13's reader list Describe's live run record, its prior-generation readers (ArchivedWebsterDirs) and the PR rework archive seam, stating that each reads or archives the active implementer's run directory and that the rework round archives _lym/weaver/ into its own prior-generation slot that Describe's prior readers list. Name these readers in the D10 or D13 acceptance criterion.
  - id: F7
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D11
    summary: D11 leaves to Plan whether the shared pieces can leave internal/websterengine at all; postBatchInputs, checkRecoveredSuspects, the fingerprint helpers and the failure digest all take Webster's State, BatchState and Geometry, so the move carries Webster's state model or Weaver keeps Webster-shaped state - a deferred research question
    fix: Settle it in D11 from internal/websterengine (recordbatch.go postBatchInputs, suspect.go checkRecoveredSuspects, fingerprint.go, recoverbatch.go failureDigestBlock) and record the answer - either the state types the checks read move into the shared package and both implementers' state files embed them, or the checks take a narrow input both states can fill - naming the evidence. Keep D3, D9 and D12 consistent with whichever state shape it picks.
  - id: F8
    severity: MEDIUM
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md D6, D7, D12
    summary: No decision stops a failed attempt's strand (timed out, gates still failing) before the next attempt or the Stuck, so the arbiter's fix can share the tree with a live stale agent and D12's re-entry attaches to it instead of running D7's checks
    fix: In D6, state that an attempt that fails or times out has its strand stopped through shuttle's stop verb before the next attempt starts or the row returns Stuck. In D12, state that the attach applies only to an attempt state records as in flight, never one recorded failed.
  - id: F9
    severity: MEDIUM
    class: design
    origin: handler
    location: _lym/discussion/decision-record.md D1 guard, D2 Bound
    summary: D1's refusal names resetting the other implementer's run directory as a way forward, but D2 adds no verb, so a Webster row refused over Weaver's begun batches has no runnable step and PATTERN-refusal-way-forward is not met
    fix: In D1, name the concrete step for each direction - lym webster reset for Webster's directory, and for Weaver's either a lym weaver reset verb (then amend D2's Bound to name it and what it may discard, and bound it to a halted run) or the PR rework archive path - and state that a test reaches each.
  - id: F10
    severity: LOW
    class: consistency
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D2 Bound, D5 Bound
    summary: The re-entry bound cites the arbiter spawn cap and the driver repair cap, but the repair cap counts only interrupted stops, and a Stuck re-enters also through the tackler's resume under tackler_max_spawns and through a parent re-step
    fix: In D2's and D5's Bound, name the caps that bound a Stuck's re-entry - the tackler's and arbiter's spawn caps, with a parent re-step unbounded by design - and drop the repair cap.
  - id: F11
    severity: LOW
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md D10
    summary: Batten's watch bounds a gatefixer verify-fixer strand by webster.yaml's RecoveryTimeoutMin, and D10 names neither the Weaver fixer's role nor its timeout, so it runs under Webster's recovery timeout
    fix: In D10, state the fixer's gatefixer role and the timeout it runs under - a weaver.yaml key with its default, which D5's config list then names - and that batten's watch reads it for a Weaver run.
  - id: F12
    severity: LOW
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md D13
    summary: lym fabric merge-in warns of a run in flight only through websterengine.RunInFlight, which reads Webster's state.json and outcome.yaml, so a Weaver run mid-flight gets no warning
    fix: Add the merge-in in-flight check to D13's reader list, stating it reads the active implementer's state through the same seam.
  - id: F13
    severity: LOW
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md D8, D12
    summary: A crash between D8's journal append and the state write makes D12's record-the-finished-report step append the same attempt's journal entry twice
    fix: In D8, state that the append is keyed by batch and attempt and skipped when the journal already holds that heading, or that state records the append before the journal write.
  - id: F14
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md D5, D6
    summary: D6's one same-rung retry for a machinery loss does not say whether it is per attempt or per batch, nor whether it applies to D7's re-entry attempt and D10's fixer strands, while D5's Bound speaks of retries in the plural
    fix: In D6, state that each attempt, re-entry attempts and fixer strands included, gets one same-rung retry after a machinery loss, and align D5's Bound wording.
  - id: F15
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/discussion/decision-record.md Scope Out
    summary: Scope puts pushing from inside the row Out while Open risks asks the parent whether Weaver should push after each batch
    fix: Drop the Scope Out line on pushing; the Open risks question carries it until the parent answers.
  - id: F16
    severity: NIT
    class: consistency
    origin: handler
    location: _lym/discussion/support-log.md Question ledger Q6, Q9, Q10
    summary: Q6 still says card gates at HEAD (D7 now runs the full post-batch checks), Q9 still runs the gates entries after the verify (D10 puts them on strands), and Q10 still says the Stuck reason names the paths and is raised under Open risks (D8 moved it into the arbiter description)
    fix: Mark Q6, Q9 and Q10 as revised after the lens review, as Q11 is, with the answers D7, D10 and D8 now give.
---

### [BLOCKING] D12's attach rests on a false FindRun premise

**Location:** `_lym/discussion/decision-record.md` D12
**Issue:** `FindRun(cfg, anchorPath, guid)` (`internal/shuttleengine/rundir.go`) resolves a guid the caller already holds.
Webster's `RecoverSpawnOrAttach` attaches only through a recorded `StrandGUID`.
In the window D12 names, where state recorded the attempt but not the strand, there is no guid to look up, so D6's gone-run path fires and a second agent starts on the same tree.

### [BLOCKING] D3's one-batch partition starves the shared post-batch checks

**Location:** `_lym/discussion/decision-record.md` D3, D11
**Issue:** `State.Partition` is recorded once and every Webster verb reads it.
`laterDeleteReferenceReasons` passes `unbegunCards(batches, st)` to the later-reference check, and `isLastBatch` picks the last-batch gate.
D3 discards all but the first batch of each partition, so the pending cards sit in no batch: the later-reference check sees none, and every batch renders as the last.
D7's claim that the drift check catches what an arbiter fix removed then fails for pending cards.

### [MEDIUM] "Active implementer" is undefined, and D1's guard fires too late

**Location:** `_lym/discussion/decision-record.md` D1, D13
**Issue:** The plan gate (`internal/loomshed/gates.go`) reads done cards on a reroute to Plan-Burler-Review, before the implementer row runs again.
If "active" means the setting, flipping it after Weaver built cards unfreezes them, which is the outcome D13's rationale names.
Neither decision says what a missing or corrupt state file of the other implementer lets through.

### [MEDIUM] The renumber bound loses its lease under Weaver

**Location:** `_lym/discussion/decision-record.md` D13
**Issue:** `internal/loomcli/renumber.go` takes `websterengine.AcquireStateMutation` around the bound check, and every Webster state writer takes the same lease.
D12's run lock is not one renumber takes, so a renumber and a Weaver batch begin can interleave.

### [MEDIUM] A held-back decision never reaches the strand D12 re-attaches to

**Location:** `_lym/discussion/decision-record.md` D13, D12
**Issue:** `heldBackReason` (`internal/loomcli/decision.go`) justifies the hold-back with "the fresh session the run's re-entry starts carries the decision".
Weaver's re-entry attaches to the live strand instead, so neither the notice nor a fresh prompt brings it the decision.

### [MEDIUM] Describe and the rework archive see only Webster's run directory

**Location:** `_lym/discussion/decision-record.md` D9, D10, D13
**Issue:** Describe's `RunRecordPath` is built from `websterGeom.WebsterDir` (`internal/loomcli/wiring.go`), its prior records come from `ArchivedWebsterDirs` over `prior-generation/webster`, and `ArchiveWebster` archives only Webster's record.
D10's "as it does Webster's" and D9's "as it does `_lym/webster/`" both rest on readers that D13's reader list leaves out.

### [MEDIUM] D11 defers whether the shared pieces can leave websterengine

**Location:** `_lym/discussion/decision-record.md` D11
**Issue:** The pieces D11 names take `*State`, `*BatchState` and `Geometry` (`postBatchInputs`, `checkRecoveredSuspects(geom, st, bs, …)`, `failureDigestBlock(prior *BatchState)`, the fingerprint helpers).
Whether the move carries Webster's state model decides Weaver's own state shape (D3, D9, D12).
That is a coarse module-boundary fact the writer may establish, and the rubric's deferred-research rule applies.

### [MEDIUM] A failed attempt's strand is never stopped

**Location:** `_lym/discussion/decision-record.md` D6, D7, D12
**Issue:** A timed-out attempt's strand can stay live after D6 climbs the ladder or the row returns Stuck.
The arbiter then fixes the tree beside it, and D12's re-entry attaches to it ahead of D7's checks.

### [MEDIUM] D1's way forward names a reset Weaver has no verb for

**Location:** `_lym/discussion/decision-record.md` D1, D2
**Issue:** "Reset the other implementer's run directory" maps to `lym webster reset` for Webster.
D2 adds no verb, so the reverse direction names no step an operator can run, against PATTERN-refusal-way-forward's rule that a refusal names its way forward and a test reaches it.

### [LOW] D2 and D5 name the wrong caps on re-entry

**Location:** `_lym/discussion/decision-record.md` D2, D5
**Issue:** The driver's repair cap counts interrupted stops; a Stuck re-enters through the tackler's resume and a parent re-step, which neither Bound names.

### [LOW] The Weaver fixer runs under Webster's recovery timeout

**Location:** `_lym/discussion/decision-record.md` D10
**Issue:** Batten's watch reads `webster.yaml`'s `RecoveryTimeoutMin` for any verify-fixer strand, and `weaver.yaml` has no timeout key.

### [LOW] merge-in's in-flight warning misses Weaver

**Location:** `_lym/discussion/decision-record.md` D13
**Issue:** `websterengine.RunInFlight` reads Webster's files only.

### [LOW] The journal can take one attempt's entry twice

**Location:** `_lym/discussion/decision-record.md` D8, D12
**Issue:** D8's append and D12's crash-time recording are not ordered or keyed against each other.

### [LOW] D6's same-rung retry has no stated scope

**Location:** `_lym/discussion/decision-record.md` D5, D6
**Issue:** D6 does not say whether the retry is per attempt or per batch, or whether it covers re-entry and fixer strands.

### [LOW] Scope rules out what Open risks asks the parent

**Location:** `_lym/discussion/decision-record.md` Scope
**Issue:** "Pushing from inside the row" is Out, yet the second Open risk asks whether Weaver should push after each batch.

### [NIT] Stale ledger answers

**Location:** `_lym/discussion/support-log.md` Question ledger
**Issue:** Q6, Q9 and Q10 still give the answers that the lens review replaced.

## Focus verdicts

- Attempt bounds (D5, D6, D7, D10): one attempt per rung and one last-rung re-entry attempt agree across D5 and D7. D10's fixer bound is stated as a climb up the ladder, with one last-rung fixer after a verify stop. The same-rung retry's scope is F14, and the caps cited for re-entry are F10.
- D1 guard and D13 seam: neither says which state decides "active" or what a bad state file lets through (F3).
- D9 per-batch commit: `fabricengine.CommitAnchoredPaths` stages only the named relPaths under the weft write lock and never pushes, so other uncommitted work is neither swept in nor lost. `_lym` is weft-side and excluded on the warp side, so the commit does not change D6's clean-tree check. Not a defect.
- D11: a deferred research question (F7), and its partition input is broken (F2).
- D8 and D13 grounding: `arbiterRecipeDescription` takes only the recipe name today, and D8 states the change plainly. `heldBackReason` and the loom readers D13 cites exist as described. D13's reader list misses Describe, the rework archive and merge-in (F6, F12), and the renumber lease (F4).
- Open risks: both questions are parent-class. The first asks what the operator's "each batch is a producer" meant. The second sets the board's "journal is pushed with the run" against the implementer contract and `push-before-act-takeover`'s ownership of pushing. D2 and D9 rest on the stated readings, which is how the record should proceed while it waits.
- Compaction: no rejected-alternative narrative remains in Decisions. D10's rationale clause about sparing hubs a `gate.yaml` entry is a reason for the choice, not a re-litigation. No requirement sits only in the support log.

## Rejected

- Walkthrough's D12 row (held): bound-breaker's FindRun evidence shows that the attach needs a recorded guid, so F1 stands.
- Filing the two Open risks as decision findings: the record already raises both under Open risks for the perch to put to the parent, so filing them again would duplicate them.
