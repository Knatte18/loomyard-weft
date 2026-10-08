---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md D7
    summary: D7's follow-up-card way forward rests on a false premise, since a new card needs a Card Index line in 00-overview.md and rebaseline never accepts an overview edit
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D1, Open risks
    summary: report-head under a held run lock is bounded only by an assumption about the caller, with no mechanical check that no fork or recovery strand is still writing
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D6
    summary: card_amended mark clears at recovery spawn, so an amendment is lost if that recovery fails or ignores it, and the ladder has no stated signal telling card_amended apart from a failed recovery
  - id: F4
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md D3, D1
    summary: D3's archive-only path does not say which PlanReset guards it keeps (run lock, mid-merge, foreign branch, dirty tree) or whether it removes live recovery strands
  - id: F5
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md D3, D4
    summary: D3 promises a following plain run starts clean, but D4's standalone keep-reset leaves the run's own uncommitted edits in the tree where the pair path discards them
  - id: F6
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md D10
    summary: begin-batch names recover-batch for a terminal dead batch while Master's ladder treats dead as exhausted and ends the run stuck
  - id: F7
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md D10
    summary: config_invalid is set on any config load failure with no bound separating invalid content from an unreadable file, so a transient read error would end the run stuck
  - id: F8
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D12
    summary: migration hook leaves a profile carrying both master_base and orientation undefined, and places startup_context and alone_above ambiguously inside the profile
  - id: F9
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md Scope, D8, Acceptance criteria
    summary: Scope has no item for D7 or D8, D8 has no acceptance line, and D8's in-flight refusal no longer exists once D6 accepts an in-flight edit
  - id: F10
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md D13, Open risks, support-log.md Q14
    summary: orientation value and the system-prompt constant have no usable source, since the 20.6K figure is not in the repo and the Q14 fallback needs the same measured runs' plans
  - id: F11
    severity: LOW
    class: scope
    location: _lyx/discussion/decision-record.md Auto-mode assumptions, Open risks
    summary: the issue-run speculation repeats support-log Interview 10, and the transcript fallback is a plan requirement filed under Open risks instead of D13
---

### [BLOCKING] D7's follow-up card cannot land through rebaseline

**Location:** `_lyx/discussion/decision-record.md` D7, Scope "Out" (amending `00-overview.md`).
**Issue:** D7 tells the operator to "add a follow-up card after the last begun batch" and calls it "an ordinary card in the tail rebaseline already accepts".
`planparser.ParsePlan` builds `plan.Cards` from the Card Index in `00-overview.md` (`internal/planparser/parse.go`, `parseCardIndex` then `parseCardFile` per entry), and `checkIndexFileConsistency` flags a card file the index does not name.
A new card therefore needs an index line in `00-overview.md`, and `Rebaseline` refuses any change to that file (`internal/websterengine/rebaseline.go`, the `planOverviewFile` branch: "never rebaselined").
The rebaseline tests accept an added card only because they build the plan in memory.
So D7's way forward ends in the very refusal D7 keeps, and Scope "Out" forbids amending the overview.
The record offers no landing for a decision against a done card, and the Rationale's "without new machinery" is false.
**Fix:** Settle the design in the record.
Either rebaseline accepts an overview change confined to appending Card Index lines for cards after the last begun batch, with that bound stated, or the way forward for a done card is something that needs no index edit.
Then rework D7, the Scope "Out" line and its acceptance line to agree.

### [MEDIUM] D1 report-head under the run lock rests on trust

**Location:** `_lyx/discussion/decision-record.md` D1 (second and third Bound bullets), Open risks.
**Issue:** `report-head` is exempted from the run-lock refusal because "its caller is Master ... when no fork of the run is in flight".
The verb cannot see who calls it, and Open risks concedes that a Master forking out of order could have a fork writing during the reset.
The stated bounds (batch begun and not terminal, report on disk, descends from the start) limit which commit is the target, not whether a writer is live.
The reset also moves the remote task branch through `ResetPairCode`, which D1's Bound does not mention.
**Fix:** State a mechanical bound for the held-lock case, for example refusing while the state records a live fork or recovery strand for batch NN, or naming that the report on disk is the fork's last act and why that suffices.
If the assumption is the intended bound, say so in the Decision and add the remote-branch effect to it.

### [MEDIUM] D6 loses the amendment when the recovery strand fails

