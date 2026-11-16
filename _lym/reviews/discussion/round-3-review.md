---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lym/discussion/decision-record.md:143-151 (D10), :166-181 (D12), :223 (PATTERN-gate-slot constraint)
    summary: D10 and D12 spawn and attach the verify fixer directly as a gated shuttle strand with its own output file, which gatefixer's contract does not offer, and the record never says how this meets PATTERN-gate-slot's rule that a cold fixer runs only via gatefixer
    fix: >-
      In D10, state which of two designs holds, and carry it through D12, the Constraints line for PATTERN-gate-slot and the acceptance criteria.
      Either (a) the verify fixer runs through gatefixer.SpawnOrAttach and gatefixer.Await with a Weaver-owned gatefixer.Store, as Webster's verify fixer does,
      in which case drop "carrying the row's gates entries" and "attaches through AttachGated with the strand's output file and gates" for the fixer,
      state that the fixer's only output file is its reply file (fix-n.md), and state that the fixer's record is saved by gatefixer only after the spawn returns,
      so D12 must bound the crash window between spawn and record (for example by a recorded intended fixer number plus a probe of the reply path through shuttle's ProbeGated before any new spawn);
      or (b) Weaver spawns the fixer itself through shuttle, in which case add a PATTERN changes entry for PATTERN-gate-slot that admits the Weaver fixer,
      and state what bounds it (one fixer per failed verify evaluation, none after a runner crash, Go's re-run judges, and the fixer's own gate calls spawn no fixer).
      Whichever is chosen, say in the Decision that a verify runner crash gets no fixer.
  - id: F2
    severity: MEDIUM
    class: design
    location: _lym/discussion/decision-record.md:49-51 (D2, reset refusals), :170-175 (D12)
    summary: lym weaver reset refuses only for an attempt recorded in flight whose strand shuttle reports live, so a strand with a recorded outcome that was not yet stopped, or an in-flight strand whose guid was never recorded, survives the reset and keeps committing on the moved-back branch
    fix: >-
      In D2, make the reset's live-strand check cover every attempt and fixer strand state records, whatever its recorded outcome, and find each strand by its recorded output file (shuttle's ProbeGated gives a handle whose Stop ends it),
      not only by a recorded guid, since D12 says the guid may never have been recorded.
      State that the check runs before the reset archives or clears any state, and that --stop-strand stops every strand it finds through that handle.
      Add the two cases (outcome recorded and strand not yet stopped; spawn returned and guid not recorded) to the acceptance criterion for the reset.
  - id: F3
    severity: MEDIUM
    class: design
    location: _lym/discussion/decision-record.md:100-102 (D6), :172-180 (D12)
    summary: The record leaves unbounded the two shuttle answers that cannot show a strand gone, an attach that returns an error and a stop that fails, so the next attempt or the arbiter can start beside a live strand
    fix: >-
      In D12, add that an AttachGated error (shuttle returns one when a candidate cannot be confirmed dead or alive, when several live runs match, or when reed state is unreadable) makes the row return Stuck naming the error and its way forward, and spawns nothing.
      In D6, add that a failed stop through shuttle's stop verb makes the row return Stuck naming the strand guid and shuttle's way forward (lym reed remove), with the attempt's outcome already recorded so D12's first step stops it again on re-entry, and that no next attempt starts.
      State that the arbiter then meets a Stuck whose reason names the live strand.
      Add both returns to the strand-lifecycle acceptance criterion.
  - id: F4
    severity: MEDIUM
    class: design
    location: _lym/discussion/decision-record.md:32 (D1, corrupt state file way forward), :138 (D9)
    summary: D1 offers restoring Weaver's last state file version from the per-batch weft commits, but that version predates the current batch's begin record, start commit and in-flight attempt identity, so a restore lets a live strand outlive its record and re-records the batch start at a later HEAD
    fix: >-
      In D1, bound the restore: state that it is a valid way forward only when the task branch holds no commit since the commit that last recorded the state, and otherwise name `lym weaver reset` as the way forward.
      Give the reason in the Decision: a restored state has no begin record for the batch in progress, so the next begin captures HEAD as the batch start and the post-batch checks then take their delta from a start that excludes the earlier attempts' commits.
      Add the refusal and its way forward to the acceptance criterion for corrupt state files.
  - id: F5
    severity: MEDIUM
    class: consistency
    location: _lym/discussion/decision-record.md:238-241 (Open risks), _lym/discussion/support-log.md:102-103 (Q16, Q17), _lym/discussion/decision-record.md:38-53 (D2), :131-141 (D9)
    summary: Both Open risks lines are still in the record although the two Added after Discussion entries answer them, and neither D2 nor D9 states the settled call the entries tell the writer to state
    fix: >-
      In decision-record.md, delete both bullets under Open risks and leave the heading with the single line `None.`.
      Add to D2, before its Rationale line, the sentence: one producer that loops over the batches in Go, each batch stop returned as the row's Stuck so it reaches shed's stop chain, is the reading of the operator's decision that each batch is a producer.
      Add to D9 directly after the line Weaver never pushes; the landing rows do, the sentence: the per-batch commits and the journal reach the remote with the run's own push at the row's end, the push every shed transition makes, Done or Stuck alike, and no push is added inside the producer.
      In support-log.md, change Q16 and Q17 from open to answered by the matching Added after Discussion entry.
  - id: F6
    severity: LOW
    class: consistency
    location: _lym/discussion/decision-record.md:102 (D6, last sentence of the lifecycle paragraph)
    summary: D6 says a failed state write leaves an attempt that re-entry counts as a machinery loss, but shuttle keeps the record of a stopped run running when its output files exist, so D12's attach harvests that report instead
    fix: >-
      In D6, replace "counts it a machinery loss (D12)" with: re-entry attaches through D12; a report on disk is judged through the attach's result, and only an attempt with no report counts as a machinery loss.
      The evidence is shuttleengine stopOutcome in stop.go, which keeps a running record when every output file exists, and dispositionCandidate in attach.go, which attaches such a record whatever reed says.
  - id: F7
    severity: LOW
    class: design
    location: _lym/discussion/decision-record.md:48 (D2, archives _lym/weaver/ before clearing it)
    summary: The reset's archive has no named destination and no commit, so it may land in the prior-generation weaver slot that Describe reads (D13) and leave the weft tree unclean
    fix: >-
      In D2, state that the reset archives under a UTC-stamped name inside Weaver's own run directory, as Webster's reset renames state.json and the reports directory in its run directory (archive.go archiveStateFile),
      that the path is named through lymdirs, that Describe's prior-generation readers do not list it, and that the reset commits the clearing together with the plan restore through fabric.