**Location:** `_lyx/discussion/decision-record.md` D6.
**Issue:** `AmendedCards` clears "once the recovery strand is spawned", but the finding is enforced only "when that attempt's report is recorded".
Two paths follow.
First, a recovery strand that reports done without reading the amended card is recorded done: the mark is already clear, so nothing enforces the amendment.
Second, a recovery that ends `stuck` or `dead` replaces the failure digest, so the `card_amended` text is gone; Master's ladder stops the run, and a later operator `recover-batch` renders no amendment instruction.
The bound "at most one extra recovery per amendment" holds, but it hides that the second recovery may never check the amendment.
D6 also says the ladder runs `recover-batch` for `card_amended` "even when the failed attempt was itself a recovery", yet the ladder today stops on any `batch_failed` from a recovery.
No refusal field or finding marker lets Master tell the two apart, and Acceptance has no line for this ladder case.
**Fix:** Decide when the mark clears (at the recovery's own terminal record, not at spawn), what a failed recovery does to it, and the signal Master reads to take the `card_amended` branch.
Add an acceptance line for recovery after a recovery.

### [MEDIUM] D3's archive-only path states no guards

**Location:** `_lyx/discussion/decision-record.md` D3 (Bound), D1 ("Every target keeps PlanReset's existing guards").
**Issue:** The archive-only branch skips the branch move, so the ancestor, remote-branch and dirty-tree checks have nothing to protect, but D3 does not say which guards remain.
The unstated cases are the run lock, a merge in progress, a foreign branch, and the live recovery strands that the current `--to start` removes before it resets (`internal/webstercli/reset.go`, `RemoveRecoveryStrands`).
Archiving a run record under a live recovery strand leaves a writer with no record.
D3's Bound lists what the path does not do; the rubric asks what it can now let through and what bounds that.
**Fix:** State in D3 which `PlanReset` refusals still apply to the archive-only path, and that live recovery strands are removed first, or why they are not.

### [MEDIUM] D3 and D4 disagree on the tree a reset to start leaves

**Location:** `_lyx/discussion/decision-record.md` D3 (Rationale), D4 (Bound).
**Issue:** D3 says every reset to start is followed by a fresh run on the reset tree.
The pair path discards uncommitted changes to the run's own paths; D4's standalone `git reset --keep` does not, and it refuses only when a dirty path differs between HEAD and the target.
After a standalone `--to start` the run's own unfinished edits may remain, and the next plain `run` starts a new run over them.
D4's refusal way forward ("commit or restore the path") also offers a commit for what was a run-owned edit.
The D3 and D4 bounds describe different guarantees without saying so.
**Fix:** State in D4 what the standalone path leaves behind for the run's own paths and untracked files, and either word the way forward as restore-only for run-owned paths or record that a dirty start is accepted.

### [MEDIUM] D10's `dead` remedy contradicts Master's ladder

**Location:** `_lyx/discussion/decision-record.md` D10 (third bullet); `contracts/stencils/webster/webster-template-master.md` (`dead` rung, line 58).
**Issue:** D10 names `lyx webster recover-batch NN` for a terminal `stuck`, `dead` or `failed` batch.
Master's template treats a `dead` batch as exhausted: write `outcome: stuck` and stop, never recovering it again.
The code allows recover-batch on a dead batch (`refuseRecoveringDoneReport`), so the remedy is reachable, but the one text Master reads after a crash sends it the opposite way.
**Fix:** Name the remedy per caller, or split `dead` into its own bullet (outcome stuck) and align the Master template line in the same commit.

### [LOW] D10 does not bound when `config_invalid` is set

**Location:** `_lyx/discussion/decision-record.md` D10, Scope.
**Issue:** The field is set on "a webster verb whose config fails to load" and ends the run `stuck` from any verb.
Batcher and webster load through the degrading `LoadOrTemplate`, so most failures are invalid content, but an unreadable file is also a load failure and would stop a run that a retry fixes.
**Fix:** Bound the field to a parse or validation failure of present config content, with an unreadable file taking the existing transient route.

### [MEDIUM] D12's migration leaves the both-keys case and key locations open

**Location:** `_lyx/discussion/decision-record.md` D12, Constraints (`PATTERN-config-strictness`).
**Issue:** The hook "replaces `weights.master_base` with `weights.orientation` at the template's value".
For a profile that carries both, it does not say whether it overwrites the operator's `orientation`, keeps it and drops `master_base`, or errors; the Bound ("only the keys a module declares retired") covers only the key set.
`alone_above` is a profile-level key and `startup_context` a `weights` key (`internal/batcher/config.go`, `estimate.go`), but D12 lists both under "inside each `profiles` entry".
Dropping a retired key is also a change to the operator's own file, which the strictness Constraint's "migration lives in reconcile" does not discuss for a profile the operator wrote.
**Fix:** State the both-keys outcome (recommended: keep the operator's `orientation`, drop `master_base`) and place each retired key at its real level.

### [MEDIUM] Scope, D8 and Acceptance criteria disagree

**Location:** `_lyx/discussion/decision-record.md` Scope, D7, D8, Acceptance criteria.
**Issue:** Scope has no item for D7's reworded done-card refusal or D8's `doc.go` sequence, though Constraints name D8's doc landing.
D7 has an acceptance line; D8 has none.
D8 says the sequence is "named in the in-flight refusal's way forward", but after D6 an in-flight card edit is accepted, so no such refusal remains unless it names the begin-batch or record-batch plan-drift refusal.
D1's five targets agree with Scope and Acceptance.
**Fix:** Add Scope items for D7 and D8, an acceptance line for D8, and name the refusal D8 means.

### [MEDIUM] D13's numeric inputs have no usable source

**Location:** `_lyx/discussion/decision-record.md` D13, Auto-mode assumptions, Open risks; `_lyx/discussion/support-log.md` Q14.
**Issue:** The system-prompt-and-tools constant is "set from the measured 20.6K start", and no file in the repo carries that figure or says what the measurement contained (a bare session or one already loaded with `CLAUDE.md`, which the computed part also counts).
`orientation` comes from the calibrate fit over "existing measured runs", and the fallback is `52000` minus "the computed base of the measured runs' plans".
Both routes need each measured run's `00-overview.md` and `CLAUDE.md`, so the fallback is not independent of the transcripts it replaces.
D12 writes this value into every migrated operator config, and Q14 stays open "for Plan".
**Fix:** Record the constant's source and contents in D13, and give a last-resort rule for `orientation` that needs no old run data, such as a stated default with the rest absorbed by the fit.

### [LOW] Deliberation narrative in Auto-mode assumptions and Open risks

**Location:** `_lyx/discussion/decision-record.md` Auto-mode assumptions (second bullet), Open risks (last bullet).
**Issue:** The second assumption is the speculation about why the issue-run's config changed; it repeats `support-log.md` Interview 10, and D10 and D12 already state what each covers, so the plan writer needs none of it.
The last Open risk carries a plan requirement, the fallback derivation of `orientation`, so it must reach D13 before anything moves.
**Fix:** Move the assumption's narrative to the support log, and extract the fallback rule into D13's Decision first (see the D13 finding).

## Focus departures

None.