---

### [BLOCKING] The verify fixer sits outside gatefixer's contract and the record does not reconcile it with PATTERN-gate-slot

**Location:** `_lym/discussion/decision-record.md` D10, D12 and the PATTERN-gate-slot constraint line.

**Issue:**
PATTERN-gate-slot's gate-fixer rule says a cold agent clears a failed gate only through `gatefixer`, one fixer per failed evaluation, with Go's re-run judging and no fixer for a runner crash.
D10 says the fixer "takes `gatefixer`'s verify-fixer role", carries the row's `gates` entries, writes `fix-<n>.md` as its output file, and D12 attaches it through `AttachGated` with that file and the gates.
In `internal/gatefixer/spawn.go`, `Spec` has no gate field and `SpawnOrAttach` starts a strand whose only output file is the reply file.
It attaches only to a live record its `Store` holds, and it saves the record after `Start` returns.
So under gatefixer a crash between the spawn and the record save leaves a live fixer that no record names, which is the double-fixer hazard focus item 5 asks about.
Under a direct shuttle spawn the strand escapes the pattern.
The record states neither horn, so the plan writer cannot satisfy D10, D12 and the listed constraint together.

### [MEDIUM] The reset's live-strand check misses two strands that outlive a crashed producer

**Location:** `_lym/discussion/decision-record.md` D2 (the refusals of `lym weaver reset`), D12.

**Issue:**
D2 refuses the reset only "while state records an attempt in flight whose strand shuttle reports live".
D6 records an attempt's outcome first and stops its strand second, so a crash between them leaves a live strand on an attempt that is no longer in flight; the reset's check passes.
D12 also says state may never record the guid of a spawned strand, and shuttle's stop verb (`Runner.StopStrand`) takes a guid, so the check cannot find that strand either.
In both cases the strand keeps committing to the task branch after the reset moves it back and archives the state D12's re-entry needs.
`ProbeGated` in `internal/shuttleengine/attach.go` returns a handle with `Stop()` found by output files alone, which is the way to find a guid-less strand.

### [MEDIUM] An attach error and a failed stop are unbounded

**Location:** `_lym/discussion/decision-record.md` D6 lifecycle paragraph, D12.

**Issue:**
D12 treats attach as found or not found.
`attach` in `internal/shuttleengine/attach.go` has a third answer, an error, for a candidate that cannot be confirmed dead or alive, for several live matches, and for unreadable reed state.
Treating it as not found would spawn beside a possibly live strand.
D6 promises the strand is stopped before the next attempt or the arbiter, but `removeStrandIfPresent` in `stop.go` returns an error when reed cannot be probed or the removal fails.
The record does not say what the row does then.
Item 2 of the focus is settled for every ending D6 names except these two; the order of recording against stopping does survive a crash, through D12's first step.

### [MEDIUM] The corrupt-state restore drops the begin record the batch checks depend on

**Location:** `_lym/discussion/decision-record.md` D1, D9.

**Issue:**
D9 commits state after each batch ends, so the last committed version predates the batch in progress.
D3 records a batch's start commit at begin and keeps it fixed for every attempt.
After a restore, the next begin captures the current HEAD, which already holds the earlier attempts' commits.
The comment in `recoverSpawn` (`internal/websterengine/recoverbatch.go`) states the cost: a narrower start delta reports a symbol an earlier attempt created as never created.
The restore also forgets an in-flight strand, so D12 has nothing to attach to.
The record names the restore as a way forward without a bound.

### [MEDIUM] Open risks still carry two questions the Added after Discussion entries answer

**Location:** `_lym/discussion/decision-record.md` Open risks, D2, D9; `_lym/discussion/support-log.md` Q16 and Q17.

**Issue:**
The rubric makes a writer's Open risks line that an added entry settles a `consistency` finding whose fix drops the line.
Both added entries tell the writer to state the call in D2 and D9, drop the line and mark the question answered.
The record carries neither the statements nor the drops, and the ledger still reads open.

### [LOW] D6's account of a failed state write contradicts shuttle's stop semantics

**Location:** `_lym/discussion/decision-record.md` D6.

**Issue:**
When the report is on disk, `stopOutcome` keeps the record running and `dispositionCandidate` attaches it by the file contract, so D12 harvests it through the gated Wait instead of counting a machinery loss.
The outcome is harmless, but the sentence specifies behavior shuttle does not have.

### [LOW] The reset's archive has no destination or commit

**Location:** `_lym/discussion/decision-record.md` D2.

**Issue:**
D13 has Describe's prior readers list every archived implementer slot of each rework round.
A reset archive placed in that slot would put an abandoned run in the change description.
The clearing of `_lym/weaver/` lives in the weft worktree, and D2 commits only the plan restore.

## Rejected

- Focus item 1 (crash re-entry order): not a defect beyond F3.
  D12 stops the live strand of every attempt with a recorded outcome, then attaches every in-flight attempt through `AttachGated`, and records a report only from the attach's result.
  `attach.go` supports this: a record still running with every output file on disk is attachable whatever reed says, and the attached gated Wait runs the gate.
- Focus item 3 (batch identity): not a defect.
  D3 fixes a batch's number as its first card's number, and the shared state keys batch records by it (`batchIdentity` in `runlevel.go`).
  `completedCards` and `DispatchScope` skip a number with no record, so a pending batch with no record reads as unbegun.
  D11 hands the shared checks the recorded partition mapped onto the plan (`mapPartition`), so `isLastBatch` is true only for the run's last batch.
- The no-spawn path of D7 is bounded: D7 states that a row with `gates` entries never takes it and that the full post-batch checks run first.
- The two `decision` findings the focus names are not re-raised; they are answered by the added entries, which F5 covers as a consistency finding.
